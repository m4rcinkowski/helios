# Helios 🐾📹

A lightweight, zero-dependency Node.js tool that turns any Mac with multiple cameras into a real-time, multi-stream pet monitor accessible over your local network.

https://github.com/user-attachments/assets/57cb7bdb-b513-4941-8ba8-802e4dc31b73

---

## 💡 Motivation

When leaving my dog home alone, I wanted a simple way to check in on him from another room or my phone without buying dedicated security cameras or subscribing to monthly cloud services. 

I already had the hardware right on my desk: a MacBook with an integrated FaceTime HD camera and a Logitech USB webcam. The goal was to build a single command-line script to capture both cameras simultaneously, stream them over low-latency WebRTC, and present a responsive dashboard on any local device.

---

## ⚡ Advantages

* **100% Local & Cloud-Free:** No third-party servers, cloud accounts, or SaaS subscriptions. Your video stays entirely within your local network.
* **Hardware Re-use:** Uses equipment you already own (macOS webcams and USB cameras).
* **Hardware Accelerated:** Leverages Apple VideoToolbox (`h264_videotoolbox`) for low-CPU, low-battery H.264 encoding.
* **Ultra-Low Latency:** WebRTC (WHEP) streaming delivers video with sub-second delay compared to standard HLS streams.
* **Mobile-Friendly Dashboard:** A clean HTML viewer with grid/solo modes, pinch-to-zoom support, and touch swipe gestures.

---

## 🔒 Security & Assumptions

### The Good
* **No External Leaks:** Video streams are bound locally. No data leaves your network to third-party processing servers.
* **Zero Cloud Attack Surface:** No remote database or authentication endpoints exposed to the open internet.

### Caveats & Assumptions
* **Unauthenticated Local Access:** Any device connected to your Wi-Fi or LAN can open `http://<MACBOOK-IP>:8000` to view live streams without a password.
* **Unencrypted HTTP/WebRTC:** HTTP traffic and WebRTC media streams run unencrypted over your local subnet by default.
* **Network Isolation:** Do not expose port `8000` or `8889` to the internet via router port-forwarding. To access the stream outside your home, use a private mesh VPN like **Tailscale** or **WireGuard**.

---

## 📋 Requirements

* **macOS** (Apple Silicon or Intel)
* **Node.js** (v18+)
* **FFmpeg** with `avfoundation` support
* **MediaMTX** (RTSP/WebRTC media server)

---

## 🚀 Installation & Running

### 1. Install System Dependencies

Install `ffmpeg` and `mediamtx` via Homebrew:

```bash
brew install ffmpeg mediamtx

```

### 2. Clone the Repository

```bash
git clone https://github.com/m4rcinkowski/helios.git
cd helios

```

### 3. Make Executable & Run

Make `./helios` executable and start the launcher:

```bash
chmod +x helios
./helios

```

1. Select your desired camera devices using the interactive CLI menu (**Space** to toggle, **Enter** to confirm).
2. Open the displayed local network URL (e.g., `http://192.168.1.X:8000`) on your phone or tablet browser.

---

## 🛠️ Architecture Overview

```
┌────────────────────────────────────────────────────────┐
│               Helios CLI Supervisor                    │
└───────┬───────────────────┬───────────────────┬────────┘
        │                   │                   │
        ▼                   ▼                   ▼
 ┌──────────────┐   ┌───────────────┐   ┌──────────────┐
 │  caffeinate  │   │   MediaMTX    │   │ Embedded HTTP│
 │(Sleep Guard) │   │ (RTSP / WHEP) │   │ (Dashboard)  │
 └──────────────┘   └───────▲───────┘   └──────────────┘
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
    ┌────────────────────┐    ┌────────────────────┐
    │ FFmpeg (FaceTime)  │    │ FFmpeg (USB Cam)   │
    └────────────────────┘    └────────────────────┘

```

---

## 📄 License

MIT
