# Configuration Files

This directory contains example configuration files for Pwnagotchi on Pi Zero 2 W.

## Files

### `config.toml.example`
Example Pwnagotchi configuration file.

**Usage:**
```bash
# Copy to Pi
scp config.toml.example admin@<pi-ip>:~/

# On Pi, move to proper location
ssh admin@<pi-ip>
sudo cp ~/config.toml.example /etc/pwnagotchi/config.toml

# Edit to customize
sudo nano /etc/pwnagotchi/config.toml

# Restart Pwnagotchi
sudo systemctl restart pwnagotchi
```

**Important Settings to Change:**
- `main.name` - Your Pwnagotchi's name
- `ui.web.password` - Web UI password
- `ui.display.enabled` - Set to `true` when ready for standalone
- `main.plugins.grid.enabled` - Enable after WiFi is working

---

### `boot-config.txt.example`
Example boot configuration for display and USB gadget.

**Usage:**
```bash
# On Pi, backup original
sudo cp /boot/firmware/config.txt /boot/firmware/config.txt.backup

# Add relevant sections to your config.txt
sudo nano /boot/firmware/config.txt

# Reboot to apply
sudo reboot
```

**Critical Sections:**
- **USB Gadget:** Enables USB ethernet for development
- **Display:** Configure your specific display driver
- **vc4-kms-v3d:** Must be disabled for SPI displays

---

## Configuration Tips

### Development vs Production

**Development Mode (USB Connected):**
```toml
ui.display.enabled = false        # No display needed during dev
main.plugins.grid.enabled = false # Grid won't work without WiFi
ui.web.enabled = true             # Web UI for testing
```

**Production Mode (Standalone):**
```toml
ui.display.enabled = true         # Show Pwnagotchi face
main.plugins.grid.enabled = true  # Share data with other Pwnagotchis
ui.web.enabled = true             # Keep enabled or false to save resources
```

---

## Display Configuration

### Sunfounder MS 3.5 IPS
```
dtoverlay=mhs35ips,rotate=90,speed=20000000
```

### Waveshare 3.5" (if compatible)
```
dtoverlay=waveshare35b-v2,rotate=270,speed=20000000
```

**Note:** Sunfounder displays often need `mhs35ips` driver, NOT Waveshare!

---

## Security Notes

⚠️ **Before deploying:**

1. **Change web UI password:**
   ```toml
   ui.web.password = "your-secure-password"
   ```

2. **Consider disabling web UI in standalone mode:**
   ```toml
   ui.web.enabled = false
   ```
   (Saves resources, but you won't be able to access web interface)

3. **Review whitelist settings:**
   ```toml
   main.whitelist = ["AA:BB:CC:DD:EE:FF"]  # MACs to never attack
   ```

---

## Plugin Configuration

Enable/disable plugins as needed:

```toml
# Essential plugins (already enabled by default)
main.plugins.auto-update.enabled = true
main.plugins.session-stats.enabled = true

# Optional plugins
main.plugins.grid.enabled = true         # PWNGrid network
main.plugins.gps.enabled = false         # GPS tracking (needs hardware)
main.plugins.bt-tether.enabled = false   # Bluetooth tethering
main.plugins.memtemp.enabled = true      # Memory/temp monitoring
```

---

## Troubleshooting

**Config not loading:**
```bash
# Check syntax
python3 -c "import toml; toml.load('/etc/pwnagotchi/config.toml')"

# Check permissions
ls -la /etc/pwnagotchi/config.toml
# Should be readable by root

# Check logs
sudo journalctl -u pwnagotchi | grep -i config
```

**Display not working:**
```bash
# Check boot config
cat /boot/firmware/config.txt | grep -i display

# Verify SPI enabled
ls -la /dev/spidev*

# Check framebuffer
ls -la /dev/fb*
```

---

## More Information

- **Full documentation:** [../docs/CONFIGURATION.md](../docs/CONFIGURATION.md)
- **Build guide:** [../docs/BUILD-GUIDE.md](../docs/BUILD-GUIDE.md)
- **Troubleshooting:** [../docs/TROUBLESHOOTING.md](../docs/TROUBLESHOOTING.md)

---

**Last Updated:** January 27, 2026
