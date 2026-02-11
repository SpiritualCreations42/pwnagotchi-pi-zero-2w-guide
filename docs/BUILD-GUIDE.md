# 🔨 Complete Build Guide - Pwnagotchi on Pi Zero 2 W

**Time Required:** ~2 hours  
**Difficulty:** 🟡 Intermediate  
**Coffee Required:** ☕☕☕

> **👋 Hey Captain!** This guide is written for YOU. Every step is documented. Take breaks when needed. You've got this! 💪

---

## 📋 Table of Contents

1. [Overview & Prerequisites](#overview--prerequisites)
2. [Phase 1: System Setup](#phase-1-system-setup-20-min)
3. [Phase 2: Display Configuration](#phase-2-display-configuration-15-min)
4. [Phase 3: Nexmon Firmware](#phase-3-nexmon-firmware-10-min)
5. [Phase 4: Build Environment](#phase-4-build-environment-15-min)
6. [Phase 5: Bettercap](#phase-5-bettercap-10-min)
7. [Phase 6: Python & AI](#phase-6-python--ai-30-min)
8. [Phase 7: Pwnagotchi](#phase-7-pwnagotchi-15-min)
9. [Phase 8: Testing](#phase-8-testing-15-min)
10. [Phase 9: Image Backup](#phase-9-image-backup-optional)

---

## Overview & Prerequisites

### What You're Building
A fully functional Pwnagotchi that:
- Runs on Pi Zero 2 W
- Uses Debian Trixie (kernel 6.12.62)
- Has working WiFi monitor mode
- Displays on 3.5" screen
- Learns and captures handshakes

### Before You Start

**⚠️ CRITICAL: Read These First!**

1. **USB Gadget Conflict**
   - Pi Zero 2 W cannot use USB AND WiFi at same time
   - Development: USB connected = no WiFi
   - Operation: USB disconnected = WiFi works
   - This is HARDWARE limitation, not a bug!

2. **Kernel Version**
   - Must use kernel 6.12.62
   - Do NOT upgrade kernel
   - Nexmon patches are kernel-specific

3. **Display Driver**
   - Sunfounder MS 3.5 IPS needs mhs35ips driver
   - Waveshare drivers = wrong colors
   - Install correct driver first!

### What You Need

**Hardware:**
- [ ] Raspberry Pi Zero 2 W
- [ ] 64GB+ microSD card (Class 10+)
- [ ] USB cable (data + power)
- [ ] Sunfounder MS 3.5 IPS display (optional)
- [ ] Computer with SD card reader

**Software:**
- [ ] [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [ ] SSH client (PuTTY/Terminal)
- [ ] Text editor

**Time & Energy:**
- [ ] 2-3 hours of focused time
- [ ] Patience for compilation steps
- [ ] Coffee or beverage of choice ☕

---

## Phase 1: System Setup (20 min)

### Step 1.1: Flash Raspberry Pi OS

1. **Download & Install Raspberry Pi Imager**

2. **Flash the SD Card:**
   - Choose OS: **"Raspberry Pi OS (other)"** → **"Raspberry Pi OS Lite (64-bit)"**
   - Choose **Debian Trixie** variant if available
   - Choose Storage: Your SD card
   
3. **Configure Settings** (⚙️ icon):
   ```
   ✅ Set hostname: pwnagotchi
   ✅ Enable SSH (password authentication)
   ✅ Set username: admin (or your choice)
   ✅ Set password: <your secure password>
   ✅ Configure WiFi: <your network> (for initial setup only)
   ✅ Set timezone: <your timezone>
   ```

4. **Write to SD Card** and wait for completion

5. **Boot the Pi:**
   - Insert SD card into Pi
   - Connect USB to computer
   - Wait 2-3 minutes for first boot

### Step 1.2: Connect via SSH

```bash
# Find the IP address (check your router or use)
ping pwnagotchi.local

# SSH in
ssh admin@pwnagotchi.local
# Or use the IP: ssh admin@<ip-address>

# Enter your password
```

**✅ Checkpoint:** You should see the Raspberry Pi welcome message!

### Step 1.3: Initial System Configuration

```bash
# Update package lists
sudo apt update

# Check kernel version (should be 6.12.x)
uname -r
# Output should be something like: 6.12.62+rpt-rpi-v8

# Hold kernel to prevent updates
sudo apt-mark hold raspberrypi-kernel
sudo apt-mark hold raspberrypi-kernel-headers

# Install kernel headers for current version
sudo apt install -y raspberrypi-kernel-headers

# Full system upgrade
sudo apt upgrade -y
```

**⏱️ This will take ~10 minutes on Pi Zero 2 W**

```bash
# Verify kernel is held
apt-mark showhold
# Should show: raspberrypi-kernel and raspberrypi-kernel-headers
```

**✅ Checkpoint:** System updated, kernel held at 6.12.x

### Step 1.4: Setup USB Gadget Networking

**⚠️ Important:** This enables USB networking for development!

```bash
# Enable USB gadget mode
echo 'dtoverlay=dwc2' | sudo tee -a /boot/firmware/config.txt

# Load modules at boot
echo ' modules-load=dwc2,g_ether' | sudo tee -a /boot/firmware/cmdline.txt

# Configure g_ether module
echo 'options g_ether use_eem=0 host_addr=f0:0d:ba:be:f0:0d dev_addr=58:70:77:6e:79:58' | \
  sudo tee /etc/modprobe.d/g_ether.conf

# Configure usb0 network interface
sudo tee /etc/network/interfaces.d/usb0-cfg << 'EOF'
allow-hotplug usb0
iface usb0 inet dhcp
  metric 900
EOF

# Reboot to apply changes
sudo reboot
```

**After reboot, your Pi will get an IP from Windows ICS (usually 10.x.x.x range)**

```bash
# Reconnect via new IP
ssh admin@<new-ip-address>

# Check USB interface
ip addr show usb0
# Should show an IP address like 10.0.0.1
```

**✅ Checkpoint:** USB networking working, can SSH via USB!

---

## Phase 2: Display Configuration (15 min)

### Step 2.1: Identify Your Display

**Important:** Different displays need different drivers!

**For Sunfounder MS 3.5 IPS:**
- Model: MS 3.5 IPS SP 125MHz 480x320
- Driver needed: **mhs35ips** (NOT Waveshare!)

### Step 2.2: Install MHS35 IPS Driver

**⚠️ Method varies - document your specific installation:**

```bash
# Clone the driver repository (example - adjust for your driver source)
cd ~
git clone <mhs35ips-driver-repo-url>

# Follow driver-specific installation
cd <driver-directory>
sudo ./install.sh # or similar

# Reboot
sudo reboot
```

### Step 2.3: Verify Display Configuration

```bash
# Check boot config
cat /boot/firmware/config.txt | grep -i display

# Should see something like:
# dtoverlay=mhs35ips
# OR specific overlay for your display

# Check framebuffer devices
ls -la /dev/fb*
# Should see /dev/fb0 and possibly /dev/fb1
```

### Step 2.4: Test Display

```bash
# Show image on display
sudo fbi -T 1 -d /dev/fb1 -noverbose -a /usr/share/pixmaps/debian-logo.png

# If display shows Debian logo with CORRECT colors → Success!
# If colors are wrong → Wrong driver, try different one
```

**✅ Checkpoint:** Display working with correct colors!

---

## Phase 3: Nexmon Firmware (10 min)

### Step 3.1: Install Build Dependencies

```bash
# Install required packages
sudo apt install -y git libgmp3-dev gawk qpdf bison flex make \
  autoconf libtool texinfo gcc-arm-none-eabi libfl-dev g++ xxd \
  tcpdump build-essential dkms libelf-dev
```

### Step 3.2: Clone and Build Nexmon

```bash
# Create git directory
mkdir -p ~/git
cd ~/git

# Clone DrSchottky's Nexmon (has 6.12 support!)
git clone --depth=1 https://github.com/DrSchottky/nexmon.git
cd nexmon

# Build as root (easier due to setup_env.sh)
sudo bash
source setup_env.sh

# Build nexmon base
make
# Takes ~3 minutes on Pi Zero 2 W

# Navigate to Pi Zero 2 W patches
cd patches/bcm43436b0/9_88_4_65/nexmon

# Build and install firmware
make
# Takes ~2 minutes

make backup-firmware
make install-firmware

# Exit root shell
exit
```

### Step 3.3: Install Nexmon Driver

```bash
# Install the .ko driver
cd /lib/modules/$(uname -r)/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac

# Backup original
sudo mv brcmfmac.ko.xz brcmfmac.ko.xz.ORIG

# Copy and compress nexmon driver
sudo cp ~/git/nexmon/patches/driver/brcmfmac_$(uname -r | cut -d. -f 1-2).y-nexmon/brcmfmac.ko .
sudo xz brcmfmac.ko

# Update module dependencies
sudo depmod -a

# Reboot to load new driver
sudo reboot
```

### Step 3.4: Verify Nexmon Loaded

```bash
# Check dmesg for nexmon
dmesg | grep -i nexmon
# Should see: "Firmware: BCM43436/2 wl0: Oct  4 2023 03:00:02 version 9.88.4.65 (nexmon...)"

# Check WiFi interface
ip link show wlan0
# Should be present

# Note: Monitor mode WON'T work while USB connected (WiFi disabled)!
```

**✅ Checkpoint:** Nexmon firmware installed, will test monitor mode later!

---

## Phase 4: Build Environment (15 min)

### Step 4.1: Setup Swap Space

**⚠️ Critical:** Pi Zero 2 W needs more swap for builds!

```bash
# Create 4GB swapfile
sudo fallocate -l 4G /var/swapfile
sudo chmod 600 /var/swapfile
sudo mkswap /var/swapfile
sudo swapon /var/swapfile

# Make permanent
echo '/var/swapfile   swap    swap     defaults   0       0' | sudo tee -a /etc/fstab

# Verify
swapon -s
```

### Step 4.2: Install Build Tools

```bash
# Core build tools
sudo apt install -y build-essential git wget curl unzip rsync vim

# Python development
sudo apt install -y python3-pip python3-dev python3-setuptools

# Network tools
sudo apt install -y libnetfilter-queue-dev libpcap-dev tcpdump lsof

# Libraries
sudo apt install -y libssl-dev libffi-dev libbz2-dev libsqlite3-dev \
  libreadline-dev libncursesw5-dev tk-dev libgdbm-dev libc6-dev \
  zlib1g-dev libopenblas-dev libatlas-base-dev libhdf5-dev

# Additional dependencies
sudo apt install -y dbus-python3 libdbus-1-dev libdbus-glib-1-dev \
  bc flex bison libelf-dev libusb-1.0-0-dev
```

**⏱️ Takes ~5 minutes**

### Step 4.3: Install Go

```bash
# Download Go 1.21.1 for ARM64
cd ~/Downloads
wget https://go.dev/dl/go1.21.1.linux-arm64.tar.gz

# Install
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.21.1.linux-arm64.tar.gz

# Add to PATH
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# Verify
go version
# Should show: go version go1.21.1 linux/arm64
```

**✅ Checkpoint:** Build environment ready!

---

## Phase 5: Bettercap (10 min)

### Step 5.1: Setup Build Environment

```bash
# Create temp directories outside /tmp (it's too small!)
mkdir -p ~/go-build-tmp
mkdir -p ~/pip-tmp

# Export environment variables
export TMPDIR=~/go-build-tmp
export GOCACHE=~/go-build-tmp/.cache
export GOPATH=~/go
```

### Step 5.2: Build Bettercap

```bash
# Clone bettercap
cd ~/git
git clone https://github.com/bettercap/bettercap.git
cd bettercap

# Build
make build
# Takes ~8 minutes on Pi Zero 2 W

# Install
sudo mv bettercap /usr/bin/

# Verify
bettercap -version
```

### Step 5.3: Install Caplets and UI

```bash
# Install caplets and web UI (must specify interface that exists!)
# wlan0 should exist even if not working due to USB
sudo bettercap -eval "caplets.update; ui.update; quit" -iface wlan0
```

**✅ Checkpoint:** Bettercap installed and caplets updated!

---

## Phase 6: Python & AI (30 min)

### Step 6.1: Install Python Packages from APT

**⚠️ Install as many from APT as possible to avoid pip issues!**

```bash
# Core Python packages
sudo apt install -y python3-flask python3-flask-cors python3-flaskext.wtf \
  python3-pil python3-smbus python3-dbus python3-yaml python3-dateutil \
  python3-requests python3-numpy python3-pandas python3-matplotlib

# Scientific packages
sudo apt install -y python3-scipy python3-networkx python3-cloudpickle
```

### Step 6.2: Install Python Packages from Pip

```bash
# Set pip temp directory
export TMPDIR=~/pip-tmp

# Install additional packages
sudo -E pip install --break-system-packages --no-cache-dir \
  pycryptodome requests PyYAML scapy tweepy file-read-backwards \
  inky smbus2 Pillow spidev gast flask flask-cors flask-wtf \
  dbus-python toml python-dateutil websockets gym shimmy \
  jinja2==2.11.3 MarkupSafe==1.1.1

# Install prctl (needed by pwnagotchi)
sudo pip install --break-system-packages python-prctl
```

### Step 6.3: Install PyTorch

```bash
# PyTorch from default pip (works on ARM64!)
sudo pip install --break-system-packages --no-cache-dir torch torchvision
```

**⏱️ Takes ~15 minutes**

### Step 6.4: Install stable-baselines3

```bash
# Install stable-baselines3 2.x
sudo pip install --break-system-packages --no-cache-dir stable-baselines3==2.7.1

# Upgrade numpy
pip install --upgrade --break-system-packages numpy
```

**✅ Checkpoint:** All Python dependencies installed!

---

## Phase 7: Pwnagotchi (15 min)

### Step 7.1: Clone Pwnagotchi

```bash
# Clone Sniffleupagus fork (has stable-baselines3 2.x support!)
cd ~/git
git clone https://github.com/Sniffleupagus/pwnagotchi-snflpgs.git
cd pwnagotchi-snflpgs
```

### Step 7.2: Fix Shebang

```bash
# Fix hardcoded virtualenv path
sed -i 's|#!/home/pwnagotchi/.venv/bin/python3|#!/usr/bin/python3|' \
  ~/git/pwnagotchi-snflpgs/bin/pwnagotchi
```

### Step 7.3: Install Pwnagotchi

```bash
# Create symlinks
sudo ln -s ~/git/pwnagotchi-snflpgs/bin/pwnagotchi /usr/local/bin/
sudo ln -s ~/git/pwnagotchi-snflpgs/pwnagotchi /usr/local/lib/python3.13/dist-packages/pwnagotchi

# Create plugin directory
sudo mkdir -p /usr/local/share/pwnagotchi/custom-plugins

# Create config directory
sudo mkdir -p /etc/pwnagotchi

# Enable SPI for display
if ! grep -q "^dtparam=spi=on" /boot/firmware/config.txt; then
  echo "dtparam=spi=on" | sudo tee -a /boot/firmware/config.txt
  sudo dtparam spi=on
fi
```

### Step 7.4: Install PWNGrid

```bash
# Download pwngrid
cd ~/Downloads
wget https://github.com/evilsocket/pwngrid/releases/download/v1.10.3/pwngrid_linux_aarch64_v1.10.3.zip
unzip pwngrid_linux_aarch64_v1.10.3.zip

# Install
sudo mv pwngrid /usr/bin/
sudo chmod +x /usr/bin/pwngrid

# Generate identity (takes ~90 seconds)
sudo pwngrid -generate -keys /etc/pwnagotchi
```

**Identity will be generated and shown!**

### Step 7.5: Install Helper Scripts

```bash
# Install all helper scripts
sudo cp ~/git/pwnagotchi-snflpgs/builder/data/usr/bin/* /usr/bin/
sudo chmod +x /usr/bin/mon*

# Install systemd services
sudo cp ~/git/pwnagotchi-snflpgs/builder/data/etc/systemd/system/*.service \
  /etc/systemd/system/

# Reload systemd
sudo systemctl daemon-reload
```

### Step 7.6: Configure Pwnagotchi

```bash
# Create basic config
sudo tee /etc/pwnagotchi/config.toml << 'EOF'
main.name = "pwnagotchi"
main.custom_plugins = "/usr/local/share/pwnagotchi/custom-plugins"

# Disable grid during USB development
main.plugins.grid.enabled = false

# LED disabled (different path on Pi Zero 2 W)
main.plugins.led.enabled = false

# Deauth disabled for Nexmon stability
personality.deauth = false

# Display will enable after USB testing
ui.display.enabled = false

# Web UI enabled
ui.web.enabled = true
ui.web.username = "pwnagotchi"
ui.web.password = "pwnagotchi"
EOF
```

**✅ Checkpoint:** Pwnagotchi installed and configured!

---

## Phase 8: Testing (15 min)

### Step 8.1: Test Pwnagotchi (USB Mode)

```bash
# Run in manual mode with debug
sudo pwnagotchi --manual --debug
```

**You should see:**
```
[INFO] pwnagotchi@<long-hash> (v2.1.2)
[INFO] entering manual mode ...
[INFO] web ui available at http://0.0.0.0:8080/
```

**✅ Success if no errors!**

**Press Ctrl+C to stop**

### Step 8.2: Access Web UI

Open browser on your computer:
```
http://<pi-ip-address>:8080
```

Login:
- Username: `pwnagotchi`
- Password: `pwnagotchi`

**✅ If web UI loads, core Pwnagotchi is working!**

### Step 8.3: Test Monitor Mode (Standalone)

**⚠️ IMPORTANT:** Must disconnect USB for this test!

```bash
# Exit SSH cleanly
exit

# Unplug USB cable from Pi

# Wait 30 seconds

# Plug USB back in and SSH back

# Check if mon0 was created
ip link show mon0

# Check logs
sudo journalctl -u pwnagotchi -n 50
```

**✅ If mon0 exists and WiFi is scanning, monitor mode works!**

### Step 8.4: Enable Display (Optional)

```bash
# Edit config
sudo nano /etc/pwnagotchi/config.toml

# Change:
ui.display.enabled = true

# Save and exit (Ctrl+X, Y, Enter)

# Reboot
sudo reboot

# After reboot, disconnect USB
# Display should show Pwnagotchi face!
```

**✅ Checkpoint:** Everything working! Time to deploy! 🎉

---

## Phase 9: Image Backup (Optional)

### Step 9.1: Clean Up Before Backup

```bash
# Clean temporary files
sudo rm -rf ~/go-build-tmp/*
sudo rm -rf ~/pip-tmp/*
sudo rm -rf ~/.cache/pip/*

# Clean package cache
sudo apt clean
sudo apt autoclean

# Clear old logs
sudo journalctl --vacuum-time=1d

# Shutdown
sudo shutdown -h now
```

### Step 9.2: Create Image

1. Remove SD card from Pi
2. Insert into computer
3. Use Win32DiskImager or Etcher:
   - **Read** image from SD card
   - Save as: `pwnagotchi-pi0-2w-trixie.img`
4. Compress: `gzip pwnagotchi-pi0-2w-trixie.img`

### Step 9.3: Deploy to Production Card

1. Flash image to 64GB SD card
2. Insert into Pi
3. Boot and test
4. Success! 🎉

**✅ You now have a deployable Pwnagotchi image!**

---

## 🎉 Success!

### What You Built

- ✅ Fully functional Pwnagotchi
- ✅ Working WiFi monitor mode
- ✅ Web UI accessible
- ✅ Display configured
- ✅ AI ready to learn
- ✅ Deployable image

### Next Steps

1. **Configure AI parameters** - See [Configuration Guide](CONFIGURATION.md)
2. **Enable plugins** - See [Plugin Guide](PLUGINS.md)
3. **Start hunting handshakes!** 🎯

### Remember

- **USB mode** = Development (no WiFi)
- **Standalone** = Operation (WiFi active)
- Always disconnect USB cleanly before unplugging!

---

## 🆘 Having Issues?

**Check these first:**
1. [Troubleshooting Guide](TROUBLESHOOTING.md)
2. [Common Issues](#common-issues-quick-fixes)
3. [GitHub Issues](../../issues)

### Common Issues Quick Fixes

**Issue:** "ModuleNotFoundError: No module named 'X'"
```bash
sudo pip install --break-system-packages X
```

**Issue:** Monitor mode not working
```bash
# Make sure USB is DISCONNECTED!
# Check Nexmon loaded: dmesg | grep nexmon
```

**Issue:** Display wrong colors
```bash
# Wrong driver! Need mhs35ips for Sunfounder
# Check: cat /boot/firmware/config.txt
```

**Issue:** Web UI not accessible
```bash
# Check if running: sudo systemctl status pwnagotchi
# Check IP: ip addr show usb0
```

---

## 📚 Additional Resources

- [Quick Reference](QUICK-REFERENCE.md) - Command cheat sheet
- [Configuration Guide](CONFIGURATION.md) - All config options
- [Hardware Guide](HARDWARE.md) - Compatible parts
- [Development Workflow](DEVELOPMENT.md) - USB vs Standalone

---

<p align="center">
  <strong>🎊 Congratulations! You built something amazing! 🎊</strong><br>
  <em>Now go capture some handshakes! 📡</em>
</p>

---

**Need help?** Open an issue or check the community resources!  
**Built successfully?** Share your success story! ⭐
