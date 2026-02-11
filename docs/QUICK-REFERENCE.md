# ⚡ Quick Reference - Pwnagotchi Commands

> **For Captain:** Your memory-friendly command cheat sheet! Bookmark this page! 📌

---

## 🚀 Essential Commands

### Check System Status
```bash
# Check if Pwnagotchi running
sudo systemctl status pwnagotchi

# View live logs
sudo journalctl -u pwnagotchi -f

# Check last 50 log lines
sudo journalctl -u pwnagotchi -n 50

# Check Pwnagotchi version
pwnagotchi --version
```

### Start/Stop Pwnagotchi
```bash
# Manual mode (testing)
sudo pwnagotchi --manual --debug

# Start service
sudo systemctl start pwnagotchi

# Stop service
sudo systemctl stop pwnagotchi

# Restart service
sudo systemctl restart pwnagotchi

# Enable auto-start on boot
sudo systemctl enable pwnagotchi
```

### Network Commands
```bash
# Check USB network
ip addr show usb0

# Check WiFi interface
ip addr show wlan0

# Check monitor interface (only when USB disconnected!)
ip link show mon0

# Test connectivity
ping -c 4 8.8.8.8
```

---

## 📡 Monitor Mode Commands

### ⚠️ REMEMBER: USB must be DISCONNECTED for monitor mode!

```bash
# Check if Nexmon loaded
dmesg | grep -i nexmon

# Check WiFi chipset
dmesg | grep brcmfmac

# Manually create monitor interface (testing)
sudo iw phy phy0 interface add mon0 type monitor
sudo ifconfig mon0 up

# Check monitor mode
sudo tcpdump -i mon0

# Remove monitor interface
sudo ifconfig mon0 down
sudo iw dev mon0 del
```

---

## 🎨 Display Commands

```bash
# Check framebuffer devices
ls -la /dev/fb*

# Show image on display
sudo fbi -T 1 -d /dev/fb1 -noverbose -a /path/to/image.png

# Check display config
cat /boot/firmware/config.txt | grep -i display

# Clear display
sudo dd if=/dev/zero of=/dev/fb1
```

---

## 📝 Configuration

### Edit Config
```bash
# Edit main config
sudo nano /etc/pwnagotchi/config.toml

# View current config
cat /etc/pwnagotchi/config.toml

# Check config syntax (run pwnagotchi to validate)
sudo pwnagotchi --check-config
```

### Common Config Changes
```toml
# Enable/disable display
ui.display.enabled = true  # or false

# Enable/disable grid
main.plugins.grid.enabled = true  # or false

# Change name
main.name = "your-name-here"

# Change web UI password
ui.web.password = "YOUR_WEB_PASSWORD"
```

---

## 🔧 Troubleshooting Commands

### Check Services
```bash
# All pwnagotchi services
sudo systemctl status pwnagotchi
sudo systemctl status bettercap
sudo systemctl status pwngrid-peer

# Check if services enabled
systemctl is-enabled pwnagotchi
systemctl is-enabled bettercap
```

### View Logs
```bash
# Pwnagotchi logs
sudo journalctl -u pwnagotchi -n 100

# Bettercap logs
sudo journalctl -u bettercap -n 100

# System logs
sudo dmesg | tail -50

# All logs since boot
sudo journalctl -b
```

### Check Python Modules
```bash
# List installed packages
pip list

# Check specific package
pip show stable-baselines3

# Check Python version
python3 --version
```

### Disk Space
```bash
# Check disk usage
df -h

# Check directory sizes
du -sh /*

# Find large files
sudo find / -type f -size +100M 2>/dev/null
```

---

## 🗄️ File Locations

### Important Files
```bash
# Main config
/etc/pwnagotchi/config.toml

# Identity keys
/etc/pwnagotchi/id_rsa
/etc/pwnagotchi/id_rsa.pub

# Pwnagotchi binary
/usr/local/bin/pwnagotchi

# Python package
/usr/local/lib/python3.13/dist-packages/pwnagotchi/

# Custom plugins
/usr/local/share/pwnagotchi/custom-plugins/

# Handshakes
/root/handshakes/

# Boot config
/boot/firmware/config.txt
```

### Service Files
```bash
# Systemd services
/etc/systemd/system/pwnagotchi.service
/etc/systemd/system/bettercap.service
/etc/systemd/system/pwngrid-peer.service

# Helper scripts
/usr/bin/monstart
/usr/bin/monstop
/usr/bin/monreload
```

---

## 🔄 Common Workflows

### USB Development Mode
```bash
# 1. Connect USB to computer
# 2. SSH in
ssh admin@<ip-address>

# 3. Make changes, test
sudo pwnagotchi --manual --debug

# 4. Access web UI
http://<ip-address>:8080

# 5. Exit cleanly
exit

# 6. Unplug USB (cleanly shutdown if making permanent changes)
sudo shutdown -h now
```

### Standalone Operation Mode
```bash
# 1. Make sure display enabled
sudo nano /etc/pwnagotchi/config.toml
# ui.display.enabled = true

# 2. SSH in via USB
ssh admin@<ip-address>

# 3. Check services will start
sudo systemctl status pwnagotchi
sudo systemctl status bettercap

# 4. Exit cleanly
exit

# 5. Unplug USB

# 6. Wait 2 minutes

# 7. Check display shows Pwnagotchi face

# 8. Reconnect USB to check logs
ssh admin@<ip-address>
sudo journalctl -u pwnagotchi -n 50
```

