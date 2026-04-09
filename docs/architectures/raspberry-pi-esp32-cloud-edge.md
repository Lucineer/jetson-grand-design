# Raspberry Pi + ESP32 Cloud-Edge Architecture

> **Codespaces as IoT IDE: Developing at the Very Edge**

## Executive Summary

This architecture enables cloud-heavy development for edge deployments using:
- **Raspberry Pi 5** as local hub and coordinator
- **ESP32** devices as distributed sensors/actuators with git-agent capability
- **GitHub Codespaces** as full-featured IoT IDE
- **Over-the-air deployment** to remote devices

This enables "develop anywhere, deploy everywhere" for marine, industrial, and remote installations.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Hardware Configuration](#2-hardware-configuration)
3. [ESP32 Git-Agent Design](#3-esp32-git-agent-design)
4. [Raspberry Pi Hub](#4-raspberry-pi-hub)
5. [GitHub Codespaces IDE](#5-github-codespaces-ide)
6. [Deployment Pipeline](#6-deployment-pipeline)
7. [Offline Considerations](#7-offline-considerations)
8. [Use Cases](#8-use-cases)
9. [Implementation Code](#9-implementation-code)
10. [Configuration Schemas](#10-configuration-schemas)

---

## 1. Architecture Overview

### 1.1 Concept Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     CLOUD-EDGE DEVELOPMENT ARCHITECTURE                          │
│                                                                                  │
│  ┌───────────────────────────────────────────────────────────────────────────┐  │
│  │                              CLOUD LAYER                                   │  │
│  │                                                                            │  │
│  │  ┌─────────────────────────────────────────────────────────────────────┐  │  │
│  │  │                    GITHUB CODESPACES (IoT IDE)                       │  │  │
│  │  │                                                                       │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │  │  │
│  │  │  │ Code Editor  │  │ Build System │  │ Test Suite   │               │  │  │
│  │  │  │ (VS Code)    │  │ (PlatformIO) │  │ (Unity/pytest)│              │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘               │  │  │
│  │  │                                                                       │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │  │  │
│  │  │  │ Simulation   │  │ Model Cache  │  │ Git Repo     │               │  │  │
│  │  │  │ (Wokwi/QEMU) │  │ (ML Models)  │  │ (Versioning) │               │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘               │  │  │
│  │  │                                                                       │  │  │
│  │  │  Developer Experience:                                                │  │  │
│  │  │  • Full VS Code in browser                                            │  │  │
│  │  │  • Pre-configured toolchains                                          │  │  │
│  │  │  • Hardware simulation                                                │  │  │
│  │  │  • One-click deployment                                               │  │  │
│  │  │                                                                       │  │  │
│  │  └─────────────────────────────────────────────────────────────────────┘  │  │
│  │                                       │                                    │  │
│  │                                       │ Secure WebSocket / HTTPS           │  │
│  │                                       │                                    │  │
│  └───────────────────────────────────────┼────────────────────────────────────┘  │
│                                          │                                       │
│                          ┌───────────────┼───────────────┐                       │
│                          │    INTERNET (when available)  │                       │
│                          └───────────────┼───────────────┘                       │
│                                          │                                       │
│  ┌───────────────────────────────────────┼───────────────────────────────────┐   │
│  │                              EDGE LAYER │                                 │   │
│  │                                        ▼                                  │   │
│  │  ┌─────────────────────────────────────────────────────────────────────┐  │   │
│  │  │                    RASPBERRY PI 5 (Local Hub)                        │  │   │
│  │  │                                                                       │  │   │
│  │  │  ┌───────────────────────────────────────────────────────────────┐   │  │   │
│  │  │  │                      SERVICES                                   │   │  │   │
│  │  │  │                                                                 │   │  │   │
│  │  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │  │   │
│  │  │  │  │ Sync Agent  │  │ Local LLM   │  │ MQTT Broker │            │   │  │   │
│  │  │  │  │ (Git ↔ ESP) │  │ (TinyLlama) │  │ (Mosquitto) │            │   │  │   │
│  │  │  │  └─────────────┘  └─────────────┘  └─────────────┘            │   │  │   │
│  │  │  │                                                                 │   │  │   │
│  │  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │   │  │   │
│  │  │  │  │ Sensor Hub  │  │ State Store │  │ API Gateway │            │   │  │   │
│  │  │  │  │ (Data Agg)  │  │ (SQLite)    │  │ (FastAPI)   │            │   │  │   │
│  │  │  │  └─────────────┘  └─────────────┘  └─────────────┘            │   │  │   │
│  │  │  │                                                                 │   │  │   │
│  │  │  └───────────────────────────────────────────────────────────────┘   │  │   │
│  │  │                                                                       │  │   │
│  │  │  Offline Capability:                                                  │  │   │
│  │  │  • Full operation without internet                                    │  │   │
│  │  │  • Local inference and control                                        │  │   │
│  │  │  • Queue changes for sync                                             │  │   │
│  │  │  • Auto-sync when connected                                           │  │   │
│  │  │                                                                       │  │   │
│  │  └─────────────────────────────────────────────────────────────────────┘  │   │
│  │                                       │                                   │   │
│  │                          ┌────────────┼────────────┐                      │   │
│  │                          │  LOCAL NETWORK (WiFi/Ethernet)                 │   │
│  │                          └────────────┼────────────┘                      │   │
│  │                                       │                                   │   │
│  │  ┌────────────────────────────────────┼───────────────────────────────┐  │   │
│  │  │                         DEVICE LAYER │                              │  │   │
│  │  │                                     ▼                               │  │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │  │   │
│  │  │  │ ESP32 #1 │ │ ESP32 #2 │ │ ESP32 #3 │ │ ESP32 #4 │ │ ESP32 #N │ │  │   │
│  │  │  │          │ │          │ │          │ │          │ │          │ │  │   │
│  │  │  │ Git-     │ │ Git-     │ │ Git-     │ │ Git-     │ │ Git-     │ │  │   │
│  │  │  │ Agent    │ │ Agent    │ │ Agent    │ │ Agent    │ │ Agent    │ │  │   │
│  │  │  │          │ │          │ │          │ │          │ │          │ │  │   │
│  │  │  │ Temp     │ │ GPS      │ │ Depth    │ │ Motor    │ │ Relay    │ │  │   │
│  │  │  │ Sensor   │ │ Logger   │ │ Sensor   │ │ Control  │ │ Board    │ │  │   │
│  │  │  │          │ │          │ │          │ │          │ │          │ │  │   │
│  │  │  │ OTA      │ │ OTA      │ │ OTA      │ │ OTA      │ │ OTA      │ │  │   │
│  │  │  │ Update   │ │ Update   │ │ Update   │ │ Update   │ │ Update   │ │  │   │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │  │   │
│  │  │                                                                     │  │   │
│  │  │  Each ESP32:                                                        │  │   │
│  │  │  • Receives code updates via git pull (simplified protocol)         │  │   │
│  │  │  • Reports status to Pi hub                                         │  │   │
│  │  │  • Runs autonomously once programmed                                │  │   │
│  │  │  • Can be updated over-the-air                                      │  │   │
│  │  │                                                                     │  │   │
│  │  └─────────────────────────────────────────────────────────────────────┘  │   │
│  │                                                                            │   │
│  └────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Key Innovation: ESP32 Git-Agent

The core innovation is the **ESP32 Git-Agent** — a microcontroller that can:
1. Pull code updates from the Pi hub (which syncs with GitHub)
2. Report its status and sensor data
3. Run autonomously for months
4. Be reprogrammed remotely without physical access

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ESP32 GIT-AGENT CONCEPT                          │
│                                                                     │
│  Traditional IoT Development:                                       │
│  ┌───────────┐      USB Cable      ┌───────────┐                   │
│  │  Laptop   │ ──────────────────▶ │   ESP32   │                   │
│  │  (IDE)    │      Physical       │  (Device) │                   │
│  └───────────┘      Access         └───────────┘                   │
│                                                                     │
│  Problem: Requires physical access, can't update remotely           │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  Git-Agent Approach:                                                │
│                                                                     │
│  ┌───────────┐    HTTPS/Push    ┌───────────┐    OTA/Git    ┌─────┴─────┐
│  │ Developer │ ───────────────▶ │  GitHub   │ ─────────────▶ │ Pi Hub    │
│  │ (Anywhere)│                  │  Repo     │                │ (Edge)    │
│  └───────────┘                  └───────────┘                └─────┬─────┘
│                                                                      │
│                                         OTA Update (HTTP/WebSocket)  │
│                                                                      │
│                                         ┌──────────┐                │
│                                         │  ESP32   │                │
│                                         │ Git-Agent│                │
│                                         │          │                │
│                                         │ 1. Check for updates      │
│                                         │ 2. Download new firmware  │
│                                         │ 3. Verify signature       │
│                                         │ 4. Apply update           │
│                                         │ 5. Report status          │
│                                         └──────────┘                │
│                                                                     │
│  Benefits:                                                          │
│  • Develop from anywhere (Codespaces)                               │
│  • No physical access needed for updates                            │
│  • Version-controlled firmware                                      │
│  • Rollback capability                                              │
│  • Fleet management                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Hardware Configuration

### 2.1 Raspberry Pi 5 Hub

| Component | Specification |
|-----------|---------------|
| **Model** | Raspberry Pi 5 (8GB RAM) |
| **Storage** | NVMe SSD 256GB (via M.2 HAT) |
| **Network** | Gigabit Ethernet + WiFi 6 |
| **Power** | 5V 5A USB-C PD |
| **Enclosure** | Marine/industrial rated |
| **OS** | Raspberry Pi OS 64-bit (Bookworm) |

### 2.2 ESP32 Device Options

| Model | Use Case | Features |
|-------|----------|----------|
| **ESP32-WROOM-32** | General sensors | WiFi, Bluetooth, 240MHz dual-core |
| **ESP32-S3** | ML/edge inference | AI acceleration, more GPIO |
| **ESP32-C3** | Low power sensors | RISC-V, ultra-low power |
| **ESP32-C6** | Thread/Matter | WiFi 6, Zigbee support |

### 2.3 Example Marine Installation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                 EXAMPLE: FISHING BOAT IoT DEPLOYMENT                             │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                      RASPBERRY PI 5 HUB                                  │    │
│  │                      (Mounted in cabin)                                  │    │
│  │                                                                          │    │
│  │  Connected to:                                                           │    │
│  │  • Main display (HDMI)                                                   │    │
│  │  • UPS battery backup                                                    │    │
│  │  • 4G/LTE modem for cloud sync                                           │    │
│  │                                                                          │    │
│  └────────────────────────────────────┬────────────────────────────────────┘    │
│                                       │                                          │
│                    ┌──────────────────┴──────────────────┐                       │
│                    │         WiFi / Ethernet              │                       │
│                    └──────────────────┬──────────────────┘                       │
│                                       │                                          │
│  ┌────────────────────────────────────┼────────────────────────────────────┐    │
│  │                           ESP32 FLEET                                  │    │
│  │                                                                         │    │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │    │
│  │  │ ESP32 #1        │  │ ESP32 #2        │  │ ESP32 #3        │         │    │
│  │  │ Helm Station    │  │ Engine Room     │  │ Deck Sensors    │         │    │
│  │  │                 │  │                 │  │                 │         │    │
│  │  │ • GPS backup    │  │ • Temp sensors  │  │ • Wind sensor   │         │    │
│  │  │ • Compass       │  │ • RPM monitor   │  │ • Bilge level   │         │    │
│  │  │ • Button input  │  │ • Fuel flow     │  │ • Hatch status  │         │    │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘         │    │
│  │                                                                         │    │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │    │
│  │  │ ESP32 #4        │  │ ESP32 #5        │  │ ESP32 #6        │         │    │
│  │  │ Fish Hold       │  │ Navigation      │  │ Safety          │         │    │
│  │  │                 │  │                 │  │                 │         │    │
│  │  │ • Temp/humidity │  │ • Lights        │  │ • Smoke detect  │         │    │
│  │  │ • Door status   │  │ • Horn          │  │ • CO detect     │         │    │
│  │  │ • Drain sensors │  │ • Bilge pump    │  │ • Manual alarm  │         │    │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘         │    │
│  │                                                                         │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.4 Bill of Materials

| Component | Quantity | Est. Cost | Purpose |
|-----------|----------|-----------|---------|
| Raspberry Pi 5 8GB | 1 | $80 | Local hub |
| NVMe HAT + 256GB SSD | 1 | $50 | Storage |
| ESP32-WROOM-32 DevKit | 6 | $8 x 6 = $48 | Sensors/actuators |
| ESP32-S3 (ML) | 1 | $12 | Edge inference |
| Marine enclosure (Pi) | 1 | $40 | Environmental protection |
| Waterproof sensor housings | 6 | $10 x 6 = $60 | ESP32 protection |
| Temperature sensors (DS18B20) | 4 | $2 x 4 = $8 | Engine/fish hold |
| GPS module (u-blox) | 1 | $15 | Backup GPS |
| Wind sensor | 1 | $25 | Weather data |
| Relay board (4-channel) | 2 | $5 x 2 = $10 | Control outputs |
| Power supply 5V 5A | 1 | $15 | Pi power |
| Buck converters (12V to 5V) | 6 | $2 x 6 = $12 | ESP32 power |
| UPS HAT | 1 | $35 | Backup power |
| 4G/LTE HAT | 1 | $45 | Cloud connectivity |
| **Total Estimated** | | **~$455** | |

---

## 3. ESP32 Git-Agent Design

### 3.1 Git-Agent Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ESP32 GIT-AGENT INTERNAL                         │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     FIRMWARE LAYOUT                          │   │
│  │                                                               │   │
│  │  ┌─────────────────────────────────────────────────────────┐│   │
│  │  │           OTA UPDATE PARTITION (~1MB)                    ││   │
│  │  │  • New firmware images                                   ││   │
│  │  │  • Rollback capability                                   ││   │
│  │  └─────────────────────────────────────────────────────────┘│   │
│  │                                                               │   │
│  │  ┌─────────────────────────────────────────────────────────┐│   │
│  │  │           APPLICATION PARTITION (~2MB)                   ││   │
│  │  │  • Main firmware                                        ││   │
│  │  │  • Sensor drivers                                       ││   │
│  │  │  • Business logic                                       ││   │
│  │  │  • MQTT client                                          ││   │
│  │  │  • Git-Agent client                                     ││   │
│  │  └─────────────────────────────────────────────────────────┘│   │
│  │                                                               │   │
│  │  ┌─────────────────────────────────────────────────────────┐│   │
│  │  │           FILESYSTEM (SPIFFS/LittleFS ~1MB)             ││   │
│  │  │  • Configuration files                                  ││   │
│  │  │  • Cached sensor data                                   ││   │
│  │  │  • Device identity                                      ││   │
│  │  │  • Last known state                                     ││   │
│  │  └─────────────────────────────────────────────────────────┘│   │
│  │                                                               │   │
│  │  ┌─────────────────────────────────────────────────────────┐│   │
│  │  │           NVS (Non-Volatile Storage)                     ││   │
│  │  │  • WiFi credentials                                     ││   │
│  │  │  • Encryption keys                                      ││   │
│  │  │  • Boot count                                           ││   │
│  │  │  • Firmware version                                     ││   │
│  │  └─────────────────────────────────────────────────────────┘│   │
│  │                                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  MEMORY MAP (4MB Flash):                                            │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ 0x000000 - 0x00FFFF: Bootloader                               │ │
│  │ 0x010000 - 0x0FFFFF: OTA Partition A (Active)                 │ │
│  │ 0x100000 - 0x1FFFFF: OTA Partition B (Update Target)          │ │
│  │ 0x200000 - 0x2FFFFF: SPIFFS/LittleFS Filesystem               │ │
│  │ 0x300000 - 0x3FFFFF: NVS + Reserved                           │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Git-Agent Protocol

```python
# Git-Agent communication protocol (conceptual)

# The ESP32 doesn't run actual git, but uses a simplified protocol
# that achieves the same result: pull updates, track versions

PROTOCOL_MESSAGES = {
    # Device -> Hub
    "ANNOUNCE": {
        "type": "announce",
        "device_id": "esp32-001",
        "firmware_version": "v1.2.3",
        "commit_hash": "abc123",
        "capabilities": ["temp_sensor", "relay_control"],
        "status": "operational"
    },
    
    "CHECK_UPDATE": {
        "type": "check_update",
        "device_id": "esp32-001",
        "current_commit": "abc123",
        "branch": "main"
    },
    
    "REQUEST_FIRMWARE": {
        "type": "request_firmware",
        "device_id": "esp32-001",
        "target_commit": "def456",
        "chunk_size": 4096
    },
    
    "ACK_CHUNK": {
        "type": "ack_chunk",
        "device_id": "esp32-001",
        "chunk_number": 42,
        "checksum": "sha256:..."
    },
    
    "REPORT_STATUS": {
        "type": "status",
        "device_id": "esp32-001",
        "uptime_seconds": 86400,
        "free_heap": 45000,
        "sensor_data": {...},
        "errors": []
    },
    
    # Hub -> Device
    "UPDATE_AVAILABLE": {
        "type": "update_available",
        "target_commit": "def456",
        "size_bytes": 1048576,
        "checksum": "sha256:...",
        "signature": "ed25519:...",
        "changelog": "Fix temperature reading bug"
    },
    
    "FIRMWARE_CHUNK": {
        "type": "firmware_chunk",
        "chunk_number": 42,
        "total_chunks": 256,
        "data": "<base64 encoded binary>",
        "checksum": "sha256:..."
    },
    
    "UPDATE_CONFIRM": {
        "type": "update_confirm",
        "message": "Firmware verified, ready to apply"
    },
    
    "ROLLBACK": {
        "type": "rollback",
        "reason": "Failed health check after update",
        "target_commit": "abc123"
    }
}
```

### 3.3 ESP32 Firmware Implementation

```cpp
// esp32-git-agent/src/main.cpp

#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <Update.h>
#include <SPIFFS.h>
#include <ESP32Ping.h>

// Configuration
const char* DEVICE_ID = "esp32-001";
const char* HUB_ADDRESS = "192.168.1.100";
const int HUB_PORT = 8080;
const char* CURRENT_COMMIT = "abc123def456";
const char* BRANCH = "main";

// Timing
const unsigned long CHECK_INTERVAL = 300000;  // 5 minutes
const unsigned long REPORT_INTERVAL = 60000;  // 1 minute

// State
unsigned long lastCheckTime = 0;
unsigned long lastReportTime = 0;
bool updateInProgress = false;
String pendingCommit = "";

// Git-Agent Client
class GitAgentClient {
private:
    String hubUrl;
    String deviceId;
    
public:
    GitAgentClient(String hub, String id) 
        : hubUrl("http://" + hub + ":8080"), deviceId(id) {}
    
    bool checkForUpdate() {
        HTTPClient http;
        String url = hubUrl + "/api/v1/update/check";
        
        http.begin(url);
        http.addHeader("Content-Type", "application/json");
        
        StaticJsonDocument<256> doc;
        doc["device_id"] = deviceId;
        doc["current_commit"] = CURRENT_COMMIT;
        doc["branch"] = BRANCH;
        
        String body;
        serializeJson(doc, body);
        
        int code = http.POST(body);
        
        if (code == 200) {
            String response = http.getString();
            StaticJsonDocument<512> respDoc;
            deserializeJson(respDoc, response);
            
            if (respDoc["update_available"]) {
                pendingCommit = respDoc["target_commit"].as<String>();
                return true;
            }
        }
        
        http.end();
        return false;
    }
    
    bool downloadAndApplyUpdate(String targetCommit) {
        HTTPClient http;
        String url = hubUrl + "/api/v1/firmware/" + targetCommit;
        
        http.begin(url);
        int code = http.GET();
        
        if (code != 200) {
            Serial.println("Failed to download firmware");
            return false;
        }
        
        int contentLength = http.getSize();
        WiFiClient* stream = http.getStreamPtr();
        
        // Verify update can begin
        if (!Update.begin(contentLength)) {
            Serial.println("Not enough space for update");
            return false;
        }
        
        // Write firmware
        size_t written = Update.writeStream(*stream);
        if (written != contentLength) {
            Serial.println("Download incomplete");
            Update.abort();
            return false;
        }
        
        // Verify and apply
        if (!Update.end()) {
            Serial.println("Update verification failed");
            return false;
        }
        
        Serial.println("Update applied successfully");
        return true;
    }
    
    void reportStatus(JsonDocument& sensorData) {
        HTTPClient http;
        String url = hubUrl + "/api/v1/status";
        
        http.begin(url);
        http.addHeader("Content-Type", "application/json");
        
        StaticJsonDocument<1024> doc;
        doc["device_id"] = deviceId;
        doc["uptime_seconds"] = millis() / 1000;
        doc["free_heap"] = ESP.getFreeHeap();
        doc["firmware_commit"] = CURRENT_COMMIT;
        doc["sensor_data"] = sensorData;
        
        String body;
        serializeJson(doc, body);
        
        http.POST(body);
        http.end();
    }
};

GitAgentClient gitAgent(HUB_ADDRESS, DEVICE_ID);

void setup() {
    Serial.begin(115200);
    
    // Initialize filesystem
    if (!SPIFFS.begin(true)) {
        Serial.println("SPIFFS mount failed");
        return;
    }
    
    // Connect to WiFi
    // (credentials stored in NVS from initial provisioning)
    WiFi.mode(WIFI_STA);
    // WiFi.begin(ssid, password); -- loaded from NVS
    
    Serial.println("Git-Agent initialized");
    Serial.printf("Device ID: %s\n", DEVICE_ID);
    Serial.printf("Current commit: %s\n", CURRENT_COMMIT);
}

void loop() {
    unsigned long now = millis();
    
    // Check for updates periodically
    if (now - lastCheckTime > CHECK_INTERVAL) {
        lastCheckTime = now;
        
        if (gitAgent.checkForUpdate()) {
            Serial.printf("Update available: %s\n", pendingCommit.c_str());
            
            if (gitAgent.downloadAndApplyUpdate(pendingCommit)) {
                // Restart to apply
                ESP.restart();
            }
        }
    }
    
    // Report status periodically
    if (now - lastReportTime > REPORT_INTERVAL) {
        lastReportTime = now;
        
        StaticJsonDocument<256> sensorData;
        // Read sensors and populate sensorData
        sensorData["temperature"] = readTemperature();
        sensorData["humidity"] = readHumidity();
        
        gitAgent.reportStatus(sensorData);
    }
    
    // Main application logic
    runApplicationLogic();
}

// Placeholder functions
float readTemperature() { return 25.0; }
float readHumidity() { return 60.0; }
void runApplicationLogic() { /* Device-specific */ }
```

---

## 4. Raspberry Pi Hub

### 4.1 Hub Services Architecture

```yaml
# docker-compose.yml for Raspberry Pi Hub

version: "3.8"

services:
  # Sync Agent - Bridges GitHub and local devices
  sync-agent:
    build: ./sync-agent
    container_name: sync-agent
    restart: unless-stopped
    volumes:
      - ./repos:/repos
      - ./config:/config
    environment:
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_REPO=${GITHUB_REPO}
      - SYNC_INTERVAL=60
    ports:
      - "8080:8080"

  # MQTT Broker - Device communication
  mqtt:
    image: eclipse-mosquitto:2
    container_name: mqtt
    restart: unless-stopped
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    ports:
      - "1883:1883"
      - "9001:9001"

  # Local LLM - TinyLlama for edge inference
  llm:
    image: ghcr.io/ggerganov/llama.cpp:server
    container_name: llm
    restart: unless-stopped
    volumes:
      - ./models:/models
    command: >
      --model /models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
      --host 0.0.0.0
      --port 8081
      --ctx-size 2048
      --n-gpu-layers 0
    ports:
      - "8081:8081"

  # API Gateway - REST API for devices
  api:
    build: ./api
    container_name: api
    restart: unless-stopped
    volumes:
      - ./data:/data
    environment:
      - DATABASE_URL=sqlite:///data/hub.db
      - MQTT_HOST=mqtt
      - LLM_HOST=http://llm:8081
    ports:
      - "8000:8000"
    depends_on:
      - mqtt
      - llm

  # Web Dashboard - Local UI
  dashboard:
    build: ./dashboard
    container_name: dashboard
    restart: unless-stopped
    environment:
      - API_URL=http://api:8000
    ports:
      - "3000:3000"
    depends_on:
      - api

  # Sync Queue - Offline operation support
  sync-queue:
    build: ./sync-queue
    container_name: sync-queue
    restart: unless-stopped
    volumes:
      - ./queue:/queue
    environment:
      - CLOUD_ENDPOINT=${CLOUD_ENDPOINT}
      - SYNC_WHEN_CONNECTED=true
```

### 4.2 Sync Agent Implementation

```python
# sync-agent/main.py

import os
import subprocess
import asyncio
import hashlib
from pathlib import Path
from typing import Dict, Optional
from dataclasses import dataclass
import json
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(title="Git-Agent Sync Hub")

@dataclass
class FirmwareVersion:
    commit_hash: str
    branch: str
    timestamp: str
    size_bytes: int
    checksum: str
    changelog: str

class SyncAgent:
    """Syncs between GitHub and local device fleet"""
    
    REPOS_PATH = Path("/repos")
    FIRMWARE_PATH = Path("/firmware")
    
    def __init__(self, github_token: str, repo: str):
        self.github_token = github_token
        self.repo = repo
        self.local_repo = self.REPOS_PATH / repo.split("/")[-1]
    
    async def clone_or_pull(self) -> str:
        """Clone or update local repository"""
        if self.local_repo.exists():
            # Pull latest
            result = subprocess.run(
                ["git", "pull"],
                cwd=self.local_repo,
                capture_output=True,
                text=True
            )
        else:
            # Clone
            result = subprocess.run(
                ["git", "clone", f"https://{self.github_token}@github.com/{self.repo}.git", 
                 str(self.local_repo)],
                capture_output=True,
                text=True
            )
        
        return self._get_current_commit()
    
    def _get_current_commit(self) -> str:
        """Get current commit hash"""
        result = subprocess.run(
            ["git", "rev-parse", "HEAD"],
            cwd=self.local_repo,
            capture_output=True,
            text=True
        )
        return result.stdout.strip()
    
    async def build_firmware(self, device_type: str) -> Optional[FirmwareVersion]:
        """Build firmware for a device type"""
        build_path = self.local_repo / "devices" / device_type
        
        if not build_path.exists():
            return None
        
        # Run PlatformIO build
        result = subprocess.run(
            ["pio", "run"],
            cwd=build_path,
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            print(f"Build failed: {result.stderr}")
            return None
        
        # Find built firmware
        firmware_file = build_path / ".pio" / "build" / "esp32dev" / "firmware.bin"
        
        if not firmware_file.exists():
            return None
        
        # Calculate checksum
        checksum = self._calculate_checksum(firmware_file)
        
        # Copy to firmware store
        commit = self._get_current_commit()
        dest = self.FIRMWARE_PATH / device_type / f"{commit}.bin"
        dest.parent.mkdir(parents=True, exist_ok=True)
        
        import shutil
        shutil.copy(firmware_file, dest)
        
        return FirmwareVersion(
            commit_hash=commit,
            branch="main",
            timestamp=self._get_commit_timestamp(),
            size_bytes=dest.stat().st_size,
            checksum=checksum,
            changelog=self._get_changelog()
        )
    
    def _calculate_checksum(self, file_path: Path) -> str:
        """Calculate SHA256 checksum"""
        sha256 = hashlib.sha256()
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                sha256.update(chunk)
        return f"sha256:{sha256.hexdigest()}"
    
    def _get_commit_timestamp(self) -> str:
        """Get commit timestamp"""
        result = subprocess.run(
            ["git", "log", "-1", "--format=%ai"],
            cwd=self.local_repo,
            capture_output=True,
            text=True
        )
        return result.stdout.strip()
    
    def _get_changelog(self) -> str:
        """Get latest commit message"""
        result = subprocess.run(
            ["git", "log", "-1", "--format=%s"],
            cwd=self.local_repo,
            capture_output=True,
            text=True
        )
        return result.stdout.strip()

# Global sync agent
sync_agent = SyncAgent(
    github_token=os.getenv("GITHUB_TOKEN"),
    repo=os.getenv("GITHUB_REPO", "your-org/iot-fleet")
)

@app.on_event("startup")
async def startup():
    """Initialize sync agent"""
    await sync_agent.clone_or_pull()

@app.get("/api/v1/version")
async def get_version():
    """Get current firmware version"""
    commit = sync_agent._get_current_commit()
    return {"commit": commit, "repo": sync_agent.repo}

@app.get("/api/v1/update/check")
async def check_update(device_id: str, current_commit: str, branch: str = "main"):
    """Check if update is available for a device"""
    latest_commit = sync_agent._get_current_commit()
    
    if current_commit != latest_commit:
        return {
            "update_available": True,
            "target_commit": latest_commit,
            "current_commit": current_commit
        }
    
    return {"update_available": False}

@app.get("/api/v1/firmware/{commit}")
async def get_firmware(commit: str):
    """Download firmware for a specific commit"""
    # Find firmware file
    for device_type in sync_agent.FIRMWARE_PATH.iterdir():
        firmware_file = device_type / f"{commit}.bin"
        if firmware_file.exists():
            from fastapi.responses import FileResponse
            return FileResponse(
                firmware_file,
                media_type="application/octet-stream",
                filename=f"firmware-{commit}.bin"
            )
    
    raise HTTPException(status_code=404, detail="Firmware not found")

@app.post("/api/v1/sync")
async def trigger_sync():
    """Manually trigger repository sync"""
    commit = await sync_agent.clone_or_pull()
    return {"status": "synced", "commit": commit}

@app.post("/api/v1/build/{device_type}")
async def build_device_firmware(device_type: str):
    """Build firmware for a device type"""
    firmware = await sync_agent.build_firmware(device_type)
    if firmware:
        return {
            "status": "built",
            "commit": firmware.commit_hash,
            "size": firmware.size_bytes,
            "checksum": firmware.checksum
        }
    raise HTTPException(status_code=400, detail="Build failed")
```

---

## 5. GitHub Codespaces IDE

### 5.1 Dev Container Configuration

```json
// .devcontainer/devcontainer.json

{
  "name": "IoT Fleet Development",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu-22.04",
  
  "features": {
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.11"
    },
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  
  "customizations": {
    "vscode": {
      "extensions": [
        "platformio.platformio-ide",
        "ms-vscode.vscode-serial-monitor",
        "ms-azuretools.vscode-docker",
        "esbenp.prettier-vscode",
        "ms-python.python",
        "wokwi.wokwi-vscode",
        "github.copilot"
      ],
      "settings": {
        "terminal.integrated.defaultProfile.linux": "bash",
        "platformio-ide.useBuiltinPIOCore": true,
        "files.exclude": {
          "**/.git": true,
          "**/.pio": true
        }
      }
    }
  },
  
  "forwardPorts": [8000, 3000],
  
  "postCreateCommand": "pip install -r requirements.txt && npm install",
  
  "remoteUser": "vscode",
  
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,type=bind,consistency=cached"
  ]
}
```

### 5.2 PlatformIO Configuration

```ini
; platformio.ini for multi-device fleet

[env:esp32-base]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
build_flags = 
    -DCORE_DEBUG_LEVEL=3
    -DBOARD_HAS_PSRAM
lib_deps = 
    bblanchon/ArduinoJson@^6.21.0
    knolleary/PubSubClient@^2.8
    ayushsharma82/EasyOTA@^1.0.5

[env:temp-sensor]
extends = env:esp32-base
build_src_filter = +<*> +<../common/> -<../motor-control/>
build_flags = ${env:esp32-base.build_flags} -DDEVICE_TYPE=\"temp-sensor\"

[env:motor-control]
extends = env:esp32-base
build_src_filter = +<*> +<../common/> -<../temp-sensor/>
build_flags = ${env:esp32-base.build_flags} -DDEVICE_TYPE=\"motor-control\"

[env:gps-logger]
extends = env:esp32-base
build_src_filter = +<*> +<../common/> -<../temp-sensor/> -<../motor-control/>
build_flags = ${env:esp32-base.build_flags} -DDEVICE_TYPE=\"gps-logger\"
lib_deps = 
    ${env:esp32-base.lib_deps}
    mikalhart/TinyGPSPlus@^1.0.3
```

### 5.3 Development Workflow

```markdown
## Codespaces IoT Development Workflow

### 1. Start Codespace
- Open GitHub repository
- Click "Code" → "Codespaces" → "Create codespace"
- Wait for container to build (~2 minutes)

### 2. Develop
- Edit code in VS Code (browser or desktop)
- Use Wokwi extension for hardware simulation
- Write tests in `tests/` directory
- Run tests: `pytest tests/` or `pio test`

### 3. Build
```bash
# Build for all device types
pio run

# Build for specific device
pio run -e temp-sensor
```

### 4. Simulate
```bash
# Launch Wokwi simulation
# (Click "Start Simulator" in VS Code)
```

### 5. Deploy
```bash
# Deploy to Pi hub (which distributes to ESP32s)
./scripts/deploy.sh --target hub.local --device temp-sensor

# Or use the deployment API
curl -X POST http://hub.local:8080/api/v1/build/temp-sensor
```

### 6. Monitor
```bash
# Watch device logs
./scripts/logs.sh --device esp32-001

# Or via API
curl http://hub.local:8000/api/v1/devices/esp32-001/logs
```

### 7. Debug
```bash
# Connect to device serial (via Pi hub)
./scripts/serial.sh --device esp32-001
```
```

---

## 6. Deployment Pipeline

### 6.1 CI/CD Workflow

```yaml
# .github/workflows/deploy.yml

name: Build and Deploy IoT Firmware

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        device: [temp-sensor, motor-control, gps-logger]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install PlatformIO
        run: pip install platformio
      
      - name: Build Firmware
        run: pio run -e ${{ matrix.device }}
      
      - name: Upload Firmware Artifact
        uses: actions/upload-artifact@v4
        with:
          name: firmware-${{ matrix.device }}
          path: devices/${{ matrix.device }}/.pio/build/
      
      - name: Run Tests
        run: pio test -e ${{ matrix.device }}
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download All Artifacts
        uses: actions/download-artifact@v4
        with:
          path: firmware
      
      - name: Deploy to Fleet
        env:
          HUB_ENDPOINT: ${{ secrets.HUB_ENDPOINT }}
          HUB_TOKEN: ${{ secrets.HUB_TOKEN }}
        run: |
          for device in temp-sensor motor-control gps-logger; do
            curl -X POST \
              -H "Authorization: Bearer $HUB_TOKEN" \
              -F "firmware=@firmware/firmware-$device/firmware.bin" \
              "$HUB_ENDPOINT/api/v1/firmware/upload?device_type=$device"
          done
      
      - name: Trigger Fleet Update
        env:
          HUB_ENDPOINT: ${{ secrets.HUB_ENDPOINT }}
          HUB_TOKEN: ${{ secrets.HUB_TOKEN }}
        run: |
          curl -X POST \
            -H "Authorization: Bearer $HUB_TOKEN" \
            "$HUB_ENDPOINT/api/v1/fleet/update?commit=${{ github.sha }}"
```

---

## 7. Offline Considerations

### 7.1 Offline Operation Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OFFLINE OPERATION FLOW                           │
│                                                                     │
│  ONLINE STATE:                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  Codespaces ────▶ GitHub ────▶ Pi Hub ────▶ ESP32 Devices     │ │
│  │     │               │              │             │             │ │
│  │     └───────────────┴──────────────┴─────────────┘             │ │
│  │                     Normal sync flow                           │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  OFFLINE STATE:                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  Pi Hub ─────────────────────▶ ESP32 Devices                  │ │
│  │     │                               │                          │ │
│  │     │  • Operates autonomously      │                          │ │
│  │     │  • Queues changes             │                          │ │
│  │     │  • Provides local LLM         │                          │ │
│  │     │  • Manages fleet              │                          │ │
│  │     │                               │                          │ │
│  │  ESP32 Devices:                                                │ │
│  │     • Continue normal operation                                │ │
│  │     • Cache sensor data                                        │ │
│  │     • Check for updates (will timeout)                         │ │
│  │                                                                 │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  RECONNECTION STATE:                                                │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  1. Pi Hub detects internet connection                         │ │
│  │  2. Pull latest from GitHub                                    │ │
│  │  3. Push queued changes (sensor logs, status)                  │ │
│  │  4. Build new firmware if needed                               │ │
│  │  5. Notify devices of available updates                        │ │
│  │  6. Resume normal sync operation                               │ │
│  │                                                                 │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Sync Queue Implementation

```python
# sync-queue/queue_manager.py

import sqlite3
import json
from datetime import datetime
from typing import List, Dict, Any
from dataclasses import dataclass
import asyncio
import httpx

@dataclass
class QueuedItem:
    id: int
    timestamp: str
    endpoint: str
    method: str
    payload: str
    retry_count: int
    last_error: str

class SyncQueue:
    """Queue for offline operation - syncs when connected"""
    
    def __init__(self, db_path: str = "/queue/sync.db"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """Initialize queue database"""
        conn = sqlite3.connect(self.db_path)
        c = conn.cursor()
        c.execute("""
            CREATE TABLE IF NOT EXISTS sync_queue (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                timestamp TEXT NOT NULL,
                endpoint TEXT NOT NULL,
                method TEXT NOT NULL,
                payload TEXT,
                retry_count INTEGER DEFAULT 0,
                last_error TEXT,
                created_at TEXT DEFAULT CURRENT_TIMESTAMP
            )
        """)
        conn.commit()
        conn.close()
    
    def enqueue(self, endpoint: str, method: str, payload: Dict = None):
        """Add item to sync queue"""
        conn = sqlite3.connect(self.db_path)
        c = conn.cursor()
        c.execute(
            "INSERT INTO sync_queue (timestamp, endpoint, method, payload) VALUES (?, ?, ?, ?)",
            (datetime.utcnow().isoformat(), endpoint, method, json.dumps(payload) if payload else None)
        )
        conn.commit()
        conn.close()
    
    def get_pending(self) -> List[QueuedItem]:
        """Get all pending items"""
        conn = sqlite3.connect(self.db_path)
        c = conn.cursor()
        c.execute("SELECT * FROM sync_queue ORDER BY timestamp ASC")
        rows = c.fetchall()
        conn.close()
        
        return [
            QueuedItem(
                id=row[0],
                timestamp=row[1],
                endpoint=row[2],
                method=row[3],
                payload=row[4],
                retry_count=row[5],
                last_error=row[6]
            )
            for row in rows
        ]
    
    async def sync_all(self, cloud_endpoint: str):
        """Attempt to sync all queued items"""
        items = self.get_pending()
        
        async with httpx.AsyncClient() as client:
            for item in items:
                try:
                    url = f"{cloud_endpoint}{item.endpoint}"
                    payload = json.loads(item.payload) if item.payload else None
                    
                    if item.method == "POST":
                        response = await client.post(url, json=payload)
                    elif item.method == "PUT":
                        response = await client.put(url, json=payload)
                    else:
                        response = await client.get(url)
                    
                    if response.status_code < 400:
                        # Success - remove from queue
                        self._remove(item.id)
                    else:
                        # Failure - increment retry
                        self._increment_retry(item.id, str(response.status_code))
                
                except Exception as e:
                    self._increment_retry(item.id, str(e))
    
    def _remove(self, item_id: int):
        """Remove item from queue"""
        conn = sqlite3.connect(self.db_path)
        c = conn.cursor()
        c.execute("DELETE FROM sync_queue WHERE id = ?", (item_id,))
        conn.commit()
        conn.close()
    
    def _increment_retry(self, item_id: int, error: str):
        """Increment retry count"""
        conn = sqlite3.connect(self.db_path)
        c = conn.cursor()
        c.execute(
            "UPDATE sync_queue SET retry_count = retry_count + 1, last_error = ? WHERE id = ?",
            (error, item_id)
        )
        conn.commit()
        conn.close()
```

---

## 8. Use Cases

### 8.1 Marine IoT Fleet

| Device | Sensors | Actuators | Update Frequency |
|--------|---------|-----------|------------------|
| Helm Station | GPS, Compass | Display, Buzzer | Real-time |
| Engine Room | Temp, RPM, Fuel | Alarm | 10 seconds |
| Fish Hold | Temp, Humidity | Refrigeration | 30 seconds |
| Deck | Wind, Bilge | Pump, Lights | 1 minute |
| Safety | Smoke, CO | Alarm | Real-time |

### 8.2 Industrial Monitoring

| Device | Sensors | Purpose |
|--------|---------|---------|
| Machine Monitor | Vibration, Temp | Predictive maintenance |
| Production Counter | IR Sensor | Output tracking |
| Quality Gate | Camera | Defect detection |
| Environmental | Temp, Humidity, Dust | Work condition monitoring |

### 8.3 Remote Installation

| Device | Sensors | Communication |
|--------|---------|---------------|
| Weather Station | Wind, Rain, Temp, Pressure | LoRa + WiFi |
| Wildlife Cam | PIR, Camera | Cellular backup |
| Water Level | Ultrasonic | LoRa mesh |

---

## 9. Implementation Code

### 9.1 Device Provisioning

```python
# scripts/provision_device.py

import subprocess
import json
import secrets
from pathlib import Path

def provision_device(device_id: str, device_type: str, wifi_ssid: str, wifi_pass: str):
    """Provision a new ESP32 device"""
    
    # Generate device credentials
    device_token = secrets.token_hex(16)
    
    # Create device configuration
    config = {
        "device_id": device_id,
        "device_type": device_type,
        "hub_address": "192.168.1.100",
        "hub_port": 8080,
        "wifi_ssid": wifi_ssid,
        "wifi_pass": wifi_pass,
        "device_token": device_token,
        "report_interval": 60,
        "check_interval": 300
    }
    
    # Write configuration to build
    config_path = Path(f"devices/{device_type}/include/config.h")
    config_path.parent.mkdir(parents=True, exist_ok=True)
    
    with open(config_path, "w") as f:
        f.write(f"""// Auto-generated device configuration
#pragma once

#define DEVICE_ID "{device_id}"
#define DEVICE_TYPE "{device_type}"
#define HUB_ADDRESS "{config['hub_address']}"
#define HUB_PORT {config['hub_port']}
#define WIFI_SSID "{wifi_ssid}"
#define WIFI_PASS "{wifi_pass}"
#define DEVICE_TOKEN "{device_token}"
#define REPORT_INTERVAL {config['report_interval']}
#define CHECK_INTERVAL {config['check_interval']}
""")
    
    # Build firmware
    print(f"Building firmware for {device_id}...")
    result = subprocess.run(
        ["pio", "run", "-e", device_type],
        capture_output=True,
        text=True
    )
    
    if result.returncode != 0:
        print(f"Build failed: {result.stderr}")
        return None
    
    # Flash firmware (requires device connected via USB)
    print(f"Flashing firmware to {device_id}...")
    result = subprocess.run(
        ["pio", "run", "-e", device_type, "-t", "upload"],
        capture_output=True,
        text=True
    )
    
    if result.returncode != 0:
        print(f"Flash failed: {result.stderr}")
        return None
    
    # Register device with hub
    print(f"Registering {device_id} with hub...")
    # (Would make API call to hub)
    
    return {
        "device_id": device_id,
        "device_type": device_type,
        "token": device_token,
        "status": "provisioned"
    }

if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description="Provision ESP32 device")
    parser.add_argument("--device-id", required=True)
    parser.add_argument("--device-type", required=True)
    parser.add_argument("--wifi-ssid", required=True)
    parser.add_argument("--wifi-pass", required=True)
    
    args = parser.parse_args()
    
    result = provision_device(
        args.device_id,
        args.device_type,
        args.wifi_ssid,
        args.wifi_pass
    )
    
    print(json.dumps(result, indent=2))
```

---

## 10. Configuration Schemas

### 10.1 Fleet Configuration

```yaml
# config/fleet.yaml

fleet:
  id: fishing-boat-001
  name: "Sea Harvest"
  type: commercial_fishing

hub:
  host: 192.168.1.100
  port: 8080
  model: raspberry-pi-5-8gb

devices:
  - id: esp32-helm
    type: helm-station
    location: bridge
    sensors:
      - gps
      - compass
    actuators:
      - display
      - buzzer
    
  - id: esp32-engine
    type: engine-monitor
    location: engine_room
    sensors:
      - temperature
      - rpm
      - fuel_flow
    actuators:
      - alarm

  - id: esp32-hold
    type: fish-hold
    location: fish_hold
    sensors:
      - temperature
      - humidity
      - door_status
    actuators:
      - refrigeration

sync:
  interval_seconds: 60
  cloud_endpoint: https://fleet.example.com/api
  offline_mode: queue

updates:
  auto_update: false  # Manual approval required
  check_interval_seconds: 300
  rollback_on_failure: true
```

---

## Summary

This architecture provides:

| Feature | Implementation |
|---------|---------------|
| **Cloud Development** | GitHub Codespaces with full IoT toolchain |
| **Edge Hub** | Raspberry Pi 5 with local LLM and sync |
| **Distributed Devices** | ESP32 with git-agent OTA updates |
| **Offline Operation** | Queue-based sync with automatic recovery |
| **Low Cost** | ~$455 for complete 6-device fleet |
| **Remote Management** | No physical access needed for updates |

---

*Raspberry Pi + ESP32 Cloud-Edge Architecture v1.0*
*Part of the Jetson Robotics Grand Design Documentation Suite*
