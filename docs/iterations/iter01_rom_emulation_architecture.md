# Iteration 1: ROM Emulation Architecture

## Jetson Thor Software Vessel Foundation Layer

**Document Version**: 1.0  
**Iteration**: 01  
**Date**: January 2025  
**Status**: Architectural Specification

---

## Executive Summary

This iteration defines the foundational architecture for deploying git-native agents on NVIDIA Jetson Thor hardware. The core design principle is a **hybrid ROM-emulation pattern**: a baked base layer providing stability and performance, combined with an interconnected application layer enabling evolution and customization.

### Key Design Decision

```
+------------------------------------------------------------------+
|                    HYBRID ARCHITECTURE                            |
+------------------------------------------------------------------+
|                                                                   |
|  ROM Base Layer (Baked)     +     Application Layer (Container)  |
|  ─────────────────────          ─────────────────────────────    |
|  • OS, Drivers, CUDA             • capitaine agent framework     |
|  • Isaac ROS 4.1                 • field-captain services        |
|  • Pre-optimized FP4 models      • nexus-runtime orchestration   |
|  • A2A server base               • User customizations           |
|                                                                   |
|  Flash every 6-12 months         Update via git pull daily       |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Part 1: ROM Base Layer Architecture

### 1.1 Philosophy: "Close to Metal, Baked-In Stability"

The ROM base layer embodies the principle of **immutable infrastructure** for the foundational components that rarely change and require maximum performance.

```
+------------------------------------------------------------------+
|                    ROM BASE LAYER STACK                           |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │ Layer 0: Hardware (Jetson Thor)                             │ |
|  │ ┌─────────────────────────────────────────────────────────┐ │ |
|  │ │ NVIDIA Thor SoC                                         │ │ |
|  │ │ ├── Blackwell GPU (2070 FP4 TFLOPS)                    │ │ |
|  │ │ ├── 128 GB Unified Memory                              │ │ |
|  │ │ ├── 12x ARM Cortex-A78AE CPU cores                     │ │ |
|  │ │ └── Deep Learning Accelerators                         │ │ |
|  │ └─────────────────────────────────────────────────────────┘ │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|  ┌───────────────────────────▼─────────────────────────────────┐ |
|  │ Layer 1: Hardware Abstraction (Baked)                       │ |
|  │ ┌─────────────────────────────────────────────────────────┐ │ |
|  │ │ JetPack 7.0 L4T (Linux for Tegra)                       │ │ |
|  │ │ ├── NVIDIA drivers (GPU, camera, GPIO)                  │ │ |
|  │ │ ├── CUDA 12.6+ runtime                                  │ │ |
|  │ │ ├── TensorRT 10.x inference engine                      │ │ |
|  │ │ └── cuDNN 9.x neural network primitives                 │ │ |
|  │ └─────────────────────────────────────────────────────────┘ │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|  ┌───────────────────────────▼─────────────────────────────────┐ |
|  │ Layer 2: Isaac ROS Foundation (Baked)                       │ |
|  │ ┌─────────────────────────────────────────────────────────┐ │ |
|  │ │ ROS 2 Humble Hawksbill                                  │ │ |
|  │ │ ├── isaac_ros_nvblox (3D reconstruction)                │ │ |
|  │ │ ├── isaac_ros_visual_slam                               │ │ |
|  │ │ ├── isaac_ros_espeak (TTS)                              │ │ |
|  │ │ ├── isaac_ros_rmp (motion planning)                     │ │ |
|  │ │ └── isaac_ros_nitros (hardware accel)                   │ │ |
|  │ └─────────────────────────────────────────────────────────┘ │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|  ┌───────────────────────────▼─────────────────────────────────┐ |
|  │ Layer 3: Pre-Optimized Models (Baked)                       │ |
|  │ ┌─────────────────────────────────────────────────────────┐ │ |
|  │ │ /opt/models/                                            │ │ |
|  │ │ ├── llm/                                                │ │ |
|  │ │ │   └── Qwen2.5-32B-Instruct-FP4.trt  (~16 GB)         │ │ |
|  │ │ ├── vision/                                             │ │ |
|  │ │ │   └── fishinglog-ai-v2.trt                            │ │ |
|  │ │ └── voice/                                              │ │ |
|  │ │     ├── whisper-large-v3.trt                            │ │ |
|  │ │     └── piper-tts.thor-optimized.onnx                   │ │ |
|  │ └─────────────────────────────────────────────────────────┘ │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|  ┌───────────────────────────▼─────────────────────────────────┐ |
|  │ Layer 4: A2A Protocol Server (Baked)                        │ |
|  │ ┌─────────────────────────────────────────────────────────┐ │ |
|  │ │ /opt/a2a/                                               │ │ |
|  │ │ ├── a2a-server (FastAPI + uvicorn)                      │ │ |
|  │ │ ├── agent-card templates                                │ │ |
|  │ │ └── latency broadcast service                           │ │ |
|  │ └─────────────────────────────────────────────────────────┘ │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 ROM Image Specification

