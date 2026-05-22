# Network Rack Layout

```
                    INTERNET
                        │
                 ┌─────────────┐
                 │ ZTE H188A   │
                 │ (Bridge /   │
                 │ Modem       │
                 └──────┬──────┘
                        │
                ┌─────────────┐
                │ TP-Link     │
                │ ER605       │
                │ Router      │
                └──────┬──────┘
                       │
                ┌──────┴──────┐
                │ TP-Link     │
                │ ES208G      │
                │ Omada       │
                │ Switch      │
                └──────┬──────┘
                       │
        ┌──────────────┼──────────────┬──────────────┐
        │              │              │              │
        │              │              │              │
┌───────┴───────┐ ┌────┴──────┐ ┌─────┴─────┐ ┌─────┴─────┐
│ CQ57 Server   │ │ TP-Link   │ │ HP Laser  │ │ Future    │
│ Debian +      │ │ EAP115 V4 │ │ Jet 1010  │ │ Devices / │
│ Docker        │ │ Wi-Fi AP  │ │ Printer   │ │ NAS       │
└───────────────┘ └───────────┘ └───────────┘ └───────────┘
```

---

# Hardware Chart

| Device | Purpose | Notes |
| --- | --- | --- |
| TP-Link ER605 | Main router/firewall |Main DHCP + routing |
| TP-Link ES208G | Main switch | Omada easy managed switch |
| TP-Link EAP115 V4 | Main Wi-Fi AP | Omada AP with VLAN SSIDs |
| Compaq Presario CQ57 | Main homelab server | Debian + Docker server |
| ZTE H188A | ISP modem | Bridge mode |
| HP LaserJet 1010 | Network printer | Shared using CUPS |
| IKEA LACK table | Rack structure | DIY rack |
| External HDD/SSD | NAS storage | Recommended |
| UPS | Power backup | APC preferred |
| USB Gigabit Adapter | Faster networking | If CQ57 only has Fast Ethernet |

---

# CQ57 Server Role

## OS

- Debian 13

---

# Services

- Home Assistant
- Omada Software Controller
- AdGuard Home
- Portainer
- Uptime Kuma
- Jellyfin
- Samba
- FileBrowser
- Tailscale
- CUPS
- HPLIP
- Avahi

---

# Docker Setup

## Docker Stack

```bash
Debian 13
└── Docker Engine
    ├── Docker Compose
    ├── Portainer
    ├── AdGuard Home
    ├── Home Assistant
    ├── Tailscale
    ├── Jellyfin
    ├── FileBrowser
    ├── Uptime Kuma
    ├── n8n
    └── Homepage
```

---

## Docker Folder Layout

```bash
/home/homelab/
├── compose/
│   └── docker-compose.yml
├── data/
│   ├── adguard/
│   ├── homeassistant/
│   ├── jellyfin/
│   ├── filebrowser/
│   ├── uptime-kuma/
│   ├── n8n/
│   └── homepage/
├── backups/
└── scripts/
```

---

## Docker Compose Stack

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ../data/portainer:/data

  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000"
      - "80:80"
    volumes:
      - ../data/adguard/work:/opt/adguardhome/work
      - ../data/adguard/conf:/opt/adguardhome/conf

  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    network_mode: host
    privileged: true
    volumes:
      - ../data/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro

  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    network_mode: host
    volumes:
      - ../data/jellyfin/config:/config
      - ../data/jellyfin/cache:/cache
      - /media:/media

  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ../data:/srv

  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ../data/uptime-kuma:/app/data

  tailscale:
    image: tailscale/tailscale:latest
    container_name: tailscale
    restart: unless-stopped
    network_mode: host
    privileged: true
    volumes:
      - ../data/tailscale:/var/lib

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    volumes:
      - ../data/n8n:/home/node/.n8n

  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    restart: unless-stopped
    ports:
      - "3002:3000"
    volumes:
      - ../data/homepage:/app/config
