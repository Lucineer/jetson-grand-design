# File Schemas: Jetson Robotics Grand Design

> Complete schema reference for every file in the repository scaffold

## Overview

This document provides detailed schemas for every file type in the Jetson Robotics Grand Design repository. Each schema includes:
- File purpose and location
- Required fields and types
- Validation rules
- Example content

---

## Table of Contents

1. [Configuration Files](#1-configuration-files)
2. [Implementation Files](#2-implementation-files)
3. [Documentation Files](#3-documentation-files)
4. [Data Files](#4-data-files)
5. [Test Files](#5-test-files)
6. [Build Files](#6-build-files)

---

## 1. Configuration Files

### 1.1 Vessel Configuration (`vessel-config.yaml`)

**Location**: `/etc/cocapn/vessel-config.yaml`

**Purpose**: Defines vessel identity, type, and core settings

```yaml
# JSON Schema
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["vessel_id", "vessel_type", "domain", "hardware"],
  "properties": {
    "vessel_id": {
      "type": "string",
      "pattern": "^vessel-[a-z0-9-]+$",
      "description": "Unique vessel identifier"
    },
    "vessel_type": {
      "type": "string",
      "enum": ["marine", "aerial", "industrial", "home", "medical"],
      "description": "Vessel domain type"
    },
    "domain": {
      "type": "string",
      "description": "Specific domain within type (e.g., 'fishing', 'warehouse')"
    },
    "hardware": {
      "type": "object",
      "required": ["jetson_model"],
      "properties": {
        "jetson_model": {
          "type": "string",
          "enum": ["thor", "agx-orin", "orin-nx", "orin-nano"]
        },
        "memory_gb": {
          "type": "integer",
          "minimum": 8,
          "maximum": 128
        },
        "storage_gb": {
          "type": "integer",
          "minimum": 64
        }
      }
    },
    "mig": {
      "type": "object",
      "required": ["enabled", "profile"],
      "properties": {
        "enabled": {"type": "boolean"},
        "profile": {
          "type": "string",
          "enum": ["balanced_4x32", "inference_optimized", "training_focused"]
        },
        "partitions": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "id": {"type": "integer"},
              "memory_gb": {"type": "integer"},
              "compute": {"type": "string"}
            }
          }
        }
      }
    },
    "network": {
      "type": "object",
      "properties": {
        "a2a_port": {"type": "integer", "default": 8080},
        "discovery_mode": {
          "type": "string",
          "enum": ["multicast", "dns-sd", "manual"],
          "default": "multicast"
        },
        "fleet_coordinator": {"type": "string"}
      }
    },
    "agents": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["name", "type", "mig_partition"],
        "properties": {
          "name": {"type": "string"},
          "type": {"type": "string"},
          "mig_partition": {"type": "integer"},
          "priority": {"type": "integer", "default": 0},
          "enabled": {"type": "boolean", "default": true}
        }
      }
    }
  }
}
```

**Example**:
```yaml
vessel_id: vessel-fishing-001
vessel_type: marine
domain: fishing

hardware:
  jetson_model: thor
  memory_gb: 128
  storage_gb: 1024

mig:
  enabled: true
  profile: balanced_4x32
  partitions:
    - id: 0
      memory_gb: 32
      compute: "25%"
    - id: 1
      memory_gb: 32
      compute: "25%"
    - id: 2
      memory_gb: 32
      compute: "25%"
    - id: 3
      memory_gb: 32
      compute: "25%"

network:
  a2a_port: 8080
  discovery_mode: multicast
  fleet_coordinator: coordinator.fleet.example.com

agents:
  - name: navigation
    type: NavigationAgent
    mig_partition: 0
    priority: 10
    enabled: true
  - name: species
    type: SpeciesAgent
    mig_partition: 1
    priority: 5
    enabled: true
  - name: safety
    type: SafetyAgent
    mig_partition: 2
    priority: 15
    enabled: true
```

---

### 1.2 Agent Configuration (`agent-config.yaml`)

**Location**: `/etc/cocapn/agents/{agent_name}.yaml`

**Purpose**: Defines agent behavior and parameters

```yaml
# JSON Schema
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "type", "model", "capabilities"],
  "properties": {
    "name": {"type": "string"},
    "type": {"type": "string"},
    "version": {"type": "string", "pattern": "^\\d+\\.\\d+\\.\\d+$"},
    "model": {
      "type": "object",
      "required": ["path", "quantization"],
      "properties": {
        "path": {"type": "string"},
        "quantization": {
          "type": "string",
          "enum": ["fp32", "fp16", "fp8", "fp4", "int8"]
        },
        "max_tokens": {"type": "integer"},
        "temperature": {"type": "number", "minimum": 0, "maximum": 2}
      }
    },
    "capabilities": {
      "type": "array",
      "items": {"type": "string"}
    },
    "input_topics": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "type": {"type": "string"},
          "rate_hz": {"type": "number"}
        }
      }
    },
    "output_topics": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "type": {"type": "string"}
        }
      }
    },
    "parameters": {
      "type": "object",
      "additionalProperties": true
    }
  }
}
```

**Example**:
```yaml
name: NavigationAgent
type: navigation
version: 1.0.0

model:
  path: /opt/models/navigation-v1
  quantization: fp4
  max_tokens: 2048
  temperature: 0.1

capabilities:
  - path_planning
  - collision_avoidance
  - waypoint_following
  - route_optimization

input_topics:
  - name: /sensors/gps
    type: sensor_msgs/NavSatFix
    rate_hz: 10
  - name: /sensors/compass
    type: sensor_msgs/MagneticField
    rate_hz: 20

output_topics:
  - name: /control/navigation
    type: cocapn/NavigationCommand
  - name: /status/navigation
    type: cocapn/AgentStatus

parameters:
  max_speed_mps: 10.0
  safety_margin_m: 5.0
  replan_interval_s: 2.0
```

---

### 1.3 Fleet Configuration (`fleet-config.yaml`)

**Location**: `/etc/cocapn/fleet-config.yaml`

**Purpose**: Defines fleet membership and coordination settings

```yaml
# JSON Schema
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["fleet_id", "coordinator"],
  "properties": {
    "fleet_id": {
      "type": "string",
      "pattern": "^fleet-[a-z0-9-]+$"
    },
    "coordinator": {
      "type": "object",
      "required": ["host"],
      "properties": {
        "host": {"type": "string"},
        "port": {"type": "integer", "default": 8080},
        "tls": {"type": "boolean", "default": true}
      }
    },
    "membership": {
      "type": "object",
      "properties": {
        "role": {
          "type": "string",
          "enum": ["member", "leader", "observer"]
        },
        "heartbeat_interval_s": {"type": "number", "default": 1.0},
        "timeout_s": {"type": "number", "default": 10.0}
      }
    },
    "knowledge_sharing": {
      "type": "object",
      "properties": {
        "enabled": {"type": "boolean", "default": true},
        "mode": {
          "type": "string",
          "enum": ["real-time", "batch", "scheduled"]
        },
        "sync_interval_s": {"type": "number", "default": 300}
      }
    },
    "federated_learning": {
      "type": "object",
      "properties": {
        "enabled": {"type": "boolean"},
        "participation_rate": {"type": "number", "minimum": 0, "maximum": 1},
        "privacy_budget": {"type": "number"}
      }
    }
  }
}
```

---

### 1.4 ROM Build Configuration (`rom-build.yaml`)

**Location**: `implementation/rom-base/rom-build.yaml`

**Purpose**: Defines ROM image build parameters

```yaml
# JSON Schema
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["target", "version", "components"],
  "properties": {
    "target": {
      "type": "string",
      "enum": ["thor", "agx-orin", "orin-nx", "orin-nano"]
    },
    "version": {"type": "string"},
    "base_image": {"type": "string"},
    "components": {
      "type": "object",
      "properties": {
        "jetpack_version": {"type": "string"},
        "cuda_version": {"type": "string"},
        "isaac_ros_version": {"type": "string"},
        "ros_distro": {"type": "string"}
      }
    },
    "optimizations": {
      "type": "object",
      "properties": {
        "fp4_enabled": {"type": "boolean"},
        "mig_preconfigured": {"type": "boolean"},
        "secure_boot": {"type": "boolean"}
      }
    },
    "size_gb": {"type": "integer"}
  }
}
```

---

## 2. Implementation Files

### 2.1 Python Module Structure

**Location**: `implementation/**/*.py`

**Purpose**: Core implementation modules

#### Module Header Template

```python
"""
Module: {module_name}
Purpose: {description}
Part of: Jetson Robotics Grand Design
Iteration: {iteration_number}

Dependencies:
    - {dependency_1}
    - {dependency_2}

Author: {author}
Created: {date}
Last Modified: {date}
"""

from typing import Dict, List, Any, Optional
from dataclasses import dataclass
import logging

# Module-level constants
MODULE_VERSION = "1.0.0"
LOGGER = logging.getLogger(__name__)


@dataclass
class {ClassName}Config:
    """Configuration for {ClassName}"""
    # Fields with type hints and defaults
    
    def validate(self) -> bool:
        """Validate configuration"""
        pass


class {ClassName}:
    """
    {ClassName} - {brief description}
    
    Attributes:
        {attribute}: {description}
    
    Methods:
        {method}: {description}
    """
    
    def __init__(self, config: {ClassName}Config):
        """Initialize {ClassName}"""
        self.config = config
        self._setup()
    
    def _setup(self):
        """Internal setup method"""
        pass
    
    def public_method(self, param: type) -> return_type:
        """Public method description"""
        pass
```

---

### 2.2 Agent Implementation Schema

**Location**: `implementation/agents/{agent_type}_agent.py`

```python
# Agent Implementation Schema
{
  "required_imports": [
    "from abc import ABC, abstractmethod",
    "from typing import Dict, List, Any, Optional",
    "from dataclasses import dataclass",
    "import asyncio"
  ],
  "required_classes": [
    "AgentConfig",
    "AgentState",
    "AgentStatus"
  ],
  "required_methods": [
    {
      "name": "initialize",
      "signature": "async def initialize(self) -> bool",
      "description": "Initialize agent resources"
    },
    {
      "name": "execute",
      "signature": "async def execute(self, params: Dict) -> Dict",
      "description": "Execute agent task"
    },
    {
      "name": "shutdown",
      "signature": "async def shutdown(self) -> bool",
      "description": "Clean shutdown"
    },
    {
      "name": "status",
      "signature": "def status(self) -> AgentStatus",
      "description": "Get current status"
    }
  ]
}
```

---

### 2.3 A2A-R Message Schema

**Location**: `implementation/a2a_r/messages/`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "A2A-R Message",
  "type": "object",
  "required": ["message_id", "sender_id", "receiver_id", "message_type", "qos", "timestamp", "payload"],
  "properties": {
    "message_id": {
      "type": "string",
      "format": "uuid"
    },
    "sender_id": {
      "type": "string",
      "pattern": "^(vessel|agent)-[a-z0-9-]+$"
    },
    "receiver_id": {
      "type": "string",
      "description": "Recipient ID or 'broadcast'"
    },
    "message_type": {
      "type": "string",
      "enum": [
        "telemetry",
        "control",
        "coordination",
        "perception",
        "status",
        "discovery",
        "learning"
      ]
    },
    "qos": {
      "type": "string",
      "enum": ["best_effort", "reliable", "real_time"]
    },
    "timestamp": {
      "type": "number",
      "description": "Unix timestamp in seconds"
    },
    "payload": {
      "type": "object",
      "description": "Message-specific payload"
    },
    "latency_budget_ms": {
      "type": "number",
      "description": "Maximum acceptable latency"
    },
    "correlation_id": {
      "type": "string",
      "description": "For request-response correlation"
    },
    "ttl": {
      "type": "integer",
      "description": "Time-to-live in seconds"
    }
  }
}
```

---

### 2.4 Experience Primitive Schema

**Location**: `implementation/training/schemas/experience.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Experience Primitive",
  "type": "object",
  "required": ["experience_id", "vessel_id", "timestamp", "context", "observations", "actions", "outcomes"],
  "properties": {
    "experience_id": {
      "type": "string",
      "pattern": "^exp-[a-f0-9]{16}$"
    },
    "vessel_id": {
      "type": "string"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "context": {
      "type": "object",
      "properties": {
        "state": {"type": "object"},
        "location": {"type": "object"},
        "environment": {"type": "object"},
        "task": {"type": "string"}
      }
    },
    "observations": {
      "type": "object",
      "properties": {
        "sensors": {"type": "object"},
        "perceptions": {"type": "array"},
        "inferences": {"type": "object"}
      }
    },
    "actions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "action_id": {"type": "string"},
          "type": {"type": "string"},
          "parameters": {"type": "object"},
          "timestamp": {"type": "string"}
        }
      }
    },
    "outcomes": {
      "type": "object",
      "properties": {
        "success": {"type": "boolean"},
        "metrics": {"type": "object"},
        "side_effects": {"type": "array"}
      }
    },
    "learning": {
      "type": "object",
      "properties": {
        "pattern_detected": {"type": "string"},
        "improvement_suggestion": {"type": "string"},
        "confidence": {"type": "number"}
      }
    },
    "quality_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "tier": {
      "type": "integer",
      "enum": [1, 2, 3],
      "description": "1=high, 2=medium, 3=low quality"
    }
  }
}
```

---

### 2.5 Innovation Portfolio Schema

**Location**: `implementation/portfolio/schemas/innovation.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Innovation Portfolio Item",
  "type": "object",
  "required": ["innovation_id", "contributor_id", "commit_hash", "innovation_type", "title", "verification_status"],
  "properties": {
    "innovation_id": {
      "type": "string",
      "pattern": "^INN-[a-f0-9]{8}$"
    },
    "contributor_id": {
      "type": "string",
      "description": "Contributor identifier (email or ID)"
    },
    "commit_hash": {
      "type": "string",
      "pattern": "^[a-f0-9]{40}$"
    },
    "innovation_type": {
      "type": "string",
      "enum": ["bug_fix", "optimization", "feature", "pattern", "integration", "documentation"]
    },
    "title": {
      "type": "string",
      "maxLength": 200
    },
    "description": {
      "type": "string"
    },
    "affected_files": {
      "type": "array",
      "items": {"type": "string"}
    },
    "verification_status": {
      "type": "string",
      "enum": ["pending", "verified", "rejected"]
    },
    "verification_layers": {
      "type": "object",
      "properties": {
        "commit_integrity": {"type": "boolean"},
        "gpg_signature": {"type": "boolean"},
        "timestamp_authority": {"type": "boolean"},
        "code_review": {"type": "boolean"},
        "testing_passed": {"type": "boolean"}
      }
    },
    "adoption_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of vessels that adopted this innovation"
    },
    "derivative_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of innovations built on this one"
    },
    "value_score": {
      "type": "number",
      "minimum": 0,
      "description": "Calculated portfolio value"
    },
    "domains": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": ["marine", "aerial", "industrial", "home", "medical", "cross-domain"]
      }
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    },
    "last_updated": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