```yaml
# rom-image-spec.yaml
# Specification for field-captain-thor-rom.img

metadata:
  name: "field-captain-thor-rom"
  version: "1.0.0"
  target_hardware: "nvidia-jetson-thor"
  build_date: "${BUILD_DATE}"
  image_size_gb: 64

# Base System
base_system:
  os: "ubuntu-22.04-l4t"
  jetpack_version: "7.0"
  kernel_version: "5.15-l4t"
  
  packages:
    essential:
      - cuda-toolkit-12.6
      - tensorrt-10.x
      - cudnn-9.x
      - nvidia-container-runtime
      - docker-ce
      - docker-compose-plugin
      
    utilities:
      - git
      - curl
      - htop
      - iotop
      - nvtop
      - python3.10
      - python3-pip

# Pre-baked Models
models:
  llm:
    name: "Qwen/Qwen2.5-32B-Instruct"
    quantization: "FP4"
    format: "TensorRT"
    path: "/opt/models/llm/"
    memory_footprint_gb: 16
    inference_latency_ms: 150  # First token
    
  vision:
    name: "fishinglog-ai-v2"
    format: "TensorRT"
    path: "/opt/models/vision/"
    supported_classes: 1500
    
  voice_stt:
    name: "whisper-large-v3"
    quantization: "FP16"
    format: "TensorRT"
    path: "/opt/models/voice/whisper/"
    
  voice_tts:
    name: "piper-tts"
    format: "ONNX"
    path: "/opt/models/voice/tts/"

# Isaac ROS Packages
isaac_ros:
  version: "4.1"
  ros_distro: "humble"
  
  packages:
    - isaac_ros_nvblox
    - isaac_ros_visual_slam
    - isaac_ros_espeak
    - isaac_ros_rmp
    - isaac_ros_nitros
    - isaac_ros_image_pipeline
    - isaac_ros_depth

# A2A Server
a2a_server:
  port: 8080
  features:
    - task_execution
    - streaming
    - latency_broadcast
    - cancellation
    
# MIG Configuration
mig:
  enabled: true
  partition_profile: "balanced_4x32"
  partitions:
    - id: 0
      memory_gb: 32
      compute_fraction: 0.25
      purpose: "llm_inference"
    - id: 1
      memory_gb: 32
      compute_fraction: 0.25
      purpose: "vision_pipeline"
    - id: 2
      memory_gb: 32
      compute_fraction: 0.25
      purpose: "voice_processing"
    - id: 3
      memory_gb: 32
      compute_fraction: 0.25
      purpose: "training_uplift"

# Power Management
power:
  default_mode: "performance"  # 130W
  available_modes:
    - name: "performance"
      power_w: 130
      cpu_governor: "performance"
    - name: "balanced"
      power_w: 80
      cpu_governor: "ondemand"
    - name: "power_saver"
      power_w: 40
      cpu_governor: "powersave"
```

### 1.3 ROM Image Build Process

```bash
#!/bin/bash
# build-rom-image.sh
# Build the field-captain ROM image for Jetson Thor

set -e

# Configuration
ROM_NAME="field-captain-thor-rom"
VERSION="1.0.0"
BUILD_DIR="/tmp/rom-build"

echo "=== Building ${ROM_NAME} v${VERSION} ==="

# Step 1: Initialize base image (JetPack 7.0 SDK Manager)
echo "[1/6] Initializing base L4T image..."
sudo ./scripts/init-jetpack-image.sh \
  --version 7.0 \
  --output "${BUILD_DIR}/base.img"

# Step 2: Install CUDA and TensorRT
echo "[2/6] Installing CUDA 12.6 and TensorRT 10.x..."
sudo chroot "${BUILD_DIR}/rootfs" /bin/bash <<EOF
  apt-get update
  apt-get install -y cuda-toolkit-12-6 tensorrt cudnn9
EOF

# Step 3: Install Isaac ROS 4.1
echo "[3/6] Installing Isaac ROS 4.1..."
sudo chroot "${BUILD_DIR}/rootfs" /bin/bash <<EOF
  # ROS 2 Humble
  apt-get install -y ros-humble-ros-base
  
  # Isaac ROS packages
  apt-get install -y \
    ros-humble-isaac-ros-nvblox \
    ros-humble-isaac-ros-visual-slam \
    ros-humble-isaac-ros-espeak \
    ros-humble-isaac-ros-rmp \
    ros-humble-isaac-ros-nitros
EOF

# Step 4: Bake optimized models (FP4 quantization)
echo "[4/6] Baking FP4-optimized models..."
python3 scripts/bake-models.py \
  --models config/models.yaml \
  --output "${BUILD_DIR}/rootfs/opt/models/" \
  --quantization fp4

# Step 5: Install A2A server
echo "[5/6] Installing A2A protocol server..."
sudo chroot "${BUILD_DIR}/rootfs" /bin/bash <<EOF
  pip3 install fastapi uvicorn pynvml
  
  mkdir -p /opt/a2a
  cp -r /build/a2a-server/* /opt/a2a/
  
  # Create systemd service
  cat > /etc/systemd/system/a2a-server.service <<SERVICE
[Unit]
Description=Field Captain A2A Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/a2a/server.py
Restart=always

[Install]
WantedBy=multi-user.target
SERVICE
  
  systemctl enable a2a-server
EOF

# Step 6: Configure MIG and finalize
echo "[6/6] Configuring MIG and finalizing..."
sudo chroot "${BUILD_DIR}/rootfs" /bin/bash <<EOF
  # Enable MIG
  nvidia-smi -mig 1
  
  # Create partition profile
  nvidia-smi mig -cgi 19,19,19,19 -C  # 4x balanced partitions
  
  # Finalize image
  apt-get clean
  rm -rf /var/lib/apt/lists/*
EOF

# Create final image
echo "Creating final image..."
sudo ./scripts/create-image.sh \
  --rootfs "${BUILD_DIR}/rootfs" \
  --output "${ROM_NAME}-${VERSION}.img"

echo "=== Build complete: ${ROM_NAME}-${VERSION}.img ==="
```