```

---

# Main Docker Commands

```bash
docker compose up -d
docker compose down
docker compose restart
docker ps
docker stats
```

---

# Container Access URLs

| Service | URL |
| --- | --- |
| Portainer | [https://SERVER-IP:9443](https://server-ip:9443/) |
| AdGuard Home | [http://SERVER-IP:3000](http://server-ip:3000/) |
| Home Assistant | [http://SERVER-IP:8123](http://server-ip:8123/) |
| Jellyfin | [http://SERVER-IP:8096](http://server-ip:8096/) |
| FileBrowser | [http://SERVER-IP:8080](http://server-ip:8080/) |
| Uptime Kuma | [http://SERVER-IP:3001](http://server-ip:3001/) |
| n8n | [http://SERVER-IP:5678](http://server-ip:5678/) |
| Homepage | [http://SERVER-IP:3002](http://server-ip:3002/) |

---

## Software Setup

# Installed Directly On Debian

| Software | Purpose |
| --- | --- |
| Omada Software Controller | Network controller |
| Samba | Network shared folders |
| CUPS | Print server |
| HPLIP | HP printer drivers |
| Avahi Daemon | Network discovery |
| Cockpit (Optional) | Web admin panel |

---

# Omada Software Controller

Controls:

- TP-Link ER605
- TP-Link ES208G
- TP-Link EAP115 V4

## Installation

```bash
sudo apt install openjdk-17-jre-headless -y
```

Download:

- Omada Software Controller Debian package

Install:

```bash
sudo dpkg -i Omada_SDN_Controller*.deb
sudo apt --fix-broken install -y
```

Start service:

```bash
sudo systemctl enable omada
sudo systemctl start omada
```

Access:

```
https://SERVER-IP:8043
```

---

# Samba

```bash
sudo apt install samba -y
```

---

# CUPS + HPLIP

```bash
sudo apt install cups hplip -y
```

Enable:

```bash
sudo systemctl enable cups
sudo systemctl start cups
```

Access:

```
http://SERVER-IP:631
```

---

# Avahi

```bash
sudo apt install avahi-daemon -y
```

Enable:

```bash
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

---

# Cockpit (Optional)

```bash
sudo apt install cockpit -y
```

Enable:

```bash
sudo systemctl enable cockpit.socket
sudo systemctl start cockpit.socket
```

Access:

```
https://SERVER-IP:9090
```

---

# Storage Layout

```
128GB SSD (Internal)
├── Debian
├── Docker
├── Containers
└── Configs
```

```
External HDD/SSD
├── Photos
├── Videos
├── Documents
├── Backups
├── Media
├── Downloads
└── Home Assistant Backups
```

---

# Power Usage Estimate

| Device | Approx Power |
| --- | --- |
| ER605 | ~5W |
| ES208G | ~5–10W |
| EAP115 V4 | ~3–5W |
| CQ57 | ~15–30W |
| ZTE H188A | ~10W |

## Total

### ~40–60W overall

Very efficient for 24/7 operation.

---

# Best Future Upgrades

## Priority Order

1. External SSD/HDD
2. UPS
3. Better AP (Wi-Fi 6)
4. Gigabit USB NIC
5. Mini PC upgrade (N100)
6. PoE managed switch

---

# Base System

- Debian
    
    Main operating system for the CQ57 server.
    
- Docker Engine
    
    Runs all containerized applications.
    
- Docker Compose
    
    Manages multi-container services.
    

---

# Container Management

- Portainer
Docker web management UI.

---

# Networking Stack

- Omada Software Controller
    
    Controls:
    
    - ER605
    - ES208G
    - EAP115 V4
- AdGuard Home
    
    DNS filtering + ad blocking.
    
- Tailscale
    
    Secure remote access VPN.
    

---

# Smart Home / Automation

- Home Assistant
    
    Smart home platform.
    
- n8n
    
    Workflow automation platform.
    

---

# Media / Files

- Jellyfin
    
    Media streaming server (Direct Play only).
    
- Samba
    
    Network shared folders.
    
- FileBrowser
    
    Web-based file manager and photo browser.
    

---

# Monitoring / Dashboard

- Uptime Kuma
    
    Service monitoring.
    
- Custom HTML Dashboard
    
    Lightweight dashboard for:
    
    - iPad Mini 1
    - old Safari
    - low-resource devices

---

# Printer Server

- CUPS
    
    Linux print server.
    
- HPLIP
    
    HP printer drivers.
    