## 3. Documentation Files

### 3.1 Iteration Document Schema

**Location**: `docs/iterations/iter{NN}_{title}.md`

```markdown
# Iteration {N}: {Title}

> **Status**: {Draft | In Progress | Complete}
> **Created**: {YYYY-MM-DD}
> **Last Updated**: {YYYY-MM-DD}

## Objective
{Clear statement of what this iteration accomplishes}

## Dependencies
- Iteration {N-1}: {Dependency name}
- {Other dependencies}

## Files Created
```
{directory}/
├── {file1}
├── {file2}
└── {file3}
```

## Implementation Steps

### Step 1: {Step Title}
{Description}

```{language}
{Code example}
```

### Step 2: {Step Title}
{Description}

## Tests
{Test requirements and examples}

## Validation Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}

## Known Issues
- {Issue 1}
- {Issue 2}

## Next Steps
{What comes next after this iteration}
```

---

### 3.2 Expert Analysis Document Schema

**Location**: `docs/expert-analysis/{type}_{name}.md`

```markdown
# {Expert Name} Analysis: {Domain}

> **Expert**: {ARCHIMEDES | MAYA | GAIA | ORACLE}
> **Domain**: {Technical | Human-Centric | Ecosystem | Paradigmatic}
> **Created**: {YYYY-MM-DD}

## Executive Summary
{2-3 paragraph summary of findings}

## Key Findings

### Finding 1: {Title}
{Detailed explanation}

### Finding 2: {Title}
{Detailed explanation}

## Architecture Recommendations
{Specific recommendations with rationale}

## Code Examples
{Implementation examples}

## Open Questions
- {Question 1}
- {Question 2}

## References
- {Reference 1}
- {Reference 2}
```

