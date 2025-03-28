## 🛠 Voicevox Engine Setup (AWS Linux / EC2)

To keep the Voicevox Engine stable and automatically restarted, it's recommended to run it as a `systemd` service instead of using `&` in the background.

### ✅ 1. Run as a systemd service

Create a service file:

```bash
sudo nano /etc/systemd/system/voicevox-engine.service
```

Paste the following (modify paths if needed):

```ini
[Unit]
Description=Voicevox Engine
After=network.target

[Service]
ExecStart=/home/ec2-user/VOICEVOX/vv-engine/run --port 2970
WorkingDirectory=/home/ec2-user/VOICEVOX/vv-engine
Restart=always
RestartSec=5
User=ec2-user
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

Reload and start the service:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable voicevox-engine
sudo systemctl start voicevox-engine
sudo systemctl status voicevox-engine
```

---

### ✅ 2. Monitor Port Usage / Conflicts

Check if another process is using port 2970:

```bash
sudo lsof -i :2970
```

If needed, try a different port:

```bash
./vv-engine/run --port 2980
```

Be sure to update `config.json` in your `daizu-tts` folder with the new port.

---

### ✅ 3. Check for Out-of-Memory (OOM) or CPU Load

On small EC2 instances, the engine may get killed due to limited memory. Check with:

```bash
sudo dmesg | grep -i kill
```

If OOM kills are found, consider:

- Upgrading your instance (more RAM/CPU)
- Adding a swap file
- Rate-limiting or queuing TTS requests

---

### ✅ 4. Log Output to a File

To monitor crashes or logs, run:

```bash
./vv-engine/run --port 2970 >> engine.log 2>&1 &
```

Or configure `StandardOutput` and `StandardError` in your `systemd` service.

---

### ✅ 5. Use a Watchdog Script (if not using systemd)

If you prefer not to use `systemd`, you can keep the engine alive with a simple watchdog script:

```bash
#!/bin/bash
while true; do
  if ! pgrep -f "vv-engine"; then
    echo "Restarting Voicevox Engine..."
    /home/ec2-user/vv-engine/run --port 2970 &
  fi
  sleep 10
done
```

Save it as `voicevox-watchdog.sh` and run it using `screen`, `tmux`, or a background process.





# 💡 Keeping `daizu-tts` Running with systemd

To ensure the `daizu-tts` bot stays running even after crashes, disconnects, or server restarts, you can use **systemd** to manage and supervise the process.

This guide helps you set up `systemd` to run `npm run production` reliably.

---

## 🛠️ 1. Create a systemd Service File

Run:

```bash
sudo nano /etc/systemd/system/daizu-tts.service
```

Paste the following (modify paths if needed):

```ini
[Unit]
Description=Daizu TTS Discord Bot
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/home/ec2-user/daizu-tts
ExecStart=/usr/bin/npm run production
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target

```

Reload and start the service:

```bash
sudo systemctl daemon-reexec        # optional but good after updates
sudo systemctl daemon-reload        # reload systemd configs
sudo systemctl enable daizu-tts     # start on system boot
sudo systemctl start daizu-tts      # start it now
```
### 🔍 3. Check Logs

To monitor the bot’s logs live:

```bash
journalctl -u daizu-tts.service -f
```

Or check startup logs with:

```bash
journalctl -u daizu-tts.service --since "10 minutes ago"
```

