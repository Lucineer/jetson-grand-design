# Iteration 3: Cocapn Software Vessel

## The Meta-Agent Architecture for Vessel Management

**Document Version**: 1.0  
**Iteration**: 03  
**Date**: January 2025  
**Status**: Architectural Specification

---

## Executive Summary

The **cocapn** (co-captain) is the meta-agent that orchestrates all other agents within a software vessel. It acts as the primary interface between humans and the vessel's capabilities, managing agent lifecycles, resource allocation, and cross-agent coordination. The cocapn embodies the "garden tender" relationship described in the 2044 vision.

### Core Concept: The Cocapn as Orchestrator

```
+------------------------------------------------------------------+
|                    COCAPN META-AGENT                               |
+------------------------------------------------------------------+
|                                                                   |
|  Human Operator                                                   |
|       │                                                           |
|       │ Voice / Text / Dashboard                                  |
|       ▼                                                           |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                        COCAPN                                │ |
|  │                                                              │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                 META-AGENT FUNCTIONS                     ││ |
|  │  │                                                          ││ |
|  │  │  • Intent Recognition    • Agent Selection              ││ |
|  │  │  • Task Decomposition    • Result Synthesis             ││ |
|  │  │  • Resource Allocation   • Conflict Resolution          ││ |
|  │  │  • Lifecycle Management  • Learning Coordination        ││ |
|  │  │                                                          ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                              │                               │ |
|  │              ┌───────────────┼───────────────┐              │ |
|  │              │               │               │              │ |
|  │              ▼               ▼               ▼              │ |
|  │         ┌─────────┐    ┌─────────┐    ┌─────────┐          │ |
|  │         │ Nav     │    │ Species │    │ Safety  │          │ |
|  │         │ Agent   │    │ Agent   │    │ Agent   │          │ |
|  │         └─────────┘    └─────────┘    └─────────┘          │ |
|  │              │               │               │              │ |
|  │              └───────────────┴───────────────┘              │ |
|  │                              │                               │ |
|  │                              ▼                               │ |
|  │              ┌─────────────────────────────────┐            │ |
|  │              │      PHYSICAL VESSEL            │            │ |
|  │              │   (Sensors, Actuators, Hull)    │            │ |
|  │              └─────────────────────────────────┘            │ |
|  │                                                            │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Part 1: The Cocapn as Meta-Agent

### 1.1 Conceptual Framework

The cocapn operates at a higher level of abstraction than domain-specific agents. While individual agents handle specific tasks (navigation, perception, manipulation), the cocapn handles:

1. **Intent Recognition** - Understanding what the human wants
2. **Task Decomposition** - Breaking complex requests into agent-level tasks
3. **Agent Selection** - Routing tasks to appropriate agents
4. **Resource Allocation** - Managing GPU partitions, memory, and compute
5. **Result Synthesis** - Combining outputs from multiple agents
6. **Learning Coordination** - Ensuring agents learn from experiences

```
+------------------------------------------------------------------+
|                    COCAPN ABSTRACTION LAYERS                       |
+------------------------------------------------------------------+
|                                                                   |
|  Layer 4: HUMAN INTENT                                           |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  "Take us to the best fishing spot and identify what we    │ │
|  │   catch while keeping us safe from other vessels."         │ │
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|                              ▼                                    |
|  Layer 3: COCAPN (Meta-Agent)                                    |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Intent: Fishing expedition                                 │ │
|  │  Decomposition:                                             │ │
|  │    1. Navigation Agent: Route to optimal fishing location   │ │
|  │    2. Species Agent: Prepare for fish identification        │ │
|  │    3. Safety Agent: Monitor AIS, radar, weather             │ │
|  │  Coordination: Parallel execution with safety priority      │ │
|  └─────────────────────────────────────────────────────────────┘ |
|                              │                                    |
|                              ▼                                    |
|  Layer 2: DOMAIN AGENTS                                          |
|  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              |
|  │ Navigation  │  │  Species    │  │   Safety    │              |
|  │ Agent       │  │  Agent      │  │   Agent     │              |
|  │             │  │             │  │             │              |
|  │ Route: 42°N │  │ Ready:      │  │ Monitoring: │              |
|  │ 12:30 ETA   │  │ vision, log │  │ 3 vessels   │              |
|  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              |
|         │                │                │                      |
|         └────────────────┴────────────────┘                      |
|                          │                                        |
|                          ▼                                        |
|  Layer 1: PHYSICAL VESSEL                                        |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Autopilot, Cameras, Sonar, GPS, AIS, Winches...           │ │
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 Cocapn Architecture

