# SiMa Modalix 16-Channel AI Video Matrix

High-performance multi-stream AI video analytics using the **SiMa.ai Modalix** platform, powered by **GStreamer**, **YOLO object detection**, and **MediaMTX RTSP streaming**.

This project demonstrates a fully hardware-accelerated pipeline capable of processing **16 simultaneous RTSP video streams** with real-time AI inference and live monitoring through the **OptiView Dashboard**.

---

## 🚀 Features

* **16-Channel AI Inference**

  * Decode and process up to 16 simultaneous RTSP streams
  * Real-time YOLO object detection with bounding boxes

* **Hardware Acceleration**

  * Utilizes the SiMa.ai MLA ASIC for:

    * 8-bit H.264 video decoding
    * ML inference acceleration
  * Optimized for low-latency, real-time performance

* **Direct USB-C Streaming**

  * RTSP streaming over USB-C Ethernet Gadget
  * Eliminates Wi-Fi bottlenecks and packet instability
  * High-bandwidth direct host-to-board communication

* **OptiView Dashboard Integration**

  * Live visualization of all streams
  * Real-time AI overlay rendering
  * MLA and CPU telemetry monitoring

* **Automated Media Optimization**

  * Includes FFmpeg batch conversion scripts
  * Safely converts unsupported 10-bit HDR videos into:

    * 8-bit SDR
    * Hardware-compatible H.264 streams

---

# 🏗️ Architecture

## Hardware

* SiMa.ai Modalix Board
* Ubuntu Host Machine

## Software Stack

| Component        | Technology                      |
| ---------------- | ------------------------------- |
| Video Pipeline   | GStreamer (`gst_app`)           |
| Streaming Server | MediaMTX                        |
| Media Processing | FFmpeg                          |
| Dashboard UI     | OptiView                        |
| Networking       | RTSP over USB-C Ethernet Gadget |

---

# 🛠️ Prerequisites

* SiMa Modalix Board running **SDK 2.0.0+**
* Ubuntu host laptop connected via USB-C
* Installed on host machine:

  * `ffmpeg`
  * `mediamtx`

---

# ⚙️ Setup & Installation

## 1. Prepare the Host Laptop (RTSP Server)

The host laptop acts as the RTSP streaming server.

### Verify USB-C Networking

Ensure the USB-C direct network interface is active:

```bash
10.42.0.1
```

### Add Video Files

Place all 16 MP4 files into a directory:

```bash
videos-720p16/
```

---

## 2. Optimize Videos for SiMa Hardware Decoder

The SiMa hardware decoder requires **strictly 8-bit H.264 video**.

Run the following FFmpeg batch conversion script:

```bash
mkdir -p safe_8bit

for f in *.mp4; do
    ffmpeg -hide_banner -loglevel error -y \
        -i "$f" \
        -c:v libx264 \
        -pix_fmt yuv420p \
        -s 1280x720 \
        "safe_8bit/$f"
done

mv safe_8bit/*.mp4 .
rm -rf safe_8bit
```

---

## 3. Configure the Media Source Script

Edit `mediasrc.sh` and set the direct USB-C IP:

```bash
# Inside mediasrc.sh
LOCAL_IP="10.42.0.1"
```

---

## 4. Install the Multi-Channel Pipeline on Modalix

SSH into the board:

```bash
ssh sima@10.42.0.57
```

Install the multi-channel sample package:

```bash
cd ~

sudo SIMA_CLI_CHECK_FOR_UPDATE=0 \
sima-cli install -v 2.0.0 samples/multichannel
```

When prompted for the RTSP source IP, enter:

```bash
10.42.0.1
```

---

## 5. Apply the OptiView Testing Patch

Patch the manifest to bypass strict dashboard validation:

```bash
sudo python3 -c "
import json
f='/data/simaai/applications/pipeline-multichannel/manifest.json'
d=json.load(open(f))
d['info']['just_test']=False
json.dump(d, open(f,'w'), indent=2)
"
```

---

# ▶️ Running the 16-Channel Matrix

## 1. Start the RTSP Broadcaster (Host Laptop)

```bash
pkill -f mediamtx
pkill -f ffmpeg

./mediasrc.sh ../videos-720p16
```

---

## 2. Start the OptiView Dashboard (Modalix Board)

### Clear Locked Memory Buffers

```bash
sudo pkill -9 gst_app

sudo ipcs -m | awk '/^0x/ {print $2}' | \
xargs -n 1 sudo ipcrm -m 2>/dev/null
```

### Launch OptiView

```bash
optiview
```

---

## 3. Open the Dashboard

Navigate to:

```text
https://10.42.0.57:9900
```

### Steps

1. Select `pipeline-multichannel`
2. Click the 🚀 Rocket icon to initialize MLA hardware
3. Click the 📺 TV icon to open the live matrix view

You should now see:

* 16 live RTSP streams
* YOLO bounding boxes
* Real-time telemetry

---

# 🛑 Troubleshooting

## Hardware Crash: `bit_depth == 8`

The SiMa decoder only supports 8-bit input.

Ensure FFmpeg uses:

```bash
-pix_fmt yuv420p
```

---

## Blank Streams / JSON Errors

The `manifest.json` may be corrupted.

Re-run the Python patch step.

---

## Video Lag or Stuttering

Ensure streaming uses:

```text
10.42.0.1
```

and not Wi-Fi routing.

If browser rendering becomes heavy:

* Switch OptiView to 4-channel mode

---

## Segmentation Fault / Locked Mutex

Clear stale shared memory buffers:

```bash
sudo ipcs -m | awk '/^0x/ {print $2}' | \
xargs -n 1 sudo ipcrm -m 2>/dev/null
```

---

# 📊 Performance Notes

* Optimized for:

  * 720p
  * H.264
  * 8-bit SDR video

* Direct USB-C networking significantly reduces:

  * latency
  * packet loss
  * synchronization issues

---

# 📁 Project Structure

```text
.
├── mediasrc.sh
├── videos-720p16/
├── README.md
└── scripts/
```

---

# 📝 License

This project is licensed under the MIT License.
