# Systemd Service Configuration

This directory contains systemd service files for running squeeze2diretta as a system service with automatic startup.

## Quick Setup

### 1. Edit Service File

Edit `squeeze2diretta.service` and replace:
- `YOURUSER` → Your Linux username (e.g., `dominique`)
- `LMS_SERVER_IP` → Your LMS server IP address (e.g., `192.168.1.104`)
- `TARGET_NUMBER` → Your Diretta target number from `--list-targets` (e.g., `1`)

### 2. Install Service

```bash
# Copy service file to systemd
sudo cp squeeze2diretta.service /etc/systemd/system/

# Reload systemd
sudo systemctl daemon-reload

# Enable service (auto-start on boot)
sudo systemctl enable squeeze2diretta

# Start service now
sudo systemctl start squeeze2diretta
```

### 3. Check Status

```bash
# View status
sudo systemctl status squeeze2diretta

# View real-time logs
sudo journalctl -u squeeze2diretta -f

# View last 100 log lines
sudo journalctl -u squeeze2diretta -n 100
```

## Service Management

### Start/Stop/Restart

```bash
# Start
sudo systemctl start squeeze2diretta

# Stop
sudo systemctl stop squeeze2diretta

# Restart
sudo systemctl restart squeeze2diretta

# Reload configuration
sudo systemctl reload squeeze2diretta
```

### Enable/Disable Auto-Start

```bash
# Enable auto-start on boot
sudo systemctl enable squeeze2diretta

# Disable auto-start
sudo systemctl disable squeeze2diretta

# Check if enabled
systemctl is-enabled squeeze2diretta
```

### Logs

```bash
# Follow logs (real-time)
sudo journalctl -u squeeze2diretta -f

# Show last 50 lines
sudo journalctl -u squeeze2diretta -n 50

# Show logs since boot
sudo journalctl -u squeeze2diretta -b

# Show logs for specific time period
sudo journalctl -u squeeze2diretta --since "1 hour ago"
sudo journalctl -u squeeze2diretta --since "2026-02-01 10:00:00"
```

## Performance Tuning

The service file includes performance optimizations:

- **Nice=-10**: Higher CPU priority
- **IOSchedulingClass=realtime**: Real-time I/O scheduling
- **Restart=on-failure**: Automatic restart on crashes

### Real-Time Priority (Advanced)

To enable real-time scheduling priority, uncomment these lines in the service file:

```ini
LimitRTPRIO=95
LimitMEMLOCK=infinity
```

Then add your user to the `audio` group and configure RT limits:

```bash
# Add user to audio group
sudo usermod -aG audio $USER

# Create RT limits configuration
sudo nano /etc/security/limits.d/audio.conf
```

Add:
```
@audio   -  rtprio     95
@audio   -  memlock    unlimited
```

Reboot for changes to take effect.

## Troubleshooting

### Service Won't Start

```bash
# Check syntax errors
sudo systemd-analyze verify squeeze2diretta.service

# View detailed status
sudo systemctl status squeeze2diretta -l

# Check logs for errors
sudo journalctl -u squeeze2diretta -n 50
```

### Common Issues

**Issue: Permission denied**
- Check that `User=` matches your username
- Verify file paths are correct
- Ensure squeeze2diretta binary is executable: `chmod +x build/squeeze2diretta`

**Issue: LMS server not found**
- Verify LMS server IP is correct
- Check network connectivity: `ping LMS_SERVER_IP`
- Check firewall allows port 3483 (SlimProto)

**Issue: Diretta target not found**
- Run `./build/squeeze2diretta --list-targets` to verify target number
- Check network connectivity to Diretta target
- Verify Diretta target is powered on

### Testing Before Installing

Test the command manually before installing the service:

```bash
# Copy the ExecStart line from service file and run it directly
/home/YOURUSER/squeeze2diretta/build/squeeze2diretta \
    --squeezelite /home/YOURUSER/squeeze2diretta/squeezelite/squeezelite \
    -r 768000 -D :u32be \
    -s LMS_SERVER_IP \
    -n squeeze2diretta \
    --target TARGET_NUMBER
```

If this works, then the service should work too.

## Multiple Instances

To run multiple squeeze2diretta instances (e.g., for different zones):

1. Copy service file:
```bash
sudo cp squeeze2diretta.service squeeze2diretta-zone2.service
```

2. Edit the new file to use different:
   - Player name (`-n zone2`)
   - Working directory (if different)
   - Diretta target (`--target 2`)

3. Install and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now squeeze2diretta-zone2
```

---

**For more information, see the main [README.md](../README.md)**