```python
# cocapn_architecture.py
"""
Cocapn meta-agent architecture
The orchestrator for all vessel agents
"""

from typing import Dict, Any, List, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import asyncio
from datetime import datetime
import json
from pathlib import Path

class AgentStatus(Enum):
    IDLE = "idle"
    BUSY = "busy"
    ERROR = "error"
    OFFLINE = "offline"

@dataclass
class AgentCapability:
    """Description of an agent's capability"""
    name: str
    description: str
    input_types: List[str]
    output_types: List[str]
    resource_requirements: Dict[str, float]
    priority: int = 0

@dataclass
class AgentRegistration:
    """Registration info for a domain agent"""
    agent_id: str
    agent_type: str
    capabilities: List[AgentCapability]
    status: AgentStatus
    endpoint: str  # A2A endpoint
    mig_partition: int
    last_heartbeat: datetime

@dataclass
class HumanIntent:
    """Parsed human intent from voice/text"""
    raw_input: str
    intent_type: str
    entities: Dict[str, Any]
    confidence: float
    context: Dict[str, Any]

@dataclass
class TaskPlan:
    """Decomposed task plan"""
    plan_id: str
    intent: HumanIntent
    subtasks: List[Dict[str, Any]]
    dependencies: List[tuple]  # (task_a, task_b) means a must complete before b
    parallel_groups: List[List[str]]
    estimated_duration: float

class Cocapn:
    """
    The Cocapn meta-agent
    
    Responsibilities:
    1. Intent recognition from human input
    2. Task decomposition into agent-level operations
    3. Agent selection and routing
    4. Resource allocation (MIG partitions)
    5. Result synthesis and presentation
    6. Learning coordination across agents
    """
    
    def __init__(self, repo_path: str, config: Dict):
        # Git-native memory
        self.repo_path = Path(repo_path)
        self.memory = self._init_memory()
        
        # Agent registry
        self.agents: Dict[str, AgentRegistration] = {}
        
        # Resource management
        self.mig_partitions = {
            0: {"allocated_to": None, "memory_gb": 32},
            1: {"allocated_to": None, "memory_gb": 32},
            2: {"allocated_to": None, "memory_gb": 32},
            3: {"allocated_to": None, "memory_gb": 32},
        }
        
        # Intent classifier
        self.intent_classifier = self._load_intent_classifier()
        
        # Task planner
        self.planner = self._init_planner()
        
        # Active tasks
        self.active_tasks: Dict[str, TaskPlan] = {}
        
        # Configuration
        self.config = config
        
    # ═══════════════════════════════════════════════════════════════
    # AGENT LIFECYCLE MANAGEMENT
    # ═══════════════════════════════════════════════════════════════
    
    def register_agent(self, registration: AgentRegistration) -> bool:
        """
        Register a domain agent with the cocapn
        Allocates resources and establishes A2A connection
        """
        # Check if MIG partition is available
        partition = registration.mig_partition
        if self.mig_partitions[partition]["allocated_to"] is not None:
            return False
            
        # Register agent
        self.agents[registration.agent_id] = registration
        
        # Allocate MIG partition
        self.mig_partitions[partition]["allocated_to"] = registration.agent_id
        
        # Store registration in memory
        self.memory.store_memory(
            content=json.dumps({
                "event": "agent_registered",
                "agent_id": registration.agent_id,
                "capabilities": [c.name for c in registration.capabilities],
                "timestamp": datetime.now().isoformat()
            }),
            metadata={"type": "lifecycle", "agent_id": registration.agent_id}
        )
        
        return True
        
    def unregister_agent(self, agent_id: str) -> bool:
        """Unregister an agent and release resources"""
        if agent_id not in self.agents:
            return False
            
        registration = self.agents[agent_id]
        
        # Release MIG partition
        self.mig_partitions[registration.mig_partition]["allocated_to"] = None
        
        # Remove from registry
        del self.agents[agent_id]
        
        return True
        
    def get_agent_status(self, agent_id: str) -> Optional[AgentStatus]:
        """Get current status of an agent"""
        if agent_id in self.agents:
            return self.agents[agent_id].status
        return None
        
    def update_agent_heartbeat(self, agent_id: str, status: AgentStatus):
        """Update agent heartbeat"""
        if agent_id in self.agents:
            self.agents[agent_id].status = status
            self.agents[agent_id].last_heartbeat = datetime.now()
            
    # ═══════════════════════════════════════════════════════════════
    # INTENT RECOGNITION
    # ═══════════════════════════════════════════════════════════════
    
    def recognize_intent(self, raw_input: str, 
                         context: Dict[str, Any] = None) -> HumanIntent:
        """
        Recognize human intent from voice or text input
        Uses local LLM for understanding
        """
        # Build context
        context = context or {}
        context.update(self._get_vessel_context())
        
        # Classify intent using LLM
        classification = self.intent_classifier.classify(
            input_text=raw_input,
            context=context
        )
        
        intent = HumanIntent(
            raw_input=raw_input,
            intent_type=classification["intent"],
            entities=classification["entities"],
            confidence=classification["confidence"],
            context=context
        )
        
        # Store in memory
        self.memory.store_memory(
            content=json.dumps({
                "raw_input": raw_input,
                "intent": intent.intent_type,
                "entities": intent.entities,
                "timestamp": datetime.now().isoformat()
            }),
            metadata={"type": "intent", "confidence": intent.confidence}
        )
        
        return intent
        
    def _get_vessel_context(self) -> Dict[str, Any]:
        """Get current vessel context for intent understanding"""
        return {
            "location": self._get_current_location(),
            "heading": self._get_current_heading(),
            "speed": self._get_current_speed(),
            "weather": self._get_weather_conditions(),
            "time": datetime.now().isoformat(),
            "active_agents": [a for a, r in self.agents.items() if r.status == AgentStatus.BUSY]
        }
        
    # ═══════════════════════════════════════════════════════════════
    # TASK DECOMPOSITION
    # ═══════════════════════════════════════════════════════════════
    
    def decompose_task(self, intent: HumanIntent) -> TaskPlan:
        """
        Decompose human intent into agent-level tasks
        Creates execution plan with dependencies
        """
        # Use planner to create plan
        plan = self.planner.create_plan(
            intent=intent,
            available_agents=list(self.agents.values()),
            resources=self.mig_partitions
        )
        
        # Store plan in memory
        self.memory.store_memory(
            content=json.dumps({
                "plan_id": plan.plan_id,
                "intent": intent.intent_type,
                "subtasks": plan.subtasks,
                "dependencies": plan.dependencies,
                "timestamp": datetime.now().isoformat()
            }),
            metadata={"type": "task_plan", "plan_id": plan.plan_id}
        )
        
        return plan
        
    # ═══════════════════════════════════════════════════════════════
    # AGENT SELECTION AND ROUTING
    # ═══════════════════════════════════════════════════════════════
    
    def select_agent(self, capability: str) -> Optional[str]:
        """
        Select best agent for a capability
        Considers status, load, and latency
        """
        candidates = []
        
        for agent_id, registration in self.agents.items():
            # Check capability
            has_capability = any(
                c.name == capability for c in registration.capabilities
            )
            
            if has_capability and registration.status == AgentStatus.IDLE:
                candidates.append(agent_id)
                
        if not candidates:
            return None
            
        # Select based on latency (use A2A latency broadcast)
        # For now, select first available
        return candidates[0]
        
    async def route_task(self, agent_id: str, task: Dict) -> Dict:
        """
        Route a task to an agent via A2A protocol
        """
        import aiohttp
        
        registration = self.agents[agent_id]
        
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{registration.endpoint}/a2a/v1/tasks",
                json=task
            ) as response:
                return await response.json()
                
    # ═══════════════════════════════════════════════════════════════
    # RESOURCE ALLOCATION
    # ═══════════════════════════════════════════════════════════════
    
    def allocate_resources(self, task_plan: TaskPlan) -> Dict[str, int]:
        """
        Allocate MIG partitions to tasks in the plan
        Returns mapping of subtask_id -> partition_id
        """
        allocation = {}
        
        for subtask in task_plan.subtasks:
            required_resources = subtask.get("resource_requirements", {})
            
            # Find available partition
            for partition_id, partition in self.mig_partitions.items():
                if partition["allocated_to"] is None:
                    allocation[subtask["id"]] = partition_id
                    partition["allocated_to"] = f"task_{subtask['id']}"
                    break
                    
        return allocation
        
    def release_resources(self, task_id: str):
        """Release resources allocated to a task"""
        for partition_id, partition in self.mig_partitions.items():
            if partition["allocated_to"] == f"task_{task_id}":
                partition["allocated_to"] = None
                
    # ═══════════════════════════════════════════════════════════════
    # EXECUTION COORDINATION
    # ═══════════════════════════════════════════════════════════════
    
    async def execute_plan(self, plan: TaskPlan) -> Dict[str, Any]:
        """
        Execute a task plan with coordination
        Handles dependencies and parallel execution
        """
        results = {}
        completed = set()
        
        # Execute in parallel groups
        for group in plan.parallel_groups:
            # Run group tasks in parallel
            group_tasks = []
            
            for subtask_id in group:
                subtask = next(s for s in plan.subtasks if s["id"] == subtask_id)
                agent_id = self.select_agent(subtask["capability"])
                
                if agent_id:
                    group_tasks.append(
                        self.route_task(agent_id, subtask)
                    )
                    self.agents[agent_id].status = AgentStatus.BUSY
                    
            # Wait for group to complete
            group_results = await asyncio.gather(*group_tasks)
            
            # Store results
            for subtask_id, result in zip(group, group_results):
                results[subtask_id] = result
                completed.add(subtask_id)
                
                # Update agent status
                for agent_id, reg in self.agents.items():
                    if reg.status == AgentStatus.BUSY:
                        reg.status = AgentStatus.IDLE
                        
        return results
        
    # ═══════════════════════════════════════════════════════════════
    # RESULT SYNTHESIS
    # ═══════════════════════════════════════════════════════════════
    
    def synthesize_results(self, plan: TaskPlan, 
                           results: Dict[str, Any]) -> Dict[str, Any]:
        """
        Synthesize results from multiple agents
        Create human-readable summary
        """
        # Use LLM to create summary
        summary_prompt = self._build_summary_prompt(plan, results)
        
        summary = self.intent_classifier.llm.generate(
            prompt=summary_prompt,
            max_tokens=512
        )
        
        synthesis = {
            "plan_id": plan.plan_id,
            "intent": plan.intent.intent_type,
            "summary": summary,
            "detailed_results": results,
            "timestamp": datetime.now().isoformat()
        }
        
        # Store in memory
        self.memory.store_memory(
            content=json.dumps(synthesis),
            metadata={"type": "synthesis", "plan_id": plan.plan_id}
        )
        
        return synthesis
        
    def _build_summary_prompt(self, plan: TaskPlan, 
                               results: Dict[str, Any]) -> str:
        """Build prompt for result synthesis"""
        return f"""
Summarize the results of the following task execution for a human operator:

Original Request: {plan.intent.raw_input}

Task Breakdown:
{json.dumps(plan.subtasks, indent=2)}

Results from Agents:
{json.dumps(results, indent=2)}

Provide a concise, natural language summary that:
1. Confirms what was accomplished
2. Highlights any important findings
3. Notes any issues or warnings
4. Suggests next actions if appropriate

Summary:
"""
        
    # ═══════════════════════════════════════════════════════════════
    # LEARNING COORDINATION
    # ═══════════════════════════════════════════════════════════════
    
    def coordinate_learning(self):
        """
        Coordinate learning across agents
        Sync experiences and trigger improvement cycles
        """
        for agent_id, registration in self.agents.items():
            # Request agent to self-improve if enough experiences
            self._trigger_improvement(agent_id)
            
    def _trigger_improvement(self, agent_id: str):
        """Trigger self-improvement cycle for an agent"""
        # Would send A2A message to agent
        pass
```

