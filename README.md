Markdown
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
2. Configure the Media Source Script
Edit mediasrc.sh on your laptop to hardcode the direct USB-C IP:

Bash
# Inside mediasrc.sh
LOCAL_IP="10.42.0.1"
3. Install the Pipeline on the Modalix Board
SSH into the Modalix board (ssh sima@10.42.0.57) and install the multi-channel package:

Bash
cd ~
sudo SIMA_CLI_CHECK_FOR_UPDATE=0 sima-cli install -v 2.0.0 samples/multichannel
When prompted for the RTSP Source IP, enter: 10.42.0.1

Apply the Testing Patch:
Bypass dashboard strict validations by safely injecting the just_test key into the manifest:

Bash
sudo python3 -c "import json; f='/data/simaai/applications/pipeline-multichannel/manifest.json'; d=json.load(open(f)); d['info']['just_test']=False; json.dump(d, open(f,'w'), indent=2)"
🏃‍♂️ Running the Matrix
1. Start the RTSP Broadcaster (On Laptop):

Bash
pkill -f mediamtx
pkill -f ffmpeg
./mediasrc.sh ../videos-720p16
2. Start the OptiView Dashboard (On Board):

Bash
# Clear any lingering locked memory pools
sudo pkill -9 gst_app
sudo ipcs -m | awk '/^0x/ {print $2}' | xargs -n 1 sudo ipcrm -m 2>/dev/null

# Launch dashboard
optiview
3. View the Demo:

Open Google Chrome and navigate to: https://10.42.0.57:9900

Select pipeline-multichannel.

Click the Rocket icon (🚀) to initialize the ML hardware.

Click the TV icon (📺) to view the live 16-channel matrix with AI bounding boxes.

🛑 Troubleshooting
Hardware crashes (bit_depth == 8 error): The hardware decoder strictly requires 8-bit video. Ensure your ffmpeg script uses -pix_fmt yuv420p.

Blank Streams / JSON Errors: The manifest.json on the board was corrupted. Re-apply the Python JSON patch.

Video Stuttering / Lag: Ensure your laptop is streaming to 10.42.0.1 (Direct USB-C) and not over the local Wi-Fi router. Switch Chrome view to 4-channels if the browser struggles to render 16 streams simultaneously.

Segmentation Fault / Locked Mutex: Run sudo ipcs -m | awk '/^0x/ {print $2}' | xargs -n 1 sudo ipcrm -m 2>/dev/null on the board to clear locked hardware memory buffers.

📝 License
This project is licensed under the MIT License.
