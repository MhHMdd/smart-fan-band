# ⚽ Smart Fan Bracelet

The Smart Fan Bracelet is a wearable device designed to enhance the football stadium experience using NFC ticketing, health monitoring, motion detection, and live updates.

## 🔥 Features

- 🎟️ NFC ticket scanning via HALJIA PN532
- 📺 Live match updates on a 2.7" display
- 📡 Motion detection via MPU-6050
- 💓 Heartbeat monitoring
- 🎉 RGB light effects
- 🛑 Vibration alerts for emergencies
- 🧠 Powered by Raspberry Pi

---

## 🧰 Hardware Components

| Component         | Model                        |
|------------------|------------------------------|
| MCU              | Raspberry Pi 4 Model B (8GB) |
| NFC Module       | HALJIA PN532                 |
| Screen           | 2.7" E-Paper or OLED         |
| Accelerometer    | MPU-6050                     |
| RGB Lighting     | WS2812 / Generic RGB LED     |
| Heartbeat Sensor | HW-827                       |
| Vibration Motor  | Generic Mini Motor           |

---

## 🧑‍💻 Software & Technologies

- Python (Flask for API)
- SPI / I2C protocols
- `adafruit-circuitpython-pn532`, `MCP3008`, `smbus2`, etc.
- Frontend (optional) for live dashboard via browser

---

## 📷 Wiring Diagram

See `hardware/wiring-diagram.png`.

---

## 🧪 Setup Instructions

```bash
sudo apt update
sudo apt install python3-pip
pip3 install -r requirements.txt
python3 software/app.py

import time
import RPi.GPIO as GPIO
import digitalio
import spidev
from adafruit_mcp3xxx.mcp3008 import MCP3008
import board
import busio
from flask import Flask, jsonify
from threading import Thread

# GPIO Setup for vibration motor
GPIO.setmode(GPIO.BCM)
VIBRATION_PIN = 26
GPIO.setup(VIBRATION_PIN, GPIO.OUT)

# SPI Setup using Adafruit's busio, which works better with MCP3008
spi = busio.SPI(clock=board.SCK, MISO=board.MISO, MOSI=board.MOSI)

# Chip select pin using digitalio
CS_PIN = digitalio.DigitalInOut(board.D8)  # Use the appropriate pin

# Initialize MCP3008 with SPI and CS pin
mcp = MCP3008(spi, CS_PIN)

# Heartbeat sensor pin (using MCP3008 channels 0)
HEARTBEAT_CHANNEL = 0

# Flask app
app = Flask(__name__)

# Function to read heartbeat from MCP3008
def read_heartbeat():
    return mcp.read(HEARTBEAT_CHANNEL)

# Function to trigger vibration motor
def trigger_vibration():
    GPIO.output(VIBRATION_PIN, GPIO.HIGH)
    time.sleep(5)  # Vibration duration
    GPIO.output(VIBRATION_PIN, GPIO.LOW)
    time.sleep(2)
    GPIO.output(VIBRATION_PIN, GPIO.HIGH)
    time.sleep(2)
    GPIO.output(VIBRATION_PIN, GPIO.LOW)

# Route to return heartbeat value as JSON
@app.route('/heartbeat', methods=['GET'])
def heartbeat():
    heartbeat_value = read_heartbeat()
    return jsonify({'heartbeat': heartbeat_value})

# Route to trigger vibration motor
@app.route('/trigger_vibration', methods=['GET', 'POST'])
def trigger_vibration_route():
    trigger_vibration()
    return jsonify({'status': 'Vibration triggered!'})

# Run the Flask app without reloader or debugger
def run_flask():
    app.run(host='0.0.0.0', port=5000, debug=False, use_reloader=False)

# Start the Flask app in a separate thread to allow it to run alongside other tasks
if __name__ == "__main__":
    flask_thread = Thread(target=run_flask)
    flask_thread.start()


  
