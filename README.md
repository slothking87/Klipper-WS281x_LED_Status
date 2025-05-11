# Klipper-SK6812_LED_Status
This script will take the printer status from Klipper/Moonraker and apply different effects to a SK6812 LED strip.

The code has been migrated from the OctoPrint-WS281x_LED_Status (https://github.com/cp2004/OctoPrint-WS281x_LED_Status) plugin to work with Klipper. Forked from https://github.com/11chrisadams11/Klipper-WS281x_LED_Status and updasted to use SK6812

----

## Directions for use
#!/bin/bash

# Exit on any error
set -e

echo "🔧 Updating and installing dependencies..."
sudo apt update
sudo apt install -y python3-pip python3-dev python3-gpiozero git

echo "📦 Installing required Python libraries..."
pip3 install --upgrade setuptools
pip3 install adafruit-circuitpython-neopixel

echo "🔌 Enabling SPI and UART (for board support)..."
sudo raspi-config nonint do_spi 0
sudo raspi-config nonint do_serial 1

echo "📁 Cloning LED control script directory..."
cd ~
git clone https://github.com/your/repo/Klipper-WS281x_LED_Status.git  # Change this to your repo if you have one
cd Klipper-WS281x_LED_Status

echo "🔧 Making the script executable..."
chmod +x led_control.py

echo "⚙️ Setting up systemd service..."

# Create the systemd service file
sudo tee /etc/systemd/system/led_control.service > /dev/null <<EOF
[Unit]
Description=LED Control Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/Klipper-WS281x_LED_Status/led_control.py
WorkingDirectory=/home/pi/Klipper-WS281x_LED_Status
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
EOF

echo "🔄 Reloading systemd, enabling and starting the service..."
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable led_control.service
sudo systemctl start led_control.service

echo "✅ Setup complete!"

``

#### To call from gcode shell commands (thanks to [JV_JV](https://www.reddit.com/user/JV_JV/) for the setup directions)
Add custom entries to printer.cfg - change the directory to match yours. 

```
[gcode_shell_command led_off]
command: ./home/pi/my_klipper_ledstrip.py 0 0 0
timeout: 2.
verbose: True

[gcode_shell_command led_white]
command: ./home/pi/my_klipper_ledstrip.py 255 255 255
timeout: 2.
verbose: True

[gcode_shell_command led_purple]
command: ./home/pi/my_klipper_ledstrip.py 255 0 255
timeout: 2.
verbose: True

[gcode_macro LED_OFF]
gcode:
    RUN_SHELL_COMMAND CMD=led_off

[gcode_macro LED_WHITE]
gcode:
    RUN_SHELL_COMMAND CMD=led_white

[gcode_macro LED_PURPLE]
gcode:
    RUN_SHELL_COMMAND CMD=led_purple
```

----

rpi_ws281x library instructions for needed changes depending on GPIO pin used: https://github.com/jgarff/rpi_ws281x