### Update Pwnagotchi Code
```bash
# Pull latest changes
cd ~/git/pwnagotchi-snflpgs
git pull

# Restart service
sudo systemctl restart pwnagotchi

# Check logs
sudo journalctl -u pwnagotchi -f
```

---

## 📦 Package Management

### Install Package
```bash
# From apt
sudo apt install package-name

# From pip (always use --break-system-packages on Debian)
sudo pip install --break-system-packages package-name

# With no cache (saves space)
sudo pip install --break-system-packages --no-cache-dir package-name
```

### Update System
```bash
# Update package lists
sudo apt update

# Upgrade packages (but NOT kernel!)
sudo apt upgrade -y

# Check held packages
apt-mark showhold
# Should show: raspberrypi-kernel and raspberrypi-kernel-headers
```

---

## 🔐 Security Commands

### PWNGrid Identity
```bash
# View identity
sudo cat /etc/pwnagotchi/id_rsa.pub

# Regenerate identity (if needed)
sudo rm /etc/pwnagotchi/id_rsa*
sudo pwngrid -generate -keys /etc/pwnagotchi
```

### Change Passwords
```bash
# System user password
passwd

# Web UI password
sudo nano /etc/pwnagotchi/config.toml
# Change: ui.web.password = "YOUR_WEB_PASSWORD"
```

---

## 💾 Backup & Restore

### Quick Backup
```bash
# Backup config and handshakes
sudo tar -czf ~/backup-$(date +%Y%m%d).tar.gz \
  /etc/pwnagotchi/ \
  /root/handshakes/

# Copy to computer
scp admin@<ip>:~/backup-*.tar.gz .
```

### Restore
```bash
# Extract backup
sudo tar -xzf backup-YYYYMMDD.tar.gz -C /

# Restart services
sudo systemctl restart pwnagotchi
```

---

## 🎯 Performance Checks

### CPU & Memory
```bash
# CPU usage
top -n 1

# Memory usage
free -h

# Swap usage
swapon -s

# Temperature
vcgencmd measure_temp
```

### WiFi Signal
```bash
# Check WiFi signal (when connected)
iwconfig wlan0

# Scan for networks (when not in monitor mode)
sudo iwlist wlan0 scan | grep ESSID
```

### Handshake Stats
```bash
# Count handshakes
sudo ls -1 /root/handshakes/*.pcap 2>/dev/null | wc -l

# List handshakes
sudo ls -lh /root/handshakes/

# Recent captures
sudo ls -lt /root/handshakes/ | head -10
```

---

## 🆘 Emergency Commands

### Pwnagotchi Not Starting
```bash
# Check why it failed
sudo systemctl status pwnagotchi -l

# View full logs
sudo journalctl -u pwnagotchi --no-pager

# Try manual mode to see errors
sudo pwnagotchi --manual --debug
```

### No WiFi
```bash
# Check USB is DISCONNECTED!
# Check if driver loaded
lsmod | grep brcmfmac

# Check firmware
ls -la /lib/firmware/brcm/brcmfmac43436*

# Reload driver
sudo modprobe -r brcmfmac
sudo modprobe brcmfmac
```

### Display Not Working
```bash
# Check config
cat /boot/firmware/config.txt | grep display

# Check framebuffer
ls -la /dev/fb*

# Test with image
sudo fbi -T 1 -d /dev/fb1 -noverbose -a /usr/share/pixmaps/debian-logo.png
```

### Out of Disk Space
```bash
# Find what's using space
df -h
sudo du -sh /* | sort -h

# Clean up
sudo apt clean
sudo apt autoclean
sudo rm -rf ~/go-build-tmp/*
sudo rm -rf ~/pip-tmp/*
sudo rm -rf ~/.cache/pip/*
sudo journalctl --vacuum-time=7d
```

---

## 📱 Web UI

### Access Web UI
```
http://<pi-ip-address>:8080

Default credentials:
Username: pwnagotchi
Password: pwnagotchi
```

### Web UI Not Loading
```bash
# Check if Pwnagotchi running
sudo systemctl status pwnagotchi

# Check port listening
sudo netstat -tlnp | grep 8080

# Check firewall (if enabled)
sudo iptables -L
```

---

## 🔄 Reboot & Shutdown

```bash
# Reboot
sudo reboot

# Shutdown (recommended before unplugging USB)
sudo shutdown -h now

# Force shutdown (emergency only)
sudo shutdown -h now -f
```

---

## 📖 More Help

- **Full Build Guide:** [BUILD-GUIDE.md](BUILD-GUIDE.md)
- **Troubleshooting:** [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **Configuration:** [CONFIGURATION.md](CONFIGURATION.md)
- **Hardware:** [HARDWARE.md](HARDWARE.md)

---

## 💡 Pro Tips

1. **Always disconnect USB cleanly** - Exit SSH before unplugging!
2. **Check logs first** - Most problems show in `journalctl`
3. **USB = No WiFi** - Hardware limitation, not a bug!
4. **Test changes in manual mode** - Before enabling service
5. **Backup before big changes** - `tar -czf backup.tar.gz /etc/pwnagotchi`

---

<p align="center">
  <em>🔖 Bookmark this page! You'll reference it often!</em>
</p>

**Last Updated:** January 27, 2026