---

### 3.3 Glossary Entry Schema

**Location**: `docs/GLOSSARY.md`

```markdown
## {Term}

**Definition**: {Concise definition}

**Context**: {Where this term is used in the ecosystem}

**Related Terms**: {comma-separated list}

**Example**:
```
{Code or usage example}
```

**See Also**: {Links to related documentation}
```

---

## 4. Data Files

### 4.1 Training Data Format

**Location**: `/data/training/{domain}/{timestamp}.jsonl`

```
# JSON Lines format, one experience per line
{"experience_id": "exp-...", "vessel_id": "...", ...}
{"experience_id": "exp-...", "vessel_id": "...", ...}
```

### 4.2 Model Checkpoint Format

**Location**: `/data/models/{model_name}/checkpoint-{step}/`

```
checkpoint-{step}/
├── model.safetensors      # Model weights
├── config.json            # Model configuration
├── tokenizer.json         # Tokenizer
├── training_args.json     # Training arguments
├── optimizer.pt           # Optimizer state
├── scheduler.pt           # LR scheduler state
└── trainer_state.json     # Training metrics
```

### 4.3 Federated Learning Aggregation

**Location**: `/data/federated/round-{N}/`

```
round-{N}/
├── contributions/
│   ├── vessel-001.json    # Vessel contribution
│   ├── vessel-002.json
│   └── ...
├── aggregated/
│   └── model-update.safetensors
└── metadata.json          # Round statistics
```

