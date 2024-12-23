# Installing a LLM on Raspberry Pi Zero 2 W with Raspberry Pi OS (Legacy, 64-bit) Lite

This guide walks you through setting up a lightweight Large Language Model (LLM) on a Raspberry Pi Zero 2 W. We’ll use Raspberry Pi OS (Legacy, 64-bit) Lite, optimize the Pi for better performance, and install the Ollama application to run the model.

---

## Prerequisites

### Hardware

- Raspberry Pi Zero 2 W
- MicroSD card (32GB or larger recommended)
- Micro USB power supply
- USB-to-Ethernet adapter or Wi-Fi connection

### Software

- Raspberry Pi Imager
- Raspberry Pi OS (Legacy, 64-bit) Lite (Released: 2024-10-22)

---

## Step 1: Flash Raspberry Pi OS

1. **Download Raspberry Pi Imager** from the [official website](https://www.raspberrypi.org/software/).
2. Insert your MicroSD card into your computer.
3. Open Raspberry Pi Imager and select:
   - **Operating System**: Raspberry Pi OS (Legacy, 64-bit) Lite
   - **Storage**: Your MicroSD card
4. Configure advanced options:
   - Enable SSH
   - Set a username and password
   - Configure Wi-Fi (if needed)
5. Click **Write** and wait for the process to complete.

---

## Step 2: Initial Setup

1. Insert the MicroSD card into the Raspberry Pi and power it on.
2. Find the Pi’s IP address:
   - Run `arp -a` from your computer.
   - Look for a device labeled `raspberrypi`.
3. SSH into the Raspberry Pi:
   ```bash
   ssh pi@raspberrypi.local
   ```
4. If you encounter an SSH error, clear the old host key:
   ```bash
   ssh-keygen -R raspberrypi.local
   ```
   Reattempt the connection and accept the new host key.

---

## Step 3: Update and Upgrade the Raspberry Pi

Run the following commands to ensure the system is up to date:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo rpi-update
```

---

## Step 4: Optimize the Raspberry Pi

### Overclocking

1. Open the configuration file:
   ```bash
   sudo nano /boot/config.txt
   ```
2. Add these lines at the end of the file:
   ```bash
   over_voltage=6
   arm_freq=1000
   force_turbo=1
   ```
   > **Warning:** Overclocking can void your warranty and potentially damage your hardware.
3. Save and exit (Ctrl+O, Enter, Ctrl+X).

### Increase Swap Memory

1. Turn off the current swap file:
   ```bash
   sudo dphys-swapfile swapoff
   ```
2. Edit the swap file configuration:
   ```bash
   sudo nano /etc/dphys-swapfile
   ```
3. Set the swap size to 1GB or higher:
   ```bash
   CONF_SWAPSIZE=1024
   ```
4. Apply the changes:
   ```bash
   sudo dphys-swapfile setup
   sudo dphys-swapfile swapon
   ```

---

## Step 5: Install Required Libraries

Install the necessary Python libraries:

```bash
pip install transformers optimum
pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/arm
```

---

## Step 6: Install Ollama

1. Download and install Ollama:
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ```
2. Verify the installation:
   ```bash
   ollama --version
   ```

---

## Step 7: Run a Model

Install and run the `smollm2:135m-instruct-q4_K_S` model:

```bash
ollama run smollm2:135m-instruct-q4_K_S
```

> This command will download the model and start interacting with it.

---

## Notes

- The Raspberry Pi Zero 2 W is a resource-constrained device. For better performance, consider upgrading to a Raspberry Pi 4.
- Always monitor the temperature during overclocking to avoid overheating.

---

Feel free to share your experience or ask questions in the comments!

