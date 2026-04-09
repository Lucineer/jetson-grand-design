# Onboarding Guide: Jetson Robotics Grand Design

> Welcome to the Jetson Robotics Grand Design ecosystem!

This guide will help you get started as a contributor, whether you're a developer, installer, educator, technician, or visionary.

---

## Table of Contents

1. [Welcome to the Ecosystem](#welcome-to-the-ecosystem)
2. [Choose Your Path](#choose-your-path)
3. [Development Environment Setup](#development-environment-setup)
4. [Understanding the Architecture](#understanding-the-architecture)
5. [Your First Contribution](#your-first-contribution)
6. [Portfolio System](#portfolio-system)
7. [Getting Help](#getting-help)
8. [Glossary of Terms](#glossary-of-terms)

---

## Welcome to the Ecosystem

### What We're Building

The Jetson Robotics Grand Design is a comprehensive vision for a **self-improving, git-native robotics ecosystem** with NVIDIA Jetson devices at its center. We're building a world where:

- **Every task becomes a learning opportunity** — The system captures and learns from every interaction
- **Every solution becomes a reusable pattern** — Innovations are automatically documented and shared
- **Every human contribution compounds in value** — Your work becomes part of your innovation portfolio
- **Novices become veterans through actual system evolution** — The system grows with its users

### Core Paradigm

**The repository IS the agent.**

- Git history is memory
- Code is body
- Commits are experiences
- Pull requests are innovations

### Human Value Trajectory

Unlike systems that replace humans, this ecosystem is designed so that **human value INCREASES** because their work becomes **repeatable portfolios of innovation**. The more the system learns, the more valuable human contributions become.

---

## Choose Your Path

Different roles contribute differently. Find your path:

### 🛠️ Developer Path

**You are**: A software engineer, AI researcher, or robotics developer

**Your focus**:
- Implementing core agent behaviors
- Building training pipelines
- Developing A2A-R protocol extensions
- Optimizing for edge deployment

**Start here**:
1. Read [Technical Deep Dive](expert-analysis/expert_technical_deep_dive.md)
2. Study [ROM Architecture](iterations/iter01_rom_emulation_architecture.md)
3. Try [Developer Guide](DEVELOPER_GUIDE.md)

### 📦 Installer Path

**You are**: A technician deploying systems on vessels

**Your focus**:
- Installing and configuring Jetson devices
- Setting up vessel-specific hardware
- Running pre-flight checks
- Troubleshooting deployment issues

**Start here**:
1. Read [Vessel Type Specifications](iterations/iter02_vessel_type_specifications.md)
2. Study [Human Workflow Guides](iterations/iter09_human_workflow_guides.md)
3. Learn [Failure Modes & Recovery](iterations/iter08_failure_modes_recovery.md)

### 🎓 Educator Path

**You are**: A trainer teaching others to use the system

**Your focus**:
- Creating training materials
- Conducting certification sessions
- Developing curriculum for different roles
- Building knowledge base content

**Start here**:
1. Read [Human-Centric Design](expert-analysis/expert_human_centric_design.md)
2. Study [Knowledge Indexes](iterations/iter07_knowledge_indexes.md)
3. Review [Glossary](GLOSSARY.md)

### 🔧 Technician Path

**You are**: A maintenance professional keeping systems running

**Your focus**:
- Diagnosing and fixing issues
- Performing preventive maintenance
- Escalating complex problems
- Documenting solutions

**Start here**:
1. Read [Human Workflow Guides](iterations/iter09_human_workflow_guides.md)
2. Study [Failure Modes & Recovery](iterations/iter08_failure_modes_recovery.md)
3. Learn [Portfolio System](iterations/iter06_portfolio_system.md)

### 💡 Visionary Path

**You are**: An investor, executive, or strategist

**Your focus**:
- Understanding the ecosystem vision
- Evaluating investment opportunities
- Planning deployment strategies
- Setting organizational direction

**Start here**:
1. Read [Synthesis & Reverse Actualization](synthesis/synthesis_reverse_actualization.md)
2. Study [Paradigmatic Analysis](expert-analysis/expert_paradigmatic_analysis.md)
3. Review [Master Index](iterations/iter10_master_index.md)

---

## Development Environment Setup

### Prerequisites

Before you begin, ensure you have:

- **Operating System**: Ubuntu 22.04 LTS or macOS 12+
- **Docker**: Version 24.x or later
- **Git**: Version 2.40 or later
- **Python**: Version 3.10 or later
- **Node.js/Bun**: Version 18+ / 1.x

### Quick Start

```bash
# 1. Fork the repository on GitHub
# 2. Clone your fork
git clone https://github.com/YOUR_USERNAME/jetson-robotics-grand-design.git
cd jetson-robotics-grand-design

# 3. Add upstream remote
git remote add upstream https://github.com/original/jetson-robotics-grand-design.git

# 4. Initialize submodules
git submodule update --init --recursive

# 5. Set up Python environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
pip install -r requirements-dev.txt

# 6. Install pre-commit hooks
pre-commit install

# 7. Run initial tests
pytest tests/ -v

# 8. Build documentation (optional)
cd docs && mkdocs serve
```

### Environment Configuration

Create your `.env` file:

```bash
# Copy the example
cp .env.example .env

# Edit with your settings
nano .env
```

Required settings for development:

```bash
# Your identity (for portfolio tracking)
CONTRIBUTOR_EMAIL=your.email@example.com
CONTRIBUTOR_NAME="Your Name"

# Development settings
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG

# Optional: Jetson hardware settings (if testing on hardware)
JETSON_MODEL=thor
MIG_ENABLED=false
```

### IDE Setup

#### VS Code (Recommended)

Install these extensions:
- Python
- Docker
- GitLens
- YAML
- Markdown All in One

Recommended `settings.json`:
```json
{
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "markdown.preview.breaks": true
}
```

#### PyCharm

1. Open the project as a Python project
2. Configure the virtual environment
3. Enable Docker integration
4. Install Markdown plugin

---

## Understanding the Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           THE COCAPN SOFTWARE VESSEL                         │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                     APPLICATION LAYER (Flexible)                     │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │    │
│  │  │  capitaine  │  │field-captain│  │    nexus    │  │   custom   │ │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │    │
│  │                          A2A-R Protocol                              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                      │                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                       ROM BASE LAYER (Stable)                        │    │
│  │       JetPack  •  Isaac ROS  •  CUDA  •  FP4 Models                 │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                      │                                       │
│                          JETSON THOR HARDWARE                                │
│                                      │                                       │
│                          PHYSICAL VESSEL (Boat/Drone/Robot)                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Concepts

#### Cocapn Software Vessel

The "cocapn" is a **meta-agent** that orchestrates all other agents on a vessel. Think of it as the captain of a ship — it doesn't do everything itself, but it coordinates all the specialized roles.

#### Git-Native Memory

The system uses git as its memory:
- **Commits** = Individual experiences
- **Branches** = Alternative approaches
- **Pull Requests** = Proposed innovations
- **Merges** = Adopted innovations
- **Tags** = Stable releases

#### Innovation Portfolio

Every contribution you make becomes part of your **innovation portfolio**:
- Verified through git history (not blockchain)
- Attributed to you with cryptographic signatures
- Increases in value as others adopt your innovations

#### Vessel Types

The ecosystem supports multiple vessel types:
- **Marine**: Fishing boats, research vessels, autonomous surface vessels
- **Aerial**: Drones, fixed-wing UAVs, eVTOLs
- **Industrial**: Warehouse robots, manipulator arms, AGVs
- **Home**: Assistant robots, kitchen robots
- **Medical**: Surgical assistants, diagnostic robots

#### Hardware Tiers

The ecosystem supports three hardware tiers, each optimized for different use cases:

| Tier | Hardware | Best For | Cost |
|------|----------|----------|------|
| **Tier 1** | Jetson Thor (128GB) | Research vessels, high-end applications | $2,000+ |
| **Tier 2** | Dual Orin Nano (8GB) | Commercial fishing, charter boats | ~$1,000 |
| **Tier 3** | Raspberry Pi 5 + ESP32 | Small vessels, edge IoT, education | ~$150 |

**Tier 1 (Thor)**: Full AI capabilities including large language models (70B parameters), advanced vision, and complex multi-agent coordination. See [ROM Architecture](iterations/iter01_rom_emulation_architecture.md).

**Tier 2 (Dual Orin Nano)**: Redundant dual-module setup with automatic failover. One module handles UI/navigation, the other runs inference (LLM, STT, TTS). Perfect for commercial operations requiring reliability. See [Dual Orin Nano Architecture](architectures/dual-orin-nano-commercial-fishing.md).

**Tier 3 (Pi + ESP32)**: Cloud-heavy development with edge execution. Uses GitHub Codespaces as full IoT IDE. ESP32 devices run "git-agents" that can be updated over-the-air. See [Raspberry Pi ESP32 Architecture](architectures/raspberry-pi-esp32-cloud-edge.md).

### Repository Structure

```
jetson-robotics-grand-design/
├── docs/                          # Documentation
│   ├── expert-analysis/           # Expert research documents
│   ├── iterations/                # 10 iteration documents
│   └── synthesis/                 # Vision and synthesis documents
├── implementation/                # Actual code
│   ├── agents/                    # Agent implementations
│   ├── a2a_r/                     # A2A-R protocol
│   ├── cocapn/                    # Meta-agent
│   ├── knowledge/                 # Knowledge indexing
│   ├── portfolio/                 # Portfolio system
│   ├── resilience/                # Failure handling
│   ├── rom-base/                  # ROM layer
│   ├── training/                  # Training pipeline
│   ├── vessel-types/              # Vessel type definitions
│   └── workflow/                  # Workflow engine
├── tests/                         # Test suites
├── scripts/                       # Utility scripts
├── requirements.txt               # Python dependencies
└── README.md                      # Project overview
```

---

## Your First Contribution

### Step 1: Find Something to Work On

Browse open issues:
```bash
gh issue list --label "good first issue"
```

Or check the project board for tasks marked "help wanted".

### Step 2: Create a Branch

```bash
# Update your main branch
git checkout main
git pull upstream main

# Create a feature branch
git checkout -b feature/your-feature-name

# Or a fix branch
git checkout -b fix/issue-number-description
```

### Step 3: Make Your Changes

Follow the coding standards:
- Use meaningful variable names
- Add docstrings to functions
- Include type hints
- Write tests for new code

### Step 4: Test Your Changes

```bash
# Run relevant tests
pytest tests/test_related_module.py -v

# Run linting
flake8 implementation/
black --check implementation/

# Run type checking
mypy implementation/
```

### Step 5: Commit with Portfolio in Mind

Your commits become part of your portfolio. Write good commit messages:

```bash
# Good commit message format
git commit -m "feat(navigation): add waypoint interpolation for smoother paths

- Implement cubic spline interpolation
- Add tests for edge cases
- Update documentation

Closes #123"
```

### Step 6: Push and Create PR

```bash
# Push to your fork
git push origin feature/your-feature-name

# Create PR (using GitHub CLI)
gh pr create --title "Add waypoint interpolation" --body "Description of changes"
```

### Step 7: Code Review

Be responsive to feedback. Each review is an opportunity to improve and learn.

---

## Portfolio System

### How It Works

Your contributions are tracked in your innovation portfolio:

```
Your Portfolio
├── Bug Fixes (Value: Base × Adoption)
│   ├── Fix GPS timeout handling (Adopted by 5 vessels)
│   └── Fix memory leak in species detection (Adopted by 12 vessels)
├── Optimizations
│   └── Improve path planning by 15% (Adopted by 23 vessels)
├── Features
│   └── Add weather routing capability (Adopted by 8 vessels)
└── Patterns
    └── Define standard error handling pattern (Built on by 3 innovations)
```

### Value Calculation

Innovation Value = Base Value × Adoption Multiplier × Derivative Multiplier

- **Base Value**: Bug fix (10), Optimization (15), Feature (20), Pattern (25), Integration (30)
- **Adoption Multiplier**: 1 + (adoption_count × 0.1)
- **Derivative Multiplier**: 1 + (derivative_count × 0.2)

### Verification Layers

All innovations are verified through git-native mechanisms:

1. **Commit Integrity** — SHA-256 hash verification
2. **GPG Signature** — Cryptographic author verification
3. **Timestamp Authority** — When the innovation was created
4. **Code Review** — Peer verification of quality
5. **Testing** — Automated verification

---

## Getting Help

### Documentation

- **Master Index**: [iterations/iter10_master_index.md](iterations/iter10_master_index.md)
- **Glossary**: [GLOSSARY.md](GLOSSARY.md)
- **Architecture**: [ARCHITECTURE.md](ARCHITECTURE.md)

### Community

- **GitHub Discussions**: For questions and discussions
- **GitHub Issues**: For bug reports and feature requests
- **Discord/Slack**: For real-time chat (if configured)

### Office Hours

Weekly office hours are held for new contributors. Check the project calendar for times.

### Mentorship

If you'd like a mentor to guide your early contributions, reach out to the maintainers.

---

## Glossary of Terms

### A

**A2A Protocol**
: Google's open standard for agent-to-agent communication. Enables agents to discover each other's capabilities and communicate effectively.

**A2A-R**
: Robotics extensions to the A2A Protocol, adding real-time guarantees, sensor streaming, and coordination primitives.

**Agent**
: A software component that performs specific tasks within the vessel ecosystem. Examples: NavigationAgent, SpeciesAgent, SafetyAgent.

**Application Layer**
: The flexible, updateable layer of the system that runs on top of the ROM base layer. Contains agent code and configurations.

### C

**Capitaine**
: The foundational repository implementing the "repo IS the agent" paradigm. The agent framework that all other components build on.

**Cocapn**
: The meta-agent that orchestrates all other agents on a vessel. Like a ship's captain coordinating the crew.

**Commit**
: In git, a snapshot of changes. In this ecosystem, also represents a single "experience" or "memory" of the agent.

### F

**Field-Captain**
: The repository for Jetson field deployment, including voice interface, local LLM, and orchestration for boats and job sites.

**Fleet**
: A collection of vessels coordinated through the nexus-runtime.

**FP4**
: 4-bit floating point quantization, enabling efficient local fine-tuning on Jetson devices.

### G

**Git-Native**
: Architecture that uses git as the fundamental storage and memory mechanism. Commits are experiences, PRs are innovations.

**GPG Signature**
: Cryptographic signature on git commits, verifying author identity for portfolio attribution.

### I

**Innovation Portfolio**
: A record of a contributor's verified innovations, with value that grows as others adopt and build on them.

**Iteration**
: One of the 10 development phases that progressively build the complete system.

### J

**Jetson**
: NVIDIA's edge AI computing platform. Thor is the latest generation with 2070 FP4 TFLOPS.

**JetPack**
: NVIDIA's software development kit for Jetson devices, including CUDA, TensorRT, and other tools.

### K

**Knowledge Index**
: Semantic search and discovery system for finding capabilities, patterns, and solutions across the fleet.

### L

**LoRA**
: Low-Rank Adaptation, a technique for efficient fine-tuning of large models on edge devices.

### M

**MIG (Multi-Instance GPU)**
: NVIDIA technology for partitioning a GPU into multiple isolated instances, enabling multiple agents to run safely on one device.

### N

**Nexus-Runtime**
: The orchestration layer for fleet coordination and multi-agent management.

### P

**Portfolio**
: See Innovation Portfolio.

**Pull Request**
: Proposed changes to the codebase. In this ecosystem, also represents a proposed innovation for peer review.

### R

**Reverse Actualization**
: Planning methodology that starts from the desired future state and works backward to identify required steps.

**ROM Base Layer**
: The stable, rarely-updated foundation of the system containing JetPack, Isaac ROS, CUDA, and base models.

### S

**Self-Improving**
: System capability to learn from experiences and automatically improve its performance over time.

### T

**Training Data Pipeline**
: System for capturing, curating, and using experiences to train and improve agents.

**Thor**
: NVIDIA's latest Jetson platform, with 2070 FP4 TFLOPS and 128GB unified memory.

### V

**Vessel**
: A physical robot or device with a Jetson computing core and the cocapn software stack. Types include marine, aerial, industrial, home, and medical.

**Vessel Bridge**
: Hardware abstraction layer connecting the software vessel to physical sensors and actuators.

---

## Next Steps

1. ✅ Complete this onboarding guide
2. 📚 Read the documents for your chosen path
3. 🛠️ Set up your development environment
4. 🔍 Find a "good first issue" to work on
5. 🤝 Join the community discussions
6. 🚀 Make your first contribution!

---

*Welcome aboard! Your journey to becoming a veteran contributor starts now.*

*Onboarding Guide v1.0 — Part of Jetson Robotics Grand Design Documentation*