---

## Part 2: Interconnected Application Layer

### 2.1 Philosophy: "Modular, Evolvable, Git-Native"

The application layer runs in containers on top of the ROM base, enabling rapid iteration, customization, and git-native agent behavior.

```
+------------------------------------------------------------------+
|              INTERCONNECTED APPLICATION LAYER                      |
+------------------------------------------------------------------+
|                                                                   |
|  Container Network: agent-network                                 |
|  Communication: A2A Protocol + ROS 2 DDS                          |
|                                                                   |
|  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐               |
|  │ Container 1 │  │ Container 2 │  │ Container 3 │               |
|  │             │  │             │  │             │               |
|  │ capitaine   │  │ field-      │  │ nexus-      │               |
|  │ ─────────── │  │ captain     │  │ runtime     │               |
|  │             │  │ ──────────  │  │ ──────────  │               |
|  │ Git-native  │  │ Jetson-     │  │ Fleet       │               |
|  │ agent       │  │ optimized   │  │ coordinator │               |
|  │ framework   │  │ services    │  │             │               |
|  │             │  │             │  │             │               |
|  │ MIG: Part 0 │  │ MIG: Part 1 │  │ MIG: Part 2 │               |
|  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘               |
|         │                │                │                       |
|         └────────────────┴────────────────┘                       |
|                          │                                        |
|                   A2A Protocol                                   |
|                   (HTTP/WebSocket)                                |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.2 Container Definitions

```yaml
# docker-compose.yaml
# Application layer containers

version: "3.9"

networks:
  agent-network:
    driver: bridge
    
volumes:
  agent-memory:
  training-data:
  model-cache:

services:
  # ═══════════════════════════════════════════════════════════════
  # CONTAINER 1: capitaine - Git-Native Agent Framework
  # ═══════════════════════════════════════════════════════════════
  capitaine:
    image: lucineer/capitaine:latest
    build:
      context: ./packages/capitaine
      dockerfile: Dockerfile.thor
    
    container_name: capitaine-agent
    
    volumes:
      # Agent repository (git history = memory)
      - ./agent-repo:/app/repo:rw
      # Persistent memory storage
      - agent-memory:/app/memory
      # Configuration
      - ./config/agent.yaml:/app/config/agent.yaml:ro
      
    environment:
      - AGENT_ID=capitaine-${HOSTNAME}
      - A2A_PORT=8080
      - MEMORY_TIER=main_context
      - GIT_USER_NAME=${AGENT_NAME:-Field Agent}
      - GIT_USER_EMAIL=${AGENT_EMAIL:-agent@field.local}
      
    ports:
      - "8080:8080"
      
    # MIG partition assignment
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['0']  # MIG partition 0
              capabilities: [gpu]
              
    networks:
      - agent-network
      
    restart: unless-stopped
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ═══════════════════════════════════════════════════════════════
  # CONTAINER 2: field-captain - Jetson-Optimized Services
  # ═══════════════════════════════════════════════════════════════
  field-captain:
    image: lucineer/field-captain:latest
    build:
      context: ./packages/field-captain
      dockerfile: Dockerfile.thor
    
    container_name: field-captain
    
    volumes:
      # Pre-baked models from ROM
      - /opt/models:/app/models:ro
      # Training data collection
      - training-data:/app/training
      # ROS configuration
      - ./config/ros:/app/config/ros:ro
      
    environment:
      - LLM_MODEL=/app/models/llm/Qwen2.5-32B-Instruct-FP4.trt
      - VISION_MODEL=/app/models/vision/fishinglog-ai-v2.trt
      - STT_MODEL=/app/models/voice/whisper/whisper-large-v3.trt
      - TTS_MODEL=/app/models/voice/tts/piper-tts.onnx
      - ROS_DOMAIN_ID=0
      - A2A_CAPITAINE_URL=http://capitaine:8080
      
    ports:
      - "8081:8081"
      
    # MIG partition assignment
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['1']  # MIG partition 1
              capabilities: [gpu]
              
    depends_on:
      capitaine:
        condition: service_healthy
        
    networks:
      - agent-network
      
    restart: unless-stopped

  # ═══════════════════════════════════════════════════════════════
  # CONTAINER 3: nexus-runtime - Fleet Orchestration
  # ═══════════════════════════════════════════════════════════════
  nexus-runtime:
    image: superinstance/nexus-runtime:latest
    build:
      context: ./packages/nexus-runtime
      dockerfile: Dockerfile.thor
    
    container_name: nexus-runtime
    
    volumes:
      - ./fleet-config:/app/config:ro
      
    environment:
      - FLEET_ID=nexus-${HOSTNAME}
      - DISCOVERY_MODE=mdns
      - DISCOVERY_DOMAIN=agent.local
      - A2A_FIELD_CAPTAIN_URL=http://field-captain:8081
      
    ports:
      - "8082:8082"
      
    # MIG partition assignment
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['2']  # MIG partition 2
              capabilities: [gpu]
              
    depends_on:
      - field-captain
      
    networks:
      - agent-network
      
    restart: unless-stopped

  # ═══════════════════════════════════════════════════════════════
  # CONTAINER 4: training - Self-Improvement Pipeline
  # ═══════════════════════════════════════════════════════════════
  training:
    image: lucineer/training-pipeline:latest
    build:
      context: ./packages/training
      dockerfile: Dockerfile.thor
    
    container_name: training-pipeline
    
    volumes:
      - training-data:/app/data
      - /opt/models:/app/base-models:ro
      - model-cache:/app/output
      
    environment:
      - TRAINING_INTERVAL_HOURS=24
      - MIN_EXPERIENCES=500
      - LORA_R=16
      - LORA_ALPHA=32
      
    # MIG partition assignment
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['3']  # MIG partition 3
              capabilities: [gpu]
              
    networks:
      - agent-network
      
    restart: unless-stopped