---

## Part 2: Software Vessel Integration with Physical Vessel

### 2.1 The Vessel Metaphor

The "software vessel" is the agent software running on a Jetson device. The "physical vessel" is the boat, drone, or robot. The cocapn bridges these two.

```
+------------------------------------------------------------------+
|              SOFTWARE VESSEL + PHYSICAL VESSEL INTEGRATION         |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    SOFTWARE VESSEL                           │ |
|  │                                                              │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                      COCAPN                              ││ |
|  │  │                                                          ││ |
|  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    ││ |
|  │  │  │ Nav     │  │ Species │  │ Safety  │  │ Voice   │    ││ |
|  │  │  │ Agent   │  │ Agent   │  │ Agent   │  │ Agent   │    ││ |
|  │  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘    ││ |
|  │  │       │            │            │            │          ││ |
|  │  └───────┼────────────┼────────────┼────────────┼──────────┘│ |
|  │          │            │            │            │            │ |
|  │          ▼            ▼            ▼            ▼            │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                    HAL LAYER                             ││ |
|  │  │  SensorHAL │ ActuatorHAL │ ComputeHAL                   ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                              │                               │ |
|  └──────────────────────────────┼───────────────────────────────┘ |
|                                 │                                 |
│ ═════════════════════════════════════════════════════════════════│
│                        HARDWARE INTERFACE                        │
│ ═════════════════════════════════════════════════════════════════│
|                                 │                                 |
|                                 ▼                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    PHYSICAL VESSEL                           │ |
|  │                                                              │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                     SENSORS                              ││ |
|  │  │  GPS │ AIS │ Radar │ Cameras │ Sonar │ IMU │ Depth     ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                              │                               │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                    ACTUATORS                             ││ |
|  │  │  Autopilot │ Throttle │ Rudder │ Winches │ Lights      ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                              │                               │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │                     HULL                                 ││ |
|  │  │  Structure │ Storage │ Power │ Crew Quarters            ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                                                            │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.2 Integration Architecture

```python
# vessel_integration.py
"""
Integration between software vessel and physical vessel
"""

