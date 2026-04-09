# Knowledge Base Wiki: Jetson Robotics Grand Design

> **Comprehensive Reference for Multi-Architecture Robotics Ecosystem**

---

## Quick Navigation

- [Architecture Overview](#architecture-overview)
- [Hardware Platforms](#hardware-platforms)
- [Software Stack](#software-stack)
- [Deployment Guides](#deployment-guides)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [FAQ](#faq)

---

## Architecture Overview

### What is the Jetson Robotics Grand Design?

The Jetson Robotics Grand Design is a comprehensive vision for a **self-improving, git-native robotics ecosystem** with NVIDIA Jetson devices at its center. The core paradigm:

> **The repository IS the agent. Git history is memory. Code is body. Commits are experiences. Pull requests are innovations.**

### Key Principles

1. **Human Value Increases** — Technology amplifies human value, not replaces it
2. **Git-Native Memory** — Every commit is a learnable experience
3. **Portfolio Economy** — Contributions compound in value over time
4. **Self-Improving** — The system learns from every interaction
5. **Open Architecture** — Fork, modify, contribute, benefit

### Supported Hardware Tiers

| Tier | Hardware | Use Case | Cost Range |
|------|----------|----------|------------|
| **Tier 1** | Jetson Thor (128GB) | Research, High-end vessels | $2,000+ |
| **Tier 2** | Jetson Orin Nano (8GB) | Commercial vessels | $500 |
| **Tier 3** | Raspberry Pi 5 + ESP32 | Small vessels, Edge IoT | $100 |

---

## Hardware Platforms

### Jetson Thor (Tier 1)

**Specifications**:
- 2070 FP4 TFLOPS AI performance
- 128GB unified memory
- Blackwell GPU architecture
- Native FP4 quantization support

**Best For**:
- Research vessels with complex AI needs
- Large commercial operations
- Vision-intensive applications
- Multi-agent coordination

**Configuration Guide**: See [Thor Architecture](../iterations/iter01_rom_emulation_architecture.md)

---

### Dual Orin Nano (Tier 2)

**Specifications**:
- 2x Orin Nano 8GB (redundant)
- 40 TOPS each (80 TOPS combined)
- Automatic failover capability
- ~$1,000 total hardware cost

**Best For**:
- Commercial fishing vessels
- Charter boats
- Industrial applications requiring reliability

**Configuration Guide**: See [Dual Orin Nano Architecture](../architectures/dual-orin-nano-commercial-fishing.md)

#### Redundancy Features

```
┌─────────────────────────────────────────────────────────────┐
│                    DUAL ORIN NANO REDUNDANCY                │
│                                                             │
│  Module A (UI/Primary)          Module B (Inference)        │
│  ┌─────────────────────┐        ┌─────────────────────┐    │
│  │ • Navigation        │        │ • LLM Chatbot       │    │
│  │ • Safety Systems    │        │ • Speech-to-Text    │    │
│  │ • Sensor Fusion     │◄──────►│ • Text-to-Speech    │    │
│  │ • Real-time Control │ Heart- │ • Ideation Engine   │    │
│  │                     │  beat  │                     │    │
│  └─────────────────────┘        └─────────────────────┘    │
│                                                             │
│  If Module A fails → Module B takes over (degraded)         │
│  If Module B fails → Module A takes over (degraded)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Raspberry Pi 5 + ESP32 Fleet (Tier 3)

**Specifications**:
- Raspberry Pi 5 8GB as hub
- Multiple ESP32 devices as sensors/actuators
- Cloud-heavy development via Codespaces
- ~$500 for 6-device fleet

**Best For**:
- Small inshore fishing boats
- Remote IoT installations
- Budget-conscious deployments
- Educational/training environments

**Configuration Guide**: See [Raspberry Pi ESP32 Architecture](../architectures/raspberry-pi-esp32-cloud-edge.md)

#### Git-Agent Concept

The ESP32 Git-Agent is a revolutionary concept where microcontrollers can:
1. Receive code updates via git-like protocol
2. Report status to Pi hub
3. Run autonomously for months
4. Be updated over-the-air without physical access

---

## Software Stack

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                    SOFTWARE STACK LAYERS                     │
│                                                             │
│  Layer 4: Applications                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Navigation • Voice Assistant • Portfolio • Safety   │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│  Layer 3: Agents                                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Capitaine (Framework) • Cocapn (Orchestrator)       │   │
│  │ Field-Captain (Deployment) • Nexus (Coordination)   │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│  Layer 2: Communication                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ A2A Protocol • A2A-R Extensions • MQTT • WebSocket  │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│  Layer 1: Platform                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ JetPack • Isaac ROS • CUDA • Docker • Linux         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Agent Types

| Agent | Purpose | Runs On |
|-------|---------|---------|
| **NavigationAgent** | Path planning, waypoint following | All tiers |
| **SafetyAgent** | Monitoring, alerts, emergency response | All tiers |
| **SpeciesAgent** | Fish detection, classification | Tier 1, 2 |
| **VoiceAgent** | Speech recognition, synthesis | Tier 1, 2 |
| **IdeationAgent** | Suggesting improvements | Tier 1 |
| **CoordinationAgent** | Fleet management | Tier 1, Cloud |

### A2A-R Protocol

The A2A-R Protocol extends Google's A2A Protocol for robotics:

| QoS Level | Latency | Use Case |
|-----------|---------|----------|
| **Safety-Critical** | <10ms | Emergency stop, collision avoidance |
| **Real-Time** | <50ms | Navigation control, sensor data |
| **Reliable** | <500ms | Telemetry, status updates |
| **Best-Effort** | No guarantee | Logs, non-critical data |

---

## Deployment Guides

### Quick Start: Dual Orin Nano

```bash
# 1. Flash both modules with JetPack
./scripts/flash-jetpack.sh --device /dev/ttyUSB0 --module-a
./scripts/flash-jetpack.sh --device /dev/ttyUSB1 --module-b

# 2. Configure network
# Module A: 192.168.1.1
# Module B: 192.168.1.2

# 3. Deploy software
ssh module-a.local
git clone https://github.com/your-org/cocapn.git
cd cocapn
./scripts/setup-module-a.sh --vessel-id your-vessel-id

ssh module-b.local
git clone https://github.com/your-org/cocapn.git
cd cocapn
./scripts/setup-module-b.sh --vessel-id your-vessel-id

# 4. Verify failover
./scripts/test-failover.sh
```

### Quick Start: Pi + ESP32 Fleet

```bash
# 1. Flash Raspberry Pi OS
# Use Raspberry Pi Imager with your settings

# 2. Install Docker
curl -fsSL https://get.docker.com | sh

# 3. Deploy hub services
git clone https://github.com/your-org/cocapn.git
cd cocapn
docker-compose -f docker-compose.pi.yml up -d

# 4. Provision ESP32 devices
./scripts/provision-device.py --device-id esp32-001 \
    --device-type temp-sensor \
    --wifi-ssid YourNetwork \
    --wifi-pass YourPassword

# 5. Verify fleet
curl http://pi-hub.local:8000/api/v1/fleet/status
```

---

## Troubleshooting

### Common Issues

#### Module Not Responding After Update

**Symptoms**: Module doesn't boot after firmware update

**Solution**:
```bash
# Enter recovery mode
# Hold recovery button while powering on

# Reflash from backup
./scripts/flash-recovery.sh --device /dev/ttyUSB0 --backup last-known-good.img
```

#### Voice Assistant Not Responding

**Symptoms**: Wake word detected but no response

**Diagnosis**:
```bash
# Check LLM service
docker logs llm

# Check TTS service
docker logs tts

# Test inference
curl -X POST http://localhost:8081/completion \
    -H "Content-Type: application/json" \
    -d '{"prompt": "Hello", "max_tokens": 50}'
```

**Solution**:
1. Restart LLM service: `docker restart llm`
2. Check memory usage: `free -h`
3. Verify model loaded: Check `/models/` directory

#### ESP32 Devices Offline

**Symptoms**: ESP32 devices not showing in fleet dashboard

**Diagnosis**:
```bash
# Check MQTT broker
docker logs mqtt

# Check device status
curl http://pi-hub.local:8000/api/v1/devices

# Ping device
ping esp32-001.local
```

**Solution**:
1. Check WiFi connectivity
2. Reboot device (power cycle)
3. Re-provision if needed

#### Split-Brain Condition

**Symptoms**: Both modules think the other is failed

**Diagnosis**:
```bash
# Check module status
ssh module-a.local './scripts/check-partner.sh'
ssh module-b.local './scripts/check-partner.sh'
```

**Solution**:
1. Check network cable between modules
2. Use physical switch to select active module
3. Restart both modules simultaneously

---

## Best Practices

### Development

1. **Use Codespaces for development** — Consistent environment, no local setup
2. **Test in simulation first** — Wokwi catches most issues
3. **Write tests for all new code** — Minimum 80% coverage
4. **Document as you go** — Update wiki with learnings
5. **Use feature branches** — Never commit directly to main

### Deployment

1. **Always backup before updates** — Use `./scripts/backup.sh`
2. **Use staged rollouts** — 10% → 50% → 100%
3. **Monitor after deployment** — Check logs for 1 hour
4. **Have rollback ready** — Know the previous good version
5. **Document configuration** — Keep `vessel-config.yaml` in repo

### Operations

1. **Regular health checks** — Daily automated checks
2. **Monitor temperatures** — Keep below 75°C
3. **Clean logs regularly** — Prevent disk filling
4. **Update models quarterly** — New versions improve performance
5. **Contribute innovations** — Build your portfolio

---

## FAQ

### General

**Q: What's the minimum hardware to run this system?**
A: Raspberry Pi 5 + one ESP32 device (~$150). Full features require Orin Nano or Thor.

**Q: Can I use this on non-marine applications?**
A: Yes! The architecture supports marine, aerial, industrial, home, and medical applications.

**Q: How do I contribute my innovations?**
A: Fork the repo, make improvements, submit PR. Your contributions become part of your portfolio.

**Q: Is there cloud connectivity required?**
A: No. All core functions work offline. Cloud adds sync, updates, and enhanced AI models.

### Hardware

**Q: Can I mix different hardware tiers in one fleet?**
A: Yes! The fleet management system handles heterogeneous fleets automatically.

**Q: What's the power consumption?**
A: 
- Thor: 60-100W
- Orin Nano: 7-15W each
- Pi 5: 5-12W
- ESP32: 0.1-0.5W

**Q: How long does battery backup last?**
A: With 12V 20Ah UPS:
- Thor: ~2 hours
- Orin Nano: ~15 hours
- Pi 5: ~20 hours

### Software

**Q: What LLM models are supported?**
A:
- Thor: Llama-3.2-70B, Llama-3.2-8B, custom models
- Orin Nano: Llama-3.2-3B-Q4, TinyLlama
- Pi 5: TinyLlama-1.1B-Q4 (cloud recommended for better models)

**Q: How does the portfolio system work?**
A: Every commit you make is verified via git-native mechanisms (not blockchain). As others adopt your innovations, your portfolio value increases.

**Q: Can I run this without internet?**
A: Yes. The system is designed for offline-first operation. Sync happens when connected.

### Commercial Fishing

**Q: What sensors are supported?**
A: GPS, compass, depth sounders, fish finders, weather stations, AIS, temperature, and custom NMEA 2000 devices.

**Q: Does it integrate with autopilot?**
A: Yes. The NavigationAgent can interface with standard marine autopilots via NMEA 2000 or 0183.

**Q: How accurate is the catch prediction?**
A: Accuracy depends on data quality. Vessels with 6+ months of logged data see 70-80% accuracy.

---

## Glossary

| Term | Definition |
|------|------------|
| **A2A** | Agent-to-Agent Protocol - Google's standard for agent communication |
| **A2A-R** | A2A Robotics - Extensions for real-time robotics operations |
| **Cocapn** | Meta-agent that orchestrates all other agents on a vessel |
| **Git-Agent** | ESP32 device that can receive code updates via git-like protocol |
| **MIG** | Multi-Instance GPU - NVIDIA technology for partitioning GPU |
| **Portfolio** | Collection of verified innovations attributed to a contributor |
| **Vessel** | A physical robot/device running the cocapn software |
| **Tier** | Hardware capability level (1=Thor, 2=Orin, 3=Pi) |

---

## Related Documentation

- [Developer Guide](../DEVELOPER_GUIDE.md)
- [File Schemas](../FILE_SCHEMAS.md)
- [Onboarding Guide](../ONBOARDING.md)
- [Roadmap](../ROADMAP.md)
- [Technical Deep Dive](../expert-analysis/expert_technical_deep_dive.md)

---

*Knowledge Base Wiki v1.0*
*Part of the Jetson Robotics Grand Design Documentation Suite*