```

### 2.3 Container Lifecycle Management

```python
# container_lifecycle.py
"""
Manage container lifecycle for the application layer
Coordinates with ROM base layer services
"""

import subprocess
import json
import time
from pathlib import Path
from typing import Dict, List, Optional
import docker
import git

class ApplicationLayerManager:
    """
    Manages the interconnected application layer
    Coordinates updates, health, and coordination with ROM base
    """
    
    def __init__(self, compose_path: str):
        self.compose_path = Path(compose_path)
        self.docker_client = docker.from_env()
        self.containers = {}
        
    def deploy(self):
        """
        Deploy the application layer
        Called after ROM base is verified
        """
        # Verify ROM base is ready
        if not self._verify_rom_base():
            raise RuntimeError("ROM base layer not ready")
            
        # Pull latest images
        subprocess.run([
            "docker", "compose", "-f", str(self.compose_path), "pull"
        ], check=True)
        
        # Start containers
        subprocess.run([
            "docker", "compose", "-f", str(self.compose_path), "up", "-d"
        ], check=True)
        
        # Wait for health checks
        self._wait_for_healthy()
        
        # Register with ROM A2A server
        self._register_with_rom()
        
    def update_container(self, container_name: str):
        """
        Update a single container (hot-swap)
        Maintains service continuity
        """
        # Pull new image
        subprocess.run([
            "docker", "compose", "-f", str(self.compose_path),
            "pull", container_name
        ], check=True)
        
        # Recreate container
        subprocess.run([
            "docker", "compose", "-f", str(self.compose_path),
            "up", "-d", "--force-recreate", container_name
        ], check=True)
        
        # Verify health
        self._wait_for_container_healthy(container_name)
        
    def update_agent_repo(self, repo_path: str):
        """
        Update the git-native agent repository
        This is how agents "learn" and "evolve"
        """
        repo = git.Repo(repo_path)
        
        # Fetch upstream
        origin = repo.remotes.origin
        origin.fetch()
        
        # Get current commit
        current = repo.head.commit.hexsha
        
        # Pull changes
        origin.pull()
        
        # Get new commit
        new = repo.head.commit.hexsha
        
        if current != new:
            # Agent has new experiences/memories
            # Restart agent to incorporate changes
            self.update_container("capitaine")
            
            return {
                "updated": True,
                "previous_commit": current[:8],
                "new_commit": new[:8],
                "changes": list(repo.iter_commits(f"{current}..{new}"))
            }
        
        return {"updated": False}
        
    def _verify_rom_base(self) -> bool:
        """Verify ROM base layer services are running"""
        checks = [
            self._check_cuda_available(),
            self._check_tensorrt_available(),
            self._check_a2a_server_running(),
            self._check_models_baked(),
        ]
        return all(checks)
        
    def _check_cuda_available(self) -> bool:
        """Check CUDA is available from ROM"""
        try:
            result = subprocess.run(
                ["nvidia-smi"],
                capture_output=True,
                text=True
            )
            return result.returncode == 0
        except:
            return False
            
    def _check_tensorrt_available(self) -> bool:
        """Check TensorRT is available"""
        try:
            import tensorrt as trt
            return True
        except ImportError:
            return False
            
    def _check_a2a_server_running(self) -> bool:
        """Check A2A server is responding"""
        import requests
        try:
            resp = requests.get("http://localhost:8080/health", timeout=5)
            return resp.status_code == 200
        except:
            return False
            
    def _check_models_baked(self) -> bool:
        """Check pre-baked models exist"""
        model_paths = [
            "/opt/models/llm/Qwen2.5-32B-Instruct-FP4.trt",
            "/opt/models/vision/fishinglog-ai-v2.trt",
            "/opt/models/voice/whisper/whisper-large-v3.trt",
        ]
        return all(Path(p).exists() for p in model_paths)
        
    def _register_with_rom(self):
        """Register application containers with ROM A2A server"""
        import requests
        
        containers = ["capitaine", "field-captain", "nexus-runtime"]
        
        for container in containers:
            try:
                resp = requests.post(
                    "http://localhost:8080/a2a/v1/register",
                    json={
                        "container_name": container,
                        "network_alias": f"{container}.agent.local"
                    }
                )
                print(f"Registered {container}: {resp.status_code}")
            except Exception as e:
                print(f"Failed to register {container}: {e}")
                
    def _wait_for_healthy(self, timeout: int = 120):
        """Wait for all containers to be healthy"""
        start = time.time()
        
        while time.time() - start < timeout:
            all_healthy = True
            
            for container in self.docker_client.containers.list():
                if container.status != "running":
                    all_healthy = False
                    continue
                    
                # Check health status
                health = container.attrs.get("State", {}).get("Health", {})
                if health.get("Status") != "healthy":
                    all_healthy = False
                    
            if all_healthy:
                return True
                
            time.sleep(2)
            
        raise TimeoutError("Containers did not become healthy in time")
        
    def _wait_for_container_healthy(self, name: str, timeout: int = 60):
        """Wait for specific container to be healthy"""
        start = time.time()
        
        while time.time() - start < timeout:
            container = self.docker_client.containers.get(name)
            
            if container.status == "running":
                health = container.attrs.get("State", {}).get("Health", {})
                if health.get("Status") == "healthy":
                    return True
                    
            time.sleep(2)
            
        raise TimeoutError(f"Container {name} did not become healthy")
