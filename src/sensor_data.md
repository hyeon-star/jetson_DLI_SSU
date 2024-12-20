---

## Collecting Data from Arduino Sensors (`collect_sensor_data.md`)

This script provides code for collecting real-time data from Arduino sensors using Python and Pandas and saving it as a CSV file.
The code is designed to run in a virtual environment on Jetson Nano through Jupyter Notebook.

---

### **ALL CODE**

```python
import serial
import time
import pandas as pd

# Serial communication setup
port = "/dev/ttyUSB0"  # Serial port for Arduino
baudrate = 9600        # Baud rate matching Arduino
arduino = serial.Serial(port, baudrate, timeout=1)

# List to store data
data = []

# Initialize current date
current_date = time.strftime("%Y-%m-%d")

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # Check if data is available on the serial port
            line = arduino.readline().decode("utf-8").strip()  # Read and decode the data
            
            # Filter data: process only sensor data
            if "Soil Moisture:" in line and "Light Intensity:" in line:
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # Store the current time in seconds
                
                # Parse sensor data
                parts = line.split(", ")
                soil_moisture = parts[0].split(":")[1].strip()  # Soil Moisture value
                light_intensity = parts[1].split(":")[1].strip()  # Light Intensity value
                humidity = parts[2].split(":")[1].strip()  # Humidity value
                temperature = parts[3].split(":")[1].strip()  # Temperature value
                
                print(f"{timestamp}, Soil Moisture: {soil_moisture}, Light Intensity: {light_intensity}, Humidity: {humidity}, Temperature: {temperature}")
                
                # Save data
                data.append({
                    "Timestamp": timestamp,
                    "Soil Moisture (%)": soil_moisture,
                    "Light Intensity (%)": light_intensity,
                    "Humidity (%)": humidity,
                    "Temperature (C)": temperature
                })

                # Create a DataFrame
                df = pd.DataFrame(data)

                # Generate file name (based on current date)
                file_name = f"sensor_data_{current_date.replace('-', '')}.csv"

                # Save DataFrame to a CSV file
                df.to_csv(file_name, index=False, encoding="utf-8")
                print(f"Data saved to {file_name}")

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # Save the remaining data
    if data:
        df = pd.DataFrame(data)
        file_name = f"sensor_data_{current_date.replace('-', '')}.csv"
        df.to_csv(file_name, index=False, encoding="utf-8")
        print(f"Final data saved to {file_name}")

    # Close the port
    arduino.close()

```
---

### **Project Overview**

1. This script performs the following functions:
2.Arduino Serial Communication Setup: Sets up a serial port connection to read data.
3.Data Filtering and Parsing: Reads sensor data (soil moisture, light intensity, temperature, and humidity) sent from Arduino and accurately parses it.
3. Real-Time Data Saving: Saves collected data to a CSV file in real-time.
4. Data Preservation on Exit: Ensures remaining data is safely stored upon program termination.

---

### **Usage Instructions**

1.Upload and run the Arduino code (mainarduino.md).
2.Run the Python script:
  ```bash
  python collect_sensor_data.py
  ```
3. You can press Ctrl+C to stop the program while it is running.
4. The collected data will be saved in a `sensor_data_YYYYMMDD.csv` file.

### **Example Output (Terminal)**

```
2024-06-01 14:23:45, Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Data saved to sensor_data_20240601.csv
2024-06-01 14:23:47, Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
Data saved to sensor_data_20240601.csv
```

---

### **Example File (CSV)**

| Timestamp           | Soil Moisture (%) | Light Intensity (%) | Humidity (%) | Temperature (C) |
|---------------------|-------------------|---------------------|-------------|-----------------|
| 2024-06-01 14:23:45 | 45                | 78                  | 60          | 24              |
| 2024-06-01 14:23:47 | 43                | 80                  | 59          | 23              |

---

### **Precautions**

1. Serial Port Configuration: Modify the serial port to match your system, such as `/dev/ttyUSB0` (Linux/Mac) or `COM3` (Windows).
2. Baud Rate: Ensure the baud rate (`9600`) matches in both the Arduino code and the Python script.
3. Data Saving on Exit: When exiting with `Ctrl+C`, all collected data up to the last moment will be saved.

---