from typing import Dict, Any, List, Callable
from dataclasses import dataclass
import asyncio

@dataclass
class SensorReading:
    """A reading from a physical sensor"""
    sensor_id: str
    sensor_type: str
    value: Any
    timestamp: str
    quality: float  # 0-1 confidence in reading

@dataclass
class ActuatorCommand:
    """A command to a physical actuator"""
    actuator_id: str
    actuator_type: str
    command: str
    parameters: Dict[str, Any]
    timestamp: str
    safety_check: bool = True

class VesselBridge:
    """
    Bridge between software vessel (cocapn + agents) and physical vessel
    
    Responsibilities:
    1. Sensor data ingestion and distribution
    2. Actuator command validation and execution
    3. Safety interlocks
    4. State synchronization
    """
    
    def __init__(self, config: Dict):
        # HAL connections
        self.hal = self._init_hal(config)
        
        # Sensor subscribers
        self.sensor_subscribers: Dict[str, List[Callable]] = {}
        
        # Actuator queue
        self.actuator_queue = asyncio.Queue()
        
        # Safety state
        self.safety_state = {
            "emergency_stop": False,
            "manual_override": False,
            "geofence_violation": False
        }
        
        # Vessel state
        self.vessel_state = {}
        
    def _init_hal(self, config: Dict):
        """Initialize Hardware Abstraction Layer"""
        from hal_implementation import HALManager
        return HALManager(config["hal_config"])
        
    async def start(self):
        """Start the vessel bridge"""
        # Start sensor polling
        asyncio.create_task(self._sensor_poll_loop())
        
        # Start actuator processing
        asyncio.create_task(self._actuator_process_loop())
        
        # Start state sync
        asyncio.create_task(self._state_sync_loop())
        
    # ═══════════════════════════════════════════════════════════════
    # SENSOR INGESTION
    # ═══════════════════════════════════════════════════════════════
    
    def subscribe_to_sensor(self, sensor_id: str, callback: Callable):
        """Subscribe to sensor updates"""
        if sensor_id not in self.sensor_subscribers:
            self.sensor_subscribers[sensor_id] = []
        self.sensor_subscribers[sensor_id].append(callback)
        
    async def _sensor_poll_loop(self):
        """Poll sensors and distribute readings"""
        while True:
            # Read all sensors
            readings = await self.hal.read_all_sensors()
            
            # Update vessel state
            for sensor_id, reading in readings.items():
                self.vessel_state[sensor_id] = reading
                
                # Notify subscribers
                if sensor_id in self.sensor_subscribers:
                    for callback in self.sensor_subscribers[sensor_id]:
                        await callback(reading)
                        
            await asyncio.sleep(0.01)  # 100Hz
            
    # ═══════════════════════════════════════════════════════════════
    # ACTUATOR CONTROL
    # ═══════════════════════════════════════════════════════════════
    
    async def send_command(self, command: ActuatorCommand) -> bool:
        """
        Send command to actuator
        Returns True if command was queued
        """
        # Safety check
        if self.safety_state["emergency_stop"]:
            return False
            
        if self.safety_state["manual_override"] and command.safety_check:
            return False
            
        await self.actuator_queue.put(command)
        return True
        
    async def _actuator_process_loop(self):
        """Process actuator commands"""
        while True:
            command = await self.actuator_queue.get()
            
            # Additional safety check
            if not self._is_safe_command(command):
                continue
                
            # Execute via HAL
            success = await self.hal.execute_action(
                command.actuator_id,
                command
            )
            
            # Log command
            self._log_actuator_command(command, success)
            
    def _is_safe_command(self, command: ActuatorCommand) -> bool:
        """Check if command is safe to execute"""
        # Check vessel state
        # Check actuator limits
        # Check for conflicts
        return True
        
    # ═══════════════════════════════════════════════════════════════
    # SAFETY INTERLOCKS
    # ═══════════════════════════════════════════════════════════════
    
    def trigger_emergency_stop(self, reason: str):
        """Trigger emergency stop"""
        self.safety_state["emergency_stop"] = True
        self.safety_state["emergency_reason"] = reason
        
        # Immediately stop all actuators
        # This would call HAL to stop everything
        
    def release_emergency_stop(self):
        """Release emergency stop (requires human confirmation)"""
        self.safety_state["emergency_stop"] = False
        self.safety_state["emergency_reason"] = None
        
    def set_manual_override(self, enabled: bool):
        """Set manual override mode"""
        self.safety_state["manual_override"] = enabled


class PhysicalVesselState:
    """
    Current state of the physical vessel
    Aggregates sensor data into a coherent state
    """
    
    def __init__(self, bridge: VesselBridge):
        self.bridge = bridge
        
        # Navigation state
        self.position = None  # (lat, lon)
        self.heading = None   # degrees
        self.speed = None     # m/s
        
        # Environmental state
        self.weather = None
        self.sea_state = None
        
        # Operational state
        self.fuel_level = None
        self.battery_level = None
        
        # Subscribe to relevant sensors
        bridge.subscribe_to_sensor("gps", self._update_position)
        bridge.subscribe_to_sensor("compass", self._update_heading)
        bridge.subscribe_to_sensor("speed_log", self._update_speed)
        
    def _update_position(self, reading: SensorReading):
        self.position = reading.value
        
    def _update_heading(self, reading: SensorReading):
        self.heading = reading.value
        
    def _update_speed(self, reading: SensorReading):
        self.speed = reading.value
        
    def get_state(self) -> Dict[str, Any]:
        """Get complete vessel state"""
        return {
            "position": self.position,
            "heading": self.heading,
            "speed": self.speed,
            "weather": self.weather,
            "fuel_level": self.fuel_level,
            "battery_level": self.battery_level,
            "timestamp": datetime.now().isoformat()
        }
