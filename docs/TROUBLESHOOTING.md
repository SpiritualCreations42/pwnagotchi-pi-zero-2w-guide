# 🔧 Troubleshooting Guide

> **Don't Panic!** Most issues have simple fixes. Work through this guide systematically.

---

## 📋 Table of Contents

1. [Before You Start](#before-you-start)
2. [USB & Networking Issues](#usb--networking-issues)
3. [WiFi & Monitor Mode Issues](#wifi--monitor-mode-issues)
4. [Display Issues](#display-issues)
5. [Pwnagotchi Won't Start](#pwnagotchi-wont-start)
6. [Python & Dependency Issues](#python--dependency-issues)
7. [Build & Compilation Issues](#build--compilation-issues)
8. [Performance Issues](#performance-issues)
9. [Advanced Debugging](#advanced-debugging)

---

## Before You Start

### ✅ Quick Diagnostics Checklist

Run these commands to gather info:

```bash
# System info
uname -a
cat /etc/os-release
df -h

# Check kernel version (should be 6.12.x)
uname -r

# Check services
sudo systemctl status pwnagotchi
sudo systemctl status bettercap

# Check recent logs
sudo journalctl -u pwnagotchi -n 50

# Check Nexmon
dmesg | grep -i nexmon

# Check network interfaces
ip link show
```

**Copy this output** before asking for help!

---

## USB & Networking Issues

### ❌ Can't SSH via USB

**Symptoms:**
- USB connected but can't ping/SSH
- No IP address on usb0 interface

**Diagnosis:**
```bash
# On Pi (if you can get console access):
ip addr show usb0
dmesg | grep usb

# On Windows:
# Check Device Manager for RNDIS/Ethernet Gadget
```

**Solutions:**

**1. Check USB gadget is enabled:**
```bash
# On Pi:
grep dwc2 /boot/firmware/config.txt
# Should see: dtoverlay=dwc2

grep g_ether /boot/firmware/cmdline.txt
# Should see: modules-load=dwc2,g_ether
```

**2. Reinstall drivers on Windows:**
- Device Manager → Update Driver
- Search for "RNDIS" or "USB Ethernet"
- May need to install Raspberry Pi RNDIS driver manually

**3. Check modprobe config:**
```bash
cat /etc/modprobe.d/g_ether.conf
# Should contain: options g_ether use_eem=0 host_addr=... dev_addr=...
```

**4. Try different USB cable:**
- Must be data cable, not power-only
- Try different USB port on computer

**5. Check usb0 network config:**
```bash
cat /etc/network/interfaces.d/usb0-cfg
# Should have: allow-hotplug usb0
#              iface usb0 inet dhcp
```

---

### ❌ USB Works But No Internet

**Symptoms:**
- Can SSH to Pi
- Pi can't reach internet

**Solutions:**

**1. Enable Internet Connection Sharing on Windows:**
- Settings → Network & Internet → Status
- Change adapter options
- Right-click your internet connection → Properties
- Sharing tab → Check "Allow other network users..."
- Select "RNDIS/Ethernet Gadget" from dropdown

**2. Check Pi can see gateway:**
```bash
ip route show
ping -c 4 10.0.0.1  # Or whatever your gateway IP is
```

**3. Check DNS:**
```bash
cat /etc/resolv.conf
# Should have: nameserver 8.8.8.8 or similar

# Test DNS
nslookup google.com

# Fix if needed
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
```

---

## WiFi & Monitor Mode Issues

### ❌ Monitor Mode Not Working

**THE MOST COMMON ISSUE!**

**Diagnosis:**
```bash
# Check if USB is connected
# If USB is connected, WiFi is DISABLED by hardware!

# Check if mon0 exists
ip link show mon0

# Check WiFi interface
ip link show wlan0
```

**Solution:**

**⚠️ USB GADGET DISABLES WIFI - THIS IS HARDWARE BEHAVIOR!**

```bash
# 1. Exit SSH cleanly
exit

# 2. Unplug USB cable

# 3. Wait 30 seconds

# 4. Plug USB back in

# 5. SSH in and check
ssh admin@<ip>
ip link show mon0
# Should exist now!

dmesg | tail -50
# Should show brcmfmac and monitor mode setup
```

---

### ❌ Nexmon Not Loading

**Symptoms:**
```bash
dmesg | grep nexmon
# No output or shows "not found"
```

**Solutions:**

**1. Check firmware file exists:**
```bash
ls -la /lib/firmware/brcm/brcmfmac43436*

# Should see multiple .bin files including .clm_blob
```

**2. Check driver installed:**
```bash
ls -la /lib/modules/$(uname -r)/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac/

# Should see brcmfmac.ko.xz
```

**3. Rebuild Nexmon:**
```bash
cd ~/git/nexmon
sudo bash
source setup_env.sh
cd patches/bcm43436b0/9_88_4_65/nexmon
make clean
make
make install-firmware
exit

# Reboot
sudo reboot
```

**4. Check kernel version matches:**
```bash
uname -r
# Should be 6.12.x

# Check held
apt-mark showhold
# Should show raspberrypi-kernel

# If kernel updated by mistake:
sudo apt-mark hold raspberrypi-kernel
sudo apt install raspberrypi-kernel=<6.12.x-version>
```

---

### ❌ WiFi Scan Shows No Networks

**Symptoms:**
- Monitor mode works
- tcpdump shows no packets
- No networks visible

**Solutions:**

**1. Check antenna/hardware:**
```bash
# Pi Zero 2 W has onboard antenna
# Check it's not physically damaged

# Try different location
# WiFi strength matters!
```

**2. Check monitor mode properly configured:**
```bash
sudo iw phy phy0 info
# Should show "monitor" in supported interface modes

sudo iw dev mon0 info
# Should show: type monitor, channel X
```

**3. Try different channel:**
```bash
# Change to channel 6 (common channel)
sudo iw dev mon0 set channel 6
sudo tcpdump -i mon0
```

**4. Check if in right frequency band:**
```bash
# BCM43436 supports 2.4 GHz only
# Won't see 5 GHz networks!
```

---

## Display Issues

### ❌ Display Shows Wrong Colors

**THE SUNFOUNDER PROBLEM!**

**Diagnosis:**
```bash
cat /boot/firmware/config.txt | grep display

# If shows "waveshare" → WRONG DRIVER!
```

**Solution:**

**Sunfounder MS 3.5 IPS needs mhs35ips driver, NOT Waveshare!**

```bash
# Remove wrong driver
sudo nano /boot/firmware/config.txt
# Comment out or remove waveshare lines

# Install correct driver (method varies)
# Check your specific driver repository
```

**Common driver issues:**
- Wrong overlay = wrong colors
- Waveshare drivers won't work on Sunfounder
- Need exact model match

---

### ❌ Display Not Working At All

**Symptoms:**
- Screen stays white/black
- No output

**Solutions:**

**1. Check framebuffer exists:**
```bash
ls -la /dev/fb*
# Should see /dev/fb0 and /dev/fb1
```

**2. Test with image:**
```bash
sudo fbi -T 1 -d /dev/fb1 -noverbose -a /usr/share/pixmaps/debian-logo.png
```

**3. Check display config:**
```bash
cat /boot/firmware/config.txt
# Should have display overlay enabled
# Should NOT have vc4-kms-v3d enabled (causes conflicts)
```

**4. Check SPI enabled:**
```bash
grep spi /boot/firmware/config.txt
# Should have: dtparam=spi=on

# Test SPI
ls -la /dev/spidev*
```

**5. Try different framebuffer:**
```bash
# Try fb0 instead of fb1
sudo fbi -T 1 -d /dev/fb0 -noverbose -a /usr/share/pixmaps/debian-logo.png
```

---

### ❌ Display Works But Pwnagotchi Face Not Showing

**Solutions:**

**1. Check display enabled in config:**
```bash
grep "ui.display.enabled" /etc/pwnagotchi/config.toml
# Should be: ui.display.enabled = true
```

**2. Check Pwnagotchi running:**
```bash
sudo systemctl status pwnagotchi
```

**3. Check logs:**
```bash
sudo journalctl -u pwnagotchi | grep -i display
```

**4. Test in manual mode:**
```bash
sudo pwnagotchi --manual --debug
# Watch for display-related errors
```

---

## Pwnagotchi Won't Start

### ❌ Service Fails to Start

**Diagnosis:**
```bash
sudo systemctl status pwnagotchi -l
sudo journalctl -u pwnagotchi -n 100 --no-pager
```

**Common Errors & Solutions:**

---

#### Error: `ModuleNotFoundError: No module named 'X'`

**Solution:**
```bash
# Install missing module
sudo pip install --break-system-packages X

# Common missing modules:
sudo pip install --break-system-packages python-prctl
sudo pip install --break-system-packages gym
sudo pip install --break-system-packages gymnasium
```

---

#### Error: `Config file not found` or `TOML syntax error`

**Solution:**
```bash
# Check config exists
ls -la /etc/pwnagotchi/config.toml

# Validate TOML syntax (use online validator or)
python3 -c "import toml; toml.load('/etc/pwnagotchi/config.toml')"

# Recreate basic config if corrupted
sudo tee /etc/pwnagotchi/config.toml << 'EOF'
main.name = "pwnagotchi"
main.plugins.grid.enabled = false
main.plugins.led.enabled = false
personality.deauth = false
ui.display.enabled = false
ui.web.enabled = true
ui.web.username = "pwnagotchi"
ui.web.password = "pwnagotchi"
EOF
```

---

#### Error: `Connection refused` (port 8666)

**This is the GRID plugin trying to connect!**

**Solution:**
```bash
# Disable grid plugin during USB development
sudo nano /etc/pwnagotchi/config.toml

# Add or change:
main.plugins.grid.enabled = false

# Restart
sudo systemctl restart pwnagotchi
```

---

#### Error: `FileNotFoundError: id_rsa`

**Solution:**
```bash
# Regenerate identity
sudo rm /etc/pwnagotchi/id_rsa* 2>/dev/null
sudo pwngrid -generate -keys /etc/pwnagotchi

# Takes ~90 seconds
```

---

### ❌ Pwnagotchi Starts But Web UI Not Accessible

**Diagnosis:**
```bash
# Check if process running
ps aux | grep pwnagotchi

# Check if port listening
sudo netstat -tlnp | grep 8080

# Check firewall (if enabled)
sudo iptables -L
```

**Solutions:**

**1. Check web UI enabled:**
```bash
grep "ui.web.enabled" /etc/pwnagotchi/config.toml
# Should be: ui.web.enabled = true
```

**2. Try different IP address:**
```bash
# Get Pi's IP
ip addr show usb0

# Try these URLs:
http://<ip>:8080
http://10.0.0.1:8080
http://pwnagotchi.local:8080
```

**3. Check logs for Flask errors:**
```bash
sudo journalctl -u pwnagotchi | grep -i flask
sudo journalctl -u pwnagotchi | grep -i web
```

---

## Python & Dependency Issues

### ❌ stable-baselines3 Import Error

**Error:** `ImportError: cannot import name 'X' from 'stable_baselines3'`

**Solution:**

**Need version 2.x, not 1.x!**

```bash
# Check version
pip show stable-baselines3

# Should be 2.7.1 or higher
# If not, reinstall:
sudo pip uninstall stable-baselines3
sudo pip install --break-system-packages stable-baselines3==2.7.1
```

---

### ❌ gym vs gymnasium Confusion

**Error:** `ModuleNotFoundError: No module named 'gym'`

**Solution:**

**Modern Python needs gymnasium, not gym!**

```bash
# Install gymnasium
sudo pip install --break-system-packages gymnasium

# If still issues, also install gym wrapper
sudo pip install --break-system-packages gym
```

---

### ❌ NumPy Version Conflicts

**Error:** `RuntimeError: module compiled against API version X but numpy is version Y`

**Solution:**
```bash
# Upgrade numpy
pip install --upgrade --break-system-packages numpy

# If still issues, reinstall PyTorch
sudo pip uninstall torch torchvision
sudo pip install --break-system-packages torch torchvision
```

---

## Build & Compilation Issues

### ❌ Out of Disk Space During Build

**Error:** `No space left on device`

**Solution:**

**Pi Zero 2 W has limited space!**

```bash
# Check disk usage
df -h

# Clean up
sudo apt clean
sudo apt autoclean
sudo rm -rf ~/go-build-tmp/*
sudo rm -rf ~/pip-tmp/*
sudo rm -rf ~/.cache/pip/*

# Clean old kernels (if any)
sudo apt autoremove

# Use external temp dirs
export TMPDIR=~/go-build-tmp
export GOCACHE=~/go-build-tmp/.cache
```

---

### ❌ Bettercap Build Fails

**Error:** Compilation errors or "killed" during build

**Solutions:**

**1. Ensure enough swap:**
```bash
swapon -s
# Should show 4GB swap

# If not:
sudo fallocate -l 4G /var/swapfile
sudo chmod 600 /var/swapfile
sudo mkswap /var/swapfile
sudo swapon /var/swapfile
```

**2. Use temp directory with more space:**
```bash
mkdir -p ~/go-build-tmp
export TMPDIR=~/go-build-tmp
export GOCACHE=~/go-build-tmp/.cache
export GOPATH=~/go
```

**3. Clean and retry:**
```bash
cd ~/git/bettercap
make clean
make build
```

---

### ❌ Nexmon Build Fails

**Error:** Compilation errors or missing tools

**Solutions:**

**1. Install all dependencies:**
```bash
sudo apt install -y git libgmp3-dev gawk qpdf bison flex make \
  autoconf libtool texinfo gcc-arm-none-eabi libfl-dev g++ xxd
```

**2. Rebuild from scratch:**
```bash
cd ~/git/nexmon
sudo bash
source setup_env.sh
make clean
make
cd patches/bcm43436b0/9_88_4_65/nexmon
make clean
make
make install-firmware
exit
sudo reboot
```

---

## Performance Issues

### ❌ System Running Slow

**Diagnosis:**
```bash
# Check CPU
top -n 1

# Check memory
free -h

# Check temperature
vcgencmd measure_temp
```

**Solutions:**

**1. Check thermal throttling:**
```bash
vcgencmd get_throttled
# 0x0 = OK
# Other values = throttling

# If throttling, improve cooling
```

**2. Reduce AI load:**
```bash
sudo nano /etc/pwnagotchi/config.toml

# Reduce learning rate
ai.params.learning_rate = 0.0001  # Was 0.001
```

**3. Disable unused plugins:**
```bash
sudo nano /etc/pwnagotchi/config.toml

# Disable plugins you don't need
main.plugins.pluginname.enabled = false
```

---

### ❌ SD Card Too Slow

**Symptoms:**
- Long boot times
- Laggy responsiveness
- Errors about timeouts

**Solutions:**

**1. Use faster SD card:**
- Class 10 minimum
- UHS-I (U3) recommended
- A1 or A2 application class

**2. Check card health:**
```bash
# Check for read errors
sudo dmesg | grep -i mmc

# Test speed
sudo dd if=/dev/zero of=~/test.img bs=1M count=100
sudo rm ~/test.img
```

---

## Advanced Debugging

### 🔍 Enable Debug Mode

```bash
# Run Pwnagotchi in debug mode
sudo pwnagotchi --manual --debug

# Will show verbose output
# Press Ctrl+C to stop
```

### 🔍 Check All Services

```bash
# Check all related services
systemctl status pwnagotchi bettercap pwngrid-peer

# Check service failures
systemctl list-units --failed
```

### 🔍 Full System Log

```bash
# View all logs since boot
sudo journalctl -b --no-pager > ~/full-boot-log.txt

# Copy to computer
scp admin@<ip>:~/full-boot-log.txt .
```

### 🔍 Network Debug

```bash
# Full network status
ip addr show
ip route show
cat /etc/resolv.conf

# Check all interfaces
ifconfig -a

# Check kernel messages
dmesg | grep -i network
dmesg | grep -i usb
dmesg | grep -i wifi
```

---

## 🆘 Still Stuck?

### Where to Get Help

1. **Read the docs:**
   - [BUILD-GUIDE.md](BUILD-GUIDE.md)
   - [QUICK-REFERENCE.md](QUICK-REFERENCE.md)
   - [HARDWARE.md](HARDWARE.md)

2. **Search existing issues:**
   - GitHub Issues tab
   - Check closed issues too!

3. **Ask the community:**
   - Pwnagotchi Discord
   - Reddit r/pwnagotchi
   - Raspberry Pi forums

4. **Open an issue:**
   - Include full error messages
   - Include output from diagnostic commands
   - Describe what you've tried

### What to Include in Bug Reports

```bash
# Run this and include output:
echo "=== System Info ==="
uname -a
cat /etc/os-release
df -h

echo "=== Kernel Version ==="
uname -r

echo "=== Services Status ==="
systemctl status pwnagotchi --no-pager
systemctl status bettercap --no-pager

echo "=== Recent Logs ==="
sudo journalctl -u pwnagotchi -n 100 --no-pager

echo "=== Network ==="
ip addr show

echo "=== Nexmon ==="
dmesg | grep -i nexmon

echo "=== Python ==="
python3 --version
pip list | grep -i "stable-baselines3\|torch\|gym"
```

---

## 💡 Prevention Tips

1. **Always disconnect USB cleanly** - Exit SSH first!
2. **Hold kernel version** - Don't upgrade beyond 6.12.62
3. **Backup before changes** - `tar -czf backup.tar.gz /etc/pwnagotchi`
4. **Test in manual mode first** - Before enabling services
5. **Read logs** - Most problems show in journalctl
6. **Document changes** - Know what you did!
7. **Use enough swap** - 4GB for compilations
8. **Keep enough disk space** - Monitor with `df -h`

---

<p align="center">
  <strong>Most problems have simple solutions! 💪</strong><br>
  <em>Work through systematically and you'll fix it!</em>
</p>

---

**Last Updated:** January 27, 2026
