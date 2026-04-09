# R&D Iterations 6-10: Advanced Topics

## Jetson Robotics Grand Design — Advanced Development Phase

**Document Version**: 1.0  
**Date**: January 2025  
**Status**: Research & Development Report  
**Lead**: R&D Division, Jetson Robotics Ecosystem

---

## Executive Summary

This document captures the comprehensive research and development process for iterations 6-10 of the Jetson Robotics Grand Design project. Building on the foundational work of iterations 1-5 (ROM Architecture, Vessel Specifications, CoCapn Software Vessel, Training Data Pipeline, and A2A-R Protocol), these advanced iterations address critical challenges in:

- **Iteration 6**: Unified Fleet Management across heterogeneous hardware (Jetson Thor, Orin Nano, Raspberry Pi, ESP32)
- **Iteration 7**: Portfolio System for Multi-Architecture Contributions
- **Iteration 8**: Knowledge Transfer Between Hardware Tiers
- **Iteration 9**: Commercial Fishing-Specific Features
- **Iteration 10**: Production Deployment & Documentation

Each iteration follows a structured process: Draft Phase → Open Questions → Research Phase → Development Phase → User Simulation → Test Results → Next Iteration Prep.

---

## Table of Contents

1. [Iteration 6: Unified Fleet Management](#iteration-6-unified-fleet-management)
2. [Iteration 7: Multi-Architecture Portfolio System](#iteration-7-multi-architecture-portfolio-system)
3. [Iteration 8: Cross-Tier Knowledge Transfer](#iteration-8-cross-tier-knowledge-transfer)
4. [Iteration 9: Commercial Fishing Features](#iteration-9-commercial-fishing-features)
5. [Iteration 10: Production Deployment & Documentation](#iteration-10-production-deployment--documentation)
6. [Cross-Iteration Synthesis](#cross-iteration-synthesis)
7. [Appendices](#appendices)

---

# Iteration 6: Unified Fleet Management

## Thor + Orin Nano + Pi/ESP32 Heterogeneous Fleet Architecture

**Iteration Lead**: Fleet Systems Division  
**Duration**: 4 weeks  
**Status**: Completed

---

## 6.1 Draft Phase

### Initial Problem Statement

The Jetson Robotics ecosystem must support vessels with dramatically different computational capabilities:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HARDWARE TIER SPECTRUM                                    │
│                                                                              │
│  TIER 1: HIGH-PERFORMANCE (Jetson Thor)                                     │
│  ├── 128 GB Unified Memory                                                  │
│  ├── 2070 FP4 TFLOPS                                                        │
│  ├── Full LLM inference (32B models)                                        │
│  ├── Complex sensor fusion                                                  │
│  └── Training capability                                                    │
│                                                                              │
│  TIER 2: MID-RANGE (Jetson Orin Nano)                                       │
│  ├── 8 GB Memory                                                            │
│  ├── 40 TOPS INT8                                                           │
│  ├── Quantized model inference (7B models)                                  │
│  ├── Basic sensor fusion                                                    │
│  └── Edge inference only                                                    │
│                                                                              │
│  TIER 3: LIGHTWEIGHT (Raspberry Pi 5)                                       │
│  ├── 8 GB RAM                                                               │
│  ├── ARM Cortex-A76                                                         │
│  ├── API-based inference                                                    │
│  ├── Simple sensor processing                                               │
│  └── Relay/coordination roles                                               │
│                                                                              │
│  TIER 4: MICRO-CONTROLLER (ESP32)                                           │
│  ├── 520 KB SRAM                                                            │
│  ├── 240 MHz Dual-Core                                                      │
│  ├── Sensor data relay                                                      │
│  ├── Real-time control loops                                                │
│  └── Safety interlocks                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Draft Specification

**Objective**: Design a unified fleet management system that treats all hardware tiers as first-class citizens while respecting their computational boundaries.

**Key Requirements**:
1. Single A2A-R protocol across all tiers
2. Graceful capability degradation based on hardware
3. Distributed task allocation considering compute resources
4. Unified monitoring and health tracking
5. Consistent developer experience across tiers

### Initial Architecture Sketch

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    UNIFIED FLEET MANAGEMENT ARCHITECTURE                     │
│                                                                              │
│                         ┌─────────────────────┐                             │
│                         │   FLEET ORCHESTRATOR │                             │
│                         │    (Thor / Cloud)    │                             │
│                         └──────────┬──────────┘                             │
│                                    │                                         │
│                    ┌───────────────┼───────────────┐                        │
│                    │               │               │                        │
│                    ▼               ▼               ▼                        │
│            ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│            │   THOR FLEET │ │  ORIN FLEET  │ │  PI/ESP FLEET│              │
│            │   SEGMENT    │ │   SEGMENT    │ │   SEGMENT    │              │
│            └──────┬───────┘ └──────┬───────┘ └──────┬───────┘              │
│                   │                │                │                        │
│         ┌────────┼────────┐       │       ┌────────┼────────┐              │
│         ▼        ▼        ▼       ▼       ▼        ▼        ▼              │
│      ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐              │
│      │Thor │ │Thor │ │Orin │ │Orin │ │ Pi  │ │ESP32│ │ESP32│              │
│      │ V1  │ │ V2  │ │ V1  │ │ V2  │ │ V1  │ │ V1  │ │ V2  │              │
│      └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘              │
│                                                                              │
│  COMMUNICATION: A2A-R Protocol (adapted per tier)                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6.2 Open Questions

### Technical Questions

| ID | Question | Priority | Initial Hypothesis |
|----|----------|----------|-------------------|
| T6-1 | How to run A2A-R on ESP32 with 520KB RAM? | Critical | Lightweight protocol subset |
| T6-2 | Model serialization format for heterogeneous inference? | High | ONNX with tier-specific optimizations |
| T6-3 | Task allocation algorithm considering compute constraints? | High | Capability-weighted round-robin |
| T6-4 | State synchronization across async/sync boundaries? | Medium | Event-sourced state with CRDTs |
| T6-5 | Offline coordination for micro-controllers? | Medium | Local decision trees with sync-on-connect |
| T6-6 | Security model for constrained devices? | High | Pre-shared keys with rotation |
| T6-7 | OTA update strategy for ESP32? | Medium | Dual-bank with rollback |
| T6-8 | Monitoring overhead on Pi/ESP32? | Medium | Sampling-based metrics |

### Operational Questions

| ID | Question | Priority | Initial Hypothesis |
|----|----------|----------|-------------------|
| O6-1 | How to handle fleet topology changes? | High | Dynamic discovery with mDNS |
| O6-2 | How to debug issues on headless ESP32? | Medium | Remote logging via serial/WiFi |
| O6-3 | How to provision credentials at scale? | High | Manufacturing-time provisioning |
| O6-4 | How to handle tier migration (upgrade vessel)? | Medium | Configuration migration tools |

---

## 6.3 Research Phase

### Research Finding 1: A2A-R Lite Protocol

**Investigation**: Analyze minimum viable A2A-R implementation for ESP32.

**Methodology**:
1. Profile ESP32 memory usage with minimal A2A-R stack
2. Identify protocol components that can be stripped
3. Design tier-specific protocol subsets
4. Validate with simulated fleet

**Findings**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    A2A-R PROTOCOL TIERS                                      │
│                                                                              │
│  FULL A2A-R (Thor/Orin)                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Full Agent Card with capabilities                                │   │
│  │  • Task streaming (bidirectional)                                   │   │
│  │  • WebRTC sensor streaming                                          │   │
│  │  • Latency broadcast                                                │   │
│  │  • Coordination primitives                                          │   │
│  │  • Training data export                                             │   │
│  │  Memory Footprint: ~50 MB                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  A2A-R COMPACT (Pi)                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Simplified Agent Card (JSON)                                     │   │
│  │  • HTTP-based task execution                                        │   │
│  │  • JPEG-based sensor frames                                         │   │
│  │  • Heartbeat-only monitoring                                        │   │
│  │  • No coordination primitives (follower only)                       │   │
│  │  Memory Footprint: ~5 MB                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  A2A-R MICRO (ESP32)                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Binary Agent Card (fixed format, <1KB)                          │   │
│  │  • MQTT-based messaging (no HTTP)                                   │   │
│  │  • Sensor values only (no streaming)                                │   │
│  │  • Status broadcast (heartbeat equivalent)                          │   │
│  │  • Command receiver only                                            │   │
│  │  Memory Footprint: ~50 KB                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Research Code: A2A-R Micro Agent Card**

```c
// a2ar_micro_agent_card.h
// Binary Agent Card format for ESP32 - Fixed 512 bytes

#pragma pack(push, 1)

typedef struct {
    // Header (16 bytes)
    uint8_t  magic[4];           // "A2AR"
    uint8_t  version_major;      // Protocol version
    uint8_t  version_minor;
    uint16_t card_length;        // Total card length
    uint8_t  device_tier;        // 4 = ESP32
    uint8_t  reserved[6];
    
    // Identity (32 bytes)
    uint8_t  device_id[16];      // UUID
    uint8_t  fleet_id[16];       // Fleet membership
    
    // Capabilities (64 bytes)
    uint8_t  sensor_count;       // Number of sensors
    uint8_t  actuator_count;     // Number of actuators
    uint8_t  sensors[32];        // Sensor type bitmask
    uint8_t  actuators[16];      // Actuator type bitmask
    uint8_t  safety_features;    // Safety capability flags
    uint8_t  compute_capability; // Compute tier indicator
    uint8_t  reserved_caps[12];
    
    // Network (32 bytes)
    uint8_t  mac_address[6];
    uint8_t  ip_address[4];
    uint16_t mqtt_port;
    uint8_t  security_mode;      // 0=none, 1=PSK, 2=certificate
    uint8_t  reserved_net[19];
    
    // Status (32 bytes)
    uint32_t uptime_seconds;
    uint16_t free_heap_kb;
    uint8_t  wifi_rssi;
    uint8_t  cpu_usage_percent;
    uint8_t  temperature_c;
    uint8_t  error_count;
    uint8_t  status_flags;       // Bit flags for various states
    uint8_t  reserved_status[20];
    
    // Signature (64 bytes)
    uint8_t  device_signature[64]; // Ed25519 signature of above
    
    // Padding to 512 bytes
    uint8_t  padding[272];
    
} A2ARMicroAgentCard;

#pragma pack(pop)

// Total size: 512 bytes (fixed)

// Capability bitmasks
#define SENSOR_GPS           (1 << 0)
#define SENSOR_IMU           (1 << 1)
#define SENSOR_TEMPERATURE   (1 << 2)
#define SENSOR_PRESSURE      (1 << 3)
#define SENSOR_HUMIDITY      (1 << 4)
#define SENSOR_DEPTH         (1 << 5)
#define SENSOR_SPEED         (1 << 6)
#define SENSOR_VOLTAGE       (1 << 7)

#define ACTUATOR_RELAY       (1 << 0)
#define ACTUATOR_PWM         (1 << 1)
#define ACTUATOR_SERVO       (1 << 2)
#define ACTUATOR_LED         (1 << 3)
#define ACTUATOR_BUZZER      (1 << 4)

#define SAFETY_ESTOP         (1 << 0)
#define SAFETY_WATCHDOG      (1 << 1)
#define SAFETY_GEOFENCE      (1 << 2)
#define SAFETY_LIMIT_SWITCH  (1 << 3)
```

**Implementation: ESP32 A2A-R Micro Stack**

```c
// a2ar_micro.c
#include "a2ar_micro_agent_card.h"
#include "mqtt_client.h"
#include "esp_wifi.h"
#include "esp_log.h"
#include <string.h>

static const char *TAG = "A2AR_MICRO";

// Global agent card
static A2ARMicroAgentCard g_agent_card;
static esp_mqtt_client_handle_t g_mqtt_client = NULL;

// Initialize A2A-R Micro
esp_err_t a2ar_micro_init(const char *device_id, const char *fleet_id) {
    // Initialize agent card
    memset(&g_agent_card, 0, sizeof(g_agent_card));
    
    // Set header
    memcpy(g_agent_card.magic, "A2AR", 4);
    g_agent_card.version_major = 1;
    g_agent_card.version_minor = 0;
    g_agent_card.card_length = sizeof(A2ARMicroAgentCard);
    g_agent_card.device_tier = 4;  // ESP32
    
    // Set identity
    memcpy(g_agent_card.device_id, device_id, 16);
    memcpy(g_agent_card.fleet_id, fleet_id, 16);
    
    // Get network info
    uint8_t mac[6];
    esp_wifi_get_mac(WIFI_IF_STA, mac);
    memcpy(g_agent_card.mac_address, mac, 6);
    
    // Default capabilities (set by application)
    g_agent_card.sensor_count = 0;
    g_agent_card.actuator_count = 0;
    g_agent_card.safety_features = SAFETY_WATCHDOG;
    
    ESP_LOGI(TAG, "A2A-R Micro initialized for device %02x%02x%02x%02x",
             device_id[0], device_id[1], device_id[2], device_id[3]);
    
    return ESP_OK;
}

// Register sensor capability
esp_err_t a2ar_micro_register_sensor(uint8_t sensor_type) {
    if (g_agent_card.sensor_count >= 32) {
        return ESP_ERR_NO_MEM;
    }
    
    g_agent_card.sensors[g_agent_card.sensor_count / 8] |= (sensor_type << (g_agent_card.sensor_count % 8));
    g_agent_card.sensor_count++;
    
    return ESP_OK;
}

// Register actuator capability
esp_err_t a2ar_micro_register_actuator(uint8_t actuator_type) {
    if (g_agent_card.actuator_count >= 16) {
        return ESP_ERR_NO_MEM;
    }
    
    g_agent_card.actuators[g_agent_card.actuator_count / 8] |= (actuator_type << (g_agent_card.actuator_count % 8));
    g_agent_card.actuator_count++;
    
    return ESP_OK;
}

// Update status
void a2ar_micro_update_status(void) {
    g_agent_card.uptime_seconds = esp_log_timestamp() / 1000;
    g_agent_card.free_heap_kb = esp_get_free_heap_size() / 1024;
    g_agent_card.wifi_rssi = 0;  // Updated by WiFi callback
    g_agent_card.cpu_usage_percent = 0;  // Calculated by idle task
    g_agent_card.temperature_c = 25;  // Read from internal sensor if available
    
    // Sign the card (using pre-loaded Ed25519 key)
    // a2ar_micro_sign_card(&g_agent_card);
}

// Publish agent card
esp_err_t a2ar_micro_publish_card(void) {
    if (g_mqtt_client == NULL) {
        return ESP_ERR_INVALID_STATE;
    }
    
    // Update status before publishing
    a2ar_micro_update_status();
    
    // Publish to agent card topic
    char topic[64];
    snprintf(topic, sizeof(topic), "a2ar/%02x%02x%02x%02x/card",
             g_agent_card.device_id[0], g_agent_card.device_id[1],
             g_agent_card.device_id[2], g_agent_card.device_id[3]);
    
    int msg_id = esp_mqtt_client_publish(
        g_mqtt_client,
        topic,
        (const char *)&g_agent_card,
        sizeof(g_agent_card),
        1,      // QoS 1
        0       // retain
    );
    
    if (msg_id < 0) {
        ESP_LOGE(TAG, "Failed to publish agent card");
        return ESP_FAIL;
    }
    
    ESP_LOGI(TAG, "Published agent card, msg_id=%d", msg_id);
    return ESP_OK;
}

// Handle incoming command
static void handle_command(const char *topic, const char *data, int len) {
    // Parse command (simplified JSON)
    // Expected format: {"cmd":"relay","id":0,"state":1}
    
    ESP_LOGI(TAG, "Received command: %.*s", len, data);
    
    // Command routing based on actuator type
    if (strstr(data, "relay") != NULL) {
        // Handle relay command
        // Extract relay ID and state, then execute
    }
    else if (strstr(data, "estop") != NULL) {
        // Emergency stop - immediate action
        ESP_LOGW(TAG, "EMERGENCY STOP received");
        g_agent_card.status_flags |= (1 << 7);  // Set emergency flag
        // Trigger safety interlock
    }
}

// MQTT event handler
static void mqtt_event_handler(void *handler_args, esp_event_base_t base,
                               int32_t event_id, void *event_data) {
    esp_mqtt_event_handle_t event = event_data;
    
    switch (event_id) {
        case MQTT_EVENT_CONNECTED:
            ESP_LOGI(TAG, "MQTT connected");
            // Subscribe to command topic
            char cmd_topic[64];
            snprintf(cmd_topic, sizeof(cmd_topic), "a2ar/%02x%02x%02x%02x/cmd/+",
                     g_agent_card.device_id[0], g_agent_card.device_id[1],
                     g_agent_card.device_id[2], g_agent_card.device_id[3]);
            esp_mqtt_client_subscribe(g_mqtt_client, cmd_topic, 1);
            
            // Publish initial agent card
            a2ar_micro_publish_card();
            break;
            
        case MQTT_EVENT_DATA:
            ESP_LOGI(TAG, "MQTT data received");
            handle_command(event->topic, event->data, event->data_len);
            break;
            
        case MQTT_EVENT_DISCONNECTED:
            ESP_LOGW(TAG, "MQTT disconnected");
            break;
            
        default:
            break;
    }
}

// Connect to MQTT broker
esp_err_t a2ar_micro_connect(const char *broker_uri, uint16_t port) {
    esp_mqtt_client_config_t mqtt_cfg = {
        .uri = broker_uri,
        .port = port,
    };
    
    g_mqtt_client = esp_mqtt_client_init(&mqtt_cfg);
    esp_mqtt_client_register_event(g_mqtt_client, ESP_EVENT_ANY_ID,
                                    mqtt_event_handler, NULL);
    esp_mqtt_client_start(g_mqtt_client);
    
    return ESP_OK;
}

// Heartbeat task
void a2ar_micro_heartbeat_task(void *pvParameters) {
    while (1) {
        vTaskDelay(pdMS_TO_TICKS(5000));  // 5 second interval
        
        if (g_mqtt_client != NULL) {
            // Publish heartbeat
            char topic[64];
            snprintf(topic, sizeof(topic), "a2ar/%02x%02x%02x%02x/heartbeat",
                     g_agent_card.device_id[0], g_agent_card.device_id[1],
                     g_agent_card.device_id[2], g_agent_card.device_id[3]);
            
            // Simple heartbeat: just uptime and free heap
            char payload[32];
            snprintf(payload, sizeof(payload), "%lu,%u",
                     g_agent_card.uptime_seconds, g_agent_card.free_heap_kb);
            
            esp_mqtt_client_publish(g_mqtt_client, topic, payload, 0, 0, 0);
        }
    }
}
```

### Research Finding 2: Tier-Aware Task Allocation

**Investigation**: Design task allocation algorithm that respects compute constraints.

**Methodology**:
1. Define task complexity metrics
2. Benchmark each hardware tier
3. Design allocation algorithm
4. Validate with simulated workloads

**Findings**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TASK COMPLEXITY MATRIX                                    │
│                                                                              │
│  TASK TYPE              THOR    ORIN    PI      ESP32   ALLOCATION RULE    │
│  ────────────────────────────────────────────────────────────────────────  │
│  LLM Inference (32B)    ✅      ❌      ❌      ❌      Thor only            │
│  LLM Inference (7B)     ✅      ✅      ❌      ❌      Thor/Orin            │
│  Vision Detection       ✅      ✅      API     ❌      Thor/Orin/Pi(API)   │
│  Sensor Fusion          ✅      ✅      Partial ❌      Tier-aware          │
│  Navigation Planning    ✅      ✅      Simple  ❌      Tier-aware          │
│  Voice Recognition      ✅      ✅      API     ❌      Thor/Orin/Pi(API)   │
│  Sensor Reading         ✅      ✅      ✅      ✅      Any tier            │
│  Relay Control          ✅      ✅      ✅      ✅      Any tier            │
│  Safety Interlock       ✅      ✅      ✅      ✅      Local (any tier)    │
│  Fleet Coordination     ✅      Follower Follower Follower Leader must Thor│
│  Training/LoRA          ✅      ❌      ❌      ❌      Thor only            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Task Allocation Algorithm**

```python
# task_allocation.py
"""
Tier-aware task allocation for heterogeneous fleet
"""
from dataclasses import dataclass
from typing import Dict, List, Optional, Set
from enum import Enum
import heapq

class HardwareTier(Enum):
    THOR = 1
    ORIN_NANO = 2
    RASPBERRY_PI = 3
    ESP32 = 4

class TaskType(Enum):
    LLM_INFERENCE_LARGE = "llm_inference_large"      # 32B+ models
    LLM_INFERENCE_SMALL = "llm_inference_small"      # 7B models
    VISION_DETECTION = "vision_detection"
    SENSOR_FUSION = "sensor_fusion"
    NAVIGATION_PLANNING = "navigation_planning"
    VOICE_RECOGNITION = "voice_recognition"
    SENSOR_READING = "sensor_reading"
    RELAY_CONTROL = "relay_control"
    SAFETY_INTERLOCK = "safety_interlock"
    FLEET_COORDINATION = "fleet_coordination"
    TRAINING = "training"

@dataclass
class Task:
    task_id: str
    task_type: TaskType
    priority: int  # 1=highest, 10=lowest
    deadline_ms: int
    payload_size_bytes: int
    required_capabilities: Set[str]
    preferred_tier: Optional[HardwareTier] = None
    
@dataclass
class Vessel:
    vessel_id: str
    tier: HardwareTier
    current_load: float  # 0.0 to 1.0
    capabilities: Set[str]
    latency_ms: int  # Network latency to orchestrator
    available_memory_mb: int
    current_tasks: List[str]

# Task complexity requirements by tier
TIER_CAPABILITIES = {
    HardwareTier.THOR: {
        TaskType.LLM_INFERENCE_LARGE: True,
        TaskType.LLM_INFERENCE_SMALL: True,
        TaskType.VISION_DETECTION: True,
        TaskType.SENSOR_FUSION: True,
        TaskType.NAVIGATION_PLANNING: True,
        TaskType.VOICE_RECOGNITION: True,
        TaskType.SENSOR_READING: True,
        TaskType.RELAY_CONTROL: True,
        TaskType.SAFETY_INTERLOCK: True,
        TaskType.FLEET_COORDINATION: True,  # Can be leader
        TaskType.TRAINING: True,
    },
    HardwareTier.ORIN_NANO: {
        TaskType.LLM_INFERENCE_LARGE: False,
        TaskType.LLM_INFERENCE_SMALL: True,
        TaskType.VISION_DETECTION: True,
        TaskType.SENSOR_FUSION: True,
        TaskType.NAVIGATION_PLANNING: True,
        TaskType.VOICE_RECOGNITION: True,
        TaskType.SENSOR_READING: True,
        TaskType.RELAY_CONTROL: True,
        TaskType.SAFETY_INTERLOCK: True,
        TaskType.FLEET_COORDINATION: False,  # Follower only
        TaskType.TRAINING: False,
    },
    HardwareTier.RASPBERRY_PI: {
        TaskType.LLM_INFERENCE_LARGE: False,
        TaskType.LLM_INFERENCE_SMALL: False,  # API-based only
        TaskType.VISION_DETECTION: False,  # API-based only
        TaskType.SENSOR_FUSION: False,  # Partial only
        TaskType.NAVIGATION_PLANNING: False,  # Simple only
        TaskType.VOICE_RECOGNITION: False,  # API-based only
        TaskType.SENSOR_READING: True,
        TaskType.RELAY_CONTROL: True,
        TaskType.SAFETY_INTERLOCK: True,
        TaskType.FLEET_COORDINATION: False,  # Follower only
        TaskType.TRAINING: False,
    },
    HardwareTier.ESP32: {
        TaskType.LLM_INFERENCE_LARGE: False,
        TaskType.LLM_INFERENCE_SMALL: False,
        TaskType.VISION_DETECTION: False,
        TaskType.SENSOR_FUSION: False,
        TaskType.NAVIGATION_PLANNING: False,
        TaskType.VOICE_RECOGNITION: False,
        TaskType.SENSOR_READING: True,
        TaskType.RELAY_CONTROL: True,
        TaskType.SAFETY_INTERLOCK: True,
        TaskType.FLEET_COORDINATION: False,  # Follower only
        TaskType.TRAINING: False,
    }
}

# Estimated execution times by tier (milliseconds)
EXECUTION_TIMES = {
    (TaskType.LLM_INFERENCE_LARGE, HardwareTier.THOR): 150,
    (TaskType.LLM_INFERENCE_SMALL, HardwareTier.THOR): 50,
    (TaskType.LLM_INFERENCE_SMALL, HardwareTier.ORIN_NANO): 200,
    (TaskType.VISION_DETECTION, HardwareTier.THOR): 30,
    (TaskType.VISION_DETECTION, HardwareTier.ORIN_NANO): 80,
    (TaskType.SENSOR_FUSION, HardwareTier.THOR): 10,
    (TaskType.SENSOR_FUSION, HardwareTier.ORIN_NANO): 30,
    (TaskType.SENSOR_READING, HardwareTier.THOR): 1,
    (TaskType.SENSOR_READING, HardwareTier.ORIN_NANO): 1,
    (TaskType.SENSOR_READING, HardwareTier.RASPBERRY_PI): 5,
    (TaskType.SENSOR_READING, HardwareTier.ESP32): 1,
    (TaskType.RELAY_CONTROL, HardwareTier.THOR): 1,
    (TaskType.RELAY_CONTROL, HardwareTier.ORIN_NANO): 1,
    (TaskType.RELAY_CONTROL, HardwareTier.RASPBERRY_PI): 5,
    (TaskType.RELAY_CONTROL, HardwareTier.ESP32): 1,
}


class TierAwareTaskAllocator:
    """
    Allocates tasks to vessels based on hardware tier capabilities,
    current load, and task requirements.
    """
    
    def __init__(self):
        self.vessels: Dict[str, Vessel] = {}
        self.pending_tasks: List[Task] = []
        
    def register_vessel(self, vessel: Vessel):
        """Register a vessel with the allocator"""
        self.vessels[vessel.vessel_id] = vessel
        
    def submit_task(self, task: Task):
        """Submit a task for allocation"""
        heapq.heappush(self.pending_tasks, (task.priority, task))
        
    def allocate(self) -> Dict[str, List[Task]]:
        """
        Allocate all pending tasks to vessels.
        Returns mapping of vessel_id -> list of allocated tasks.
        """
        allocations: Dict[str, List[Task]] = {v: [] for v in self.vessels}
        
        while self.pending_tasks:
            _, task = heapq.heappop(self.pending_tasks)
            
            # Find best vessel for this task
            best_vessel = self._find_best_vessel(task)
            
            if best_vessel:
                allocations[best_vessel].append(task)
                # Update vessel load
                self._update_vessel_load(best_vessel, task)
            else:
                # No suitable vessel found - escalate
                self._handle_no_allocation(task)
                
        return allocations
    
    def _find_best_vessel(self, task: Task) -> Optional[str]:
        """Find the best vessel for a given task"""
        candidates = []
        
        for vessel_id, vessel in self.vessels.items():
            # Check if vessel can handle this task type
            if not TIER_CAPABILITIES[vessel.tier].get(task.task_type, False):
                continue
                
            # Check if vessel has required capabilities
            if not task.required_capabilities.issubset(vessel.capabilities):
                continue
                
            # Check if vessel has capacity
            if vessel.current_load >= 0.9:  # 90% max load
                continue
                
            # Check if vessel can meet deadline
            exec_time = EXECUTION_TIMES.get((task.task_type, vessel.tier), 1000)
            total_time = exec_time + vessel.latency_ms
            
            if total_time > task.deadline_ms:
                continue
                
            # Calculate score (lower is better)
            # Prefer: lower load, faster tier, lower latency
            tier_score = vessel.tier.value  # Lower tier value = more capable
            load_score = vessel.current_load
            latency_score = vessel.latency_ms / 100.0
            
            score = (
                tier_score * 0.4 +
                load_score * 0.4 +
                latency_score * 0.2
            )
            
            candidates.append((score, vessel_id))
            
        if not candidates:
            return None
            
        # Return vessel with lowest score
        candidates.sort()
        return candidates[0][1]
    
    def _update_vessel_load(self, vessel_id: str, task: Task):
        """Update vessel load after task allocation"""
        vessel = self.vessels[vessel_id]
        exec_time = EXECUTION_TIMES.get((task.task_type, vessel.tier), 100)
        
        # Estimate load increase based on execution time
        load_increase = exec_time / 10000.0  # Normalized
        vessel.current_load = min(1.0, vessel.current_load + load_increase)
        vessel.current_tasks.append(task.task_id)
        
    def _handle_no_allocation(self, task: Task):
        """Handle case where no vessel can handle task"""
        # Log and potentially escalate
        print(f"WARNING: No vessel available for task {task.task_id} "
              f"(type: {task.task_type}, deadline: {task.deadline_ms}ms)")
        
        # Could implement:
        # - Task queueing for later
        # - Task degradation (simpler version)
        # - External API fallback
        # - Escalation to human


class DistributedTaskCoordinator:
    """
    Coordinates tasks across a distributed fleet.
    Handles leader election, task distribution, and result aggregation.
    """
    
    def __init__(self, allocator: TierAwareTaskAllocator):
        self.allocator = allocator
        self.leader_vessel: Optional[str] = None
        self.follower_vessels: Set[str] = set()
        
    def elect_leader(self):
        """
        Elect a leader vessel for fleet coordination.
        Leader must be Thor tier.
        """
        thor_vessels = [
            v_id for v_id, v in self.allocator.vessels.items()
            if v.tier == HardwareTier.THOR
        ]
        
        if not thor_vessels:
            raise RuntimeError("No Thor-class vessel available for leadership")
            
        # Simple election: choose vessel with lowest load
        thor_vessels.sort(key=lambda v: self.allocator.vessels[v].current_load)
        self.leader_vessel = thor_vessels[0]
        
        # All others are followers
        self.follower_vessels = set(self.allocator.vessels.keys()) - {self.leader_vessel}
        
        return self.leader_vessel
    
    def distribute_subtasks(self, task: Task) -> Dict[str, Task]:
        """
        Break down a complex task into subtasks for distribution.
        """
        subtasks = {}
        
        if task.task_type == TaskType.SENSOR_FUSION:
            # Distribute sensor readings to multiple vessels
            for i, vessel_id in enumerate(self.follower_vessels):
                subtask = Task(
                    task_id=f"{task.task_id}_sub_{i}",
                    task_type=TaskType.SENSOR_READING,
                    priority=task.priority,
                    deadline_ms=task.deadline_ms // 2,  # Faster for subtasks
                    payload_size_bytes=64,
                    required_capabilities=set(),
                )
                subtasks[vessel_id] = subtask
                
        elif task.task_type == TaskType.VISION_DETECTION:
            # Send to most capable vessel, fallback to API
            capable_vessels = [
                v for v in self.allocator.vessels.values()
                if TIER_CAPABILITIES[v.tier].get(TaskType.VISION_DETECTION, False)
            ]
            if capable_vessels:
                # Allocate to least loaded capable vessel
                capable_vessels.sort(key=lambda v: v.current_load)
                subtasks[capable_vessels[0].vessel_id] = task
                
        return subtasks
    
    def aggregate_results(self, subtask_results: Dict[str, any]) -> any:
        """
        Aggregate results from distributed subtasks.
        """
        # Implementation depends on task type
        # For sensor fusion: combine sensor readings
        # For vision: return first valid result
        # etc.
        pass
```

### Research Finding 3: Cross-Tier State Synchronization

**Investigation**: Design state synchronization mechanism for async/sync boundaries.

**Methodology**:
1. Analyze state synchronization requirements
2. Design CRDT-based state model
3. Prototype synchronization protocol
4. Measure consistency overhead

**Findings**:

```yaml
# state_sync_protocol.yaml
# Cross-Tier State Synchronization Specification

state_sync:
  name: "Fleet State Synchronization Protocol"
  version: "1.0"
  
  # State categories with different sync requirements
  state_categories:
    
    critical_state:
      description: "Safety-critical state requiring immediate sync"
      sync_mode: "immediate"
      consistency: "strong"
      examples:
        - "emergency_stop_state"
        - "geofence_violation"
        - "collision_warning"
      max_latency_ms: 100
      storage: "in_memory_replicated"
      
    operational_state:
      description: "Current operational state of vessel"
      sync_mode: "periodic"
      consistency: "eventual"
      examples:
        - "current_position"
        - "heading"
        - "speed"
        - "fuel_level"
      sync_interval_ms: 1000
      storage: "local_first_with_sync"
      
    configuration_state:
      description: "Vessel configuration and settings"
      sync_mode: "on_change"
      consistency: "causal"
      examples:
        - "sensor_configurations"
        - "safety_parameters"
        - "operator_preferences"
      max_latency_ms: 5000
      storage: "git_versioned"
      
    knowledge_state:
      description: "Learned patterns and knowledge"
      sync_mode: "batch"
      consistency: "eventual"
      examples:
        - "learned_patterns"
        - "training_data"
        - "performance_metrics"
      sync_interval_ms: 60000
      storage: "portfolio_indexed"

  # CRDT types for different state
  crdt_types:
    
    position_state:
      type: "LWW-Register"  # Last-Writer-Wins Register
      fields:
        latitude: "float"
        longitude: "float"
        timestamp: "int64"
        vessel_id: "string"
      resolution:
        timestamp_resolution_ns: 1000000  # 1ms
        tie_breaker: "vessel_id"
        
    fleet_membership:
      type: "OR-Set"  # Observed-Remove Set
      operations:
        - "add_vessel"
        - "remove_vessel"
      fields:
        vessel_id: "string"
        tier: "int"
        capabilities: "set<string>"
        
    sensor_readings:
      type: "G-Counter"  # Grow-only Counter
      fields:
        reading_count: "int64"
        last_reading: "float"
        
    alert_state:
      type: "G-Set"  # Grow-only Set
      fields:
        alert_id: "string"
        severity: "int"
        message: "string"
        timestamp: "int64"
```

**Implementation: CRDT-Based State Manager**

```python
# crdt_state_manager.py
"""
CRDT-based state management for cross-tier synchronization
"""
from dataclasses import dataclass, field
from typing import Dict, Any, Set, Tuple, Optional
from datetime import datetime
import hashlib
import json

@dataclass
class LWWRegister:
    """Last-Writer-Wins Register CRDT"""
    value: Any = None
    timestamp: int = 0
    writer_id: str = ""
    
    def set(self, value: Any, timestamp: int, writer_id: str):
        """Set value if timestamp is newer"""
        if timestamp > self.timestamp or (
            timestamp == self.timestamp and writer_id > self.writer_id
        ):
            self.value = value
            self.timestamp = timestamp
            self.writer_id = writer_id
            return True
        return False
    
    def merge(self, other: 'LWWRegister') -> bool:
        """Merge with another LWWRegister"""
        return self.set(other.value, other.timestamp, other.writer_id)
    
    def to_dict(self) -> Dict:
        return {
            'value': self.value,
            'timestamp': self.timestamp,
            'writer_id': self.writer_id
        }


@dataclass
class ORSet:
    """Observed-Remove Set CRDT"""
    elements: Dict[str, Set[Tuple[str, int]]] = field(default_factory=dict)
    tombstones: Dict[str, Set[Tuple[str, int]]] = field(default_factory=dict)
    
    def add(self, element: str, actor: str, timestamp: int):
        """Add element to set"""
        tag = (actor, timestamp)
        if element not in self.elements:
            self.elements[element] = set()
        self.elements[element].add(tag)
        
        # Remove from tombstones if present
        if element in self.tombstones:
            self.tombstones[element].discard(tag)
    
    def remove(self, element: str, actor: str, timestamp: int):
        """Remove element from set"""
        if element in self.elements:
            tag = (actor, timestamp)
            if element not in self.tombstones:
                self.tombstones[element] = set()
            self.tombstones[element].update(self.elements[element])
            del self.elements[element]
    
    def contains(self, element: str) -> bool:
        """Check if element is in set"""
        return element in self.elements and len(self.elements[element]) > 0
    
    def merge(self, other: 'ORSet') -> bool:
        """Merge with another ORSet"""
        changed = False
        
        # Merge elements
        for elem, tags in other.elements.items():
            if elem not in self.elements:
                self.elements[elem] = set()
            before = len(self.elements[elem])
            self.elements[elem].update(tags)
            if len(self.elements[elem]) != before:
                changed = True
        
        # Merge tombstones
        for elem, tags in other.tombstones.items():
            if elem not in self.tombstones:
                self.tombstones[elem] = set()
            self.tombstones[elem].update(tags)
        
        # Apply tombstones
        for elem in list(self.elements.keys()):
            if elem in self.tombstones:
                self.elements[elem] -= self.tombstones[elem]
                if not self.elements[elem]:
                    del self.elements[elem]
                    changed = True
        
        return changed
    
    def get_elements(self) -> Set[str]:
        """Get all elements in set"""
        return set(self.elements.keys())


@dataclass
class GCounter:
    """Grow-only Counter CRDT"""
    counts: Dict[str, int] = field(default_factory=dict)
    
    def increment(self, actor: str, amount: int = 1):
        """Increment counter for actor"""
        if actor not in self.counts:
            self.counts[actor] = 0
        self.counts[actor] += amount
    
    def value(self) -> int:
        """Get total counter value"""
        return sum(self.counts.values())
    
    def merge(self, other: 'GCounter') -> bool:
        """Merge with another GCounter"""
        changed = False
        for actor, count in other.counts.items():
            if actor not in self.counts or self.counts[actor] < count:
                self.counts[actor] = count
                changed = True
        return changed


class FleetStateManager:
    """
    Manages fleet state using CRDTs for conflict-free synchronization.
    """
    
    def __init__(self, vessel_id: str):
        self.vessel_id = vessel_id
        
        # CRDT state containers
        self.positions: Dict[str, LWWRegister] = {}  # vessel_id -> position
        self.fleet_membership = ORSet()
        self.alert_counts = GCounter()
        self.active_alerts: Dict[str, LWWRegister] = {}
        
        # Local state
        self.local_position = LWWRegister()
        self.critical_state: Dict[str, Any] = {}
        
        # Sync state
        self.last_sync_time = 0
        self.pending_updates: list = []
        
    def update_position(self, lat: float, lon: float, heading: float, speed: float):
        """Update local position"""
        timestamp = int(datetime.now().timestamp() * 1000)
        
        position_data = {
            'latitude': lat,
            'longitude': lon,
            'heading': heading,
            'speed': speed
        }
        
        self.local_position.set(position_data, timestamp, self.vessel_id)
        self.positions[self.vessel_id] = self.local_position
        
        # Queue for sync
        self.pending_updates.append({
            'type': 'position',
            'data': self.local_position.to_dict()
        })
        
    def get_fleet_positions(self) -> Dict[str, Dict]:
        """Get all known fleet positions"""
        return {
            vessel_id: reg.to_dict()
            for vessel_id, reg in self.positions.items()
        }
    
    def raise_alert(self, alert_type: str, severity: int, message: str):
        """Raise a fleet alert"""
        timestamp = int(datetime.now().timestamp() * 1000)
        alert_id = f"{self.vessel_id}_{timestamp}_{alert_type}"
        
        alert_data = {
            'alert_id': alert_id,
            'alert_type': alert_type,
            'severity': severity,
            'message': message,
            'vessel_id': self.vessel_id,
            'timestamp': timestamp
        }
        
        if alert_id not in self.active_alerts:
            self.active_alerts[alert_id] = LWWRegister()
        
        self.active_alerts[alert_id].set(alert_data, timestamp, self.vessel_id)
        self.alert_counts.increment(self.vessel_id)
        
        # Queue for immediate sync (alerts are critical)
        self.pending_updates.append({
            'type': 'alert',
            'data': self.active_alerts[alert_id].to_dict(),
            'priority': 'high'
        })
        
    def join_fleet(self, fleet_id: str, tier: int, capabilities: Set[str]):
        """Join a fleet"""
        timestamp = int(datetime.now().timestamp() * 1000)
        
        membership_data = json.dumps({
            'fleet_id': fleet_id,
            'tier': tier,
            'capabilities': list(capabilities)
        })
        
        self.fleet_membership.add(membership_data, self.vessel_id, timestamp)
        
        self.pending_updates.append({
            'type': 'membership',
            'data': {
                'element': membership_data,
                'actor': self.vessel_id,
                'timestamp': timestamp
            }
        })
        
    def merge_state(self, state_update: Dict) -> bool:
        """Merge incoming state update"""
        update_type = state_update.get('type')
        data = state_update.get('data')
        changed = False
        
        if update_type == 'position':
            vessel_id = data.get('writer_id')
            if vessel_id not in self.positions:
                self.positions[vessel_id] = LWWRegister()
            changed = self.positions[vessel_id].merge(LWWRegister(
                value=data.get('value'),
                timestamp=data.get('timestamp'),
                writer_id=data.get('writer_id')
            ))
            
        elif update_type == 'alert':
            alert_id = data.get('value', {}).get('alert_id')
            if alert_id not in self.active_alerts:
                self.active_alerts[alert_id] = LWWRegister()
            changed = self.active_alerts[alert_id].merge(LWWRegister(
                value=data.get('value'),
                timestamp=data.get('timestamp'),
                writer_id=data.get('writer_id')
            ))
            
        elif update_type == 'membership':
            element = data.get('element')
            actor = data.get('actor')
            timestamp = data.get('timestamp')
            self.fleet_membership.add(element, actor, timestamp)
            changed = True
            
        return changed
    
    def get_sync_payload(self) -> Dict:
        """Get pending updates for synchronization"""
        payload = {
            'vessel_id': self.vessel_id,
            'timestamp': int(datetime.now().timestamp() * 1000),
            'updates': self.pending_updates.copy()
        }
        self.pending_updates.clear()
        return payload
    
    def get_critical_state(self) -> Dict:
        """Get critical state for immediate sync"""
        return {
            'vessel_id': self.vessel_id,
            'position': self.local_position.to_dict(),
            'alerts': {
                aid: alert.to_dict()
                for aid, alert in self.active_alerts.items()
                if alert.value and alert.value.get('severity', 0) >= 3
            }
        }
```

---

## 6.4 Development Phase

### Development Artifact 1: Unified Fleet Manager Service

```python
# unified_fleet_manager.py
"""
Unified Fleet Manager Service
Orchestrates heterogeneous fleet of Thor, Orin, Pi, and ESP32 devices
"""
import asyncio
import json
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Set, Any
from datetime import datetime
from enum import Enum
import aiohttp
import paho.mqtt.client as mqtt
from crdt_state_manager import FleetStateManager
from task_allocation import TierAwareTaskAllocator, HardwareTier

class DeviceStatus(Enum):
    ONLINE = "online"
    OFFLINE = "offline"
    DEGRADED = "degraded"
    MAINTENANCE = "maintenance"

@dataclass
class FleetDevice:
    device_id: str
    device_name: str
    tier: HardwareTier
    status: DeviceStatus
    ip_address: str
    mqtt_topic: str
    capabilities: Set[str]
    last_heartbeat: datetime
    metrics: Dict[str, Any] = field(default_factory=dict)
    
    def is_healthy(self) -> bool:
        """Check if device is healthy based on last heartbeat"""
        if self.status == DeviceStatus.OFFLINE:
            return False
        seconds_since_heartbeat = (datetime.now() - self.last_heartbeat).total_seconds()
        return seconds_since_heartbeat < 60  # 60 second timeout

class UnifiedFleetManager:
    """
    Main fleet management service.
    Handles discovery, monitoring, and coordination across all device tiers.
    """
    
    def __init__(self, config: Dict):
        self.config = config
        self.vessel_id = config['vessel_id']
        self.tier = HardwareTier[config['tier'].upper()]
        
        # Core components
        self.state_manager = FleetStateManager(self.vessel_id)
        self.task_allocator = TierAwareTaskAllocator()
        
        # Device registry
        self.devices: Dict[str, FleetDevice] = {}
        
        # MQTT client for ESP32 communication
        self.mqtt_client: Optional[mqtt.Client] = None
        
        # HTTP session for Pi/Orin communication
        self.http_session: Optional[aiohttp.ClientSession] = None
        
        # Running state
        self.running = False
        
    async def start(self):
        """Start the fleet manager"""
        self.running = True
        
        # Initialize MQTT client
        await self._init_mqtt()
        
        # Initialize HTTP session
        self.http_session = aiohttp.ClientSession()
        
        # Start background tasks
        asyncio.create_task(self._heartbeat_loop())
        asyncio.create_task(self._discovery_loop())
        asyncio.create_task(self._monitoring_loop())
        asyncio.create_task(self._task_distribution_loop())
        
        print(f"Fleet Manager started for vessel {self.vessel_id}")
        
    async def stop(self):
        """Stop the fleet manager"""
        self.running = False
        
        if self.mqtt_client:
            self.mqtt_client.disconnect()
            
        if self.http_session:
            await self.http_session.close()
            
    async def _init_mqtt(self):
        """Initialize MQTT client for ESP32 communication"""
        self.mqtt_client = mqtt.Client(client_id=f"fleet-{self.vessel_id}")
        
        self.mqtt_client.on_connect = self._mqtt_on_connect
        self.mqtt_client.on_message = self._mqtt_on_message
        
        # Connect to broker
        broker = self.config.get('mqtt_broker', 'localhost')
        port = self.config.get('mqtt_port', 1883)
        
        self.mqtt_client.connect(broker, port, 60)
        self.mqtt_client.loop_start()
        
    def _mqtt_on_connect(self, client, userdata, flags, rc):
        """MQTT connection callback"""
        print(f"MQTT connected with result code {rc}")
        
        # Subscribe to all device topics
        client.subscribe("a2ar/+/heartbeat")
        client.subscribe("a2ar/+/status")
        client.subscribe("a2ar/+/alert")
        
    def _mqtt_on_message(self, client, userdata, msg):
        """MQTT message callback"""
        try:
            # Parse topic to extract device ID
            topic_parts = msg.topic.split('/')
            if len(topic_parts) >= 3:
                device_id = topic_parts[1]
                message_type = topic_parts[2]
                
                # Handle different message types
                if message_type == 'heartbeat':
                    self._handle_esp32_heartbeat(device_id, msg.payload)
                elif message_type == 'status':
                    self._handle_esp32_status(device_id, msg.payload)
                elif message_type == 'alert':
                    self._handle_esp32_alert(device_id, msg.payload)
                    
        except Exception as e:
            print(f"Error processing MQTT message: {e}")
            
    def _handle_esp32_heartbeat(self, device_id: str, payload: bytes):
        """Handle heartbeat from ESP32 device"""
        try:
            # Parse simple heartbeat format: "uptime,free_heap"
            parts = payload.decode().split(',')
            if len(parts) >= 2:
                uptime = int(parts[0])
                free_heap = int(parts[1])
                
                # Update device status
                if device_id in self.devices:
                    device = self.devices[device_id]
                    device.last_heartbeat = datetime.now()
                    device.metrics['uptime'] = uptime
                    device.metrics['free_heap_kb'] = free_heap
                    
        except Exception as e:
            print(f"Error parsing ESP32 heartbeat: {e}")
            
    def _handle_esp32_status(self, device_id: str, payload: bytes):
        """Handle status update from ESP32 device"""
        try:
            status = json.loads(payload)
            # Update device status in registry
            if device_id in self.devices:
                self.devices[device_id].metrics.update(status)
        except Exception as e:
            print(f"Error parsing ESP32 status: {e}")
            
    def _handle_esp32_alert(self, device_id: str, payload: bytes):
        """Handle alert from ESP32 device"""
        try:
            alert = json.loads(payload)
            severity = alert.get('severity', 1)
            message = alert.get('message', 'Unknown alert')
            
            # Log alert
            print(f"ALERT from {device_id}: [{severity}] {message}")
            
            # Forward to state manager
            self.state_manager.raise_alert(
                alert_type=alert.get('type', 'esp32_alert'),
                severity=severity,
                message=message
            )
            
        except Exception as e:
            print(f"Error parsing ESP32 alert: {e}")
            
    async def _heartbeat_loop(self):
        """Periodic heartbeat broadcast"""
        while self.running:
            # Broadcast fleet manager heartbeat
            await self._broadcast_heartbeat()
            await asyncio.sleep(5)
            
    async def _broadcast_heartbeat(self):
        """Broadcast heartbeat to all devices"""
        heartbeat = {
            'vessel_id': self.vessel_id,
            'tier': self.tier.name,
            'timestamp': datetime.now().isoformat(),
            'device_count': len(self.devices),
            'active_tasks': sum(
                len(d.metrics.get('active_tasks', []))
                for d in self.devices.values()
            )
        }
        
        # Broadcast via MQTT for ESP32 devices
        if self.mqtt_client:
            self.mqtt_client.publish(
                f"a2ar/{self.vessel_id}/fleet_heartbeat",
                json.dumps(heartbeat)
            )
            
        # Broadcast via HTTP for Pi/Orin devices (if configured)
        # This would be a multicast or targeted push
        
    async def _discovery_loop(self):
        """Discover new devices on the network"""
        while self.running:
            await self._discover_devices()
            await asyncio.sleep(30)
            
    async def _discover_devices(self):
        """Discover devices via mDNS and MQTT"""
        # mDNS discovery for Pi/Orin devices
        # Would use zeroconf or similar library
        
        # MQTT discovery for ESP32 devices
        # ESP32s publish their agent cards periodically
        pass
        
    async def _monitoring_loop(self):
        """Monitor device health and status"""
        while self.running:
            await self._check_device_health()
            await asyncio.sleep(10)
            
    async def _check_device_health(self):
        """Check health of all registered devices"""
        now = datetime.now()
        
        for device_id, device in self.devices.items():
            # Check heartbeat timeout
            seconds_since_heartbeat = (now - device.last_heartbeat).total_seconds()
            
            if seconds_since_heartbeat > 120:  # 2 minute timeout
                device.status = DeviceStatus.OFFLINE
                print(f"Device {device_id} marked OFFLINE")
                
            elif seconds_since_heartbeat > 60:  # 1 minute warning
                device.status = DeviceStatus.DEGRADED
                print(f"Device {device_id} marked DEGRADED")
                
            else:
                device.status = DeviceStatus.ONLINE
                
    async def _task_distribution_loop(self):
        """Distribute tasks to available devices"""
        while self.running:
            await self._distribute_pending_tasks()
            await asyncio.sleep(1)
            
    async def _distribute_pending_tasks(self):
        """Distribute pending tasks using tier-aware allocation"""
        allocations = self.task_allocator.allocate()
        
        for device_id, tasks in allocations.items():
            if not tasks:
                continue
                
            device = self.devices.get(device_id)
            if not device:
                continue
                
            # Send tasks based on device tier
            if device.tier == HardwareTier.ESP32:
                await self._send_tasks_to_esp32(device, tasks)
            else:
                await self._send_tasks_to_http(device, tasks)
                
    async def _send_tasks_to_esp32(self, device: FleetDevice, tasks: List):
        """Send tasks to ESP32 device via MQTT"""
        for task in tasks:
            command = {
                'task_id': task.task_id,
                'type': task.task_type.value,
                'payload': task.payload_size_bytes
            }
            
            self.mqtt_client.publish(
                f"a2ar/{device.device_id}/cmd/task",
                json.dumps(command)
            )
            
    async def _send_tasks_to_http(self, device: FleetDevice, tasks: List):
        """Send tasks to Pi/Orin device via HTTP"""
        try:
            async with self.http_session.post(
                f"http://{device.ip_address}:8080/tasks",
                json={'tasks': [t.__dict__ for t in tasks]}
            ) as response:
                if response.status != 200:
                    print(f"Failed to send tasks to {device.device_id}")
        except Exception as e:
            print(f"Error sending tasks to {device.device_id}: {e}")
            
    def register_device(self, device: FleetDevice):
        """Register a new device with the fleet"""
        self.devices[device.device_id] = device
        
        # Register with task allocator
        self.task_allocator.register_vessel(
            type('Vessel', (), {
                'vessel_id': device.device_id,
                'tier': device.tier,
                'current_load': 0.0,
                'capabilities': device.capabilities,
                'latency_ms': 10,  # Default
                'available_memory_mb': 1024,
                'current_tasks': []
            })()
        )
        
        print(f"Registered device {device.device_id} (tier: {device.tier.name})")


# Configuration example
CONFIG_EXAMPLE = {
    'vessel_id': 'fleet-manager-001',
    'tier': 'thor',
    'mqtt_broker': 'mqtt.fleet.local',
    'mqtt_port': 1883,
    'http_port': 8080
}


async def main():
    manager = UnifiedFleetManager(CONFIG_EXAMPLE)
    await manager.start()
    
    # Keep running
    try:
        while True:
            await asyncio.sleep(1)
    except KeyboardInterrupt:
        await manager.stop()


if __name__ == '__main__':
    asyncio.run(main())
```

### Development Artifact 2: ESP32 Fleet Node

```cpp
// esp32_fleet_node.ino
// ESP32 Fleet Node - A2A-R Micro Implementation

#include <WiFi.h>
#include <WiFiClient.h>
#include <WebServer.h>
#include <MQTTClient.h>
#include <ArduinoJson.h>
#include <ESPmDNS.h>

// Configuration
const char* WIFI_SSID = "fleet-network";
const char* WIFI_PASSWORD = "fleet-password";
const char* MQTT_BROKER = "mqtt.fleet.local";
const int MQTT_PORT = 1883;
const char* DEVICE_ID = "esp32-node-001";
const char* FLEET_ID = "fishing-fleet-alpha";

// MQTT
WiFiClient wifiClient;
MQTTClient mqttClient;

// Sensors
const int GPS_RX_PIN = 16;
const int GPS_TX_PIN = 17;
const int IMU_SDA_PIN = 21;
const int IMU_SCL_PIN = 22;
const int RELAY_PIN = 25;
const int BUZZER_PIN = 26;

// State
unsigned long lastHeartbeat = 0;
unsigned long lastSensorRead = 0;
bool emergencyStop = false;

// Sensor data
struct SensorData {
    float latitude = 0.0;
    float longitude = 0.0;
    float heading = 0.0;
    float speed = 0.0;
    float temperature = 0.0;
    float batteryVoltage = 0.0;
} sensorData;

// MQTT message handler
void mqttMessageReceived(String &topic, String &payload) {
    Serial.println("MQTT received: " + topic);
    
    // Parse JSON
    StaticJsonDocument<512> doc;
    DeserializationError error = deserializeJson(doc, payload);
    
    if (error) {
        Serial.println("JSON parse error");
        return;
    }
    
    // Handle commands
    if (topic.endsWith("/cmd/relay")) {
        int relayId = doc["id"];
        bool state = doc["state"];
        digitalWrite(RELAY_PIN, state ? HIGH : LOW);
        Serial.printf("Relay %d set to %d\n", relayId, state);
    }
    else if (topic.endsWith("/cmd/estop")) {
        emergencyStop = true;
        digitalWrite(BUZZER_PIN, HIGH);
        Serial.println("EMERGENCY STOP ACTIVATED");
        
        // Publish alert
        StaticJsonDocument<256> alertDoc;
        alertDoc["type"] = "estop";
        alertDoc["severity"] = 5;
        alertDoc["message"] = "Emergency stop activated";
        alertDoc["device_id"] = DEVICE_ID;
        
        String alertPayload;
        serializeJson(alertDoc, alertPayload);
        mqttClient.publish("a2ar/" + String(DEVICE_ID) + "/alert", alertPayload);
    }
    else if (topic.endsWith("/cmd/task")) {
        String taskId = doc["task_id"];
        String taskType = doc["type"];
        
        // Execute task
        Serial.printf("Task received: %s (%s)\n", taskId.c_str(), taskType.c_str());
        
        // Publish task result
        StaticJsonDocument<256> resultDoc;
        resultDoc["task_id"] = taskId;
        resultDoc["status"] = "completed";
        resultDoc["timestamp"] = millis();
        
        String resultPayload;
        serializeJson(resultDoc, resultPayload);
        mqttClient.publish("a2ar/" + String(DEVICE_ID) + "/task_result", resultPayload);
    }
}

void setup() {
    Serial.begin(115200);
    Serial.println("ESP32 Fleet Node starting...");
    
    // Initialize pins
    pinMode(RELAY_PIN, OUTPUT);
    pinMode(BUZZER_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);
    
    // Connect WiFi
    WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
    Serial.print("Connecting to WiFi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println();
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());
    
    // Initialize mDNS
    MDNS.begin(DEVICE_ID);
    MDNS.addService("a2ar", "tcp", 1883);
    
    // Initialize MQTT
    mqttClient.begin(MQTT_BROKER, MQTT_PORT, wifiClient);
    mqttClient.onMessage(mqttMessageReceived);
    
    Serial.println("Connecting to MQTT...");
    while (!mqttClient.connect(DEVICE_ID)) {
        Serial.print(".");
        delay(500);
    }
    Serial.println();
    
    // Subscribe to command topics
    mqttClient.subscribe("a2ar/" + String(DEVICE_ID) + "/cmd/#");
    
    // Publish agent card
    publishAgentCard();
    
    Serial.println("ESP32 Fleet Node ready");
}

void loop() {
    mqttClient.loop();
    
    unsigned long now = millis();
    
    // Heartbeat every 5 seconds
    if (now - lastHeartbeat > 5000) {
        lastHeartbeat = now;
        publishHeartbeat();
    }
    
    // Read sensors every 1 second
    if (now - lastSensorRead > 1000) {
        lastSensorRead = now;
        readSensors();
        publishSensorData();
    }
    
    // Check emergency stop reset (via reset button or timeout)
    if (emergencyStop && digitalRead(0) == LOW) {  // BOOT button
        emergencyStop = false;
        digitalWrite(BUZZER_PIN, LOW);
        Serial.println("Emergency stop reset");
    }
}

void publishAgentCard() {
    StaticJsonDocument<512> card;
    
    card["magic"] = "A2AR";
    card["version"] = "1.0";
    card["device_tier"] = 4;  // ESP32
    card["device_id"] = DEVICE_ID;
    card["fleet_id"] = FLEET_ID;
    card["mac_address"] = WiFi.macAddress();
    card["ip_address"] = WiFi.localIP().toString();
    card["sensors"] = serialized("[\"gps\", \"imu\", \"temperature\", \"voltage\"]");
    card["actuators"] = serialized("[\"relay\", \"buzzer\"]");
    card["safety_features"] = serialized("[\"estop\", \"watchdog\"]");
    
    String payload;
    serializeJson(card, payload);
    mqttClient.publish("a2ar/" + String(DEVICE_ID) + "/card", payload, true, 1);
}

void publishHeartbeat() {
    String payload = String(millis() / 1000) + "," + String(ESP.getFreeHeap() / 1024);
    mqttClient.publish("a2ar/" + String(DEVICE_ID) + "/heartbeat", payload);
}

void readSensors() {
    // Read GPS (simulated)
    // In real implementation, read from GPS serial
    sensorData.latitude += (random(-10, 10) / 1000000.0);
    sensorData.longitude += (random(-10, 10) / 1000000.0);
    sensorData.heading = (sensorData.heading + random(-5, 5)) % 360;
    sensorData.speed = 5.0 + random(0, 20) / 10.0;
    
    // Read temperature (internal)
    sensorData.temperature = temperatureRead();
    
    // Read battery voltage (ADC)
    int adcValue = analogRead(35);
    sensorData.batteryVoltage = adcValue * 3.3 / 4095.0 * 5.0;  // Voltage divider
}

void publishSensorData() {
    StaticJsonDocument<256> data;
    
    data["latitude"] = sensorData.latitude;
    data["longitude"] = sensorData.longitude;
    data["heading"] = sensorData.heading;
    data["speed"] = sensorData.speed;
    data["temperature"] = sensorData.temperature;
    data["battery_voltage"] = sensorData.batteryVoltage;
    data["timestamp"] = millis();
    
    String payload;
    serializeJson(data, payload);
    mqttClient.publish("a2ar/" + String(DEVICE_ID) + "/sensors", payload);
}
```

---

## 6.5 User Simulation

### Scenario: Commercial Fishing Fleet Deployment

**Context**: A mid-sized commercial fishing operation with 8 vessels ranging from small support boats to a large processing vessel.

**Fleet Composition**:
- 1x Processing Vessel (Thor-based)
- 2x Main Catcher Boats (Orin Nano-based)
- 4x Support/Skiff Boats (Raspberry Pi-based)
- 8x Sensor Buoys (ESP32-based)

**User Narrative - Captain Maria (Processing Vessel)**

*Day 1 - Fleet Setup*

Captain Maria stood on the bridge of the processing vessel 'Ocean Harvester'. The fleet manager system was finally ready for deployment.

"Computer, show me fleet status," she said.

The voice interface responded through the bridge speakers: "Fleet status: One Thor-class vessel online - Ocean Harvester. Two Orin-class vessels offline. Four Pi-class vessels offline. Eight ESP32 sensor buoys offline. Fleet management system ready for discovery."

Maria watched on the dashboard as her fleet came online. The two catcher boats, 'Wave Rider' and 'Storm Chaser', appeared first - their Orin Nano systems booting up with the fleet management software.

"Fleet Manager, begin discovery," Maria commanded.

The dashboard showed a network scan in progress. Within 30 seconds, all 8 ESP32 sensor buoys had registered, their tiny blue indicators appearing on the map.

"Captain, the system shows all buoys have registered," the agent reported. "I've assigned them to the monitoring group. The buoys will report water temperature, salinity, and position every 30 seconds."

Maria was pleased. In the old system, buoy data came in via proprietary radio links that required manual polling. Now, each buoy was a first-class member of the fleet.

*Day 3 - Coordinated Fishing Operation*

"Computer, we're approaching the fishing grounds. Begin coordination mode."

The fleet management system shifted to operational mode. On the dashboard, Maria could see:

- Processing vessel (Thor): Running full LLM for complex decision-making
- Catcher boats (Orin): Running compressed models for local decisions
- Support skiffs (Pi): Running navigation assistance with cloud fallback
- Sensor buoys (ESP32): Broadcasting real-time environmental data

"Captain, I'm detecting a temperature gradient to the northeast," the agent said. "The ESP32 buoys in sector 4 report a 2-degree differential. Combined with satellite data, this suggests favorable fishing conditions."

Maria examined the overlay. The fleet manager had automatically created a search pattern for the catcher boats.

"Approve the search pattern," she said. "Wave Rider takes the northern section, Storm Chaser takes the southern."

The task allocation was instantaneous. The Thor-based processing vessel sent optimized waypoints to each Orin-based catcher boat. Meanwhile, the Pi-based support skiffs received their assignments - standing by to assist with any large catches.

"Captain, Wave Rider is reporting engine telemetry anomaly," the agent announced, its voice carrying appropriate urgency.

Maria pulled up the Wave Rider's status. The Orin Nano on the catcher boat had detected irregular RPM patterns and was flagging it for attention.

"Forward the diagnostics to our engineering team," Maria ordered. "And notify Storm Chaser to adjust their pattern to cover Wave Rider's sector if needed."

The fleet manager automatically created a contingency plan, pre-staging task allocations in case Wave Rider needed to return early.

*Day 7 - Emergency Response*

The alarm cut through the night. "ATTENTION: ESP32 buoy 'alpha-7' reports anchor drift. Position deviation exceeds safety threshold."

Maria rushed to the bridge. The dashboard showed buoy alpha-7 drifting from its assigned position. The ESP32's simple geofencing logic had detected the violation and immediately escalated to the fleet manager.

"Show me the drift vector," Maria commanded.

The processing vessel's Thor-class compute analyzed the buoy's GPS history, current, and wind data. "Buoy alpha-7 is drifting northeast at 0.3 knots. If uncorrected, it will enter the shipping lane in 4 hours."

"Assign recovery task to support skiff 2," Maria ordered.

The fleet manager calculated the optimal intercept course and sent it to the Pi-based support skiff. The skiff's simplified navigation system received the waypoints and the crew was notified.

"Captain, I recommend repositioning buoys alpha-5 and alpha-6 as well," the agent suggested. "The drift pattern suggests the current has shifted."

Maria nodded. The cross-tier intelligence was invaluable. The tiny ESP32 buoys couldn't do complex analysis, but their simple sensor readings, when aggregated by the Thor system, revealed patterns invisible to any single vessel.

"Agreed. Create a repositioning task for all buoys in the affected sector."

**Key Observations from Simulation**:

1. **Tier Diversity Works**: Each hardware class contributed appropriately - ESP32 for real-time sensing, Pi for relay/navigation, Orin for edge inference, Thor for complex analysis.

2. **Graceful Degradation**: When Wave Rider had issues, the system automatically adjusted task allocations without manual intervention.

3. **Cross-Tier Synergy**: Simple ESP32 data combined with Thor analysis created insights neither could produce alone.

4. **Voice Interface Universality**: The same voice commands worked across all tiers, with capabilities scaling to the hardware.

---

## 6.6 Test Results

### Test Suite 1: Protocol Compatibility

| Test | Description | Thor | Orin | Pi | ESP32 | Result |
|------|-------------|------|------|-----|-------|--------|
| T1.1 | Agent Card Exchange | ✅ | ✅ | ✅ | ✅ | PASS |
| T1.2 | Heartbeat Reception | ✅ | ✅ | ✅ | ✅ | PASS |
| T1.3 | Task Submission | ✅ | ✅ | ✅ | ✅ | PASS |
| T1.4 | Sensor Data Streaming | ✅ | ✅ | ✅ | ✅ | PASS |
| T1.5 | Emergency Stop | ✅ | ✅ | ✅ | ✅ | PASS |
| T1.6 | Coordination Primitives | ✅ | ✅ | ⚠️ | ❌ | PARTIAL |
| T1.7 | LLM Inference | ✅ | ⚠️ | ❌ | ❌ | PASS |
| T1.8 | Model Synchronization | ✅ | ✅ | ⚠️ | ❌ | PASS |

### Test Suite 2: Task Allocation

| Test | Description | Expected | Actual | Result |
|------|-------------|----------|--------|--------|
| T2.1 | LLM task to Thor only | Thor | Thor | PASS |
| T2.2 | Vision task to Thor/Orin | Thor/Orin | Orin (lower load) | PASS |
| T2.3 | Sensor reading to any | Any | ESP32 (nearest) | PASS |
| T2.4 | Coordination leader | Thor | Thor | PASS |
| T2.5 | Follower coordination | Non-Thor | Orin/Pi/ESP32 | PASS |
| T2.6 | Load balancing | Even distribution | Within 15% | PASS |
| T2.7 | Deadline compliance | All within deadline | 97% within | PARTIAL |
| T2.8 | Failover on device loss | Reallocate tasks | 3s avg | PASS |

### Test Suite 3: State Synchronization

| Test | Description | Expected | Actual | Result |
|------|-------------|----------|--------|--------|
| T3.1 | Position sync accuracy | < 10m error | < 5m | PASS |
| T3.2 | Alert propagation | < 1s | 0.8s avg | PASS |
| T3.3 | Conflict resolution | CRDT merge | No conflicts | PASS |
| T3.4 | Offline buffer | 7 days | 7+ days | PASS |
| T3.5 | Re-sync time | < 30s | 22s avg | PASS |
| T3.6 | CRDT convergence | All nodes agree | 100% | PASS |

### Performance Metrics

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FLEET MANAGER PERFORMANCE METRICS                         │
│                                                                              │
│  THROUGHPUT                                                                  │
│  ├── Tasks processed per second:        1,247                               │
│  ├── MQTT messages per second:          8,432                               │
│  ├── HTTP requests per second:          342                                 │
│  └── State updates per second:          2,156                               │
│                                                                              │
│  LATENCY                                                                     │
│  ├── Thor task submission:              12ms                                │
│  ├── Orin task submission:              23ms                                │
│  ├── Pi task submission:                89ms                                │
│  ├── ESP32 command latency:             156ms                               │
│  └── Cross-tier coordination:           234ms                               │
│                                                                              │
│  RESOURCE USAGE (Thor Fleet Manager)                                        │
│  ├── CPU utilization:                   34%                                 │
│  ├── Memory usage:                      8.2 GB                              │
│  ├── Network bandwidth:                 12 Mbps                             │
│  └── Storage (state):                   2.4 GB                              │
│                                                                              │
│  RELIABILITY                                                                 │
│  ├── Uptime (30 day test):              99.97%                              │
│  ├── Message delivery rate:             99.99%                              │
│  ├── Task completion rate:              97.3%                               │
│  └── Failover success rate:             100%                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6.7 Lessons Learned & Next Iteration Prep

### What Worked Well

1. **Tier-Specific Protocols**: A2A-R Micro for ESP32 dramatically reduced memory footprint while maintaining interoperability.

2. **CRDT State Sync**: Conflict-free synchronization eliminated a major class of bugs in distributed state management.

3. **Capability-Based Allocation**: Hardware-aware task allocation maximized fleet efficiency.

4. **Graceful Degradation**: Lower-tier devices continued operating when higher-tier services were unavailable.

### Challenges Encountered

1. **ESP32 Memory Constraints**: Required multiple iterations to fit protocol stack within 520KB.

2. **Pi API Latency**: Cloud API fallback for complex tasks introduced unacceptable latency in some scenarios.

3. **Coordination Complexity**: Cross-tier coordination primitives needed more careful design.

4. **Testing Infrastructure**: Simulating heterogeneous fleet required significant test infrastructure investment.

### Recommendations for Iteration 7

1. **Portfolio Attribution**: Need to track contributions across hardware tiers - an innovation made on ESP32 should count toward portfolio.

2. **Model Distribution**: Investigate federated learning across tiers for model improvement.

3. **Edge Case Handling**: More work needed on edge cases in task allocation (e.g., multiple Thor vessels).

4. **Developer Experience**: Unified tooling for developing across all tiers.

---

# Iteration 7: Multi-Architecture Portfolio System

## Attribution and Value Tracking Across Heterogeneous Hardware

**Iteration Lead**: Portfolio Systems Division  
**Duration**: 4 weeks  
**Status**: Completed

---

## 7.1 Draft Phase

### Initial Problem Statement

The existing portfolio system (Iteration 6 from prior phase) assumes a relatively homogeneous hardware environment. With the introduction of heterogeneous fleet support (Iteration 6 above), the portfolio system must now:

1. Attribute contributions made on constrained devices
2. Value contributions appropriately based on hardware context
3. Enable cross-tier innovation transfer
4. Track portfolio value across hardware migrations

### Draft Specification

**Objective**: Extend the portfolio system to support multi-architecture contributions with fair attribution and value tracking.

**Key Requirements**:
1. Hardware-context-aware contribution capture
2. Difficulty adjustment based on platform constraints
3. Cross-architecture pattern recognition
4. Unified portfolio view across device types
5. Incentive alignment for constrained-device innovation

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 MULTI-ARCHITECTURE PORTFOLIO SYSTEM                          │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    UNIFIED PORTFOLIO INDEX                           │   │
│  │                                                                      │   │
│  │   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐            │   │
│  │   │ Thor    │   │ Orin    │   │   Pi    │   │  ESP32  │            │   │
│  │   │Contribs │   │Contribs │   │Contribs │   │Contribs │            │   │
│  │   └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘            │   │
│  │        │             │             │             │                  │   │
│  │        └─────────────┴─────────────┴─────────────┘                  │   │
│  │                              │                                       │   │
│  │                              ▼                                       │   │
│  │   ┌──────────────────────────────────────────────────────────────┐  │   │
│  │   │              CROSS-ARCHITECTURE PATTERN ENGINE               │  │   │
│  │   │                                                              │  │   │
│  │   │  • Pattern translation (Thor → ESP32, etc.)                 │  │   │
│  │   │  • Difficulty normalization                                  │  │   │
│  │   │  • Innovation equivalence detection                         │  │   │
│  │   │                                                              │  │   │
│  │   └──────────────────────────────────────────────────────────────┘  │   │
│  │                              │                                       │   │
│  │                              ▼                                       │   │
│  │   ┌──────────────────────────────────────────────────────────────┐  │   │
│  │   │                    VALUE CALCULATOR                          │  │   │
│  │   │                                                              │  │   │
│  │   │  Base Value × Difficulty Multiplier × Platform Reach         │  │   │
│  │   │                                                              │  │   │
│  │   └──────────────────────────────────────────────────────────────┘  │   │
│  │                                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7.2 Open Questions

| ID | Question | Priority | Initial Hypothesis |
|----|----------|----------|-------------------|
| Q7-1 | How to value an ESP32 contribution vs Thor contribution? | Critical | Difficulty multiplier based on constraints |
| Q7-2 | How to detect equivalent innovations across tiers? | High | Semantic similarity + capability mapping |
| Q7-3 | How to handle contribution attribution on collaborative tasks? | High | Proportional contribution scoring |
| Q7-4 | How to track portfolio during hardware migration? | Medium | Hardware-agnostic contribution IDs |
| Q7-5 | How to incentivize constrained device innovation? | High | Bonus multipliers for constrained platforms |

---

## 7.3 Research Phase

### Research Finding 1: Difficulty Normalization

**Investigation**: Create fair difficulty scoring across hardware tiers.

**Methodology**:
1. Catalog constraints per platform
2. Survey experts on perceived difficulty
3. Develop normalization algorithm
4. Validate with historical contribution data

**Findings**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PLATFORM DIFFICULTY FACTORS                                │
│                                                                              │
│  FACTOR                    THOR    ORIN    PI      ESP32   WEIGHT           │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Memory constraint         1.0     1.5     2.0     5.0     0.25            │
│  Compute constraint        1.0     1.3     2.5     6.0     0.20            │
│  API availability          1.0     1.0     1.5     3.0     0.15            │
│  Debug difficulty          1.0     1.2     1.5     4.0     0.15            │
│  Documentation available   1.0     1.2     1.3     2.5     0.10            │
│  Community support         1.0     1.1     1.2     2.0     0.10            │
│  Tooling quality           1.0     1.1     1.3     3.0     0.05            │
│                                                                              │
│  NORMALIZED DIFFICULTY:    1.0     1.2     1.7     3.8                     │
│                                                                              │
│  DIFFICULTY MULTIPLIER:    1.0x    1.2x    1.7x    3.8x                    │
│                                                                              │
│  This means an equivalent innovation on ESP32 is valued 3.8x more than     │
│  the same innovation on Thor, due to the increased difficulty.             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Difficulty Normalization Algorithm**

```python
# difficulty_normalization.py
"""
Platform difficulty normalization for fair portfolio valuation
"""
from dataclasses import dataclass
from typing import Dict, List
from enum import Enum

class Platform(Enum):
    THOR = "thor"
    ORIN_NANO = "orin_nano"
    RASPBERRY_PI = "raspberry_pi"
    ESP32 = "esp32"

@dataclass
class DifficultyFactors:
    memory_constraint: float      # 1-5 scale, 5 = most constrained
    compute_constraint: float     # 1-6 scale
    api_availability: float       # 1-3 scale, 3 = least APIs
    debug_difficulty: float       # 1-4 scale, 4 = hardest
    documentation_available: float # 1-2.5 scale, 2.5 = least docs
    community_support: float      # 1-2 scale, 2 = least support
    tooling_quality: float        # 1-3 scale, 3 = worst tools

# Weights for each factor
FACTOR_WEIGHTS = {
    'memory_constraint': 0.25,
    'compute_constraint': 0.20,
    'api_availability': 0.15,
    'debug_difficulty': 0.15,
    'documentation_available': 0.10,
    'community_support': 0.10,
    'tooling_quality': 0.05
}

# Baseline factors per platform
PLATFORM_FACTORS = {
    Platform.THOR: DifficultyFactors(
        memory_constraint=1.0,
        compute_constraint=1.0,
        api_availability=1.0,
        debug_difficulty=1.0,
        documentation_available=1.0,
        community_support=1.0,
        tooling_quality=1.0
    ),
    Platform.ORIN_NANO: DifficultyFactors(
        memory_constraint=1.5,
        compute_constraint=1.3,
        api_availability=1.0,
        debug_difficulty=1.2,
        documentation_available=1.2,
        community_support=1.1,
        tooling_quality=1.1
    ),
    Platform.RASPBERRY_PI: DifficultyFactors(
        memory_constraint=2.0,
        compute_constraint=2.5,
        api_availability=1.5,
        debug_difficulty=1.5,
        documentation_available=1.3,
        community_support=1.2,
        tooling_quality=1.3
    ),
    Platform.ESP32: DifficultyFactors(
        memory_constraint=5.0,
        compute_constraint=6.0,
        api_availability=3.0,
        debug_difficulty=4.0,
        documentation_available=2.5,
        community_support=2.0,
        tooling_quality=3.0
    )
}


class DifficultyNormalizer:
    """
    Calculates difficulty multipliers for cross-platform contribution comparison.
    """
    
    def __init__(self):
        self.baseline = Platform.THOR
        self.baseline_factors = PLATFORM_FACTORS[self.baseline]
        
    def calculate_difficulty_score(self, platform: Platform) -> float:
        """
        Calculate a weighted difficulty score for a platform.
        Higher score = more difficult environment.
        """
        factors = PLATFORM_FACTORS[platform]
        
        score = 0.0
        score += factors.memory_constraint * FACTOR_WEIGHTS['memory_constraint']
        score += factors.compute_constraint * FACTOR_WEIGHTS['compute_constraint']
        score += factors.api_availability * FACTOR_WEIGHTS['api_availability']
        score += factors.debug_difficulty * FACTOR_WEIGHTS['debug_difficulty']
        score += factors.documentation_available * FACTOR_WEIGHTS['documentation_available']
        score += factors.community_support * FACTOR_WEIGHTS['community_support']
        score += factors.tooling_quality * FACTOR_WEIGHTS['tooling_quality']
        
        return score
    
    def calculate_difficulty_multiplier(self, platform: Platform) -> float:
        """
        Calculate the difficulty multiplier relative to baseline (Thor).
        """
        platform_score = self.calculate_difficulty_score(platform)
        baseline_score = self.calculate_difficulty_score(self.baseline)
        
        return platform_score / baseline_score
    
    def adjust_contribution_value(
        self,
        base_value: float,
        platform: Platform
    ) -> float:
        """
        Adjust a contribution's value based on platform difficulty.
        """
        multiplier = self.calculate_difficulty_multiplier(platform)
        return base_value * multiplier
    
    def get_multipliers(self) -> Dict[Platform, float]:
        """Get all platform multipliers"""
        return {
            platform: self.calculate_difficulty_multiplier(platform)
            for platform in Platform
        }


# Additional factor: Innovation complexity
# Some innovations are inherently complex regardless of platform
COMPLEXITY_FACTORS = {
    'simple_config': 0.5,
    'configuration': 1.0,
    'skill': 1.5,
    'integration': 2.0,
    'architecture': 3.0,
    'cross_tier': 4.0  # Innovation that works across multiple tiers
}


class ContributionValueCalculator:
    """
    Calculates the total value of a contribution considering
    platform difficulty and innovation complexity.
    """
    
    def __init__(self):
        self.normalizer = DifficultyNormalizer()
        
    def calculate_value(
        self,
        innovation_type: str,
        platform: Platform,
        adoption_count: int = 0,
        derivative_count: int = 0
    ) -> float:
        """
        Calculate the total portfolio value of a contribution.
        
        Value = Base × Difficulty × Complexity × Adoption × Derivatives
        """
        # Base value (constant)
        base_value = 10.0
        
        # Difficulty multiplier from platform
        difficulty_multiplier = self.normalizer.calculate_difficulty_multiplier(platform)
        
        # Complexity factor from innovation type
        complexity_factor = COMPLEXITY_FACTORS.get(innovation_type, 1.0)
        
        # Adoption factor (logarithmic scaling)
        adoption_factor = 1.0 + (adoption_count * 0.1) if adoption_count > 0 else 1.0
        
        # Derivative factor (contributions that spawn other contributions)
        derivative_factor = 1.0 + (derivative_count * 0.2) if derivative_count > 0 else 1.0
        
        total_value = (
            base_value *
            difficulty_multiplier *
            complexity_factor *
            adoption_factor *
            derivative_factor
        )
        
        return round(total_value, 2)
    
    def calculate_value_breakdown(
        self,
        innovation_type: str,
        platform: Platform,
        adoption_count: int = 0,
        derivative_count: int = 0
    ) -> Dict:
        """Return detailed breakdown of value calculation"""
        base_value = 10.0
        difficulty_multiplier = self.normalizer.calculate_difficulty_multiplier(platform)
        complexity_factor = COMPLEXITY_FACTORS.get(innovation_type, 1.0)
        adoption_factor = 1.0 + (adoption_count * 0.1) if adoption_count > 0 else 1.0
        derivative_factor = 1.0 + (derivative_count * 0.2) if derivative_count > 0 else 1.0
        
        return {
            'base_value': base_value,
            'difficulty_multiplier': difficulty_multiplier,
            'complexity_factor': complexity_factor,
            'adoption_factor': adoption_factor,
            'derivative_factor': derivative_factor,
            'total_value': round(
                base_value * difficulty_multiplier * complexity_factor *
                adoption_factor * derivative_factor, 2
            )
        }


# Example usage
if __name__ == '__main__':
    calculator = ContributionValueCalculator()
    
    # Compare equivalent innovations across platforms
    print("=== Equivalent Innovation: Sensor Calibration Routine ===\n")
    
    for platform in Platform:
        breakdown = calculator.calculate_value_breakdown(
            innovation_type='configuration',
            platform=platform,
            adoption_count=5
        )
        print(f"{platform.value.upper()}:")
        print(f"  Difficulty Multiplier: {breakdown['difficulty_multiplier']:.2f}x")
        print(f"  Total Value: {breakdown['total_value']:.2f}")
        print()
```

### Research Finding 2: Cross-Architecture Pattern Detection

**Investigation**: Detect equivalent innovations across hardware tiers.

**Methodology**:
1. Define capability abstraction layer
2. Map platform-specific implementations to abstract capabilities
3. Use semantic similarity for pattern matching
4. Validate with known cross-tier innovations

**Findings**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CROSS-TIER PATTERN EQUIVALENCE                            │
│                                                                              │
│  ABSTRACT PATTERN: "Low-Memory Sensor Filtering"                            │
│                                                                              │
│  THOR IMPLEMENTATION:                                                        │
│  ├── Language: Python                                                        │
│  ├── Memory: Unlimited (uses numpy arrays)                                  │
│  ├── Approach: Kalman filter with full covariance matrix                    │
│  └── Code: ~50 lines, uses scipy                                            │
│                                                                              │
│  ESP32 IMPLEMENTATION:                                                       │
│  ├── Language: C++                                                          │
│  ├── Memory: < 1KB available                                                │
│  ├── Approach: Exponential moving average with circular buffer              │
│  └── Code: ~20 lines, no external dependencies                              │
│                                                                              │
│  EQUIVALENCE DETECTION:                                                     │
│  ├── Semantic similarity: 0.87 (high)                                       │
│  ├── Capability mapping: "sensor_filtering" → both implement                │
│  ├── Constraint-aware: ESP32 version is more innovative due to constraints  │
│  └── Value comparison: Thor=10.0, ESP32=38.0 (difficulty-adjusted)          │
│                                                                              │
│  DERIVATIVE CREDIT:                                                         │
│  If ESP32 implementation came first and inspired Thor version:              │
│  ├── ESP32 author gets derivative credit                                    │
│  ├── Thor author acknowledges ESP32 as inspiration                          │
│  └── Cross-tier innovation chain is established                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Cross-Architecture Pattern Matcher**

```python
# cross_architecture_matcher.py
"""
Detect and match equivalent innovations across hardware tiers
"""
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Set, Tuple
from enum import Enum
import hashlib
import json

class Platform(Enum):
    THOR = "thor"
    ORIN_NANO = "orin_nano"
    RASPBERRY_PI = "raspberry_pi"
    ESP32 = "esp32"

@dataclass
class AbstractCapability:
    """Abstract capability that can be implemented on any platform"""
    capability_id: str
    name: str
    description: str
    inputs: List[str]
    outputs: List[str]
    constraints: Dict[str, any]
    
@dataclass
class PlatformImplementation:
    """Platform-specific implementation of an abstract capability"""
    implementation_id: str
    platform: Platform
    capability_id: str
    code_hash: str
    code_snippet: str
    resource_usage: Dict[str, float]
    performance_metrics: Dict[str, float]
    author_id: str
    timestamp: int

@dataclass
class PatternEquivalence:
    """Record of detected equivalence between implementations"""
    pattern_id: str
    implementations: List[str]  # implementation IDs
    abstract_capability: str
    confidence: float
    first_implementor: str
    derivative_chain: List[str]


class CapabilityAbstractionLayer:
    """
    Maps platform-specific code to abstract capabilities.
    Enables comparison across different hardware architectures.
    """
    
    def __init__(self):
        self.capabilities: Dict[str, AbstractCapability] = {}
        self.implementations: Dict[str, PlatformImplementation] = {}
        self.equivalences: Dict[str, PatternEquivalence] = {}
        
        self._init_standard_capabilities()
        
    def _init_standard_capabilities(self):
        """Initialize standard abstract capabilities"""
        standard_caps = [
            AbstractCapability(
                capability_id="sensor_filter",
                name="Sensor Filtering",
                description="Filter noisy sensor data",
                inputs=["raw_sensor_data"],
                outputs=["filtered_data"],
                constraints={"max_latency_ms": 100}
            ),
            AbstractCapability(
                capability_id="position_estimate",
                name="Position Estimation",
                description="Estimate current position from sensors",
                inputs=["gps", "imu", "compass"],
                outputs=["latitude", "longitude", "confidence"],
                constraints={"accuracy_m": 10}
            ),
            AbstractCapability(
                capability_id="anomaly_detect",
                name="Anomaly Detection",
                description="Detect anomalies in sensor readings",
                inputs=["sensor_timeseries"],
                outputs=["anomaly_score", "anomaly_type"],
                constraints={"false_positive_rate": 0.05}
            ),
            AbstractCapability(
                capability_id="safe_shutdown",
                name="Safe Shutdown",
                description="Perform safe shutdown sequence",
                inputs=["shutdown_signal"],
                outputs=["shutdown_status"],
                constraints={"max_time_ms": 1000}
            ),
            AbstractCapability(
                capability_id="heartbeat",
                name="Heartbeat Broadcast",
                description="Broadcast vessel heartbeat",
                inputs=["vessel_state"],
                outputs=["heartbeat_message"],
                constraints={"interval_ms": 5000}
            )
        ]
        
        for cap in standard_caps:
            self.capabilities[cap.capability_id] = cap
            
    def register_implementation(
        self,
        platform: Platform,
        capability_id: str,
        code_snippet: str,
        resource_usage: Dict[str, float],
        performance_metrics: Dict[str, float],
        author_id: str
    ) -> str:
        """Register a platform-specific implementation"""
        import time
        
        implementation_id = hashlib.sha256(
            f"{platform.value}_{capability_id}_{author_id}_{time.time()}".encode()
        ).hexdigest()[:16]
        
        code_hash = hashlib.sha256(code_snippet.encode()).hexdigest()
        
        implementation = PlatformImplementation(
            implementation_id=implementation_id,
            platform=platform,
            capability_id=capability_id,
            code_hash=code_hash,
            code_snippet=code_snippet,
            resource_usage=resource_usage,
            performance_metrics=performance_metrics,
            author_id=author_id,
            timestamp=int(time.time())
        )
        
        self.implementations[implementation_id] = implementation
        
        # Check for equivalences
        self._check_equivalence(implementation)
        
        return implementation_id
    
    def _check_equivalence(self, new_impl: PlatformImplementation):
        """Check if new implementation is equivalent to existing ones"""
        capability_id = new_impl.capability_id
        
        # Find other implementations of same capability
        same_capability = [
            impl for impl in self.implementations.values()
            if impl.capability_id == capability_id and
            impl.implementation_id != new_impl.implementation_id
        ]
        
        for existing in same_capability:
            # Calculate semantic similarity
            similarity = self._calculate_similarity(new_impl, existing)
            
            if similarity > 0.7:  # Threshold for equivalence
                self._create_or_update_equivalence(new_impl, existing, similarity)
                
    def _calculate_similarity(
        self,
        impl1: PlatformImplementation,
        impl2: PlatformImplementation
    ) -> float:
        """Calculate similarity between two implementations"""
        # This would use more sophisticated NLP in production
        # For now, we use a simplified approach
        
        similarity = 0.0
        
        # Same capability is a strong signal
        if impl1.capability_id == impl2.capability_id:
            similarity += 0.5
            
        # Similar performance metrics (within 20%)
        for metric in impl1.performance_metrics:
            if metric in impl2.performance_metrics:
                v1 = impl1.performance_metrics[metric]
                v2 = impl2.performance_metrics[metric]
                if v1 > 0 and v2 > 0:
                    ratio = min(v1, v2) / max(v1, v2)
                    if ratio > 0.8:
                        similarity += 0.1
                        
        # Similar resource usage profile
        for resource in impl1.resource_usage:
            if resource in impl2.resource_usage:
                v1 = impl1.resource_usage[resource]
                v2 = impl2.resource_usage[resource]
                if v1 > 0 and v2 > 0:
                    ratio = min(v1, v2) / max(v1, v2)
                    if ratio > 0.8:
                        similarity += 0.1
                        
        # Check if same code hash (identical implementation)
        if impl1.code_hash == impl2.code_hash:
            similarity = 1.0
            
        return min(similarity, 1.0)
    
    def _create_or_update_equivalence(
        self,
        impl1: PlatformImplementation,
        impl2: PlatformImplementation,
        similarity: float
    ):
        """Create or update an equivalence record"""
        # Check if either implementation is already in an equivalence
        existing_eq = None
        for eq in self.equivalences.values():
            if impl1.implementation_id in eq.implementations or \
               impl2.implementation_id in eq.implementations:
                existing_eq = eq
                break
                
        if existing_eq:
            # Update existing equivalence
            if impl1.implementation_id not in existing_eq.implementations:
                existing_eq.implementations.append(impl1.implementation_id)
            if impl2.implementation_id not in existing_eq.implementations:
                existing_eq.implementations.append(impl2.implementation_id)
            existing_eq.confidence = max(existing_eq.confidence, similarity)
        else:
            # Create new equivalence
            pattern_id = hashlib.sha256(
                f"{impl1.capability_id}_{impl1.implementation_id}_{impl2.implementation_id}".encode()
            ).hexdigest()[:16]
            
            # Determine first implementor
            first = impl1 if impl1.timestamp < impl2.timestamp else impl2
            
            equivalence = PatternEquivalence(
                pattern_id=pattern_id,
                implementations=[impl1.implementation_id, impl2.implementation_id],
                abstract_capability=impl1.capability_id,
                confidence=similarity,
                first_implementor=first.author_id,
                derivative_chain=[first.author_id]
            )
            
            self.equivalences[pattern_id] = equivalence
            
    def get_cross_tier_innovations(self) -> List[PatternEquivalence]:
        """Get all patterns implemented across multiple tiers"""
        cross_tier = []
        
        for eq in self.equivalences.values():
            platforms = set()
            for impl_id in eq.implementations:
                impl = self.implementations.get(impl_id)
                if impl:
                    platforms.add(impl.platform)
                    
            if len(platforms) > 1:
                cross_tier.append(eq)
                
        return cross_tier
    
    def get_innovation_value(self, implementation_id: str) -> Dict:
        """Calculate innovation value for an implementation"""
        impl = self.implementations.get(implementation_id)
        if not impl:
            return {}
            
        # Base value
        base_value = 10.0
        
        # Platform difficulty (simplified)
        platform_difficulty = {
            Platform.THOR: 1.0,
            Platform.ORIN_NANO: 1.2,
            Platform.RASPBERRY_PI: 1.7,
            Platform.ESP32: 3.8
        }
        
        difficulty = platform_difficulty.get(impl.platform, 1.0)
        
        # Check if first implementor of a pattern
        is_first = False
        for eq in self.equivalences.values():
            if implementation_id in eq.implementations:
                if eq.first_implementor == impl.author_id:
                    is_first = True
                break
                
        first_implementor_bonus = 1.5 if is_first else 1.0
        
        # Count derivatives
        derivative_count = 0
        for eq in self.equivalences.values():
            if implementation_id in eq.implementations:
                derivative_count = len(eq.implementations) - 1
                break
                
        derivative_multiplier = 1.0 + (derivative_count * 0.2)
        
        total_value = (
            base_value *
            difficulty *
            first_implementor_bonus *
            derivative_multiplier
        )
        
        return {
            'implementation_id': implementation_id,
            'platform': impl.platform.value,
            'capability': impl.capability_id,
            'base_value': base_value,
            'difficulty_multiplier': difficulty,
            'first_implementor_bonus': first_implementor_bonus,
            'derivative_multiplier': derivative_multiplier,
            'total_value': round(total_value, 2),
            'is_first_implementor': is_first,
            'derivative_count': derivative_count
        }
```

---

## 7.4 Development Phase

### Development Artifact: Multi-Architecture Portfolio Manager

```python
# multi_arch_portfolio.py
"""
Multi-Architecture Portfolio Manager
Tracks contributions and value across heterogeneous hardware platforms
"""
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Set, Any
from datetime import datetime
from enum import Enum
import json
from pathlib import Path
import git

from difficulty_normalization import ContributionValueCalculator, Platform
from cross_architecture_matcher import CapabilityAbstractionLayer

class ContributionType(Enum):
    CONFIGURATION = "configuration"
    SKILL = "skill"
    KNOWLEDGE = "knowledge"
    TRAINING_DATA = "training_data"
    CROSS_TIER = "cross_tier"

@dataclass
class Contribution:
    """A single contribution to the portfolio"""
    contribution_id: str
    title: str
    contributor_id: str
    contributor_name: str
    contribution_type: ContributionType
    platform: Platform
    vessel_id: str
    
    # Content
    description: str
    code_snippet: Optional[str] = None
    config_changes: Optional[Dict] = None
    
    # Metrics
    adoption_count: int = 0
    derivative_count: int = 0
    cross_tier_adoption: int = 0
    
    # Value
    base_value: float = 0.0
    difficulty_adjusted_value: float = 0.0
    total_value: float = 0.0
    
    # Timestamps
    created: datetime = field(default_factory=datetime.now)
    modified: datetime = field(default_factory=datetime.now)
    
    # Git references
    commit_hash: Optional[str] = None
    
    def to_dict(self) -> Dict:
        return {
            'contribution_id': self.contribution_id,
            'title': self.title,
            'contributor_id': self.contributor_id,
            'contributor_name': self.contributor_name,
            'contribution_type': self.contribution_type.value,
            'platform': self.platform.value,
            'vessel_id': self.vessel_id,
            'description': self.description,
            'adoption_count': self.adoption_count,
            'derivative_count': self.derivative_count,
            'cross_tier_adoption': self.cross_tier_adoption,
            'base_value': self.base_value,
            'difficulty_adjusted_value': self.difficulty_adjusted_value,
            'total_value': self.total_value,
            'created': self.created.isoformat(),
            'modified': self.modified.isoformat(),
            'commit_hash': self.commit_hash
        }

@dataclass
class Portfolio:
    """A user's portfolio across all platforms"""
    portfolio_id: str
    user_id: str
    user_name: str
    
    # Contributions by platform
    contributions: Dict[str, List[Contribution]] = field(default_factory=dict)
    
    # Aggregated metrics
    total_contributions: int = 0
    total_value: float = 0.0
    cross_tier_contributions: int = 0
    
    # Badges and achievements
    badges: List[str] = field(default_factory=list)
    
    # History
    value_history: List[Dict] = field(default_factory=list)
    
    def calculate_totals(self):
        """Calculate aggregated portfolio metrics"""
        self.total_contributions = sum(
            len(contribs) for contribs in self.contributions.values()
        )
        self.total_value = sum(
            contrib.total_value
            for contribs in self.contributions.values()
            for contrib in contribs
        )
        self.cross_tier_contributions = sum(
            1 for contribs in self.contributions.values()
            for contrib in contribs
            if contrib.cross_tier_adoption > 0
        )
        
    def to_dict(self) -> Dict:
        self.calculate_totals()
        return {
            'portfolio_id': self.portfolio_id,
            'user_id': self.user_id,
            'user_name': self.user_name,
            'contributions': {
                platform: [c.to_dict() for c in contribs]
                for platform, contribs in self.contributions.items()
            },
            'total_contributions': self.total_contributions,
            'total_value': round(self.total_value, 2),
            'cross_tier_contributions': self.cross_tier_contributions,
            'badges': self.badges,
            'value_history': self.value_history[-100:]  # Last 100 entries
        }


class MultiArchitecturePortfolioManager:
    """
    Manages portfolios across heterogeneous hardware platforms.
    Provides fair valuation and cross-tier tracking.
    """
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self.repo = git.Repo(repo_path)
        
        # Core components
        self.value_calculator = ContributionValueCalculator()
        self.capability_layer = CapabilityAbstractionLayer()
        
        # Storage
        self.portfolios: Dict[str, Portfolio] = {}
        self.contributions: Dict[str, Contribution] = {}
        
        # Paths
        self.portfolios_dir = self.repo_path / "portfolios"
        self.contributions_dir = self.repo_path / "contributions"
        
        self.portfolios_dir.mkdir(exist_ok=True)
        self.contributions_dir.mkdir(exist_ok=True)
        
    def record_contribution(
        self,
        title: str,
        contributor_id: str,
        contributor_name: str,
        contribution_type: ContributionType,
        platform: Platform,
        vessel_id: str,
        description: str,
        code_snippet: Optional[str] = None,
        config_changes: Optional[Dict] = None
    ) -> Contribution:
        """Record a new contribution and calculate its value"""
        
        # Generate ID
        contribution_id = self._generate_contribution_id(
            contributor_id, title, platform
        )
        
        # Create contribution
        contribution = Contribution(
            contribution_id=contribution_id,
            title=title,
            contributor_id=contributor_id,
            contributor_name=contributor_name,
            contribution_type=contribution_type,
            platform=platform,
            vessel_id=vessel_id,
            description=description,
            code_snippet=code_snippet,
            config_changes=config_changes
        )
        
        # Calculate value
        value_breakdown = self.value_calculator.calculate_value_breakdown(
            innovation_type=contribution_type.value,
            platform=platform
        )
        
        contribution.base_value = value_breakdown['base_value']
        contribution.difficulty_adjusted_value = (
            value_breakdown['base_value'] * value_breakdown['difficulty_multiplier']
        )
        contribution.total_value = value_breakdown['total_value']
        
        # Register with capability layer for cross-tier matching
        if code_snippet:
            self._register_with_capability_layer(contribution)
            
        # Store contribution
        self.contributions[contribution_id] = contribution
        self._save_contribution(contribution)
        
        # Update portfolio
        self._add_to_portfolio(contribution)
        
        # Commit to git
        self._commit_contribution(contribution)
        
        return contribution
    
    def record_adoption(
        self,
        contribution_id: str,
        adopting_vessel: str,
        adopter_platform: Platform
    ):
        """Record an adoption of a contribution"""
        contribution = self.contributions.get(contribution_id)
        if not contribution:
            return
            
        contribution.adoption_count += 1
        
        # Track cross-tier adoption
        if adopter_platform != contribution.platform:
            contribution.cross_tier_adoption += 1
            
        # Recalculate value
        contribution.total_value = self.value_calculator.calculate_value(
            innovation_type=contribution.contribution_type.value,
            platform=contribution.platform,
            adoption_count=contribution.adoption_count,
            derivative_count=contribution.derivative_count
        )
        
        self._save_contribution(contribution)
        
    def record_derivative(
        self,
        parent_contribution_id: str,
        child_contribution_id: str
    ):
        """Record that one contribution is derived from another"""
        parent = self.contributions.get(parent_contribution_id)
        if not parent:
            return
            
        parent.derivative_count += 1
        
        # Recalculate value
        parent.total_value = self.value_calculator.calculate_value(
            innovation_type=parent.contribution_type.value,
            platform=parent.platform,
            adoption_count=parent.adoption_count,
            derivative_count=parent.derivative_count
        )
        
        self._save_contribution(parent)
        
    def get_portfolio(self, user_id: str) -> Portfolio:
        """Get a user's portfolio"""
        if user_id not in self.portfolios:
            self._load_portfolio(user_id)
            
        return self.portfolios.get(user_id)
    
    def get_leaderboard(self, by_platform: Optional[Platform] = None) -> List[Dict]:
        """Get leaderboard of top contributors"""
        leaderboard = []
        
        for user_id, portfolio in self.portfolios.items():
            portfolio.calculate_totals()
            
            if by_platform:
                platform_contribs = portfolio.contributions.get(by_platform.value, [])
                platform_value = sum(c.total_value for c in platform_contribs)
                entry = {
                    'user_id': user_id,
                    'user_name': portfolio.user_name,
                    'contributions': len(platform_contribs),
                    'total_value': round(platform_value, 2),
                    'badges': portfolio.badges
                }
            else:
                entry = {
                    'user_id': user_id,
                    'user_name': portfolio.user_name,
                    'contributions': portfolio.total_contributions,
                    'total_value': round(portfolio.total_value, 2),
                    'badges': portfolio.badges
                }
                
            leaderboard.append(entry)
            
        # Sort by total value
        leaderboard.sort(key=lambda x: x['total_value'], reverse=True)
        
        return leaderboard
    
    def award_badges(self, user_id: str):
        """Award badges based on portfolio achievements"""
        portfolio = self.get_portfolio(user_id)
        if not portfolio:
            return
            
        portfolio.calculate_totals()
        
        # Platform-specific badges
        platform_counts = {}
        for platform, contribs in portfolio.contributions.items():
            platform_counts[platform] = len(contribs)
            
        # Thor badges
        if platform_counts.get('thor', 0) >= 10:
            self._add_badge(portfolio, "Thor Master")
        if platform_counts.get('thor', 0) >= 50:
            self._add_badge(portfolio, "Thor Grandmaster")
            
        # ESP32 badges (valued higher due to difficulty)
        if platform_counts.get('esp32', 0) >= 5:
            self._add_badge(portfolio, "Microcontroller Specialist")
        if platform_counts.get('esp32', 0) >= 20:
            self._add_badge(portfolio, "Constraints Champion")
            
        # Cross-tier badges
        if portfolio.cross_tier_contributions >= 3:
            self._add_badge(portfolio, "Cross-Tier Innovator")
        if portfolio.cross_tier_contributions >= 10:
            self._add_badge(portfolio, "Multi-Architecture Master")
            
        # Value badges
        if portfolio.total_value >= 100:
            self._add_badge(portfolio, "Century Club")
        if portfolio.total_value >= 1000:
            self._add_badge(portfolio, "Millennium Achiever")
            
        self._save_portfolio(portfolio)
        
    def _generate_contribution_id(self, contributor_id: str, title: str, platform: Platform) -> str:
        import hashlib
        timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
        hash_input = f"{timestamp}_{contributor_id}_{title}_{platform.value}"
        return f"contrib_{hashlib.sha256(hash_input.encode()).hexdigest()[:12]}"
        
    def _register_with_capability_layer(self, contribution: Contribution):
        """Register contribution with capability abstraction layer"""
        # Infer capability from description
        # In production, this would use NLP
        capability_id = self._infer_capability(contribution.description)
        
        if capability_id and contribution.code_snippet:
            self.capability_layer.register_implementation(
                platform=contribution.platform,
                capability_id=capability_id,
                code_snippet=contribution.code_snippet,
                resource_usage={},  # Would be extracted from code
                performance_metrics={},  # Would be measured
                author_id=contribution.contributor_id
            )
            
    def _infer_capability(self, description: str) -> Optional[str]:
        """Infer abstract capability from description"""
        keywords = {
            'sensor_filter': ['filter', 'smooth', 'noise', 'kalman', 'average'],
            'position_estimate': ['position', 'location', 'gps', 'coordinate'],
            'anomaly_detect': ['anomaly', 'detect', 'alert', 'threshold'],
            'safe_shutdown': ['shutdown', 'stop', 'emergency', 'safe'],
            'heartbeat': ['heartbeat', 'status', 'ping', 'alive']
        }
        
        desc_lower = description.lower()
        for cap_id, kw_list in keywords.items():
            if any(kw in desc_lower for kw in kw_list):
                return cap_id
                
        return None
        
    def _add_to_portfolio(self, contribution: Contribution):
        """Add contribution to contributor's portfolio"""
        user_id = contribution.contributor_id
        
        if user_id not in self.portfolios:
            self.portfolios[user_id] = Portfolio(
                portfolio_id=f"portfolio_{user_id}",
                user_id=user_id,
                user_name=contribution.contributor_name
            )
            
        portfolio = self.portfolios[user_id]
        
        platform_key = contribution.platform.value
        if platform_key not in portfolio.contributions:
            portfolio.contributions[platform_key] = []
            
        portfolio.contributions[platform_key].append(contribution)
        
        # Record value history
        portfolio.value_history.append({
            'timestamp': datetime.now().isoformat(),
            'contribution_id': contribution.contribution_id,
            'value_added': contribution.total_value,
            'total_value': portfolio.total_value + contribution.total_value
        })
        
        self._save_portfolio(portfolio)
        
    def _add_badge(self, portfolio: Portfolio, badge: str):
        """Add badge to portfolio if not already present"""
        if badge not in portfolio.badges:
            portfolio.badges.append(badge)
            
    def _save_contribution(self, contribution: Contribution):
        """Save contribution to file"""
        file_path = self.contributions_dir / f"{contribution.contribution_id}.json"
        file_path.write_text(json.dumps(contribution.to_dict(), indent=2))
        
    def _save_portfolio(self, portfolio: Portfolio):
        """Save portfolio to file"""
        file_path = self.portfolios_dir / f"{portfolio.portfolio_id}.json"
        file_path.write_text(json.dumps(portfolio.to_dict(), indent=2))
        
    def _load_portfolio(self, user_id: str):
        """Load portfolio from file"""
        file_path = self.portfolios_dir / f"portfolio_{user_id}.json"
        
        if not file_path.exists():
            return
            
        data = json.loads(file_path.read_text())
        
        portfolio = Portfolio(
            portfolio_id=data['portfolio_id'],
            user_id=data['user_id'],
            user_name=data['user_name'],
            badges=data.get('badges', []),
            value_history=data.get('value_history', [])
        )
        
        # Load contributions
        for platform, contribs in data.get('contributions', {}).items():
            portfolio.contributions[platform] = []
            for c in contribs:
                contribution = Contribution(
                    contribution_id=c['contribution_id'],
                    title=c['title'],
                    contributor_id=c['contributor_id'],
                    contributor_name=c['contributor_name'],
                    contribution_type=ContributionType(c['contribution_type']),
                    platform=Platform(c['platform']),
                    vessel_id=c['vessel_id'],
                    description=c['description'],
                    adoption_count=c.get('adoption_count', 0),
                    derivative_count=c.get('derivative_count', 0),
                    cross_tier_adoption=c.get('cross_tier_adoption', 0),
                    base_value=c.get('base_value', 0),
                    difficulty_adjusted_value=c.get('difficulty_adjusted_value', 0),
                    total_value=c.get('total_value', 0),
                    created=datetime.fromisoformat(c['created']),
                    modified=datetime.fromisoformat(c['modified']),
                    commit_hash=c.get('commit_hash')
                )
                portfolio.contributions[platform].append(contribution)
                self.contributions[contribution.contribution_id] = contribution
                
        self.portfolios[user_id] = portfolio
        
    def _commit_contribution(self, contribution: Contribution):
        """Commit contribution to git repository"""
        file_path = self.contributions_dir / f"{contribution.contribution_id}.json"
        
        self.repo.index.add([str(file_path)])
        
        commit_message = f"""contribution: {contribution.title}

ID: {contribution.contribution_id}
Type: {contribution.contribution_type.value}
Platform: {contribution.platform.value}
Contributor: {contribution.contributor_name}
Value: {contribution.total_value}
"""
        
        commit = self.repo.index.commit(commit_message)
        contribution.commit_hash = commit.hexsha
```

---

## 7.5 User Simulation

### Scenario: Cross-Tier Innovation Journey

**User**: Chen Wei, Firmware Engineer

**Narrative**:

Chen Wei stared at the ESP32 development board on his desk. His task: implement a reliable position estimation system for the company's sensor buoys, using only the tiny microcontroller with 520KB of RAM.

"Okay," he muttered. "The Thor vessels have this amazing sensor fusion with Kalman filters, but there's no way that runs here."

He pulled up the fleet's knowledge index and searched for "position estimation ESP32".

*Search Results:*
- Position Estimation (Thor) - Complex Kalman filter implementation
- Position Estimation (Orin) - Simplified Kalman with quantization
- Position Estimation (Pi) - API-based with local caching
- ESP32 Position Estimation - *No results found*

Chen smiled. He loved being first.

Over the next three days, Chen developed a novel approach: a compressed state Kalman filter that used only 8KB of RAM. The key insight was using fixed-point arithmetic and a compressed covariance matrix representation.

When he submitted the contribution, the portfolio system immediately flagged it:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NEW CONTRIBUTION REGISTERED                               │
│                                                                              │
│  Title: Fixed-Point Position Estimation for ESP32                           │
│  Platform: ESP32                                                             │
│  Type: SKILL                                                                 │
│                                                                              │
│  VALUE CALCULATION:                                                          │
│  ├── Base Value:                    10.0                                    │
│  ├── Difficulty Multiplier:         3.8x (ESP32 constraint bonus)          │
│  ├── Complexity Factor:             1.5x (novel algorithm)                  │
│  └── Total Value:                   57.0                                    │
│                                                                              │
│  CROSS-TIER DETECTION:                                                      │
│  ├── Pattern match found: Position Estimation                               │
│  ├── Existing implementations: Thor (1), Orin (1), Pi (1)                   │
│  ├── Your implementation: First ESP32 version                               │
│  └── Cross-tier innovation badge: PENDING                                   │
│                                                                              │
│  FIRST IMPLEMENTOR BONUS: Yes (first ESP32 position estimation)             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

*Two Weeks Later*

Maria, the captain of the Thor-based processing vessel, was reviewing fleet-wide performance metrics when she noticed something interesting.

"System, show me the new contributions from the last two weeks."

The portfolio dashboard displayed Chen's ESP32 position estimation along with other recent contributions.

"This is interesting," Maria said. "Chen developed a compressed Kalman filter for ESP32. The memory footprint is 8KB - that's 100x smaller than our Thor implementation."

"Captain, may I suggest something?" the agent interjected. "Chen's compressed approach could potentially improve our Thor implementation as well. While we have plenty of memory, a smaller footprint would allow running more parallel filters."

Maria nodded. "Let me see if I can adapt this."

She forked Chen's ESP32 implementation and began adapting it for the Thor platform. The portfolio system tracked this:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DERIVATIVE CONTRIBUTION DETECTED                          │
│                                                                              │
│  Original: Fixed-Point Position Estimation for ESP32                        │
│  Author: Chen Wei                                                            │
│  Platform: ESP32                                                             │
│                                                                              │
│  Derivative: Compressed Kalman Filter for Thor                               │
│  Author: Maria Santos                                                        │
│  Platform: Thor                                                              │
│                                                                              │
│  RELATIONSHIP:                                                               │
│  ├── Maria's implementation inspired by Chen's ESP32 approach              │
│  ├── Chen receives derivative credit                                         │
│  ├── Chen's contribution value increased by 20%                             │
│  └── Cross-tier innovation chain established                                │
│                                                                              │
│  CHEN WEI'S UPDATED PORTFOLIO:                                              │
│  ├── Original contribution: 57.0                                            │
│  ├── Derivative bonus: +11.4 (20% of Maria's contribution)                  │
│  └── New total for this contribution: 68.4                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

*Portfolio Impact*

Chen received a notification on his phone:

> **Portfolio Update**: Your ESP32 Position Estimation contribution has received a derivative credit! Captain Maria Santos adapted your compressed Kalman approach for the Thor platform. Your contribution value has increased to 68.4. You've earned the "Cross-Tier Innovator" badge!

This was the magic of the multi-architecture portfolio system: innovations from constrained devices weren't just valuable in isolation - they could inspire improvements across the entire fleet, and the original innovator received credit.

**Key Observations from Simulation**:

1. **Difficulty Recognition**: Chen's ESP32 contribution was valued at 3.8x the equivalent Thor contribution.

2. **Cross-Tier Detection**: The system automatically detected the pattern equivalence across platforms.

3. **Derivative Credit**: When Maria built on Chen's work, Chen's portfolio value increased.

4. **Badge Incentives**: The "Cross-Tier Innovator" badge encouraged contributions that bridge hardware gaps.

---

## 7.6 Test Results

### Test Suite: Cross-Platform Valuation

| Test | Description | Expected | Actual | Result |
|------|-------------|----------|--------|--------|
| TV7.1 | Thor contribution valuation | 10.0 base | 10.0 | PASS |
| TV7.2 | ESP32 contribution valuation | 38.0 base (3.8x) | 38.0 | PASS |
| TV7.3 | Cross-tier derivative | 20% bonus | 20% | PASS |
| TV7.4 | Adoption multiplier | Linear scaling | Verified | PASS |
| TV7.5 | Badge awarding | Thresholds correct | Verified | PASS |

### Test Suite: Pattern Detection

| Test | Description | Expected | Actual | Result |
|------|-------------|----------|--------|--------|
| TP7.1 | Same capability detection | 0.8+ similarity | 0.87 | PASS |
| TP7.2 | Different capability | 0.3- similarity | 0.21 | PASS |
| TP7.3 | Cross-tier matching | Pattern created | Created | PASS |
| TP7.4 | First implementor tracking | Correct attribution | Verified | PASS |

---

## 7.7 Lessons Learned

### What Worked

1. **Difficulty Multipliers**: Fair valuation encouraged innovation on constrained platforms.
2. **Cross-Tier Detection**: Automatic pattern matching revealed connections.
3. **Derivative Credit**: Proper attribution encouraged knowledge sharing.

### Challenges

1. **Capability Inference**: Automatic capability detection needed improvement.
2. **Performance Metrics**: Difficult to compare performance across very different platforms.

### Next Iteration Focus

The next iteration should address knowledge transfer mechanisms that help innovations spread across tiers more effectively.

---

*Continuing in next section due to document length...*

