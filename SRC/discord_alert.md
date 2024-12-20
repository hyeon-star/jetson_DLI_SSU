---
# **Arduino Sensor Data Discord Notification System**

This project is a Python script that reads Arduino sensor data and sends notifications
via Discord webhooks based on specific conditions. A warning message is automatically sent when the soil moisture level falls below the threshold.

---

## **1. Project Overview**

### **Features**

1. Reads sensor data through serial communication with Arduino.
2. Sends a notification message to Discord webhook when soil moisture level is low.
3. Transmits all sensor data to Discord every 10 seconds.

---

## **2. Full Code**

```python
import serial  # For serial communication
import requests  # For HTTP requests
import time  # For time delays

# Configuration
WEBHOOK_URL = "please enter your webhook url"  # Enter the Discord webhook URL
SERIAL_PORT = "/dev/ttyUSB0"  # Check Arduino serial port (can use `ls /dev/tty*` to find)
BAUD_RATE = 9600  # Set baud rate to match Arduino

# Send a message to Discord
def send_to_discord(message):
    payload = {"content": message}  # Discord message format
    try:
        response = requests.post(WEBHOOK_URL, json=payload)
        if response.status_code == 204:
            print("Message successfully sent to Discord!")
        else:
            print(f"Failed to send: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Read sensor data from Arduino
def read_sensor_data():
    try:
        # Start serial communication
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            print("Connecting to Arduino...")
            time.sleep(2)  # Wait for stable connection
            while True:
                if ser.in_waiting > 0:
                    # Read data
                    data = ser.readline().decode('utf-8').strip()  # Read data from Arduino
                    print(f"Sensor data received: {data}")

                    # Parse data (e.g., "Soil Moisture:75, Light Intensity:28, Humidity:15, Temperature:24")
                    try:
                        values = {kv.split(":")[0].strip(): int(kv.split(":")[1].strip())
                                  for kv in data.split(",")}

                        # Check soil moisture condition
                        soil_moisture = values.get("Soil Moisture", 0)  # Use English keys from sensor data
                        if soil_moisture < 30:
                            alert_message = f"⚠️ Soil moisture is low! Please water the plants! Current moisture level: {soil_moisture}"
                            print(alert_message)
                            send_to_discord(alert_message)

                        # Send all sensor data to Discord
                        send_to_discord(f"Sensor data alert: {data}")

                    except Exception as parse_error:
                        print(f"Data parsing error: {parse_error}")

                    time.sleep(10)  # Send data every 10 seconds

    except Exception as e:
        print(f"Serial communication error: {e}")

# Execute the main function
if __name__ == "__main__":
    read_sensor_data()
```

---

## **3. Setup and Usage**

### **Environment Setup**
1. **Install Python Libraries**
   Install the required libraries:
    ```bash
   pip install pyserial requests
   ```

2. **Set Discord Webhook URL**
   Generate a webhook URL from your Discord channel and input it into the `WEBHOOK_URL` in the code.

3. **Check Arduino Serial Port**
   Identify the appropriate port for your system and set it in `SERIAL_PORT`:
   - **Linux/Mac**: `/dev/ttyUSB0` or `/dev/ttyACM0`
   - **Windows**: `COM3`, `COM4`, etc.
4. **Arduino Baud Rate**
   Set the `BAUD_RATE` to match the one used in your Arduino code.

---
### **Execution**
```bash
python your_script_name.py
```

---

## **4. Key Features Description**
1. **Serial Communication**
   Reads sensor data from Arduino and decodes it in UTF-8.

2. **Data Parsing**
   Parses sensor data when in the following format:
   ```
    "Soil Moisture: 250, Light Intensity: 500, Temperature: 25"
   ```
3. **Discord Notifications**
   - **Low Soil Moisture Condition**: Sends a warning message to Discord if `Soil Moisture < 30`.
   - **Periodic Data Transmission:** Sends all sensor data to Discord every 10 seconds.

---

## **5. Example Output**

### **Terminal Output**

```

Connecting to Arduino...
Sensor data received: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Message successfully sent to Discord!
Sensor data alert: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```
### **Example Discord Message**
```
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Sensor data alert: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```

---

## **6. Precautions**
1. **Discord Webhook Security**: Ensure the webhook URL is not exposed to external sources.
2. **Port Configuration**: The SERIAL_PORT may vary depending on your system, so verify it carefully.
3. **Sensor Value Adjustment**: Adjust condition values (e.g., soilMoisture < 30) to match your sensors and environment.

---