---

## 5. Test Files

### 5.1 Unit Test Schema

**Location**: `tests/test_{module}.py`

```python
"""
Tests for {module_name}
"""

import pytest
from unittest.mock import Mock, patch
from {module_path} import {ClassName}


class Test{ClassName}:
    """Unit tests for {ClassName}"""
    
    @pytest.fixture
    def config(self):
        """Test configuration fixture"""
        return {
            "param1": "value1",
            "param2": 42
        }
    
    @pytest.fixture
    def instance(self, config):
        """Instance fixture"""
        return {ClassName}(config)
    
    def test_initialization(self, instance):
        """Test proper initialization"""
        assert instance is not None
    
    def test_{method_name}_success(self, instance):
        """Test {method_name} success case"""
        result = instance.{method_name}({params})
        assert result == {expected}
    
    def test_{method_name}_failure(self, instance):
        """Test {method_name} failure case"""
        with pytest.raises({ExceptionType}):
            instance.{method_name}({invalid_params})
    
    @pytest.mark.parametrize("input,expected", [
        ({input1}, {expected1}),
        ({input2}, {expected2}),
    ])
    def test_{method_name}_cases(self, instance, input, expected):
        """Test {method_name} with multiple cases"""
        assert instance.{method_name}(input) == expected
```

---

