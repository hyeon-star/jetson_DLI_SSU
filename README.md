# 🌱 **Project Name: Smart Environmental Data Monitoring System**

## 📖 **Project Overview**
This project utilizes **Jetson Nano** and **Arduino sensors** to monitor and manage environmental data. It collects real-time data on soil moisture, temperature, humidity, and illuminance, and notifies users of water scarcity situations via **Discord alerts**. Additionally, it implements a **Gradio UI-based chatbot** to store sensor data and make it easily accessible to users.

## 🚀 **Key Features**
- **Data Collection and Storage**:
  - **Soil Moisture Sensor**: Measures soil moisture in real-time.
  - **DHT11 Sensor**: Collects real-time temperature and humidity data.
  - **Illuminance Sensor**: Collects real-time brightness data.
  - All collected data is stored in a **CSV file**.

- **Automated Notification System**:
  - Sends instant Discord alerts to users in case of water scarcity.
- **Gradio UI-Based Chatbot**:
  - Provides a conversational chatbot interface for users to access sensor information.
  - Enhances accessibility with data visualization through Gradio UI.

## 🛠️ **Tech Stack**
- **Programming Languages**: Python, C++
- **Platforms/Frameworks**: Jetson Nano, Arduino, Gradio

- **Hardware**:
  - Jetson Nano
  - Arduino
  - Sensors: Soil Moisture Sensor, DHT11 (Temperature and Humidity Sensor), Illuminance Sensor

- **Other Tools**: Discord API, CSV Data Storage

## 📂 **Project Structure**
```plaintext
📁 Smart Environmental Data Monitoring System/
├── README.md             # Project description
├── src/                  # Source code
│   ├── main_arduino.md   # Main Arduino sensor measurement code
│   ├── sensor_data.md    # Sensor data collection code
│   ├── discord_alert.md  # Discord notification system code
│   └── chatbot_ui.md     # Gradio UI-based chatbot code
├── failed_attempts/
│   └── gradio_graph_attempt.md  # Gradio UI graph attempt code
├── data/                 # CSV data storage folder
│   └── sensor_data.csv   # Collected sensor data **sensor data 20241213.csv**
└── requirements.txt      # Dependency list