```

---

## Part 3: Jetson Thor Specific Patterns

### 3.1 MIG Partition Management

```
+------------------------------------------------------------------+
|               MIG PARTITION STRATEGY FOR THOR                      |
+------------------------------------------------------------------+
|                                                                   |
|  Total GPU Memory: 128 GB                                         |
|  Partition Strategy: Balanced 4x32 GB                             |
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    GPU (128 GB Unified)                      │ |
|  │                                                              │ |
|  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │ |
|  │  │ Partition 0  │ │ Partition 1  │ │ Partition 2  │        │ |
|  │  │ 32 GB        │ │ 32 GB        │ │ 32 GB        │        │ |
|  │  │ ─────────── │ │ ─────────── │ │ ─────────── │        │ |
|  │  │ LLM          │ │ Vision       │ │ Voice        │        │ |
|  │  │ Inference    │ │ Pipeline     │ │ Processing   │        │ |
|  │  │              │ │              │ │              │        │ |
|  │  │ capitaine    │ │ field-       │ │ nexus-       │        │ |
|  │  │ container    │ │ captain      │ │ runtime      │        │ |
|  │  └──────────────┘ └──────────────┘ └──────────────┘        │ |
|  │                                                              │ |
|  │  ┌──────────────┐                                            │ |
|  │  │ Partition 3  │                                            │ |
|  │  │ 32 GB        │                                            │ |
|  │  │ ─────────── │                                            │ |
|  │  │ Training     │                                            │ |
|  │  │ Pipeline     │                                            │ |
|  │  │              │                                            │ |
|  │  │ LoRA fine-   │                                            │ |
|  │  │ tuning       │                                            │ |
|  │  └──────────────┘                                            │ |
|  │                                                              │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  Benefits:                                                        |
|  ✓ Complete isolation between agents                              |
|  ✓ No noisy neighbor problems                                     |
|  ✓ Independent scaling per service                                |
|  ✓ Fault containment                                              |
|                                                                   |
+------------------------------------------------------------------+
```

### 3.2 FP4 Quantization Integration

```python
# thor_fp4_integration.py
"""
Jetson Thor FP4 quantization integration
Native support for 4-bit inference and training
"""

import torch
from pathlib import Path
from typing import Dict, Any, Optional
import tensorrt as trt