```

---

## Part 3: Agent Lifecycle Management

### 3.1 Lifecycle States

```
+------------------------------------------------------------------+
|                    AGENT LIFECYCLE STATES                          |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────┐                                                      |
|  │ CREATED │ ◄────────────────────────────────────┐              |
|  └────┬────┘                                      │              |
|       │ register()                                │              |
|       ▼                                           │              |
|  ┌─────────┐                                      │              |
|  │  IDLE   │◄─────────────────────────┐          │              |
|  └────┬────┘                          │          │              |
|       │ assign_task()                 │          │              |
|       ▼                               │          │              |
|  ┌─────────┐     task_complete()      │          │              |
|  │  BUSY   │──────────────────────────┘          │              |
|  └────┬────┘                                     │              |
|       │ error()                                  │              |
|       ▼                                           │              |
|  ┌─────────┐     recover()                        │              |
|  │  ERROR  │──────────────────────────┘          │              |
|  └────┬────┘                                     │              |
|       │ unregister()                             │              |
|       ▼                                           │              |
|  ┌─────────┐                                      │              |
|  │OFFLINE  │──────────────────────────────────────┘              |
|  └─────────┘                                                    │
|                                                                   |
+------------------------------------------------------------------+
```

### 3.2 Lifecycle Manager Implementation

```python
# agent_lifecycle.py
"""
Agent lifecycle management
"""

from typing import Dict, Any, Optional
from dataclasses import dataclass
from enum import Enum
from datetime import datetime, timedelta
import asyncio

class LifecycleState(Enum):
    CREATED = "created"
    IDLE = "idle"
    BUSY = "busy"
    ERROR = "error"
    OFFLINE = "offline"

@dataclass
class AgentInstance:
    """Instance of a managed agent"""
    agent_id: str
    agent_type: str
    state: LifecycleState
    created_at: datetime
    last_activity: datetime
    error_count: int
    current_task: Optional[str]
    health_score: float  # 0-1

class AgentLifecycleManager:
    """
    Manages agent lifecycle within the cocapn
    
    Responsibilities:
    1. Agent instantiation and destruction
    2. Health monitoring
    3. Error recovery
    4. Resource cleanup
    """
    
    def __init__(self, cocapn):
        self.cocapn = cocapn
        self.instances: Dict[str, AgentInstance] = {}
        self.health_check_interval = 30  # seconds
        self.max_errors_before_restart = 3
        
    async def start(self):
        """Start lifecycle manager"""
        asyncio.create_task(self._health_monitor_loop())
        
    # ═══════════════════════════════════════════════════════════════
    # INSTANTIATION
    # ═══════════════════════════════════════════════════════════════
    
    async def instantiate_agent(self, agent_type: str, 
                                 config: Dict[str, Any]) -> str:
        """
        Create a new agent instance
        Returns agent_id
        """
        # Generate ID
        agent_id = f"{agent_type}-{datetime.now().strftime('%Y%m%d%H%M%S')}"
        
        # Create instance record
        instance = AgentInstance(
            agent_id=agent_id,
            agent_type=agent_type,
            state=LifecycleState.CREATED,
            created_at=datetime.now(),
            last_activity=datetime.now(),
            error_count=0,
            current_task=None,
            health_score=1.0
        )
        
        # Register with cocapn
        registration = AgentRegistration(
            agent_id=agent_id,
            agent_type=agent_type,
            capabilities=self._get_capabilities(agent_type),
            status=AgentStatus.IDLE,
            endpoint=f"http://{agent_id}:8080",
            mig_partition=self._allocate_partition()
        )
        
        success = self.cocapn.register_agent(registration)
        
        if success:
            instance.state = LifecycleState.IDLE
            self.instances[agent_id] = instance
            return agent_id
        else:
            raise RuntimeError(f"Failed to register agent {agent_id}")
            
    # ═══════════════════════════════════════════════════════════════
    # TASK ASSIGNMENT
    # ═══════════════════════════════════════════════════════════════
    
    async def assign_task(self, agent_id: str, task: Dict) -> bool:
        """Assign a task to an agent"""
        if agent_id not in self.instances:
            return False
            
        instance = self.instances[agent_id]
        
        if instance.state != LifecycleState.IDLE:
            return False
            
        instance.state = LifecycleState.BUSY
        instance.current_task = task.get("task_id")
        instance.last_activity = datetime.now()
        
        # Update cocapn registry
        self.cocapn.update_agent_heartbeat(agent_id, AgentStatus.BUSY)
        
        return True
        
    async def complete_task(self, agent_id: str, result: Dict):
        """Mark task as complete"""
        if agent_id not in self.instances:
            return
            
        instance = self.instances[agent_id]
        instance.state = LifecycleState.IDLE
        instance.current_task = None
        instance.last_activity = datetime.now()
        
        # Update cocapn registry
        self.cocapn.update_agent_heartbeat(agent_id, AgentStatus.IDLE)
        
    # ═══════════════════════════════════════════════════════════════
    # ERROR HANDLING
    # ═══════════════════════════════════════════════════════════════
    
    async def report_error(self, agent_id: str, error: Dict):
        """Report an error from an agent"""
        if agent_id not in self.instances:
            return
            
        instance = self.instances[agent_id]
        instance.error_count += 1
        instance.health_score = max(0, instance.health_score - 0.1)
        
        if instance.error_count >= self.max_errors_before_restart:
            # Attempt recovery
            await self._recover_agent(agent_id)
        else:
            # Log error and continue
            instance.state = LifecycleState.ERROR
            
    async def _recover_agent(self, agent_id: str):
        """Attempt to recover a failing agent"""
        instance = self.instances[agent_id]
        old_state = instance.state
        
        # Unregister from cocapn
        self.cocapn.unregister_agent(agent_id)
        
        # Reset instance
        instance.state = LifecycleState.OFFLINE
        instance.error_count = 0
        instance.current_task = None
        
        # Re-instantiate
        config = self._get_agent_config(instance.agent_type)
        await self.instantiate_agent(instance.agent_type, config)
        
    # ═══════════════════════════════════════════════════════════════
    # HEALTH MONITORING
    # ═══════════════════════════════════════════════════════════════
    
    async def _health_monitor_loop(self):
        """Monitor agent health"""
        while True:
            for agent_id, instance in list(self.instances.items()):
                # Check heartbeat
                time_since_activity = (datetime.now() - instance.last_activity).total_seconds()
                
                if time_since_activity > 60:  # 1 minute timeout
                    # Agent might be hung
                    instance.health_score -= 0.1
                    
                    if instance.health_score < 0.5:
                        await self._recover_agent(agent_id)
                        
            await asyncio.sleep(self.health_check_interval)
            
    # ═══════════════════════════════════════════════════════════════
    # CLEANUP
    # ═══════════════════════════════════════════════════════════════
    
    async def destroy_agent(self, agent_id: str):
        """Destroy an agent instance"""
        if agent_id not in self.instances:
            return
            
        instance = self.instances[agent_id]
        
        # Wait for current task to complete
        while instance.state == LifecycleState.BUSY:
            await asyncio.sleep(1)
            
        # Unregister
        self.cocapn.unregister_agent(agent_id)
        
        # Remove instance
        del self.instances[agent_id]