### 5.2 Integration Test Schema

**Location**: `tests/integration/test_{feature}.py`

```python
"""
Integration tests for {feature}
Tests multiple components working together
"""

import pytest
import asyncio
from {module1} import {Class1}
from {module2} import {Class2}


class Test{Feature}Integration:
    """Integration tests for {feature}"""
    
    @pytest.fixture(autouse=True)
    async def setup(self):
        """Set up test environment"""
        # Initialize components
        self.component1 = {Class1}(...)
        self.component2 = {Class2}(...)
        
        await self.component1.initialize()
        await self.component2.initialize()
        
        yield
        
        # Cleanup
        await self.component1.shutdown()
        await self.component2.shutdown()
    
    @pytest.mark.asyncio
    async def test_full_pipeline(self):
        """Test complete pipeline"""
        # Execute pipeline
        result = await self.component1.process(...)
        
        # Verify component2 received
        state = await self.component2.get_state()
        assert state == expected_state
```

---

## 6. Build Files

### 6.1 Dockerfile Schema

**Location**: `implementation/{component}/Dockerfile`

```dockerfile
# Dockerfile Schema
# Base image (required)
FROM nvcr.io/nvidia/l4t-jetpack:{version}

# Labels (required)
LABEL maintainer="{maintainer}"
LABEL version="{version}"
LABEL description="{description}"

# Build arguments (optional)
ARG TARGETARCH
ARG VERSION=1.0.0

# Environment variables (as needed)
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive

# System dependencies (as needed)
RUN apt-get update && apt-get install -y \
    {package1} \
    {package2} \
    && rm -rf /var/lib/apt/lists/*

# Python dependencies (if Python project)
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt

# Application code (required)
COPY . /app/
WORKDIR /app

# Health check (recommended)
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:{port}/health || exit 1

# Entry point (required)
ENTRYPOINT ["python", "-m", "{module}"]
CMD ["--config", "/etc/cocapn/{config}.yaml"]
```

---

### 6.2 Build Script Schema

**Location**: `scripts/{script_name}.sh`

```bash
#!/bin/bash
# {Script Name}
# Purpose: {What this script does}
# Usage: ./script_name.sh [options]
#
# Options:
#   --target {target}    Target platform
#   --version {version}  Version to build
#   --help               Show this help

set -euo pipefail

# Script constants
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(dirname "$SCRIPT_DIR")"

# Default values
TARGET="${TARGET:-thor}"
VERSION="${VERSION:-1.0.0}"

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --target)
            TARGET="$2"
            shift 2
            ;;
        --version)
            VERSION="$2"
            shift 2
            ;;
        --help)
            sed -n '2,8p' "$0" | sed 's/^# //'
            exit 0
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done

# Main execution
main() {
    echo "Building for target: $TARGET, version: $VERSION"
    # Implementation
}

main "$@"
```

---

## Summary

This schema reference provides the structural foundation for all files in the Jetson Robotics Grand Design repository. Key principles:

1. **Consistency**: All files follow established patterns
2. **Validation**: Schemas enable automated validation
3. **Documentation**: Self-documenting through structure
4. **Extensibility**: New files follow existing patterns

For questions about specific schemas, consult the relevant iteration document or the master index.

---

*File Schemas v1.0 — Part of Jetson Robotics Grand Design Documentation*
