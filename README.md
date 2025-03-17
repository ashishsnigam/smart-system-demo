# Smart System Demo

Install Home Automation Setup on your local development machine using any of the methods given here - https://www.home-assistant.io/installation/. For the scope of this demo, we are using docker based solution to do hands-on session and playaround.

```bash
# Create a directory on your local VM to save configuration file. It will be used for volume mapping in docker container. Eg:
mkdir -p C:\smart_system_demo\config

# Spin the docker container using command
docker run -d --name homeassistant --privileged --restart=unless-stopped -v C:\smart_system_demo\config:/config -p 8123:8123 ghcr.io/home-assistant/home-assistant:stable

# Open the front-end
http://localhost:8123/

# First time setup will require you to create a username/password. Don't loose it.
```

# Home Assistant Setup Troubleshooting Guide

This document provides solutions for common issues when working with Home Assistant in Docker containers, particularly on Windows systems, and integrating with various sensors and API services.

## Table of Contents
- [Docker Container Internet Access Issues](#docker-container-internet-access-issues)
- [PyPI and pip Connection Problems](#pypi-and-pip-connection-problems)
- [Certificate Errors](#certificate-errors)
- [Proxy Configuration](#proxy-configuration)
- [VPN Issues](#vpn-issues)
- [OpenWeatherMap Integration](#openweathermap-integration)
- [Device Simulation](#device-simulation)

## Docker Container Internet Access Issues

When Home Assistant Docker containers can't access the internet on Windows:

### Network Configuration
```bash
# Try host network mode
docker run --network=host --name homeassistant ...

# Or create a custom bridge network
docker network create ha-network
docker run --network=ha-network --name homeassistant ...
```

### DNS Issues
```bash
# Specify DNS servers explicitly
docker run --dns 8.8.8.8 --dns 8.8.4.4 --name homeassistant ...
```
Restart Docker Desktop afterward.

### Test Connectivity
```bash
# Test network connection from inside container
docker ps --> to get container_name or container_id
docker exec -it homeassistant ping google.com
```

### Windows Firewall
1. Check if Windows Defender Firewall is blocking Docker
2. Add exceptions for Docker and the container ports (typically 8123)

### WSL2 Network Issues
```bash
# Restart WSL
wsl --shutdown
```

## PyPI and pip Connection Problems

### Temporary Solution
```bash
# Use trusted-host to bypass verification
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org PACKAGE_NAME
```

### Permanent Solution
Create or edit `~/.config/pip/pip.conf`:
```
[global]
trusted-host = 
    pypi.org
    files.pythonhosted.org
```

### Timeout Configuration
```bash
# Increase the connection timeout
pip install --timeout=100 PACKAGE_NAME
```

## Certificate Errors

### Update System Certificates
```bash
# Debian/Ubuntu
sudo apt update
sudo apt install --reinstall ca-certificates

# CentOS/RHEL
sudo yum update ca-certificates

# Arch Linux
sudo pacman -S --needed ca-certificates
```

### Update Python Certificates
```bash
# Update certifi package
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org certifi
python -m pip install --upgrade certifi
```

### Check System Time
Incorrect system time can cause certificate validation failures. Ensure your system clock is accurate.

## Proxy Configuration

### Command-line Proxy
```bash
# Specify proxy for a single command
pip install --proxy http://user:password@proxyserver:port PACKAGE_NAME
```

### Environment Variables
```bash
# HTTP proxy
export HTTP_PROXY="http://username:password@proxyserver:port"
export HTTPS_PROXY="http://username:password@proxyserver:port"
```

### Make Proxy Settings Permanent
```bash
# Add to shell configuration file
echo 'export HTTP_PROXY="http://username:password@proxyserver:port"' >> ~/.bashrc
echo 'export HTTPS_PROXY="http://username:password@proxyserver:port"' >> ~/.bashrc
source ~/.bashrc
```

## VPN Issues

### Split Tunneling
Configure your VPN to use split tunneling and exclude:
- Docker subnets
- Home Assistant server addresses
- API endpoints (like pypi.org, openweathermap.org)

### Temporary VPN Disconnect
Temporarily disable VPN to test if it's causing the connectivity issues.

### Check VPN DNS Settings
Some VPNs override DNS settings which can interfere with Docker networking.

## OpenWeatherMap Integration

### Get API Key
1. Create an account at [OpenWeatherMap](https://openweathermap.org/)
2. Generate a free API key (1,000 calls/day limit)

### UI Integration
1. Go to Configuration > Integrations
2. Click "+ ADD INTEGRATION"
3. Search for "OpenWeatherMap"
4. Enter your API key and location

### YAML Configuration
```yaml
# Example configuration.yaml entry
weather:
  - platform: openweathermap
    api_key: YOUR_API_KEY_HERE
    latitude: YOUR_LATITUDE
    longitude: YOUR_LONGITUDE
    mode: daily
```

### Optimize API Calls
```yaml
# Reduce API calls by increasing scan interval
weather:
  - platform: openweathermap
    api_key: YOUR_API_KEY_HERE
    scan_interval: 1800  # 30 minutes
```

## Device Simulation

### Using Existing Simulators

1. **MQTT Device Emulator**
   ```bash
   git clone https://github.com/stjohnjohnson/mqtt-device-emulator.git
   cd mqtt-device-emulator
   npm install
   npm start -- --broker mqtt://broker-address:1883
   ```

### Custom Python Simulator

When you need a custom simulator, install the required dependencies:
```bash
pip install paho-mqtt
```

Then run the simulator:
```bash
python simulator.py --host YOUR_BROKER_IP --type temperature --id kitchen
```

### MQTT Configuration in Home Assistant
```yaml
# In configuration.yaml
mqtt:
  broker: YOUR_BROKER_IP
```

---

This troubleshooting guide covers the most common issues encountered when working with Home Assistant in Docker containers. For more specific problems, consult the [Home Assistant Documentation](https://www.home-assistant.io/docs/) or the [Community Forums](https://community.home-assistant.io/).
