# 🤖 Running Local LLMs on a $15 Raspberry Pi Zero 2 W (with Ollama)

[![Hardware](https://img.shields.io/badge/Hardware-Raspberry_Pi_Zero_2_W-C51A4A?style=for-the-badge&logo=raspberry-pi&logoColor=white)](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/)
[![Memory](https://img.shields.io/badge/RAM-512MB_LPDDR2-blue?style=for-the-badge)](https://github.com/)
[![Engine](https://img.shields.io/badge/Inference-Ollama_(ARM64)-black?style=for-the-badge&logo=ollama&logoColor=white)](https://ollama.com/)
[![Default Model](https://img.shields.io/badge/Model-SmolLM2_135M_(Q4__K__S)-orange?style=for-the-badge)](https://huggingface.co/HuggingFaceTB/SmolLM2-135M-Instruct)
[![OS](https://img.shields.io/badge/OS-Raspberry_Pi_OS_64--bit_Lite-A22846?style=for-the-badge)](https://www.raspberrypi.com/software/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

A complete, battle-tested guide to running a local Large Language Model (LLM) on the ultra-budget **Raspberry Pi Zero 2 W**. 

While large AI models normally require high-end GPUs and gigabytes of VRAM, this guide proves that with proper 64-bit kernel configuration, kernel swap tuning, CPU frequency optimization, and modern 4-bit quantized micro-architectures (like **SmolLM2 135M**), you can run a responsive, private, offline conversational AI on a **$15 board with just 512MB of RAM**.

---

## 📸 Live Terminal Demo (Running SmolLM2 on Pi Zero 2 W)

<p align="center">
  <img src="docs/terminal_demo.svg" width="100%" alt="Pi Zero 2 W LLM Terminal Demo" />
</p>

---

## 📊 Memory & Resource Budget (How it Fits in 512MB)

```text
+-------------------------------------------------------------------------+
|                  TOTAL PHYSICAL RAM: 512 MB LPDDR2                      |
+------------------------------------+------------------------------------+
|  Base Linux OS (Headless Lite)     |  Ollama Daemon + Model Context     |
|  ~45 MB active kernel + systemd    |  ~180 MB reserved runtime buffer   |
+------------------------------------+------------------------------------+
|                  VIRTUAL MEMORY (SWAP): 1024 MB - 2048 MB              |
+-------------------------------------------------------------------------+
|  Model Weights (SmolLM2 135M Q4_K_S): ~85 MB resident                   |
|  KV Cache + Working Context: ~60 MB                                     |
+-------------------------------------------------------------------------+
```

---

## ⚡ Quick Specs & Benchmarks

| Metric | Measurement on Pi Zero 2 W |
| :--- | :--- |
| **SoC** | Broadcom BCM2710A1 (Quad-core 64-bit ARM Cortex-A53) |
| **Base Clock** | 1.0 GHz (Overclockable to 1.1–1.2 GHz with passive heatsink) |
| **Model Tested** | `smollm2:135m-instruct-q4_K_S` |
| **Disk Footprint** | ~92 MB |
| **RAM at Idle** | ~55 MB |
| **RAM Under Generation** | ~290 MB (No OOM kills) |
| **Inference Speed** | **~3.2 – 5.5 tokens/sec** |

---

## 🛠️ Prerequisites

### Hardware
* **Raspberry Pi Zero 2 W** (or Pi 3 / 4 / 5).
* **MicroSD Card**: 16GB or 32GB+ (Class 10 / A1 or A2 rated recommended for swap speed).
* **Power Supply**: 5V / 2.5A Micro-USB adapter.
* **Cooling**: A small adhesive aluminum or copper heatsink (strongly recommended for sustained inference).

### Software
* **[Raspberry Pi Imager](https://www.raspberrypi.org/software/)**
* **OS**: Raspberry Pi OS (Legacy, 64-bit) Lite or Debian Bookworm 64-bit Lite (*64-bit ARM64 is mandatory for Ollama*).

---

## 🚀 Step-by-Step Setup Guide

### Step 1: Flash 64-bit Raspberry Pi OS Lite
1. Launch **Raspberry Pi Imager**.
2. Click **Choose OS** $\rightarrow$ **Raspberry Pi OS (other)** $\rightarrow$ **Raspberry Pi OS (Legacy, 64-bit) Lite**.
3. In the OS Customization settings (gear icon):
   - Set hostname (e.g. `pizero`).
   - Configure your username and password.
   - Enter your Wi-Fi credentials.
   - Check **Enable SSH** (use password authentication or your public key).
4. Click **Write** to flash the MicroSD card.

---

### Step 2: Boot and Initial Connect
Insert the MicroSD card into your Pi Zero 2 W and connect power. Wait ~60 seconds for the first boot, then connect via SSH:

```bash
ssh <username>@pizero.local
```

*(If you get a host key warning after re-flashing, clear it using `ssh-keygen -R pizero.local`)*

Update the system packages:
```bash
sudo apt update && sudo apt full-upgrade -y
```

---

### Step 3: Kernel & Swap Optimization (Crucial for 512MB RAM)
Because 512MB of RAM leaves little room for unexpected memory spikes, expanding your swap space is essential to prevent the Linux Out-Of-Memory (OOM) killer from terminating Ollama.

1. Turn off the default small swap:
   ```bash
   sudo dphys-swapfile swapoff
   ```
2. Open the swap configuration:
   ```bash
   sudo nano /etc/dphys-swapfile
   ```
3. Set `CONF_SWAPSIZE` to `1024` (or `2048` if using a large SD card):
   ```ini
   CONF_SWAPSIZE=1024
   ```
4. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`), then recreate and enable the swap:
   ```bash
   sudo dphys-swapfile setup
   sudo dphys-swapfile swapon
   ```
5. Tune Linux memory swappiness:
   ```bash
   sudo sysctl vm.swappiness=80
   echo "vm.swappiness=80" | sudo tee -a /etc/sysctl.conf
   ```

---

### Step 4: Overclocking & Headless Power Tuning (Optional)
To boost inference speeds from ~2.5 tokens/sec to ~4.5+ tokens/sec:

1. Open `/boot/firmware/config.txt` (or `/boot/config.txt` on older releases):
   ```bash
   sudo nano /boot/firmware/config.txt
   ```
2. Append the following performance settings at the bottom:
   ```ini
   # --- Pi Zero 2 W Performance Tuning ---
   arm_freq=1100
   over_voltage=4
   force_turbo=0
   
   # Headless memory savings (releases GPU RAM to system)
   gpu_mem=16
   
   # Disable HDMI output if running headless to save power & RAM
   hdmi_blanking=2
   ```
   > ⚠️ **Note:** Ensure you have a heatsink attached before overclocking. If the board becomes unstable, reduce `arm_freq` back to `1000`.
3. Reboot to apply:
   ```bash
   sudo reboot
   ```

---

### Step 5: Install Ollama
Download and run the official Linux install script:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Verify that the Ollama daemon service is active:
```bash
ollama --version
systemctl status ollama
```

---

### Step 6: Pull & Run Lightweight Models

#### 🏆 Recommended: SmolLM2 135M (Fastest & Most Reliable)
Hugging Face's **SmolLM2-135M-Instruct** quantized to 4-bit runs smoothly within 512MB RAM:

```bash
ollama run smollm2:135m-instruct-q4_K_S
```

You will immediately be greeted with an interactive prompt:
```text
>>> Why is the sky blue?
The sky appears blue because molecules in Earth's atmosphere scatter sunlight in all directions...
```

#### Other Lightweight Models to Test:
| Model Tag | Parameters | Quant Size | Recommended Board |
| :--- | :--- | :--- | :--- |
| `smollm2:135m-instruct-q4_K_S` | 135M | ~85 MB | **Pi Zero 2 W (Ideal)** |
| `smollm:135m` | 135M | ~90 MB | **Pi Zero 2 W** |
| `tinydolphin` | 1.1B | ~630 MB | Pi Zero 2 W *(heavy swap use)* / Pi 4 |
| `qwen2.5:0.5b-instruct-q4_K_M` | 490M | ~390 MB | Pi 3B+ / Pi 4 |

---

### Step 7: Accessing Ollama as a Local API
Ollama runs an HTTP REST API on port `11434`. You can query your Pi Zero 2 W over your local network:

```bash
curl http://pizero.local:11434/api/generate -d '{
  "model": "smollm2:135m-instruct-q4_K_S",
  "prompt": "Write a 3-line haiku about a robot.",
  "stream": false
}'
```

---

## 🔧 Troubleshooting & Tips

* **Error: `signal: killed` (OOM Crash):**  
  If the model terminates mid-generation, your swap is either full or not enabled. Run `free -m` and verify that `Swap` shows at least `1024`.
* **High Thermals:**  
  Check your CPU temperature with `vcgencmd measure_temp`. Keep temperatures below 80°C to avoid automatic CPU thermal throttling.
* **Disable Unused Background Services:**  
  If you need more free RAM, disable Bluetooth:
  ```bash
  sudo systemctl disable hciuart.service
  sudo systemctl disable bluetooth.service
  ```

---

## 📜 License
Open-source under the **MIT License**. Created to explore the boundaries of edge AI on low-power, low-cost hardware!
