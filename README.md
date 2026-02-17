🚁 MAVProxy Auto-Start Setup on Raspberry Pi

This guide helps you set up MAVProxy on a Raspberry Pi, connect it to a flight controller (FC), and configure it to auto-start on boot using a systemd service.

📌 Prerequisites

Raspberry Pi with Raspberry Pi OS

Internet connection

Flight Controller (Pixhawk/Ardupilot compatible)

USB cable to connect FC to Pi

Laptop running GCS (e.g., QGroundControl)

Know your Laptop IP address

⚙️ Installation & Setup
✅ Step 1 — Update Raspberry Pi
sudo apt update
sudo apt upgrade -y

✅ Step 2 — Install Python Tools
sudo apt install python3-pip python3-venv -y

✅ Step 3 — Create Virtual Environment
python3 -m venv mavenv
source mavenv/bin/activate

✅ Step 4 — Install MAVProxy
pip install MAVProxy --resume-retries 10 --no-cache-dir

🔌 Flight Controller Setup
✅ Step 5 — Verify FC Detection

Plug the flight controller into the Pi via USB.

Check if it is detected:

ls /dev/ttyACM*

Expected Output
/dev/ttyACM0


If not detected, try another USB port or cable.

✅ Step 6 — Manual MAVProxy Test (IMPORTANT)

Install dependency:

pip install future


Run MAVProxy manually:

mavproxy.py --master=/dev/ttyACM0 --out udp:<LAPTOP_IP>:14550


Replace:

<LAPTOP_IP>


with your laptop’s actual IP.

If successful, your GCS should connect.

🚀 Auto-Start on Boot
✅ Step 7 — Create systemd Service

Create the service file:

sudo nano /etc/systemd/system/mavproxy.service


Paste:

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


Replace:

<LAPTOP_IP>


with your laptop IP.

Save and exit.

✅ Step 8 — Enable Service
sudo systemctl daemon-reload
sudo systemctl enable mavproxy.service
sudo systemctl start mavproxy.service

✅ Step 9 — Check Service Status
sudo systemctl status mavproxy.service

Expected
active (running)

🔁 Step 10 — Reboot Test
sudo reboot


After reboot, MAVProxy should auto-start and connect automatically.

📂 Log File

Telemetry logs are saved at:

/home/pi/mav.tlog

🛠️ Troubleshooting
FC not detected

Try different USB cable

Check power supply

Run:

dmesg | grep tty

Service not starting

Check logs:

journalctl -u mavproxy.service -f

No GCS connection

Confirm laptop IP

Check firewall settings

Ensure both devices are on same network

🎯 Done!

Your Raspberry Pi now automatically runs MAVProxy and streams telemetry on boot.
