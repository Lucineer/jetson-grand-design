# Developer Guide: Jetson Robotics Grand Design

> **Iterative Development Walkthrough** — From Zero to Production-Ready Vessel

This guide walks you through the complete development process for the Jetson Robotics Grand Design ecosystem, following the same iterative methodology used to create this architecture.

## Table of Contents

1. [Development Philosophy](#development-philosophy)
2. [Prerequisites](#prerequisites)
3. [Environment Setup](#environment-setup)
4. [Iteration 1: ROM Emulation Architecture](#iteration-1-rom-emulation-architecture)
5. [Iteration 2: Vessel Type Specifications](#iteration-2-vessel-type-specifications)
6. [Iteration 3: Cocapn Software Vessel](#iteration-3-cocapn-software-vessel)
7. [Iteration 4: Training Data Pipeline](#iteration-4-training-data-pipeline)
8. [Iteration 5: A2A-R Protocol](#iteration-5-a2a-r-protocol)
9. [Iteration 6: Portfolio System](#iteration-6-portfolio-system)
10. [Iteration 7: Knowledge Indexes](#iteration-7-knowledge-indexes)
11. [Iteration 8: Failure Modes & Recovery](#iteration-8-failure-modes--recovery)
12. [Iteration 9: Human Workflow Guides](#iteration-9-human-workflow-guides)
13. [Iteration 10: Master Index & Documentation](#iteration-10-master-index--documentation)
14. [Testing & Validation](#testing--validation)
15. [Deployment Checklist](#deployment-checklist)

---

## Development Philosophy

### The Iterative Approach

This project follows a **10-iteration development methodology** where each iteration:

1. **Builds on previous work** — Each iteration assumes all prior iterations are complete
2. **Delivers working code** — Not just specifications, but executable implementations
3. **Includes tests** — Unit tests, integration tests, and validation scripts
4. **Documents decisions** — Architecture Decision Records (ADRs) capture the "why"
5. **Enables rollback** — Git-native means every iteration is a stable checkpoint

### Reverse-Actualization Methodology

We started from the **20-year future vision** and worked backward:

```
2044 Vision → What must exist by 2034? → What by 2027? → What NOW?
```

This ensures every iteration contributes to the long-term goal.

---

## Prerequisites

### Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| NVIDIA Jetson | AGX Orin (64GB) | Thor (128GB) |
| Storage | 256GB NVMe | 1TB NVMe |
| RAM | 32GB | 64GB+ |
| Network | Gigabit Ethernet | 10GbE + WiFi 6 |

### Software Requirements

```bash
# Operating System
Ubuntu 22.04 LTS (JetPack 6.x) or Ubuntu 24.04 LTS (JetPack 7.x)

# Core Dependencies
- Docker 24.x+
- NVIDIA Container Toolkit
- Python 3.10+
- Node.js 18+ / Bun 1.x
- Rust 1.75+ (for performance-critical components)

# Robotics Stack
- ROS 2 Humble (JetPack 6) or ROS 2 Jazzy (JetPack 7)
- Isaac ROS 4.0+
```

### Knowledge Prerequisites

- **Required**: Python, Docker, Git, basic robotics concepts
- **Recommended**: CUDA programming, ROS 2, transformer architectures
- **Helpful**: A2A Protocol, federated learning, edge AI deployment

---

## Environment Setup

### Step 1: Clone the Repository

```bash
# Clone the grand design repository
git clone https://github.com/your-org/jetson-robotics-grand-design.git
cd jetson-robotics-grand-design

# Initialize submodules (capitaine, field-captain, nexus-runtime)
git submodule update --init --recursive
```

### Step 2: Install Development Dependencies

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Install Node.js dependencies (for dashboard/web UI)
bun install

# Install Rust toolchain (optional, for high-performance components)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Step 3: Configure Environment

```bash
# Copy example environment file
cp .env.example .env

# Edit with your settings
nano .env
```

Required environment variables:

```bash
# Vessel Identity
VESSEL_ID=vessel-001
VESSEL_TYPE=marine  # marine, aerial, industrial, home, medical
VESSEL_DOMAIN=fishing  # domain-specific

# Jetson Configuration
JETSON_MODEL=thor  # thor, agx-orin, orin-nx
MIG_ENABLED=true
MIG_PARTITIONS=4

# A2A Protocol
A2A_PORT=8080
A2A_DISCOVERY=multicast  # multicast, dns-sd, manual

# Fleet Configuration
FLEET_JOIN=true
FLEET_COORDINATOR=coordinator.example.com
```

### Step 4: Verify Installation

```bash
# Run system check
python scripts/system_check.py

# Expected output:
# ✓ JetPack version: 7.0
# ✓ CUDA version: 12.x
# ✓ Docker: running
# ✓ MIG: configured (4 partitions)
# ✓ Network: connected
# ✓ All checks passed!
```

---

## Iteration 1: ROM Emulation Architecture

### Objective

Establish the stable ROM base layer and flexible application layer architecture.

### Files Created

```
implementation/
├── rom-base/
│   ├── Dockerfile.rom-base
│   ├── jetpack-thor.yaml
│   ├── rom-build.sh
│   └── rom-verify.sh
├── application-layer/
│   ├── Dockerfile.app
│   ├── app-manager.py
│   └── container-config/
│       ├── capitaine.yaml
│       ├── field-captain.yaml
│       └── nexus.yaml
└── tests/
    ├── test_rom_integrity.py
    └── test_app_isolation.py
```

### Implementation Steps

#### 1.1 Build ROM Base Image

```bash
cd implementation/rom-base

# Build the ROM image (takes 30-60 minutes first time)
./rom-build.sh --target thor --version 1.0.0

# Verify ROM integrity
./rom-verify.sh --image rom-thor-1.0.0.img

# Expected output:
# ✓ SHA-256 checksum verified
# ✓ JetPack components: all present
# ✓ Isaac ROS: installed
# ✓ CUDA 12.x: installed
# ✓ FP4 support: enabled
# ✓ ROM verification PASSED
```

#### 1.2 Configure MIG Partitions

```python
# implementation/rom-base/mig_config.py

from typing import List, Dict
import subprocess

class MIGConfigurator:
    """Configure MIG partitions on Jetson Thor"""
    
    PARTITION_PROFILES = {
        "balanced_4x32": [
            {"id": 0, "memory_gb": 32, "compute": "25%"},
            {"id": 1, "memory_gb": 32, "compute": "25%"},
            {"id": 2, "memory_gb": 32, "compute": "25%"},
            {"id": 3, "memory_gb": 32, "compute": "25%"},
        ],
        "inference_optimized": [
            {"id": 0, "memory_gb": 64, "compute": "50%"},  # Primary inference
            {"id": 1, "memory_gb": 32, "compute": "25%"},  # Secondary
            {"id": 2, "memory_gb": 32, "compute": "25%"},  # Training
        ],
        "training_focused": [
            {"id": 0, "memory_gb": 96, "compute": "75%"},  # Training
            {"id": 1, "memory_gb": 32, "compute": "25%"},  # Inference
        ],
    }
    
    def __init__(self, profile: str = "balanced_4x32"):
        self.profile = profile
        self.partitions = self.PARTITION_PROFILES[profile]
    
    def apply(self) -> bool:
        """Apply MIG configuration"""
        for partition in self.partitions:
            cmd = [
                "nvidia-smi", "mig", "-cgi", str(partition["id"]),
                "-C", partition["compute"],
                "-mem", str(partition["memory_gb"])
            ]
            result = subprocess.run(cmd, capture_output=True)
            if result.returncode != 0:
                print(f"Failed to configure partition {partition['id']}")
                return False
        return True
    
    def verify(self) -> Dict:
        """Verify MIG configuration"""
        result = subprocess.run(
            ["nvidia-smi", "mig", "-lgip"],
            capture_output=True, text=True
        )
        return {"status": "ok", "output": result.stdout}

# Usage
configurator = MIGConfigurator("balanced_4x32")
configurator.apply()
print(configurator.verify())
```

#### 1.3 Implement Application Layer Manager

```python
# implementation/application-layer/app_manager.py

import docker
import yaml
from pathlib import Path
from typing import List, Dict, Optional

class ApplicationLayerManager:
    """Manages containerized applications on top of ROM base"""
    
    def __init__(self, config_path: str = "container-config/"):
        self.client = docker.from_env()
        self.config_path = Path(config_path)
        self.containers: Dict[str, docker.models.containers.Container] = {}
    
    def load_config(self, app_name: str) -> Dict:
        """Load application configuration"""
        config_file = self.config_path / f"{app_name}.yaml"
        with open(config_file) as f:
            return yaml.safe_load(f)
    
    def start_application(self, app_name: str, mig_partition: int = 0) -> bool:
        """Start an application container with MIG partition assignment"""
        config = self.load_config(app_name)
        
        # Set MIG partition environment
        environment = config.get("environment", {})
        environment["NVIDIA_MIG_GPU_DEVICES"] = f"{mig_partition}"
        environment["CUDA_VISIBLE_DEVICES"] = f"MIG-{mig_partition}"
        
        try:
            container = self.client.containers.run(
                image=config["image"],
                name=f"cocapn-{app_name}",
                environment=environment,
                volumes=config.get("volumes", {}),
                network=config.get("network", "bridge"),
                detach=True,
                restart_policy={"Name": "unless-stopped"},
                device_requests=[
                    docker.types.DeviceRequest(
                        device_ids=[str(mig_partition)],
                        capabilities=[["gpu"]]
                    )
                ]
            )
            self.containers[app_name] = container
            return True
        except Exception as e:
            print(f"Failed to start {app_name}: {e}")
            return False
    
    def stop_application(self, app_name: str) -> bool:
        """Stop an application container"""
        if app_name in self.containers:
            self.containers[app_name].stop()
            self.containers[app_name].remove()
            del self.containers[app_name]
            return True
        return False
    
    def status(self) -> Dict:
        """Get status of all managed containers"""
        return {
            name: container.status
            for name, container in self.containers.items()
        }

# Usage
manager = ApplicationLayerManager()
manager.start_application("capitaine", mig_partition=0)
manager.start_application("field-captain", mig_partition=1)
manager.start_application("nexus", mig_partition=2)
print(manager.status())
```

### Tests for Iteration 1

```python
# implementation/tests/test_iteration1.py

import pytest
import subprocess
from pathlib import Path

class TestROMEmulation:
    """Tests for ROM Emulation Architecture (Iteration 1)"""
    
    def test_rom_image_exists(self):
        """Verify ROM image was built"""
        rom_path = Path("/opt/cocapn/rom/rom-thor-1.0.0.img")
        assert rom_path.exists(), "ROM image not found"
    
    def test_rom_checksum(self):
        """Verify ROM image integrity"""
        result = subprocess.run(
            ["sha256sum", "/opt/cocapn/rom/rom-thor-1.0.0.img"],
            capture_output=True, text=True
        )
        assert result.returncode == 0
        # Compare with stored checksum
        stored_checksum = Path("/opt/cocapn/rom/rom-thor-1.0.0.sha256").read_text().strip()
        actual_checksum = result.stdout.split()[0]
        assert actual_checksum == stored_checksum
    
    def test_mig_configuration(self):
        """Verify MIG partitions are configured"""
        result = subprocess.run(
            ["nvidia-smi", "mig", "-lgip"],
            capture_output=True, text=True
        )
        assert result.returncode == 0
        assert "MIG" in result.stdout
    
    def test_application_isolation(self):
        """Verify applications are isolated"""
        # Start two applications on different MIG partitions
        # Verify they cannot access each other's memory
        pass  # Implementation details
    
    def test_app_manager_operations(self):
        """Test ApplicationLayerManager operations"""
        from implementation.application_layer.app_manager import ApplicationLayerManager
        
        manager = ApplicationLayerManager()
        
        # Start application
        assert manager.start_application("test-app", mig_partition=3)
        
        # Check status
        status = manager.status()
        assert "test-app" in status
        assert status["test-app"] == "running"
        
        # Stop application
        assert manager.stop_application("test-app")
        
        # Verify stopped
        status = manager.status()
        assert "test-app" not in status
```

### Run Iteration 1 Tests

```bash
cd implementation
pytest tests/test_iteration1.py -v

# Expected output:
# test_rom_image_exists PASSED
# test_rom_checksum PASSED
# test_mig_configuration PASSED
# test_application_isolation PASSED
# test_app_manager_operations PASSED
```

---

## Iteration 2: Vessel Type Specifications

### Objective

Define and implement domain-specific vessel configurations for Marine, Aerial, Industrial, Home, and Medical applications.

### Files Created

```
implementation/
├── vessel-types/
│   ├── base_vessel.py
│   ├── marine/
│   │   ├── __init__.py
│   │   ├── fishing_vessel.py
│   │   ├── research_vessel.py
│   │   └── asv_vessel.py
│   ├── aerial/
│   │   ├── __init__.py
│   │   ├── drone_vessel.py
│   │   └── fixed_wing_vessel.py
│   ├── industrial/
│   │   ├── __init__.py
│   │   ├── warehouse_vessel.py
│   │   └── manipulator_vessel.py
│   ├── home/
│   │   ├── __init__.py
│   │   └── assistant_vessel.py
│   └── medical/
│       ├── __init__.py
│       └── surgical_vessel.py
└── tests/
    └── test_vessel_types.py
```

### Implementation Steps

#### 2.1 Base Vessel Class

```python
# implementation/vessel-types/base_vessel.py

from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import List, Dict, Any, Optional
from enum import Enum
import time

class VesselDomain(Enum):
    MARINE = "marine"
    AERIAL = "aerial"
    INDUSTRIAL = "industrial"
    HOME = "home"
    MEDICAL = "medical"

class VesselState(Enum):
    UNINITIALIZED = "uninitialized"
    INITIALIZING = "initializing"
    READY = "ready"
    ACTIVE = "active"
    PAUSED = "paused"
    ERROR = "error"
    SHUTTING_DOWN = "shutting_down"

@dataclass
class SensorConfig:
    """Sensor configuration for a vessel"""
    name: str
    type: str
    topic: str
    rate_hz: float
    enabled: bool = True

@dataclass
class AgentConfig:
    """Agent configuration for a vessel"""
    name: str
    type: str
    mig_partition: int
    priority: int = 0
    capabilities: List[str] = field(default_factory=list)

class BaseVessel(ABC):
    """Abstract base class for all vessel types"""
    
    def __init__(
        self,
        vessel_id: str,
        domain: VesselDomain,
        config: Dict[str, Any]
    ):
        self.vessel_id = vessel_id
        self.domain = domain
        self.config = config
        self.state = VesselState.UNINITIALIZED
        self.sensors: List[SensorConfig] = []
        self.agents: List[AgentConfig] = []
        self.heartbeat_interval = 1.0  # seconds
        self.last_heartbeat = 0.0
    
    @abstractmethod
    def initialize(self) -> bool:
        """Initialize vessel-specific hardware and agents"""
        pass
    
    @abstractmethod
    def get_capabilities(self) -> List[str]:
        """Return list of vessel capabilities"""
        pass
    
    @abstractmethod
    def emergency_stop(self) -> bool:
        """Execute emergency stop procedure"""
        pass
    
    def heartbeat(self) -> Dict:
        """Generate heartbeat message"""
        now = time.time()
        self.last_heartbeat = now
        return {
            "vessel_id": self.vessel_id,
            "domain": self.domain.value,
            "state": self.state.value,
            "timestamp": now,
            "capabilities": self.get_capabilities(),
            "sensors_active": sum(1 for s in self.sensors if s.enabled),
            "agents_active": len(self.agents),
        }
    
    def transition_state(self, new_state: VesselState) -> bool:
        """Transition to a new state"""
        valid_transitions = {
            VesselState.UNINITIALIZED: [VesselState.INITIALIZING],
            VesselState.INITIALIZING: [VesselState.READY, VesselState.ERROR],
            VesselState.READY: [VesselState.ACTIVE, VesselState.SHUTTING_DOWN],
            VesselState.ACTIVE: [VesselState.PAUSED, VesselState.ERROR, VesselState.SHUTTING_DOWN],
            VesselState.PAUSED: [VesselState.ACTIVE, VesselState.SHUTTING_DOWN],
            VesselState.ERROR: [VesselState.READY, VesselState.SHUTTING_DOWN],
            VesselState.SHUTTING_DOWN: [VesselState.UNINITIALIZED],
        }
        
        if new_state in valid_transitions.get(self.state, []):
            old_state = self.state
            self.state = new_state
            print(f"Vessel {self.vessel_id}: {old_state.value} → {new_state.value}")
            return True
        return False
```

#### 2.2 Marine Vessel Implementation

```python
# implementation/vessel-types/marine/fishing_vessel.py

from typing import List, Dict, Any
from ..base_vessel import BaseVessel, VesselDomain, VesselState, SensorConfig, AgentConfig

class FishingVessel(BaseVessel):
    """Fishing vessel with navigation, species detection, and safety agents"""
    
    def __init__(self, vessel_id: str, config: Dict[str, Any] = None):
        super().__init__(vessel_id, VesselDomain.MARINE, config or {})
        self._configure_sensors()
        self._configure_agents()
    
    def _configure_sensors(self):
        """Configure fishing vessel sensors"""
        self.sensors = [
            SensorConfig("gps", "gps", "/sensors/gps", 10.0),
            SensorConfig("compass", "magnetometer", "/sensors/compass", 20.0),
            SensorConfig("depth", "sonar", "/sensors/depth", 5.0),
            SensorConfig("fish_finder", "sonar_imaging", "/sensors/fish_finder", 2.0),
            SensorConfig("camera_bow", "camera", "/sensors/camera/bow", 30.0),
            SensorConfig("camera_stern", "camera", "/sensors/camera/stern", 30.0),
            SensorConfig("weather", "weather_station", "/sensors/weather", 1.0),
            SensorConfig("ais", "ais_receiver", "/sensors/ais", 0.5),
        ]
    
    def _configure_agents(self):
        """Configure fishing vessel agents"""
        self.agents = [
            AgentConfig(
                "navigation",
                "NavigationAgent",
                mig_partition=0,
                priority=10,
                capabilities=["path_planning", "collision_avoidance", "waypoint_following"]
            ),
            AgentConfig(
                "species",
                "SpeciesAgent",
                mig_partition=1,
                priority=5,
                capabilities=["fish_detection", "species_classification", "catch_optimization"]
            ),
            AgentConfig(
                "safety",
                "SafetyAgent",
                mig_partition=2,
                priority=15,
                capabilities=["weather_monitoring", "proximity_alerts", "emergency_protocols"]
            ),
        ]
    
    def initialize(self) -> bool:
        """Initialize fishing vessel systems"""
        self.transition_state(VesselState.INITIALIZING)
        
        try:
            # Initialize sensors
            for sensor in self.sensors:
                print(f"Initializing {sensor.name}...")
                # Hardware-specific initialization would go here
            
            # Initialize agents
            for agent in self.agents:
                print(f"Starting {agent.name} agent on MIG partition {agent.mig_partition}...")
                # Agent initialization would go here
            
            self.transition_state(VesselState.READY)
            return True
        except Exception as e:
            print(f"Initialization failed: {e}")
            self.transition_state(VesselState.ERROR)
            return False
    
    def get_capabilities(self) -> List[str]:
        """Return fishing vessel capabilities"""
        capabilities = []
        for agent in self.agents:
            capabilities.extend(agent.capabilities)
        return capabilities
    
    def emergency_stop(self) -> bool:
        """Execute emergency stop for fishing vessel"""
        print(f"EMERGENCY STOP on {self.vessel_id}")
        
        # Stop all propulsion
        # Surface to safe depth if submerged
        # Activate safety beacons
        # Alert nearby vessels via AIS
        
        self.transition_state(VesselState.ERROR)
        return True
```

### Tests for Iteration 2

```python
# implementation/tests/test_iteration2.py

import pytest
from vessel_types.base_vessel import VesselDomain, VesselState
from vessel_types.marine.fishing_vessel import FishingVessel

class TestVesselTypes:
    """Tests for Vessel Type Specifications (Iteration 2)"""
    
    def test_fishing_vessel_creation(self):
        """Test fishing vessel instantiation"""
        vessel = FishingVessel("test-fishing-001")
        assert vessel.vessel_id == "test-fishing-001"
        assert vessel.domain == VesselDomain.MARINE
        assert vessel.state == VesselState.UNINITIALIZED
    
    def test_fishing_vessel_sensors(self):
        """Test fishing vessel sensor configuration"""
        vessel = FishingVessel("test-fishing-001")
        assert len(vessel.sensors) == 8
        sensor_names = [s.name for s in vessel.sensors]
        assert "gps" in sensor_names
        assert "fish_finder" in sensor_names
    
    def test_fishing_vessel_agents(self):
        """Test fishing vessel agent configuration"""
        vessel = FishingVessel("test-fishing-001")
        assert len(vessel.agents) == 3
        agent_names = [a.name for a in vessel.agents]
        assert "navigation" in agent_names
        assert "species" in agent_names
        assert "safety" in agent_names
    
    def test_vessel_initialization(self):
        """Test vessel initialization sequence"""
        vessel = FishingVessel("test-fishing-001")
        assert vessel.initialize()
        assert vessel.state == VesselState.READY
    
    def test_vessel_heartbeat(self):
        """Test vessel heartbeat generation"""
        vessel = FishingVessel("test-fishing-001")
        vessel.initialize()
        heartbeat = vessel.heartbeat()
        
        assert heartbeat["vessel_id"] == "test-fishing-001"
        assert heartbeat["domain"] == "marine"
        assert heartbeat["state"] == "ready"
        assert len(heartbeat["capabilities"]) > 0
    
    def test_emergency_stop(self):
        """Test emergency stop procedure"""
        vessel = FishingVessel("test-fishing-001")
        vessel.initialize()
        vessel.transition_state(VesselState.ACTIVE)
        
        assert vessel.emergency_stop()
        assert vessel.state == VesselState.ERROR
```

---

## Iteration 3: Cocapn Software Vessel

### Objective

Implement the meta-agent orchestration layer that coordinates all domain agents on a vessel.

### Key Implementation

```python
# implementation/cocapn/cocapn_agent.py

from typing import Dict, List, Any, Optional
from dataclasses import dataclass
import asyncio
import json

@dataclass
class Intent:
    """User intent representation"""
    raw_input: str
    intent_type: str
    confidence: float
    entities: Dict[str, Any]
    context: Dict[str, Any]

@dataclass
class Task:
    """Decomposed task representation"""
    task_id: str
    task_type: str
    assigned_agent: str
    parameters: Dict[str, Any]
    priority: int
    status: str = "pending"

class CocapnAgent:
    """Meta-agent orchestrating all vessel agents"""
    
    def __init__(self, vessel_id: str):
        self.vessel_id = vessel_id
        self.agents: Dict[str, Any] = {}  # Agent instances
        self.capabilities_index: Dict[str, str] = {}  # capability → agent
        self.active_tasks: Dict[str, Task] = {}
        self.intent_classifier = None  # Will be loaded
    
    def register_agent(self, agent_name: str, agent_instance: Any, capabilities: List[str]):
        """Register an agent with its capabilities"""
        self.agents[agent_name] = agent_instance
        for capability in capabilities:
            self.capabilities_index[capability] = agent_name
        print(f"Registered {agent_name} with capabilities: {capabilities}")
    
    def classify_intent(self, user_input: str, context: Dict = None) -> Intent:
        """Classify user intent from natural language input"""
        # This would use a local LLM for intent classification
        # Simplified version for illustration
        
        intent_patterns = {
            "navigate": ["go to", "head to", "navigate", "move to", "set course"],
            "search": ["find", "search for", "look for", "detect"],
            "safety": ["emergency", "danger", "stop", "alert", "help"],
            "status": ["status", "how are", "report", "condition"],
            "configure": ["set", "configure", "change", "adjust"],
        }
        
        user_input_lower = user_input.lower()
        detected_intent = "unknown"
        confidence = 0.0
        
        for intent_type, patterns in intent_patterns.items():
            for pattern in patterns:
                if pattern in user_input_lower:
                    detected_intent = intent_type
                    confidence = 0.8  # Simplified confidence
                    break
        
        return Intent(
            raw_input=user_input,
            intent_type=detected_intent,
            confidence=confidence,
            entities={},  # Entity extraction would happen here
            context=context or {}
        )
    
    def decompose_task(self, intent: Intent) -> List[Task]:
        """Decompose an intent into executable tasks"""
        tasks = []
        
        if intent.intent_type == "navigate":
            tasks.append(Task(
                task_id=f"nav-{len(self.active_tasks)}",
                task_type="navigation",
                assigned_agent="navigation",
                parameters={"destination": intent.entities.get("destination")},
                priority=10
            ))
        elif intent.intent_type == "search":
            tasks.append(Task(
                task_id=f"search-{len(self.active_tasks)}",
                task_type="perception",
                assigned_agent="species",
                parameters={"target": intent.entities.get("target")},
                priority=5
            ))
        elif intent.intent_type == "safety":
            tasks.append(Task(
                task_id=f"safety-{len(self.active_tasks)}",
                task_type="emergency",
                assigned_agent="safety",
                parameters={},
                priority=15
            ))
        
        return tasks
    
    async def execute_task(self, task: Task) -> Dict:
        """Execute a task using the assigned agent"""
        if task.assigned_agent not in self.agents:
            return {"status": "error", "message": f"Agent {task.assigned_agent} not found"}
        
        agent = self.agents[task.assigned_agent]
        self.active_tasks[task.task_id] = task
        task.status = "executing"
        
        try:
            result = await agent.execute(task.parameters)
            task.status = "completed"
            return {"status": "success", "result": result}
        except Exception as e:
            task.status = "failed"
            return {"status": "error", "message": str(e)}
    
    async def process_input(self, user_input: str, context: Dict = None) -> Dict:
        """Process user input through the complete pipeline"""
        # 1. Classify intent
        intent = self.classify_intent(user_input, context)
        
        # 2. Decompose into tasks
        tasks = self.decompose_task(intent)
        
        # 3. Execute tasks
        results = []
        for task in tasks:
            result = await self.execute_task(task)
            results.append(result)
        
        return {
            "intent": intent.intent_type,
            "tasks_executed": len(tasks),
            "results": results
        }
```

---

## Iteration 4: Training Data Pipeline

### Objective

Implement the experience capture, curation, and learning pipeline.

### Key Implementation

```python
# implementation/training/experience_capture.py

from dataclasses import dataclass, asdict
from typing import Dict, List, Any, Optional
from datetime import datetime
import json
import hashlib

@dataclass
class ExperiencePrimitive:
    """Fundamental unit of learning experience"""
    experience_id: str
    vessel_id: str
    timestamp: str
    context: Dict[str, Any]  # State when action was taken
    observations: Dict[str, Any]  # What was observed
    actions: List[Dict[str, Any]]  # What actions were taken
    outcomes: Dict[str, Any]  # What resulted
    learning: Dict[str, Any]  # What was learned
    quality_score: float = 0.0
    tier: int = 0  # 1=high, 2=medium, 3=low

class ExperienceCapture:
    """Captures and stores experiences from vessel operations"""
    
    def __init__(self, vessel_id: str, storage_path: str = "/data/experiences"):
        self.vessel_id = vessel_id
        self.storage_path = storage_path
        self.experience_buffer: List[ExperiencePrimitive] = []
        self.capture_enabled = True
    
    def capture(
        self,
        context: Dict,
        observations: Dict,
        actions: List[Dict],
        outcomes: Dict,
        learning: Dict
    ) -> ExperiencePrimitive:
        """Capture an experience during operation"""
        
        timestamp = datetime.utcnow().isoformat()
        experience_id = self._generate_id(timestamp, context, actions)
        
        experience = ExperiencePrimitive(
            experience_id=experience_id,
            vessel_id=self.vessel_id,
            timestamp=timestamp,
            context=context,
            observations=observations,
            actions=actions,
            outcomes=outcomes,
            learning=learning
        )
        
        if self.capture_enabled:
            self.experience_buffer.append(experience)
        
        return experience
    
    def _generate_id(self, timestamp: str, context: Dict, actions: List) -> str:
        """Generate unique experience ID"""
        content = f"{timestamp}{json.dumps(context, sort_keys=True)}{json.dumps(actions, sort_keys=True)}"
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def flush_to_storage(self) -> int:
        """Write buffered experiences to persistent storage"""
        if not self.experience_buffer:
            return 0
        
        import os
        os.makedirs(self.storage_path, exist_ok=True)
        
        count = 0
        for exp in self.experience_buffer:
            filename = f"{self.storage_path}/{exp.experience_id}.json"
            with open(filename, 'w') as f:
                json.dump(asdict(exp), f, indent=2)
            count += 1
        
        self.experience_buffer.clear()
        return count
```

---

## Iteration 5: A2A-R Protocol

### Objective

Implement robotics-extended A2A protocol for real-time communication.

### Key Implementation

```python
# implementation/a2a_r/protocol.py

from dataclasses import dataclass
from typing import Dict, Any, Optional, List
from enum import Enum
import json
import time

class QoSLevel(Enum):
    BEST_EFFORT = "best_effort"  # No guarantees
    RELIABLE = "reliable"  # Guaranteed delivery
    REAL_TIME = "real_time"  # Guaranteed latency

@dataclass
class A2ARMessage:
    """A2A-R message structure"""
    message_id: str
    sender_id: str
    receiver_id: str  # or "broadcast"
    message_type: str
    qos: QoSLevel
    timestamp: float
    payload: Dict[str, Any]
    latency_budget_ms: Optional[float] = None
    
    def to_json(self) -> str:
        return json.dumps({
            "message_id": self.message_id,
            "sender_id": self.sender_id,
            "receiver_id": self.receiver_id,
            "message_type": self.message_type,
            "qos": self.qos.value,
            "timestamp": self.timestamp,
            "payload": self.payload,
            "latency_budget_ms": self.latency_budget_ms
        })

class A2ARProtocol:
    """A2A-R Protocol implementation"""
    
    LATENCY_TARGETS = {
        "safety_critical": 10,  # ms
        "real_time_control": 50,  # ms
        "coordination": 100,  # ms
        "telemetry": 500,  # ms
        "discovery": 5000,  # ms
    }
    
    def __init__(self, agent_id: str):
        self.agent_id = agent_id
        self.message_handlers: Dict[str, callable] = {}
        self.pending_acks: Dict[str, float] = {}
    
    def register_handler(self, message_type: str, handler: callable):
        """Register a handler for a message type"""
        self.message_handlers[message_type] = handler
    
    def create_message(
        self,
        receiver_id: str,
        message_type: str,
        payload: Dict,
        qos: QoSLevel = QoSLevel.BEST_EFFORT
    ) -> A2ARMessage:
        """Create a new A2A-R message"""
        import uuid
        
        latency_budget = self.LATENCY_TARGETS.get(message_type, 1000)
        
        return A2ARMessage(
            message_id=str(uuid.uuid4()),
            sender_id=self.agent_id,
            receiver_id=receiver_id,
            message_type=message_type,
            qos=qos,
            timestamp=time.time(),
            payload=payload,
            latency_budget_ms=latency_budget
        )
    
    async def send(self, message: A2ARMessage) -> bool:
        """Send a message"""
        # Implementation would use WebSocket, MQTT, or similar
        if message.qos in [QoSLevel.RELIABLE, QoSLevel.REAL_TIME]:
            self.pending_acks[message.message_id] = time.time()
        
        # Actual transmission logic here
        return True
    
    async def receive(self, raw_message: str) -> Optional[Dict]:
        """Process received message"""
        data = json.loads(raw_message)
        message_type = data.get("message_type")
        
        if message_type in self.message_handlers:
            handler = self.message_handlers[message_type]
            return await handler(data)
        
        return None
```

---

## Iteration 6: Portfolio System

### Objective

Implement the innovation portfolio capture and verification system.

### Key Implementation

```python
# implementation/portfolio/portfolio_system.py

from dataclasses import dataclass
from typing import Dict, List, Any, Optional
from datetime import datetime
import subprocess
import hashlib

@dataclass
class Innovation:
    """Innovation portfolio item"""
    innovation_id: str
    contributor_id: str
    commit_hash: str
    innovation_type: str  # bug_fix, optimization, feature, pattern, integration
    title: str
    description: str
    affected_files: List[str]
    verification_status: str
    adoption_count: int
    derivative_count: int
    value_score: float

class PortfolioSystem:
    """Git-native innovation portfolio system"""
    
    def __init__(self, repo_path: str = "."):
        self.repo_path = repo_path
    
    def capture_innovation(
        self,
        innovation_type: str,
        title: str,
        description: str
    ) -> Innovation:
        """Capture an innovation from current git state"""
        
        # Get current commit info
        commit_hash = self._get_current_commit()
        contributor = self._get_contributor()
        affected_files = self._get_affected_files()
        
        innovation = Innovation(
            innovation_id=self._generate_innovation_id(commit_hash),
            contributor_id=contributor,
            commit_hash=commit_hash,
            innovation_type=innovation_type,
            title=title,
            description=description,
            affected_files=affected_files,
            verification_status="pending",
            adoption_count=0,
            derivative_count=0,
            value_score=0.0
        )
        
        return innovation
    
    def verify_innovation(self, innovation: Innovation) -> bool:
        """Verify innovation using git-native verification"""
        
        # Layer 1: Commit Integrity
        if not self._verify_commit_integrity(innovation.commit_hash):
            return False
        
        # Layer 2: GPG Signature
        if not self._verify_gpg_signature(innovation.commit_hash):
            return False
        
        # Layer 3: Code Review
        if not self._verify_code_review(innovation.commit_hash):
            return False
        
        innovation.verification_status = "verified"
        return True
    
    def calculate_value(self, innovation: Innovation) -> float:
        """Calculate portfolio value of an innovation"""
        
        # Value = Base × Adoption × Derivatives × Time
        base_value = self._get_base_value(innovation.innovation_type)
        adoption_multiplier = 1 + (innovation.adoption_count * 0.1)
        derivative_multiplier = 1 + (innovation.derivative_count * 0.2)
        
        innovation.value_score = base_value * adoption_multiplier * derivative_multiplier
        return innovation.value_score
    
    def _get_current_commit(self) -> str:
        result = subprocess.run(
            ["git", "-C", self.repo_path, "rev-parse", "HEAD"],
            capture_output=True, text=True
        )
        return result.stdout.strip()
    
    def _get_contributor(self) -> str:
        result = subprocess.run(
            ["git", "-C", self.repo_path, "config", "user.email"],
            capture_output=True, text=True
        )
        return result.stdout.strip()
    
    def _get_affected_files(self) -> List[str]:
        result = subprocess.run(
            ["git", "-C", self.repo_path, "diff", "--name-only", "HEAD~1"],
            capture_output=True, text=True
        )
        return result.stdout.strip().split('\n') if result.stdout.strip() else []
    
    def _generate_innovation_id(self, commit_hash: str) -> str:
        return f"INN-{commit_hash[:8]}"
    
    def _verify_commit_integrity(self, commit_hash: str) -> bool:
        result = subprocess.run(
            ["git", "-C", self.repo_path, "verify-commit", commit_hash],
            capture_output=True
        )
        return result.returncode == 0
    
    def _verify_gpg_signature(self, commit_hash: str) -> bool:
        result = subprocess.run(
            ["git", "-C", self.repo_path, "log", "--format=%G?", "-1", commit_hash],
            capture_output=True, text=True
        )
        return result.stdout.strip() in ['G', 'U']  # Good or Unknown (no key)
    
    def _verify_code_review(self, commit_hash: str) -> bool:
        # Check for review approval in commit message or PR
        return True  # Simplified
    
    def _get_base_value(self, innovation_type: str) -> float:
        base_values = {
            "bug_fix": 10.0,
            "optimization": 15.0,
            "feature": 20.0,
            "pattern": 25.0,
            "integration": 30.0,
        }
        return base_values.get(innovation_type, 10.0)
```

---

## Iteration 7: Knowledge Indexes

### Objective

Implement semantic search and capability discovery.

### Key Implementation

```python
# implementation/knowledge/capability_index.py

from dataclasses import dataclass
from typing import Dict, List, Any, Optional
import json

@dataclass
class Capability:
    """Capability definition"""
    capability_id: str
    name: str
    description: str
    domain: str
    prerequisites: List[str]
    parameters: Dict[str, Any]
    performance_characteristics: Dict[str, float]

class CapabilityIndex:
    """Index for discovering and matching capabilities"""
    
    def __init__(self):
        self.capabilities: Dict[str, Capability] = {}
        self.domain_index: Dict[str, List[str]] = {}
        self.keyword_index: Dict[str, List[str]] = {}
    
    def register(self, capability: Capability):
        """Register a capability in the index"""
        self.capabilities[capability.capability_id] = capability
        
        # Index by domain
        if capability.domain not in self.domain_index:
            self.domain_index[capability.domain] = []
        self.domain_index[capability.domain].append(capability.capability_id)
        
        # Index by keywords
        keywords = self._extract_keywords(capability)
        for keyword in keywords:
            if keyword not in self.keyword_index:
                self.keyword_index[keyword] = []
            self.keyword_index[keyword].append(capability.capability_id)
    
    def search(self, query: str, domain: str = None) -> List[Capability]:
        """Search for capabilities matching a query"""
        results = []
        
        # Extract keywords from query
        query_keywords = set(query.lower().split())
        
        # Find matching capability IDs
        matching_ids = set()
        for keyword in query_keywords:
            if keyword in self.keyword_index:
                matching_ids.update(self.keyword_index[keyword])
        
        # Filter by domain if specified
        if domain and domain in self.domain_index:
            domain_ids = set(self.domain_index[domain])
            matching_ids = matching_ids.intersection(domain_ids)
        
        # Retrieve capabilities
        for cap_id in matching_ids:
            results.append(self.capabilities[cap_id])
        
        return results
    
    def match_intent(self, intent_type: str, context: Dict) -> Optional[Capability]:
        """Find best capability for an intent"""
        candidates = self.search(intent_type, context.get("domain"))
        
        if not candidates:
            return None
        
        # Rank by performance characteristics
        # Simplified: return first match
        return candidates[0]
    
    def _extract_keywords(self, capability: Capability) -> List[str]:
        """Extract searchable keywords from a capability"""
        keywords = []
        keywords.extend(capability.name.lower().split())
        keywords.extend(capability.description.lower().split())
        keywords.append(capability.domain.lower())
        return keywords
```

---

## Iteration 8: Failure Modes & Recovery

### Objective

Implement failure detection, isolation, and recovery mechanisms.

### Key Implementation

```python
# implementation/resilience/failure_manager.py

from dataclasses import dataclass
from typing import Dict, List, Any, Optional, Callable
from enum import Enum
import asyncio
import time

class Severity(Enum):
    SEV_0 = 0  # Catastrophic - vessel loss risk
    SEV_1 = 1  # Critical - safety compromised
    SEV_2 = 2  # Major - functionality degraded
    SEV_3 = 3  # Minor - workaround available
    SEV_4 = 4  # Cosmetic - no operational impact

@dataclass
class Failure:
    """Failure event"""
    failure_id: str
    severity: Severity
    category: str  # hardware, software, network, human, ecosystem
    component: str
    message: str
    timestamp: float
    detected_by: str
    recovery_actions: List[str]

class FailureManager:
    """Manages failure detection, isolation, and recovery"""
    
    def __init__(self, vessel_id: str):
        self.vessel_id = vessel_id
        self.active_failures: Dict[str, Failure] = {}
        self.recovery_handlers: Dict[str, Callable] = {}
        self.isolation_level = 0  # 0=normal, 1=component, 2=agent, 3=vessel, 4=fleet
    
    def register_handler(self, failure_category: str, handler: Callable):
        """Register a recovery handler for a failure category"""
        self.recovery_handlers[failure_category] = handler
    
    def detect(self, category: str, component: str, message: str) -> Optional[Failure]:
        """Detect and record a failure"""
        import uuid
        
        severity = self._assess_severity(category, component, message)
        
        failure = Failure(
            failure_id=str(uuid.uuid4()),
            severity=severity,
            category=category,
            component=component,
            message=message,
            timestamp=time.time(),
            detected_by="system",
            recovery_actions=self._get_recovery_actions(category, severity)
        )
        
        self.active_failures[failure.failure_id] = failure
        self._update_isolation_level(severity)
        
        return failure
    
    def isolate(self, failure: Failure) -> bool:
        """Isolate the failing component"""
        if failure.severity.value <= 1:
            # Critical isolation - stop affected agent
            return self._isolate_agent(failure.component)
        elif failure.severity.value <= 3:
            # Partial isolation - disable feature
            return self._isolate_component(failure.component)
        return True
    
    async def recover(self, failure: Failure) -> bool:
        """Attempt to recover from a failure"""
        if failure.category in self.recovery_handlers:
            handler = self.recovery_handlers[failure.category]
            try:
                result = await handler(failure)
                if result:
                    del self.active_failures[failure.failure_id]
                return result
            except Exception as e:
                print(f"Recovery failed: {e}")
                return False
        return False
    
    def _assess_severity(self, category: str, component: str, message: str) -> Severity:
        """Assess failure severity"""
        # Catastrophic conditions
        if "fire" in message.lower() or "flood" in message.lower():
            return Severity.SEV_0
        
        # Critical conditions
        if category == "hardware" and any(x in component for x in ["gps", "compass", "motor"]):
            return Severity.SEV_1
        
        # Major conditions
        if category == "network" or "timeout" in message.lower():
            return Severity.SEV_2
        
        # Default to minor
        return Severity.SEV_3
    
    def _get_recovery_actions(self, category: str, severity: Severity) -> List[str]:
        """Get recommended recovery actions"""
        actions = {
            "hardware": ["restart_component", "switch_to_backup", "notify_technician"],
            "software": ["restart_service", "rollback_version", "clear_cache"],
            "network": ["reconnect", "switch_to_offline_mode", "queue_operations"],
            "human": ["alert_operator", "provide_guidance", "wait_for_input"],
            "ecosystem": ["isolate_vessel", "notify_coordinator", "enable_local_mode"],
        }
        return actions.get(category, ["investigate"])
    
    def _update_isolation_level(self, severity: Severity):
        """Update system isolation level based on failure severity"""
        if severity == Severity.SEV_0:
            self.isolation_level = 4  # Fleet-level isolation
        elif severity == Severity.SEV_1:
            self.isolation_level = 3  # Vessel-level isolation
        elif severity == Severity.SEV_2:
            self.isolation_level = 2  # Agent-level isolation
        elif severity == Severity.SEV_3:
            self.isolation_level = 1  # Component-level isolation
    
    def _isolate_agent(self, component: str) -> bool:
        """Isolate an agent"""
        # Implementation would stop the agent container
        return True
    
    def _isolate_component(self, component: str) -> bool:
        """Isolate a component"""
        # Implementation would disable the component
        return True
```

---

## Iteration 9: Human Workflow Guides

### Objective

Implement workflow support for installers, educators, and technicians.

### Key Implementation

```python
# implementation/workflow/workflow_engine.py

from dataclasses import dataclass
from typing import Dict, List, Any, Optional, Callable
from enum import Enum
import time

class WorkflowRole(Enum):
    INSTALLER = "installer"
    EDUCATOR = "educator"
    TECHNICIAN = "technician"
    OPERATOR = "operator"

class WorkflowStatus(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    BLOCKED = "blocked"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass
class WorkflowStep:
    """Single step in a workflow"""
    step_id: str
    title: str
    description: str
    verification: Optional[Callable]
    timeout_seconds: int
    dependencies: List[str]
    status: WorkflowStatus = WorkflowStatus.PENDING

@dataclass
class Workflow:
    """Complete workflow definition"""
    workflow_id: str
    name: str
    role: WorkflowRole
    description: str
    steps: List[WorkflowStep]
    current_step: int = 0
    started_at: Optional[float] = None
    completed_at: Optional[float] = None

class WorkflowEngine:
    """Engine for executing guided workflows"""
    
    def __init__(self):
        self.active_workflows: Dict[str, Workflow] = {}
        self.workflow_templates: Dict[str, Workflow] = {}
    
    def register_template(self, workflow: Workflow):
        """Register a workflow template"""
        self.workflow_templates[workflow.name] = workflow
    
    def start_workflow(self, name: str, context: Dict = None) -> Optional[Workflow]:
        """Start a workflow from a template"""
        if name not in self.workflow_templates:
            return None
        
        import copy
        workflow = copy.deepcopy(self.workflow_templates[name])
        workflow.workflow_id = f"{name}-{time.time()}"
        workflow.started_at = time.time()
        
        self.active_workflows[workflow.workflow_id] = workflow
        return workflow
    
    def get_current_step(self, workflow_id: str) -> Optional[WorkflowStep]:
        """Get the current step of a workflow"""
        if workflow_id not in self.active_workflows:
            return None
        
        workflow = self.active_workflows[workflow_id]
        if workflow.current_step >= len(workflow.steps):
            return None
        
        return workflow.steps[workflow.current_step]
    
    def complete_step(self, workflow_id: str, step_id: str, result: Dict = None) -> bool:
        """Complete a workflow step"""
        if workflow_id not in self.active_workflows:
            return False
        
        workflow = self.active_workflows[workflow_id]
        current_step = self.get_current_step(workflow_id)
        
        if not current_step or current_step.step_id != step_id:
            return False
        
        # Run verification if present
        if current_step.verification:
            try:
                if not current_step.verification(result):
                    current_step.status = WorkflowStatus.FAILED
                    return False
            except Exception as e:
                current_step.status = WorkflowStatus.FAILED
                return False
        
        current_step.status = WorkflowStatus.COMPLETED
        workflow.current_step += 1
        
        # Check if workflow is complete
        if workflow.current_step >= len(workflow.steps):
            workflow.completed_at = time.time()
        
        return True
    
    def get_progress(self, workflow_id: str) -> Dict:
        """Get workflow progress"""
        if workflow_id not in self.active_workflows:
            return {}
        
        workflow = self.active_workflows[workflow_id]
        completed = sum(1 for s in workflow.steps if s.status == WorkflowStatus.COMPLETED)
        
        return {
            "workflow_id": workflow_id,
            "name": workflow.name,
            "role": workflow.role.value,
            "total_steps": len(workflow.steps),
            "completed_steps": completed,
            "current_step": workflow.current_step,
            "progress_percent": (completed / len(workflow.steps)) * 100 if workflow.steps else 0,
            "status": "completed" if workflow.completed_at else "in_progress"
        }
```

---

## Iteration 10: Master Index & Documentation

### Objective

Create comprehensive documentation, indexes, and navigation.

### Files Created

```
docs/
├── MASTER_INDEX.md
├── GLOSSARY.md
├── QUICK_REFERENCE.md
├── API_REFERENCE.md
├── ARCHITECTURE_DIAGRAMS.md
└── CONTRIBUTING.md

implementation/
├── documentation/
│   ├── generate_docs.py
│   ├── doc_templates/
│   └── api_extractor.py
└── tests/
    └── test_documentation.py
```

### Key Implementation

```python
# implementation/documentation/master_index.py

from dataclasses import dataclass
from typing import Dict, List, Any
import os
import json

@dataclass
class DocumentEntry:
    """Entry in the master document index"""
    doc_id: str
    title: str
    path: str
    category: str
    audience: List[str]
    dependencies: List[str]
    keywords: List[str]
    last_updated: str

class MasterIndex:
    """Master index for all documentation"""
    
    def __init__(self, docs_path: str = "docs"):
        self.docs_path = docs_path
        self.entries: Dict[str, DocumentEntry] = {}
        self.category_index: Dict[str, List[str]] = {}
        self.audience_index: Dict[str, List[str]] = {}
        self.keyword_index: Dict[str, List[str]] = {}
    
    def scan_documents(self):
        """Scan all documents and build index"""
        for root, dirs, files in os.walk(self.docs_path):
            for file in files:
                if file.endswith('.md'):
                    path = os.path.join(root, file)
                    entry = self._parse_document(path)
                    if entry:
                        self._index_entry(entry)
    
    def _parse_document(self, path: str) -> Optional[DocumentEntry]:
        """Parse a document and extract metadata"""
        try:
            with open(path, 'r') as f:
                content = f.read()
            
            # Extract title from first heading
            title = "Untitled"
            for line in content.split('\n'):
                if line.startswith('# '):
                    title = line[2:].strip()
                    break
            
            # Extract doc_id from filename
            doc_id = os.path.basename(path).replace('.md', '')
            
            return DocumentEntry(
                doc_id=doc_id,
                title=title,
                path=path,
                category=self._categorize(path),
                audience=self._extract_audience(content),
                dependencies=self._extract_dependencies(content),
                keywords=self._extract_keywords(content),
                last_updated=self._get_last_updated(path)
            )
        except Exception as e:
            print(f"Error parsing {path}: {e}")
            return None
    
    def _index_entry(self, entry: DocumentEntry):
        """Add entry to all indexes"""
        self.entries[entry.doc_id] = entry
        
        # Category index
        if entry.category not in self.category_index:
            self.category_index[entry.category] = []
        self.category_index[entry.category].append(entry.doc_id)
        
        # Audience index
        for audience in entry.audience:
            if audience not in self.audience_index:
                self.audience_index[audience] = []
            self.audience_index[audience].append(entry.doc_id)
        
        # Keyword index
        for keyword in entry.keywords:
            kw = keyword.lower()
            if kw not in self.keyword_index:
                self.keyword_index[kw] = []
            self.keyword_index[kw].append(entry.doc_id)
    
    def search(self, query: str, audience: str = None) -> List[DocumentEntry]:
        """Search the documentation index"""
        results = []
        query_keywords = set(query.lower().split())
        
        matching_ids = set()
        for keyword in query_keywords:
            if keyword in self.keyword_index:
                matching_ids.update(self.keyword_index[keyword])
        
        if audience and audience in self.audience_index:
            matching_ids = matching_ids.intersection(set(self.audience_index[audience]))
        
        for doc_id in matching_ids:
            results.append(self.entries[doc_id])
        
        return results
    
    def generate_reading_order(self, audience: str) -> List[DocumentEntry]:
        """Generate optimal reading order for an audience"""
        if audience not in self.audience_index:
            return []
        
        # Topological sort based on dependencies
        doc_ids = self.audience_index[audience]
        ordered = []
        visited = set()
        
        def visit(doc_id):
            if doc_id in visited:
                return
            visited.add(doc_id)
            for dep in self.entries[doc_id].dependencies:
                if dep in self.entries:
                    visit(dep)
            ordered.append(self.entries[doc_id])
        
        for doc_id in doc_ids:
            visit(doc_id)
        
        return ordered
    
    def _categorize(self, path: str) -> str:
        """Categorize document by path"""
        if "iterations" in path:
            return "iteration"
        elif "expert-analysis" in path:
            return "analysis"
        elif "synthesis" in path:
            return "synthesis"
        else:
            return "reference"
    
    def _extract_audience(self, content: str) -> List[str]:
        """Extract target audience from content"""
        audiences = []
        content_lower = content.lower()
        
        if "installer" in content_lower or "installation" in content_lower:
            audiences.append("installer")
        if "developer" in content_lower or "implementation" in content_lower:
            audiences.append("developer")
        if "educator" in content_lower or "training" in content_lower:
            audiences.append("educator")
        if "technician" in content_lower or "maintenance" in content_lower:
            audiences.append("technician")
        if "architect" in content_lower or "architecture" in content_lower:
            audiences.append("architect")
        
        return audiences if audiences else ["general"]
    
    def _extract_dependencies(self, content: str) -> List[str]:
        """Extract document dependencies from links"""
        import re
        links = re.findall(r'\[([^\]]+)\]\(([^)]+\.md)\)', content)
        return [os.path.basename(link).replace('.md', '') for _, link in links]
    
    def _extract_keywords(self, content: str) -> List[str]:
        """Extract keywords from content"""
        # Simple keyword extraction
        # Would use NLP in production
        words = content.lower().split()
        # Filter out common words and short words
        stop_words = {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for'}
        keywords = [w for w in words if len(w) > 4 and w not in stop_words]
        return list(set(keywords))[:20]
    
    def _get_last_updated(self, path: str) -> str:
        """Get last updated timestamp"""
        import datetime
        stat = os.stat(path)
        return datetime.datetime.fromtimestamp(stat.st_mtime).isoformat()
```

---

## Testing & Validation

### Complete Test Suite

```bash
# Run all tests
cd implementation
pytest tests/ -v --cov=. --cov-report=html

# Run specific iteration tests
pytest tests/test_iteration1.py tests/test_iteration2.py -v

# Run with coverage threshold
pytest tests/ --cov=. --cov-fail-under=80
```

### Integration Tests

```bash
# Run integration test suite
python tests/integration/test_full_pipeline.py

# Test vessel startup sequence
python tests/integration/test_vessel_startup.py --vessel-type marine

# Test fleet coordination
python tests/integration/test_fleet_coordination.py --vessels 5
```

---

## Deployment Checklist

### Pre-Deployment

- [ ] All tests passing
- [ ] Code coverage > 80%
- [ ] Documentation updated
- [ ] Configuration reviewed
- [ ] Security audit completed
- [ ] Performance benchmarks met

### Deployment Steps

1. **Build ROM Image**
   ```bash
   ./implementation/rom-base/rom-build.sh --target thor --version 1.0.0
   ```

2. **Verify ROM Integrity**
   ```bash
   ./implementation/rom-base/rom-verify.sh --image rom-thor-1.0.0.img
   ```

3. **Configure Vessel**
   ```bash
   python scripts/configure_vessel.py --type marine --domain fishing
   ```

4. **Initialize Agents**
   ```bash
   python scripts/initialize_agents.py --config vessel-config.yaml
   ```

5. **Run Pre-Flight Checks**
   ```bash
   python scripts/preflight_check.py --verbose
   ```

6. **Deploy to Vessel**
   ```bash
   python scripts/deploy.py --vessel-id vessel-001 --dry-run
   python scripts/deploy.py --vessel-id vessel-001
   ```

### Post-Deployment

- [ ] Vessel heartbeat received
- [ ] Fleet coordinator connected
- [ ] All agents operational
- [ ] A2A-R protocol active
- [ ] Knowledge index synchronized
- [ ] Portfolio system recording

---

## Next Steps

After completing all 10 iterations:

1. **Validation Testing** — Run comprehensive test suite on target hardware
2. **Performance Tuning** — Optimize for specific vessel configurations
3. **Documentation Review** — Ensure all docs are accurate and complete
4. **Training Development** — Create training materials for human operators
5. **Production Deployment** — Deploy to production vessels
6. **Monitoring Setup** — Configure fleet monitoring and alerting
7. **Feedback Loop** — Establish process for continuous improvement

---

*This Developer Guide is part of the Jetson Robotics Grand Design documentation suite.*