class ThorFP4Manager:
    """
    Manage FP4 quantization on Jetson Thor
    Thor has native FP4 support via Blackwell Transformer Engine
    """
    
    def __init__(self):
        self.device = torch.device("cuda:0")
        self.fp4_supported = self._check_fp4_support()
        
    def _check_fp4_support(self) -> bool:
        """Check if device supports native FP4"""
        # Thor (Blackwell) has native FP4
        # Orin and earlier do not
        major, _ = torch.cuda.get_device_capability()
        return major >= 10  # Blackwell is SM 100+
        
    def quantize_model_to_fp4(
        self,
        model_path: str,
        output_path: str,
        calibration_data: Optional[str] = None
    ) -> Dict[str, Any]:
        """
        Quantize a model to FP4 for Thor
        Uses TensorRT for optimal inference
        """
        if not self.fp4_supported:
            raise RuntimeError("FP4 not supported on this device")
            
        # Build TensorRT engine with FP4
        logger = trt.Logger(trt.Logger.WARNING)
        builder = trt.Builder(logger)
        network = builder.create_network(
            1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
        )
        config = builder.create_builder_config()
        
        # Set FP4 precision
        config.set_flag(trt.BuilderFlag.FP4)
        config.set_flag(trt.BuilderFlag.INT4)  # For weights
        
        # Optimization profiles for dynamic shapes
        profile = builder.create_optimization_profile()
        profile.set_shape(
            "input",
            min=(1, 1, 4096),      # Min sequence length
            opt=(1, 512, 4096),     # Optimal sequence length
            max=(1, 4096, 4096)     # Max sequence length
        )
        config.add_optimization_profile(profile)
        
        # Build engine
        engine = builder.build_serialized_network(network, config)
        
        # Save engine
        Path(output_path).write_bytes(engine)
        
        return {
            "output_path": output_path,
            "precision": "FP4",
            "memory_reduction": "4x",
            "latency_improvement": "2-3x"
        }
        
    def load_fp4_model(self, model_path: str):
        """
        Load FP4-quantized model for inference
        Uses Thor's native FP4 Tensor Cores
        """
        # Load TensorRT engine
        logger = trt.Logger(trt.Logger.WARNING)
        runtime = trt.Runtime(logger)
        
        engine_data = Path(model_path).read_bytes()
        engine = runtime.deserialize_cuda_engine(engine_data)
        
        # Create execution context
        context = engine.create_execution_context()
        
        return {
            "engine": engine,
            "context": context,
            "device": self.device
        }
        
    def inference_fp4(
        self,
        model: Dict,
        input_tensor: torch.Tensor
    ) -> torch.Tensor:
        """
        Run inference on FP4 model
        Thor-optimized execution
        """
        engine = model["engine"]
        context = model["context"]
        
        # Allocate buffers
        bindings = []
        for i in range(engine.num_bindings):
            binding = engine[i]
            size = trt.volume(engine.get_binding_shape(i))
            
            if engine.binding_is_input(i):
                # Input buffer
                input_buffer = input_tensor.contiguous()
                bindings.append(input_buffer.data_ptr())
            else:
                # Output buffer
                output_buffer = torch.empty(
                    size, dtype=torch.float32, device=self.device
                )
                bindings.append(output_buffer.data_ptr())
                
        # Execute
        context.execute_async_v2(
            bindings,
            torch.cuda.current_stream().cuda_stream
        )
        
        return output_buffer
```

### 3.3 Power Management Profiles

```python
# power_management.py
"""
Jetson Thor power management
Critical for edge deployment scenarios
"""

import subprocess
from dataclasses import dataclass
from typing import Dict, Any, Optional
from enum import Enum
import time

class PowerMode(Enum):
    PERFORMANCE = "performance"   # 130W max
    BALANCED = "balanced"         # 80W typical
    POWER_SAVER = "power_saver"   # 40W min

@dataclass
class PowerProfile:
    mode: PowerMode
    power_budget_w: int
    cpu_governor: str
    gpu_freq_mhz: int
    cpu_freq_mhz: int
    expected_inference_latency_ms: int

POWER_PROFILES = {
    PowerMode.PERFORMANCE: PowerProfile(
        mode=PowerMode.PERFORMANCE,
        power_budget_w=130,
        cpu_governor="performance",
        gpu_freq_mhz=1600,  # Max GPU clock
        cpu_freq_mhz=2200,  # Max CPU clock
        expected_inference_latency_ms=150
    ),
    PowerMode.BALANCED: PowerProfile(
        mode=PowerMode.BALANCED,
        power_budget_w=80,
        cpu_governor="ondemand",
        gpu_freq_mhz=1200,
        cpu_freq_mhz=1800,
        expected_inference_latency_ms=250
    ),
    PowerMode.POWER_SAVER: PowerProfile(
        mode=PowerMode.POWER_SAVER,
        power_budget_w=40,
        cpu_governor="powersave",
        gpu_freq_mhz=800,
        cpu_freq_mhz=1200,
        expected_inference_latency_ms=500
    )
}

class ThorPowerManager:
    """
    Manage Jetson Thor power modes
    Adapts to workload and battery constraints
    """
    
    def __init__(self):
        self.current_mode = PowerMode.PERFORMANCE
        self.profile = POWER_PROFILES[self.current_mode]
        
    def set_mode(self, mode: PowerMode):
        """Set power mode on Jetson Thor"""
        profile = POWER_PROFILES[mode]
        
        # Set nvpmodel (power mode)
        nvpmodel_map = {
            PowerMode.PERFORMANCE: 0,
            PowerMode.BALANCED: 8,
            PowerMode.POWER_SAVER: 15
        }
        
        subprocess.run([
            "sudo", "nvpmodel", "-m", str(nvpmodel_map[mode])
        ], check=True)
        
        # Set CPU governor
        subprocess.run([
            "sudo", "cpupower", "frequency-set", "-g", profile.cpu_governor
        ], check=True)
        
        # Maximize clocks in performance mode
        if mode == PowerMode.PERFORMANCE:
            subprocess.run(["sudo", "jetson_clocks"], check=True)
            
        self.current_mode = mode
        self.profile = profile
        
    def adapt_to_workload(self, current_load: float, battery_percent: Optional[float] = None):
        """
        Adapt power mode based on workload and battery
        Implements intelligent power management
        """
        # Battery priority
        if battery_percent is not None and battery_percent < 20:
            self.set_mode(PowerMode.POWER_SAVER)
            return
            
        if battery_percent is not None and battery_percent < 50:
            self.set_mode(PowerMode.BALANCED)
            return
            
        # Workload-based adaptation
        if current_load > 0.8:
            # High load: need performance
            if self.current_mode != PowerMode.PERFORMANCE:
                self.set_mode(PowerMode.PERFORMANCE)
        elif current_load < 0.3:
            # Low load: save power
            if self.current_mode == PowerMode.PERFORMANCE:
                self.set_mode(PowerMode.BALANCED)
                
    def get_current_stats(self) -> Dict[str, Any]:
        """Get current power statistics"""
        import pynvml
        
        pynvml.nvmlInit()
        handle = pynvml.nvmlDeviceGetHandleByIndex(0)
        
        return {
            "mode": self.current_mode.value,
            "power_mw": pynvml.nvmlDeviceGetPowerUsage(handle),
            "temperature_c": pynvml.nvmlDeviceGetTemperature(
                handle, pynvml.NVML_TEMPERATURE_GPU
            ),
            "gpu_utilization": pynvml.nvmlDeviceGetUtilizationRates(handle).gpu,
            "memory_utilization": pynvml.nvmlDeviceGetUtilizationRates(handle).memory,
            "profile": {
                "power_budget_w": self.profile.power_budget_w,
                "expected_latency_ms": self.profile.expected_inference_latency_ms
            }
        }
