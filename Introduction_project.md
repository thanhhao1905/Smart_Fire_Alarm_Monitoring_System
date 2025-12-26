# Smart Fire Alarm Monitoring System

## Project Overview
The Smart Fire Alarm Monitoring System is an Internet of Things (IoT) based safety solution that combines Field-Programmable Gate Array (FPGA) hardware processing with ESP32 microcontroller wireless communication capabilities. This system implements real-time environmental monitoring and emergency alert functionality through the integration of multiple sensor technologies. The project provides comprehensive fire safety monitoring by combining temperature/humidity detection, smoke sensing, and wireless connectivity in a unified platform.

## Key Features

### Multi-Sensor Integration
- **DHT11 Sensor**: Real-time temperature and humidity monitoring with FPGA-based precision timing control
- **MQ-2 Smoke Sensor**: Analog and digital smoke detection with configurable sensitivity thresholds
- **ESP32 Processing**: Advanced data processing and wireless communication capabilities

### Dual Communication Architecture
- **UART Communication**: High-speed data transfer between FPGA and ESP32 processors
- **WiFi Connectivity**: Cloud integration via ThingsBoard IoT platform
- **Real-time Telemetry**: Continuous data streaming to cloud dashboard

### Intelligent Alert System
- **Multi-level Warning**: Visual (LED indicators), audible (buzzer), and remote notifications
- **Configurable Thresholds**: Dynamic adjustment of alarm parameters through cloud interface
- **Emergency Calling**: GSM-based automatic emergency dialing system activation

### Advanced Display Interfaces
- **LCD Display**: I2C-controlled 16×2 LCD for local sensor readings
- **7-Segment Display**: Four-digit LED display for smoke concentration values
- **Real-time Updates**: Simultaneous local and remote data visualization

## System Architecture

### FPGA Subsystem (Verilog Implementation)
- **Sensor Management**: Precise timing control for DHT11 communication protocol
- **Data Processing**: Real-time sensor data acquisition and validation
- **Display Control**: LCD and 7-segment display drivers
- **UART Communication**: Efficient data packaging and transmission to ESP32

### ESP32 Subsystem (C++ Implementation)
- **Wireless Connectivity**: WiFi management and cloud communication
- **Data Aggregation**: Collection and processing from multiple sensor sources
- **Cloud Integration**: ThingsBoard telemetry and Remote Procedure Call (RPC) command handling
- **Emergency Response**: GSM-based calling system activation

## Technical Specifications

### Hardware Components
- **FPGA Board**: Main processing unit with sensor interfaces
- **ESP32**: WiFi/BLE module for wireless communication
- **DHT11**: Digital temperature and humidity sensor
- **MQ-2**: Analog smoke/gas detection sensor
- **LCD Display**: I2C 16×2 character display
- **7-Segment Display**: Four-digit LED display module
- **GSM Module**: SIM800L for emergency calling functionality

### Communication Protocols
- **UART**: FPGA ↔ ESP32 data exchange (9600 baud rate)
- **I2C**: LCD display control interface
- **SPI**: 74HC595 shift register interface for 7-segment display
- **HTTP/RPC**: Cloud communication with ThingsBoard platform

### Performance Parameters
- **Sampling Interval**: 2 seconds for environmental sensors
- **Response Time**: <100 milliseconds for alarm conditions
- **Temperature Accuracy**: ±2°C
- **Humidity Accuracy**: ±5%
- **Communication Latency**: <1 second for cloud telemetry

## Implementation Details

### FPGA Code Structure
The FPGA implementation consists of modular Verilog components:
1. **Clock Management**: Frequency division for precise timing
2. **Sensor Interface**: DHT11 protocol state machine
3. **Display Controllers**: LCD and 7-segment driver modules
4. **Communication Interface**: UART transmitter for ESP32 communication

### ESP32 Software Architecture
The ESP32 firmware includes:
1. **WiFi Management**: Connection handling and network stability
2. **Cloud Communication**: ThingsBoard MQTT client implementation
3. **Data Processing**: Sensor data aggregation and formatting
4. **Emergency System**: GSM module control and call management

## Safety Features
1. **Data Validation**: Checksum verification for sensor communications
2. **Redundant Alerts**: Multiple independent warning systems
3. **Manual Override**: Physical button control for warning mode
4. **Fault Detection**: Communication timeout and error handling

## Applications
- Residential fire safety systems
- Industrial environmental monitoring
- Laboratory safety equipment
- Educational IoT demonstrations
