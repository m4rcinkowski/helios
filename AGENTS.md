# AGENTS.md — Helios PetCam Project Guide

## Overview
Helios is a single-file Node.js CLI tool (`./helios`) that streams macOS `avfoundation` webcams via MediaMTX over RTSP/WebRTC (WHEP) to a local HTTP viewer on port 8000.

* **Capture:** FFmpeg (`avfoundation` + `h264_videotoolbox` hardware acceleration).
* **Streaming Server:** MediaMTX (v1.x+).
* **Flow:** Encoders publish RTSP to `rtsp://127.0.0.1:8554/camX` $\rightarrow$ MediaMTX translates to WHEP $\rightarrow$ Browser consumes WebRTC at `http://<IP>:8889/camX/whep`.

---

## Failure Post-Mortems & Rules

### 1. MediaMTX Auth Fatal Crash (`authInternalUsers` vs Legacy Keys)
* **Error:** `ERR: authInternalUsers and legacy credentials (publishUser, publishPass, ...) cannot be used together`
* **Root Cause:** In MediaMTX v1.x+, defining `authInternalUsers` while leaving legacy auth options (`readUser`, `publishUser`) in `paths` or environment variables triggers a strict startup validation crash.
* **Rule:** If `authInternalUsers` is defined, **NEVER** include `readUser`, `publishUser`, `readPass`, or `publishPass` anywhere in the config file, CLI args, or env vars.

### 2. RTSP Ingestion Rejection (`path 'camX' is not configured` / 400 Bad Request)
* **Error:** FFmpeg exits immediately with `Server returned 400 Bad Request` and MediaMTX logs `path 'camX' is not configured`.
* **Root Cause:** If `paths` is omitted or empty, MediaMTX rejects dynamic stream paths by default.
* **Rule:** You MUST define `paths: { all_others: {} }` to allow dynamic stream ingestion (`/cam1`, `/cam2`, etc.).

### 3. Environment Variable Parsing Inconsistencies
* **Error:** Browser WebRTC connections fail with `401 Unauthorized` or `"authentication error"` despite passing `MTX_AUTHINTERNAL_USERS_*` env vars.
* **Root Cause:** Env var array syntax varies across MediaMTX binary builds and OS environments.
* **Rule:** **Always use a temporary runtime YAML config file** (`/tmp/helios-mediamtx-TIMESTAMP.yml`) generated at boot and passed as a direct argument to `mediamtx`. Clean it up on exit (`shutdown()`).

### 4. macOS Binary Resolution
* **Error:** `execSync('which mediamtx')` or `spawn('mediamtx')` fails in non-interactive shells.
* **Root Cause:** Apple Silicon Homebrew uses `/opt/homebrew/bin/`, while Intel Macs use `/usr/local/bin/`.
* **Rule:** Explicitly resolve paths across `/opt/homebrew/bin`, `/usr/local/bin`, and `/usr/bin` before relying on `PATH` or `which`.

---

## Verified Minimal MediaMTX Runtime Config

The temporary runtime config file passed to `mediamtx` must strictly match this structure:

```yaml
paths:
  all_others: {}

authInternalUsers:
  - user: any
    pass: ""
    ips: []
    permissions:
      - action: publish
        path: ""
      - action: read
        path: ""