```

---

## Part 4: Resource Allocation and Isolation

### 4.1 MIG-Based Resource Isolation

```
+------------------------------------------------------------------+
|                    MIG RESOURCE ALLOCATION                         |
+------------------------------------------------------------------+
|                                                                   |
|  Jetson Thor GPU (128 GB, 2070 TFLOPS)                            |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                              │ |
|  │  Partition 0 (32 GB)         Partition 1 (32 GB)            │ |
|  │  ┌─────────────────────┐     ┌─────────────────────┐        │ |
|  │  │                     │     │                     │        │ |
|  │  │   COCAPN            │     │   NAVIGATION        │        │ |
|  │  │   Meta-Agent        │     │   AGENT             │        │ |
|  │  │                     │     │                     │        │ |
|  │  │   • LLM Inference   │     │   • Path Planning   │        │ |
|  │  │   • Intent Recog    │     │   • SLAM            │        │ |
|  │  │   • Coordination    │     │   • Route Opt       │        │ |
|  │  │                     │     │                     │        │ |
|  │  └─────────────────────┘     └─────────────────────┘        │ |
|  │                                                              │ |
|  │  Partition 2 (32 GB)         Partition 3 (32 GB)            │ |
|  │  ┌─────────────────────┐     ┌─────────────────────┐        │ |
|  │  │                     │     │                     │        │ |
|  │  │   PERCEPTION        │     │   TRAINING          │        │ |
|  │  │   AGENT             │     │   PIPELINE          │        │ |
|  │  │                     │     │                     │        │ |
|  │  │   • Vision Pipeline │     │   • LoRA Fine-tune  │        │ |
|  │  │   • Species ID      │     │   • Experience     │        │ |
|  │  │   • Voice STT/TTS   │     │     Replay         │        │ |
|  │  │                     │     │                     │        │ |
|  │  └─────────────────────┘     └─────────────────────┘        │ |
|  │                                                              │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  Benefits of MIG Partitioning:                                    |
|  ✓ Complete isolation - no noisy neighbors                       |
|  ✓ Independent scaling per agent                                  |
|  ✓ Fault containment - one agent crash doesn't affect others     |
|  ✓ Memory protection - each agent has dedicated memory           |
|  ✓ Performance guarantee - compute is guaranteed per partition   |
|                                                                   |
+------------------------------------------------------------------+
```

### 4.2 Resource Manager Implementation

```python
# resource_manager.py
"""
Resource allocation and isolation for agents
Uses MIG for GPU partitioning
"""

from typing import Dict, Any, Optional, List
from dataclasses import dataclass
from enum import Enum
import subprocess

class ResourceType(Enum):
    GPU_MEMORY = "gpu_memory"
    GPU_COMPUTE = "gpu_compute"
    CPU_CORES = "cpu_cores"
    MEMORY = "memory"
    STORAGE = "storage"

@dataclass
class ResourceAllocation:
    """Resource allocation for an agent"""
    agent_id: str
    mig_partition: int
    gpu_memory_gb: float
    gpu_compute_fraction: float
    cpu_cores: List[int]
    memory_gb: float
    storage_gb: float

@dataclass
class ResourceRequest:
    """Request for resources"""
    agent_id: str
    gpu_memory_gb: float
    gpu_compute_fraction: float
    cpu_cores: int
    memory_gb: float
    storage_gb: float
    priority: int  # Higher = more important