```

---

## Part 4: Update Mechanisms and Versioning

### 4.1 ROM Update Strategy

```
+------------------------------------------------------------------+
|                    ROM UPDATE STRATEGY                             |
+------------------------------------------------------------------+
|                                                                   |
|  Update Types:                                                    |
|  ─────────────                                                    |
|                                                                   |
|  1. MAJOR (Full Reflash)                                          |
|     └── JetPack upgrade, CUDA major version                       |
|     └── Frequency: 12-18 months                                   |
|     └── Downtime: 30-60 minutes                                   |
|     └── Method: SD card / USB flash                               |
|                                                                   |
|  2. MINOR (OTA Update)                                            |
|     └── Isaac ROS updates, driver patches                         |
|     └── Frequency: 3-6 months                                     |
|     └── Downtime: 5-15 minutes                                    |
|     └── Method: A/B partition swap                                |
|                                                                   |
|  3. PATCH (Hot-fix)                                               |
|     └── Security patches, critical fixes                          |
|     └── Frequency: As needed                                      |
|     └── Downtime: < 5 minutes (container restart)                 |
|     └── Method: Container image update                            |
|                                                                   |
+------------------------------------------------------------------+
```

### 4.2 Application Layer Update Strategy

```yaml
# update-strategy.yaml
# Defines how application layer components are updated

application_updates:
  
  # Git-native agent updates
  capitaine:
    method: "git_pull"
    frequency: "continuous"
    trigger: "manual_or_scheduled"
    
    update_process:
      1. "git fetch origin"
      2. "git merge origin/main"
      3. "docker compose restart capitaine"
      
    rollback:
      method: "git revert"
      trigger: "health_check_failure"
      
  # Service updates
  field_captain:
    method: "docker_pull"
    frequency: "weekly"
    trigger: "manual_or_ci_cd"
    
    update_process:
      1. "docker compose pull field-captain"
      2. "docker compose up -d --no-deps field-captain"
      
    rollback:
      method: "image_tag_rollback"
      previous_versions_kept: 3
      
  # Model updates
  models:
    method: "hot_swap"
    frequency: "monthly"
    trigger: "validation_pass"
    
    update_process:
      1. "Download new model to /opt/models/new/"
      2. "Validate model on partition 3"
      3. "Symlink swap: /opt/models/llm -> /opt/models/new/llm"
      4. "Restart inference containers"
      
    rollback:
      method: "symlink_rollback"
      previous_model_kept: true

# Version tracking
versioning:
  rom_version_file: "/etc/field-captain/rom-version"
  app_version_file: "/app/repo/.version"
  model_version_file: "/opt/models/versions.json"
  
  format: "semver"
  example: "1.2.3-thor.20250115"
```

### 4.3 Version Compatibility Matrix

```
+------------------------------------------------------------------+
|                VERSION COMPATIBILITY MATRIX                        |
+------------------------------------------------------------------+
|                                                                   |
|  ROM Version  │ JetPack │ Isaac ROS │ capitaine │ field-captain  │
|  ────────────┼─────────┼───────────┼───────────┼─────────────── │
|  1.0.x        │ 7.0     │ 4.1       │ 1.x       │ 1.x            │
|  1.1.x        │ 7.0     │ 4.2       │ 1.x, 2.x  │ 1.x, 2.x       │
|  2.0.x        │ 7.1     │ 5.0       │ 2.x, 3.x  │ 2.x, 3.x       │
|                                                                   |
|  ─────────────────────────────────────────────────────────────────│
|                                                                   |
|  Model Version │ ROM Min │ LLM Container │ Vision Container       │
|  ─────────────┼─────────┼───────────────┼─────────────────────── │
|  Qwen2.5-32B   │ 1.0     │ 1.x+          │ N/A                    │
|  Qwen3-70B     │ 2.0     │ 3.x+          │ N/A                    │
|  fishinglog-v3 │ 1.1     │ N/A           │ 2.x+                   │
|                                                                   |
+------------------------------------------------------------------+
```

---

## Part 5: Deployment Checklist

### 5.1 Pre-Deployment Verification

```bash
#!/bin/bash
# pre-deployment-check.sh
# Verify ROM base layer before application deployment