- Avahi Daemon
    
    Apple printer/service discovery.
    

---

# Optional Server Management

- Cockpit
Lightweight Linux web administration UI.

---

# VLAN Plan (Future)

| VLAN | Purpose |
| --- | --- |
| VLAN 10 | Main devices |
| VLAN 20 | IoT |
| VLAN 30 | Guest Wi-Fi |
| VLAN 40 | Servers |

---

# Wi-Fi SSIDs

| SSID | VLAN | Purpose |
| --- | --- | --- |
| Home | 10 | Main devices |
| IoT | 20 | Smart home devices |
| Guest | 30 | Isolated guest Wi-Fi |

---

# Hardware Roles

| Device | Role |
| --- | --- |
| Compaq Presario CQ57 | Main homelab server |
| TP-Link ER605 | Main router/DHCP |
| TP-Link ES208G | Main switch |
| TP-Link EAP115 V4 | Main Wi-Fi AP |
| ZTE H188A | ISP modem/bridge |
| HP LaserJet 1010 | Network printer |
| iPad Mini 1 | Dashboard display |
| External HDD/SSD | Storage/media/photos |

---

# Main Goals Of The Rack

- Compact homelab
- Network appliance
- Media/file server
- Smart home hub
- Remote access node
- Print server
- Custom dashboard system
- Low-power always-on server
- VLAN-ready network
- Omada ecosystem setup

---

# esp32

## **🛠️ Hardware Wiring Diagram (ESP32 DevKit V1)**

Both W5500 modules share the hardware **SPI pins (GPIO 18, 19, 23)**. They use separate **Chip Select (CS)** and **Reset (RST)** pins so the ESP32 can test them one at a time. The OLED uses the standard **I2C pins (GPIO 21, 22)**.

| Component | Component Pin | ESP32 DevKit V1 Pin | Notes |
| --- | --- | --- | --- |
| **W5500 #1 (Net 1)** | **MISO**  **MOSI**  **SCLK / SCK**  **CS**  **RST**  VCC / GND | **D19**  **D23**  **D18**  **D5**  **D2**  3V3 / GND | Shared SPI BusShared SPI BusShared SPI BusDedicated CS for Net 1Dedicated Reset for Net 1Power |
| **W5500 #2 (Net 2)** | **MISO**  **MOSI**  **SCLK / SCK**  **CS**  **RST**  VCC / GND | **D19**  **D23**  **D18**  **D15**  **D4**  3V3 / GND | Shared SPI BusShared SPI BusShared SPI BusDedicated CS for Net 2Dedicated Reset for Net 2Power |
| **OLED Display** | **SDA**  **SCL**  VCC / GND | **D21**  **D22**  3V3 / GND | I2C DataI2C Clock |
| **DS18B20 Temp** | **DATA**  VCC / GND | **D27**  3V3 / GND | **Requires 4.7kΩ resistor** between DATA & 3V3 |
| **Status LEDs** | **RED LED**  **GREEN LED**  **BLUE LED** | **D12**  **D13**  **D14** | Connect each via a 220Ω resistor to **GND** |

---

## **📜 Complete Production Code**

Make sure your Arduino IDE has the **Adafruit SSD1306**, **DallasTemperature**, and standard **Ethernet** libraries installed before flashing this.