class ResourceManager:
    """
    Manages resource allocation for agents
    
    Uses:
    - MIG for GPU isolation
    - cgroups for CPU/memory isolation
    - Docker for container isolation
    """
    
    def __init__(self, config: Dict):
        self.config = config
        
        # MIG partitions
        self.mig_partitions: Dict[int, ResourceAllocation] = {}
        self._init_mig_partitions()
        
        # CPU cores
        self.cpu_cores = list(range(config.get("cpu_cores", 12)))
        self.available_cpu_cores = self.cpu_cores.copy()
        
        # Memory
        self.total_memory_gb = config.get("memory_gb", 128)
        self.allocated_memory_gb = 0
        
    def _init_mig_partitions(self):
        """Initialize MIG partition configuration"""
        # Partition profile: 4x balanced partitions
        partition_config = [
            {"id": 0, "memory_gb": 32, "compute_fraction": 0.25},
            {"id": 1, "memory_gb": 32, "compute_fraction": 0.25},
            {"id": 2, "memory_gb": 32, "compute_fraction": 0.25},
            {"id": 3, "memory_gb": 32, "compute_fraction": 0.25},
        ]
        
        for p in partition_config:
            self.mig_partitions[p["id"]] = None  # Not allocated
            
    # ═══════════════════════════════════════════════════════════════
    # ALLOCATION
    # ═══════════════════════════════════════════════════════════════
    
    def allocate(self, request: ResourceRequest) -> Optional[ResourceAllocation]:
        """
        Allocate resources for an agent
        Returns allocation if successful, None if insufficient resources
        """
        # Find suitable MIG partition
        partition = self._find_mig_partition(request)
        if partition is None:
            return None
            
        # Allocate CPU cores
        cpu_cores = self._allocate_cpu_cores(request.cpu_cores)
        if cpu_cores is None:
            return None
            
        # Allocate memory
        if self.allocated_memory_gb + request.memory_gb > self.total_memory_gb:
            return None
            
        # Create allocation
        allocation = ResourceAllocation(
            agent_id=request.agent_id,
            mig_partition=partition,
            gpu_memory_gb=request.gpu_memory_gb,
            gpu_compute_fraction=request.gpu_compute_fraction,
            cpu_cores=cpu_cores,
            memory_gb=request.memory_gb,
            storage_gb=request.storage_gb
        )
        
        # Mark as allocated
        self.mig_partitions[partition] = allocation
        for core in cpu_cores:
            self.available_cpu_cores.remove(core)
        self.allocated_memory_gb += request.memory_gb
        
        # Apply cgroup configuration
        self._apply_cgroup(allocation)
        
        return allocation
        
    def deallocate(self, agent_id: str):
        """Deallocate resources for an agent"""
        # Find allocation
        allocation = None
        for partition_id, alloc in self.mig_partitions.items():
            if alloc and alloc.agent_id == agent_id:
                allocation = alloc
                partition_id = partition_id
                break
                
        if allocation is None:
            return
            
        # Release MIG partition
        self.mig_partitions[partition_id] = None
        
        # Release CPU cores
        for core in allocation.cpu_cores:
            self.available_cpu_cores.append(core)
            
        # Release memory
        self.allocated_memory_gb -= allocation.memory_gb
        
        # Remove cgroup
        self._remove_cgroup(allocation)
        
    # ═══════════════════════════════════════════════════════════════
    # PARTITION SELECTION
    # ═══════════════════════════════════════════════════════════════
    
    def _find_mig_partition(self, request: ResourceRequest) -> Optional[int]:
        """Find suitable MIG partition"""
        for partition_id, allocation in self.mig_partitions.items():
            if allocation is None:
                # Check if partition meets requirements
                partition_memory = 32  # GB per partition
                partition_compute = 0.25  # Fraction
                
                if (request.gpu_memory_gb <= partition_memory and
                    request.gpu_compute_fraction <= partition_compute):
                    return partition_id
                    
        return None
        
    def _allocate_cpu_cores(self, count: int) -> Optional[List[int]]:
        """Allocate CPU cores"""
        if len(self.available_cpu_cores) < count:
            return None
        return self.available_cpu_cores[:count]
        
    # ═══════════════════════════════════════════════════════════════
    # CGROUP MANAGEMENT
    # ═══════════════════════════════════════════════════════════════
    
    def _apply_cgroup(self, allocation: ResourceAllocation):
        """Apply cgroup configuration for isolation"""
        cgroup_path = f"/sys/fs/cgroup/agent_{allocation.agent_id}"
        
        # Create cgroup
        subprocess.run(["mkdir", "-p", cgroup_path], check=True)
        
        # Set CPU cores
        core_list = ",".join(str(c) for c in allocation.cpu_cores)
        subprocess.run([
            "echo", core_list, ">", f"{cgroup_path}/cpuset.cpus"
        ], shell=True)
        
        # Set memory limit
        memory_bytes = int(allocation.memory_gb * 1024 * 1024 * 1024)
        subprocess.run([
            "echo", str(memory_bytes), ">", f"{cgroup_path}/memory.max"
        ], shell=True)
        
    def _remove_cgroup(self, allocation: ResourceAllocation):
        """Remove cgroup"""
        cgroup_path = f"/sys/fs/cgroup/agent_{allocation.agent_id}"
        subprocess.run(["rmdir", cgroup_path])
        
    # ═══════════════════════════════════════════════════════════════
    # MONITORING
    # ═══════════════════════════════════════════════════════════════
    
    def get_utilization(self) -> Dict[str, Any]:
        """Get current resource utilization"""
        import pynvml
        
        pynvml.nvmlInit()
        handle = pynvml.nvmlDeviceGetHandleByIndex(0)
        
        # GPU utilization per partition
        partition_util = {}
        for partition_id, allocation in self.mig_partitions.items():
            if allocation:
                # Would query MIG-specific stats
                partition_util[partition_id] = {
                    "agent_id": allocation.agent_id,
                    "status": "allocated"
                }
            else:
                partition_util[partition_id] = {
                    "agent_id": None,
                    "status": "available"
                }
                
        return {
            "mig_partitions": partition_util,
            "cpu_cores_available": len(self.available_cpu_cores),
            "cpu_cores_total": len(self.cpu_cores),
            "memory_allocated_gb": self.allocated_memory_gb,
            "memory_total_gb": self.total_memory_gb
        }
