# 🚁 MAVProxy Auto-Start Setup on Raspberry Pi

> Set up MAVProxy on a Raspberry Pi, connect to a flight controller, and auto-start on boot using systemd.

---

# 📌 Prerequisites

* Raspberry Pi with Raspberry Pi OS
* Internet connection
* Flight Controller (Pixhawk / Ardupilot compatible)
* USB cable
* Laptop running GCS (QGroundControl / Mission Planner)
* Your laptop IP address

---

# ⚙️ Installation & Setup

---

## ✅ Step 1 — Update Raspberry Pi

```bash
sudo apt update
sudo apt upgrade -y
```

---

## ✅ Step 2 — Install Python Tools

```bash
sudo apt install python3-pip python3-venv -y
```

---

## ✅ Step 3 — Create Virtual Environment

```bash
python3 -m venv mavenv
source mavenv/bin/activate
```

---

## ✅ Step 4 — Install MAVProxy

```bash
pip install MAVProxy --resume-retries 10 --no-cache-dir
```

---

# 🔌 Flight Controller Setup

---

## ✅ Step 5 — Verify FC Detection

Plug the flight controller into the Pi.

Check detection:

```bash
ls /dev/ttyACM*
```

### Expected Output

```bash
/dev/ttyACM0
```

If not detected:

* Try another USB cable
* Try another port
* Check power supply

---

## ✅ Step 6 — Manual MAVProxy Test (IMPORTANT)

Install dependency:

```bash
pip install future
```

Run MAVProxy:

```bash
mavproxy.py --master=/dev/ttyACM0 --out udp:<LAPTOP_IP>:14550
```

Replace `<LAPTOP_IP>` with your laptop IP.

✅ If telemetry appears in GCS → Working!

---

# 🚀 Auto-Start on Boot

---

## ✅ Step 7 — Create systemd Service

```bash
sudo nano /etc/systemd/system/mavproxy.service
```

Paste:

```ini
[Unit]
Description=MAVProxy Autostart Service
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi

ExecStart=/bin/bash -c 'until [ -e /dev/ttyACM0 ]; do sleep 2; done; /home/pi/mavenv/bin/mavproxy.py --master=/dev/ttyACM0 --out udp:<LAPTOP_IP>:14550 --logfile=/home/pi/mav.tlog --daemon'

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

## ✅ Step 8 — Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable mavproxy.service
sudo systemctl start mavproxy.service
```

---

## ✅ Step 9 — Check Service Status

```bash
sudo systemctl status mavproxy.service
```

Expected:

```
active (running)
```

---

## 🔁 Step 10 — Reboot Test

```bash
sudo reboot
```

MAVProxy should now auto-start 🎉

---

# 📂 Logs

Telemetry logs:

```
/home/pi/mav.tlog
```

---

# 🛠️ Troubleshooting

### FC Not Detected

```bash
dmesg | grep tty
```

---

### Service Issues

```bash
journalctl -u mavproxy.service -f
```

---

### No GCS Connection

* Confirm laptop IP
* Same network
* Check firewall

---

# 🎯 Done!

Your Pi now:

* Auto connects to FC
* Streams telemetry
* Starts on boot
* Logs flight data

---

## 💡 Pro Tips for Good README Formatting

✅ Leave **blank lines** between sections
✅ Use `---` to create separators
✅ Always put commands in code blocks
✅ Keep one idea per line
✅ Use bullets instead of paragraphs

