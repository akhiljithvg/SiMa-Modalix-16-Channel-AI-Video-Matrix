# SiMa Modalix 16-Channel AI Video Matrix

![SiMa.ai](https://img.shields.io/badge/Platform-SiMa.ai_Modalix-blue)
![GStreamer](https://img.shields.io/badge/Framework-GStreamer-green)
![RTSP](https://img.shields.io/badge/Protocol-RTSP-orange)

A high-performance, hardware-accelerated GStreamer pipeline designed to process 16 simultaneous video streams (720p/480p) using YOLO object detection on the **SiMa.ai Machine Learning Accelerator (MLA)**. This project includes a local RTSP streaming architecture via MediaMTX and integrates with the OptiView web dashboard for real-time monitoring.

## 🚀 Features

* **16-Channel AI Inference:** Simultaneously decode and run YOLO-based object detection on 16 RTSP video streams.
* **Hardware Acceleration:** Fully utilizes the SiMa.ai ASIC for 8-bit H.264 video decoding and ML processing, ensuring real-time performance.
* **Zero-Latency Direct Streaming:** Bypasses local Wi-Fi networks by utilizing a Direct USB-C Network (`10.42.0.1`) for flawless, high-bandwidth RTSP streaming.
* **OptiView Dashboard Integration:** View all 16 streams, bounding boxes, and live MLA/CPU telemetry via a web browser.
* **Automated Media Optimization:** Includes FFmpeg batch scripts to safely downscale and transcode unsupported 10-bit HDR video into hardware-safe 8-bit SDR video formats.

## 🏗️ Architecture & Tech Stack

* **Hardware:** SiMa.ai Modalix Board, Ubuntu Host Machine
* **Video Pipeline:** GStreamer (`gst_app`)
* **Streaming Server:** MediaMTX, FFmpeg
* **UI/Dashboard:** SiMa OptiView
* **Networking:** RTSP over USB-C Ethernet Gadget

---

## 🛠️ Prerequisites

1. **SiMa Modalix Board** running SDK 2.0.0+
2. **Ubuntu Host Laptop** connected to the board via USB-C.
3. `ffmpeg` and `mediamtx` installed on the host machine.

---

## ⚙️ Setup & Installation

### 1. Prepare the Host Laptop (Streaming Server)
The host laptop acts as the RTSP server. It must push video directly over the USB-C cable to avoid network lag.

* Ensure the USB-C direct network interface is active (usually `10.42.0.1`).
* Place your 16 MP4 files in a directory (e.g., `videos-720p16`).

**Optimize Videos (Crucial for Hardware Decoder):**
The SiMa hardware decoder requires strictly **8-bit** video. Run this batch script to convert any 10-bit HDR videos to safe 8-bit 720p files:
```bash
mkdir -p safe_8bit
for f in *.mp4; do 
    ffmpeg -hide_banner -loglevel error -y -i "$f" -c:v libx264 -pix_fmt yuv420p -s 1280x720 "safe_8bit/$f"
done
mv safe_8bit/*.mp4 .
rm -rf safe_8bit