```

---

## Part 5: Human-Cocapn Interaction

### 5.1 Interaction Modes

```
+------------------------------------------------------------------+
|                    HUMAN-COCAPN INTERACTION                        |
+------------------------------------------------------------------+
|                                                                   |
|  Mode 1: VOICE (Primary for field operations)                    |
|  ─────────────────────────────────────────                       |
|  Human: "What's the best route to the fishing grounds?"          |
|  Cocapn: "Based on current weather and known fish patterns,       |
|           I recommend heading 45° NE for 12 nautical miles.       |
|           ETA 2 hours 15 minutes. Should I set the course?"       |
|                                                                   |
|  Mode 2: DASHBOARD (Secondary for planning/review)               |
|  ─────────────────────────────────────────                       |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │ [Map View] │ [Status] │ [Portfolio] │ [Settings]           │ |
|  │             │                                       │        │ |
|  │   ┌─────┐   │  Navigation Agent: Active            │        │ |
|  │   │ 🚢  │   │  Species Agent: Ready                │        │ |
|  │   │     │   │  Safety Agent: Monitoring            │        │ |
|  │   └─────┘   │                                       │        │ |
|  │             │  [Voice] [Text] [Manual]             │        │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  Mode 3: TEXT (Alternative for noisy environments)               |
|  ─────────────────────────────────────────                       |
|  Human: > identify fish on deck camera 2                         |
|  Cocapn: Identified: Atlantic Cod (Gadus morhua)                 |
|          Confidence: 94%                                          |
|          Estimated weight: 8.5 kg                                 |
|          Regulations: Legal size, season open                     |
|          Log this catch? [Y/n]                                    |
|                                                                   |
+------------------------------------------------------------------+
```

### 5.2 Interaction Handler

```python
# interaction_handler.py
"""
Human-cocapn interaction handler
"""

from typing import Dict, Any, Optional
from dataclasses import dataclass
from enum import Enum
import asyncio

class InteractionMode(Enum):
    VOICE = "voice"
    TEXT = "text"
    DASHBOARD = "dashboard"

@dataclass
class Interaction:
    """A single human-cocapn interaction"""
    interaction_id: str
    mode: InteractionMode
    input: str
    intent: Optional[HumanIntent] = None
    response: Optional[str] = None
    actions_taken: list = None
    timestamp: str = None

class InteractionHandler:
    """
    Handles human-cocapn interactions
    
    Responsibilities:
    1. Voice interface integration
    2. Text command parsing
    3. Dashboard state management
    4. Response generation
    """
    
    def __init__(self, cocapn):
        self.cocapn = cocapn
        self.voice_engine = self._init_voice_engine()
        self.dashboard_state = {}
        self.interaction_history = []
        
    async def handle_voice_input(self, audio_data: bytes) -> str:
        """Handle voice input from human"""
        # Transcribe
        text = await self.voice_engine.transcribe(audio_data)
        
        # Process as text
        response = await self.handle_text_input(text, InteractionMode.VOICE)
        
        # Synthesize response
        audio_response = await self.voice_engine.synthesize(response)
        
        return audio_response
        
    async def handle_text_input(self, text: str, 
                                 mode: InteractionMode = InteractionMode.TEXT) -> str:
        """Handle text input from human"""
        # Recognize intent
        intent = self.cocapn.recognize_intent(text)
        
        # Create interaction record
        interaction = Interaction(
            interaction_id=self._generate_id(),
            mode=mode,
            input=text,
            intent=intent,
            timestamp=datetime.now().isoformat()
        )
        
        # Determine response based on intent
        if intent.intent_type == "query":
            response = await self._handle_query(intent)
        elif intent.intent_type == "command":
            response = await self._handle_command(intent)
        elif intent.intent_type == "clarification":
            response = await self._handle_clarification(intent)
        else:
            response = await self._handle_unknown(intent)
            
        interaction.response = response
        self.interaction_history.append(interaction)
        
        return response
        
    async def _handle_query(self, intent: HumanIntent) -> str:
        """Handle a query intent"""
        # Query relevant agents
        # Synthesize response
        
        if "route" in intent.entities:
            # Ask navigation agent
            nav_agent = self.cocapn.select_agent("navigation")
            result = await self.cocapn.route_task(nav_agent, {
                "type": "query_route",
                "destination": intent.entities.get("destination")
            })
            return self._format_route_response(result)
            
        elif "species" in intent.entities:
            # Ask species agent
            species_agent = self.cocapn.select_agent("species_identification")
            result = await self.cocapn.route_task(species_agent, {
                "type": "identify",
                "source": intent.entities.get("source", "deck_camera")
            })
            return self._format_species_response(result)
            
        # Default response
        return "I'm not sure how to answer that. Could you clarify?"
        
    async def _handle_command(self, intent: HumanIntent) -> str:
        """Handle a command intent"""
        # Decompose task
        plan = self.cocapn.decompose_task(intent)
        
        # Confirm with human
        confirmation = self._generate_confirmation(plan)
        
        # Execute plan
        results = await self.cocapn.execute_plan(plan)
        
        # Synthesize results
        synthesis = self.cocapn.synthesize_results(plan, results)
        
        return synthesis["summary"]
        
    def _format_route_response(self, result: Dict) -> str:
        """Format route information as human-readable response"""
        return f"""Based on current conditions, I recommend:
- Heading: {result['heading']}°
- Distance: {result['distance_nm']} nautical miles
- ETA: {result['eta']}
- Fuel required: {result['fuel_required']} gallons

Should I set this course?"""
```

---

## Summary

### Cocapn Responsibilities Matrix

| Responsibility | Implementation | Priority |
|----------------|----------------|----------|
| Intent Recognition | LLM-based classifier | Critical |
| Task Decomposition | Planning module | Critical |
| Agent Selection | Capability matching | High |
| Resource Allocation | MIG partitioning | Critical |
| Lifecycle Management | State machine | High |
| Learning Coordination | Git-native memory | Medium |
| Human Interface | Voice/Text/Dashboard | Critical |

### Key Integration Points

1. **Cocapn ↔ Agents**: A2A Protocol for all agent communication
2. **Cocapn ↔ Physical Vessel**: HAL layer for sensor/actuator access
3. **Cocapn ↔ Human**: Voice, text, and dashboard interfaces
4. **Cocapn ↔ Git**: Memory and learning via git history

### Next Steps

- **Iteration 4**: Fleet Coordination (nexus-runtime deep dive)
- **Iteration 5**: Cross-Domain Learning Patterns
- **Iteration 6**: Portfolio System Architecture

---

*Document prepared for Jetson Robotics Ecosystem Grand Design*  
*Iteration 03 - Cocapn Software Vessel*  
*Date: January 2025*