set -e

echo "=== Pre-Deployment Verification ==="

# Check Jetson Thor
echo "[1/8] Checking Jetson Thor hardware..."
if ! command -v nvidia-smi &> /dev/null; then
    echo "ERROR: nvidia-smi not found"
    exit 1
fi
nvidia-smi -L | grep -q "Thor" || echo "WARNING: Not Thor GPU"

# Check CUDA
echo "[2/8] Checking CUDA..."
nvcc --version | grep -q "12.6" || echo "WARNING: CUDA version mismatch"

# Check TensorRT
echo "[3/8] Checking TensorRT..."
python3 -c "import tensorrt as trt; print(f'TensorRT {trt.__version__}')"

# Check MIG
echo "[4/8] Checking MIG configuration..."
nvidia-smi mig -cgi | grep -q "MIG" || echo "ERROR: MIG not enabled"

# Check Isaac ROS
echo "[5/8] Checking Isaac ROS..."
if [ -d "/opt/ros/humble" ]; then
    echo "ROS 2 Humble found"
else
    echo "ERROR: ROS 2 not found"
fi

# Check models
echo "[6/8] Checking pre-baked models..."
MODELS=(
    "/opt/models/llm/Qwen2.5-32B-Instruct-FP4.trt"
    "/opt/models/vision/fishinglog-ai-v2.trt"
    "/opt/models/voice/whisper/whisper-large-v3.trt"
)
for model in "${MODELS[@]}"; do
    if [ -f "$model" ]; then
        echo "  ✓ $model"
    else
        echo "  ✗ $model NOT FOUND"
    fi
done

# Check A2A server
echo "[7/8] Checking A2A server..."
if curl -s http://localhost:8080/health | grep -q "healthy"; then
    echo "  ✓ A2A server healthy"
else
    echo "  ✗ A2A server not responding"
fi

# Check Docker
echo "[8/8] Checking Docker..."
docker info | grep -q "nvidia" || echo "WARNING: NVIDIA runtime not configured"

echo "=== Verification Complete ==="
```

### 5.2 Deployment Procedure

```bash
#!/bin/bash
# deploy.sh
# Deploy application layer on ROM base

set -e

COMPOSE_FILE="docker-compose.yaml"
CONFIG_DIR="./config"

echo "=== Deploying Application Layer ==="

# Step 1: Pull latest images
echo "[1/6] Pulling container images..."
docker compose -f $COMPOSE_FILE pull

# Step 2: Verify configuration
echo "[2/6] Verifying configuration..."
python3 scripts/verify-config.py --config $CONFIG_DIR

# Step 3: Initialize agent repository
echo "[3/6] Initializing agent repository..."
if [ ! -d "./agent-repo/.git" ]; then
    git clone https://github.com/lucineer/capitaine-template ./agent-repo
    cd agent-repo
    git config user.name "Field Agent"
    git config user.email "agent@field.local"
    cd ..
fi

# Step 4: Start containers
echo "[4/6] Starting containers..."
docker compose -f $COMPOSE_FILE up -d

# Step 5: Wait for health
echo "[5/6] Waiting for containers to be healthy..."
python3 scripts/wait-healthy.py --timeout 120

# Step 6: Verify deployment
echo "[6/6] Verifying deployment..."
python3 scripts/verify-deployment.py

echo "=== Deployment Complete ==="
echo ""
echo "Services:"
echo "  - capitaine:      http://localhost:8080"
echo "  - field-captain:  http://localhost:8081"
echo "  - nexus-runtime:  http://localhost:8082"
```

---

## Summary

This iteration establishes the foundational architecture for the Jetson Robotics Ecosystem:

1. **ROM Base Layer**: Immutable, baked-in foundation providing stability, performance, and consistency
2. **Application Layer**: Containerized, evolvable services enabling customization and git-native agent behavior
3. **Jetson Thor Patterns**: MIG partitioning, FP4 quantization, power management
4. **Update Mechanisms**: Tiered update strategy from full reflash to git pull
5. **Versioning**: Compatibility matrix ensuring ecosystem coherence

### Key Benefits

| Aspect | ROM Base | Application Layer |
|--------|----------|-------------------|
| Stability | Maximum (baked) | High (containerized) |
| Flexibility | Low | Maximum (git-native) |
| Update Speed | Slow (reflash) | Fast (git pull) |
| Customization | Limited | Unlimited |
| Performance | Optimal (close-to-metal) | High (GPU passthrough) |

### Next Steps

- **Iteration 2**: Vessel Type Specifications (marine, aerial, industrial, home, medical)
- **Iteration 3**: Cocapn Software Vessel (meta-agent architecture)

---

*Document prepared for Jetson Robotics Ecosystem Grand Design*  
*Iteration 01 - ROM Emulation Architecture*  
*Date: January 2025*