```jsx
#include <SPI.h>
#include <Ethernet.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <OneWire.h>
#include <DallasTemperature.h>

// --- ESP32 DevKit V1 Pin Mapping ---
#define W5500_N1_CS   5   // D5
#define W5500_N1_RST  2   // D2
#define W5500_N2_CS   15  // D15
#define W5500_N2_RST  4   // D4

#define ONE_WIRE_BUS 27  // D27 (Requires 4.7k pull-up resistor)

#define LED_RED      12  // D12
#define LED_GREEN    13  // D13
#define LED_BLUE     14  // D14

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// Unique MAC addresses for each Ethernet controller
byte mac1[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0x11 };
byte mac2[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0x22 };

// Independent remote targets to verify real internet connectivity
IPAddress target1(8, 8, 8, 8); // Google DNS
IPAddress target2(1, 1, 1, 1); // Cloudflare DNS

bool net1_up = false;
bool net2_up = false;
float currentTemp = 0.0;

// Hard reset both W5500 modules to clear old cache registers
void resetEthernetHardware() {
  pinMode(W5500_N1_RST, OUTPUT);
  pinMode(W5500_N2_RST, OUTPUT);
  
  digitalWrite(W5500_N1_RST, LOW);
  digitalWrite(W5500_N2_RST, LOW);
  delay(50);
  digitalWrite(W5500_N1_RST, HIGH);
  digitalWrite(W5500_N2_RST, HIGH);
  delay(300); 
}

void setup() {
  Serial.begin(115200);
  
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_BLUE, OUTPUT);

  // Turn all LEDs on briefly to verify they work
  digitalWrite(LED_RED, HIGH);
  digitalWrite(LED_GREEN, HIGH);
  digitalWrite(LED_BLUE, HIGH);
  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("OLED allocation failed"));
    for(;;);
  }
  
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);
  display.setCursor(0,0);
  display.println("Initializing System...");
  display.display();

  sensors.begin();
  resetEthernetHardware();

  // Initialize Network Port 1 via DHCP
  Ethernet.init(W5500_N1_CS);
  Serial.println("Configuring Net 1...");
  if (Ethernet.begin(mac1, 3000) == 0) {
    Serial.println("Net 1 DHCP Failed");
  } else {
    Serial.print("Net 1 IP assigned: "); Serial.println(Ethernet.localIP());
  }

  // Initialize Network Port 2 via DHCP
  Ethernet.init(W5500_N2_CS);
  Serial.println("Configuring Net 2...");
  if (Ethernet.begin(mac2, 3000) == 0) {
    Serial.println("Net 2 DHCP Failed");
  } else {
    Serial.print("Net 2 IP assigned: "); Serial.println(Ethernet.localIP());
  }
  
  // Turn off startup test LEDs
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_GREEN, LOW);
  digitalWrite(LED_BLUE, LOW);
}

// Tests isolated connection performance by switching SPI chip lines
bool validateNetworkPath(uint8_t csPin, IPAddress destination) {
  Ethernet.init(csPin);
  Ethernet.maintain(); // Keep DHCP leases refreshed
  
  if (Ethernet.linkStatus() == LinkOFF) {
    return false; // Physical cable is disconnected
  }

  EthernetClient client;
  // Attempt to open a temporary TCP socket handshake to verify external gateway path
  if (client.connect(destination, 53, 1500)) { 
    client.stop();
    return true; // Successfully reached the target server
  }
  return false; // Host unreachable
}

void loop() {
  // 1. Gather environmental metrics
  sensors.requestTemperatures();
  currentTemp = sensors.getTempCByIndex(0);

  // 2. Query network availability independently 
  net1_up = validateNetworkPath(W5500_N1_CS, target1);
  net2_up = validateNetworkPath(W5500_N2_CS, target2);

  // 3. Manage LED Notification Status
  if (net1_up && net2_up) {
    digitalWrite(LED_GREEN, HIGH);
    digitalWrite(LED_RED, LOW);
  } else {
    // If either or both networks drop out, trigger the alert line
    digitalWrite(LED_GREEN, LOW);
    digitalWrite(LED_RED, HIGH); 
  }

  // Flash the Blue LED if server rack ambient temperature is over 35°C
  if (currentTemp > 35.0) {
    digitalWrite(LED_BLUE, !digitalRead(LED_BLUE)); 
  } else {
    digitalWrite(LED_BLUE, LOW);
  }

  // 4. Draw Diagnostic UI to OLED Screen
  display.clearDisplay();
  display.setTextSize(1);
  
  display.setCursor(0, 0);
  display.print("NET 1 (D5):  ");
  if(net1_up) display.println("ONLINE");
  else display.println("OFFLINE");
  
  display.setCursor(0, 16);
  display.print("NET 2 (D15): ");
  if(net2_up) display.println("ONLINE");
  else display.println("OFFLINE");
  
  // Display current sensor temp tracking at the bottom
  display.setCursor(0, 40);
  display.setTextSize(2);
  display.print("Temp: ");
  display.print(currentTemp, 1);
  display.print("C");
  
  display.display();

  delay(4000); // Rerun validation sweep every 4 seconds
}

```
