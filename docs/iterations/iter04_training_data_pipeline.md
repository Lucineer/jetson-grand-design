# Iteration 04: Training Data Pipeline Architecture

## Experience Capture, Curation, and Fleet-Level Learning for Jetson Robotics

**Document Version**: 1.0  
**Phase**: Iteration 4 of 6  
**Date**: January 2025  
**Status**: Implementation Specification

---

## Executive Summary

This document specifies the complete training data pipeline for the Jetson Robotics Ecosystem, enabling self-improving agents that learn from every interaction. The pipeline captures experiences from individual vessels, curates quality data locally on Jetson Thor devices, aggregates insights across fleets, and incorporates human feedback for continuous improvement.

**Core Principle**: Every interaction is a learning opportunity. Git commits become training data. The repository IS the training dataset.

---

## Table of Contents

1. [Experience Capture Mechanisms](#1-experience-capture-mechanisms)
2. [Curation and Quality Filtering](#2-curation-and-quality-filtering)
3. [Local Fine-Tuning on Jetson Thor](#3-local-fine-tuning-on-jetson-thor)
4. [Fleet-Level Aggregation](#4-fleet-level-aggregation)
5. [Human-in-the-Loop Feedback](#5-human-in-the-loop-feedback)
6. [Implementation Specifications](#6-implementation-specifications)
7. [Integration with Git-Native Architecture](#7-integration-with-git-native-architecture)

---

## 1. Experience Capture Mechanisms

### 1.1 The Experience Primitive

Every interaction between agent and environment is captured as an **Experience Primitive** - the atomic unit of learning data.

```
+------------------------------------------------------------------+
|                    EXPERIENCE PRIMITIVE                           |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  experience_id: string (UUID)                               │ |
|  │  timestamp: ISO 8601                                        │ |
|  │  vessel_id: string                                          │ |
|  │  agent_version: semver                                      │ |
|  │                                                             │ |
|  │  CONTEXT                                                    │ |
|  │  ├── environment_state: dict                                │ |
|  │  ├── task_specification: dict                               │ |
|  │  ├── constraints: dict                                      │ |
|  │  └── human_intent: string (optional)                        │ |
|  │                                                             │ |
|  │  OBSERVATIONS                                               │ |
|  │  ├── sensor_data: tensor[]                                  │ |
|  │  ├── perception_outputs: dict                               │ |
|  │  └── internal_state: dict                                   │ |
|  │                                                             │ |
|  │  ACTIONS                                                    │ |
|  │  ├── action_sequence: list[Action]                          │ |
|  │  ├── action_metadata: dict                                  │ |
|  │  └── human_overrides: list (optional)                       │ |
|  │                                                             │ |
|  │  OUTCOMES                                                   │ |
|  │  ├── success: boolean                                       │ |
|  │  ├── reward_signal: float                                   │ |
|  │  ├── human_feedback: enum (optional)                        │ |
|  │  └── unexpected_events: list                                │ |
|  │                                                             │ |
|  │  LEARNING_SIGNALS                                           │ |
|  │  ├── confidence_before: float                               │ |
|  │  ├── confidence_after: float                                │ |
|  │  ├── surprise_metric: float                                 │ |
|  │  └── improvement_opportunity: float                         │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 Capture Points

```
+------------------------------------------------------------------+
|                    CAPTURE ARCHITECTURE                           |
+------------------------------------------------------------------+
|                                                                   |
|                     JETSON THOR EDGE                              |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │   SENSORS              AGENT              ACTUATORS         │ |
|  │  ┌─────────┐         ┌─────────┐        ┌─────────┐        │ |
|  │  │ Camera  │────────►│ Vision  │───────►│ Motor   │        │ |
|  │  │ Lidar   │         │ Pipeline│        │ Control │        │ |
|  │  │ GPS     │────────►│         │◄───────│         │        │ |
|  │  │ IMU     │         │         │        │         │        │ |
|  │  │ Audio   │────────►│   LLM   │◄───────│ Voice   │        │ |
|  │  │ Temp    │         │ Engine  │        │ Output  │        │ |
|  │  └─────────┘         └─────────┘        └─────────┘        │ |
|  │        │                   │                   │            │ |
|  │        └───────────────────┼───────────────────┘            │ |
|  │                            ▼                                │ |
|  │                   ┌─────────────────┐                       │ |
|  │                   │ EXPERIENCE      │                       │ |
|  │                   │ CAPTURE ENGINE  │                       │ |
|  │                   │                 │                       │ |
|  │                   │ - Buffer        │                       │ |
|  │                   │ - Preprocess    │                       │ |
|  │                   │ - Annotate      │                       │ |
|  │                   │ - Queue         │                       │ |
|  │                   └─────────────────┘                       │ |
|  │                            │                                │ |
|  │                            ▼                                │ |
|  │                   ┌─────────────────┐                       │ |
|  │                   │ GIT COMMIT      │                       │ |
|  │                   │ (Experience     │                       │ |
|  │                   │  Persistence)   │                       │ |
|  │                   └─────────────────┘                       │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.3 Capture Implementation

```python
# experience_capture.py
import torch
import json
import uuid
from datetime import datetime
from dataclasses import dataclass, field, asdict
from typing import List, Dict, Any, Optional, Tuple
from pathlib import Path
import git
import numpy as np

@dataclass
class Action:
    """Single action in a sequence"""
    action_type: str
    parameters: Dict[str, Any]
    timestamp: str
    confidence: float
    human_override: bool = False

@dataclass  
class Experience:
    """Complete experience record for learning"""
    experience_id: str
    timestamp: str
    vessel_id: str
    agent_version: str
    
    # Context
    environment_state: Dict[str, Any]
    task_specification: Dict[str, Any]
    constraints: Dict[str, Any]
    human_intent: Optional[str]
    
    # Observations
    sensor_data_refs: List[str]  # References to stored tensors
    perception_outputs: Dict[str, Any]
    internal_state: Dict[str, Any]
    
    # Actions
    action_sequence: List[Dict[str, Any]]
    action_metadata: Dict[str, Any]
    human_overrides: List[Dict[str, Any]]
    
    # Outcomes
    success: bool
    reward_signal: float
    human_feedback: Optional[str]  # 'positive', 'negative', 'corrective', None
    unexpected_events: List[Dict[str, Any]]
    
    # Learning Signals
    confidence_before: float
    confidence_after: float
    surprise_metric: float
    improvement_opportunity: float
    
    # Computed after capture
    quality_score: float = 0.0
    transferable: bool = False
    domains: List[str] = field(default_factory=list)


class ExperienceCaptureEngine:
    """
    Captures experiences from agent interactions.
    Optimized for Jetson Thor edge deployment.
    """
    
    def __init__(self, repo_path: str, vessel_id: str):
        self.repo = git.Repo(repo_path)
        self.vessel_id = vessel_id
        self.experience_dir = Path(repo_path) / "experiences"
        self.experience_dir.mkdir(exist_ok=True)
        
        # Ring buffer for active experiences (in-memory)
        self.buffer_size = 1000
        self.experience_buffer: List[Experience] = []
        
        # Tensor storage for sensor data
        self.tensor_dir = Path(repo_path) / "tensors"
        self.tensor_dir.mkdir(exist_ok=True)
        
        # Capture statistics
        self.capture_count = 0
        self.last_capture_time = None
        
    def start_capture(self, context: Dict[str, Any]) -> str:
        """Initialize a new experience capture"""
        experience_id = str(uuid.uuid4())
        
        # Store initial context
        experience = Experience(
            experience_id=experience_id,
            timestamp=datetime.now().isoformat(),
            vessel_id=self.vessel_id,
            agent_version=self._get_agent_version(),
            environment_state=context.get('environment', {}),
            task_specification=context.get('task', {}),
            constraints=context.get('constraints', {}),
            human_intent=context.get('human_intent'),
            sensor_data_refs=[],
            perception_outputs={},
            internal_state={},
            action_sequence=[],
            action_metadata={},
            human_overrides=[],
            success=False,
            reward_signal=0.0,
            human_feedback=None,
            unexpected_events=[],
            confidence_before=context.get('confidence', 0.5),
            confidence_after=0.0,
            surprise_metric=0.0,
            improvement_opportunity=0.0
        )
        
        self.experience_buffer.append(experience)
        return experience_id
    
    def capture_observation(self, experience_id: str, 
                           sensor_data: Dict[str, torch.Tensor],
                           perception_outputs: Dict[str, Any]):
        """Capture sensor observations and perception outputs"""
        experience = self._get_experience(experience_id)
        if not experience:
            return
            
        # Store tensors efficiently (FP4 on Thor)
        for sensor_name, tensor in sensor_data.items():
            tensor_ref = self._store_tensor(experience_id, sensor_name, tensor)
            experience.sensor_data_refs.append(tensor_ref)
            
        # Store perception outputs (JSON-serializable)
        experience.perception_outputs.update(perception_outputs)
        
    def capture_action(self, experience_id: str, action: Action):
        """Capture a single action execution"""
        experience = self._get_experience(experience_id)
        if not experience:
            return
            
        action_dict = {
            'action_type': action.action_type,
            'parameters': action.parameters,
            'timestamp': action.timestamp,
            'confidence': action.confidence,
            'human_override': action.human_override
        }
        experience.action_sequence.append(action_dict)
        
        if action.human_override:
            experience.human_overrides.append(action_dict)
    
    def finalize_capture(self, experience_id: str, 
                        outcome: Dict[str, Any]) -> Experience:
        """Finalize experience capture and commit to git"""
        experience = self._get_experience(experience_id)
        if not experience:
            raise ValueError(f"Experience {experience_id} not found")
            
        # Record outcome
        experience.success = outcome.get('success', False)
        experience.reward_signal = outcome.get('reward', 0.0)
        experience.human_feedback = outcome.get('human_feedback')
        experience.unexpected_events = outcome.get('unexpected_events', [])
        
        # Compute learning signals
        experience.confidence_after = outcome.get('confidence', 0.5)
        experience.surprise_metric = abs(
            experience.confidence_after - experience.confidence_before
        )
        experience.improvement_opportunity = self._compute_improvement_opportunity(
            experience
        )
        
        # Compute quality score
        experience.quality_score = self._compute_quality_score(experience)
        
        # Determine transferability
        experience.transferable = self._is_transferable(experience)
        experience.domains = self._infer_domains(experience)
        
        # Commit to git
        self._commit_experience(experience)
        
        self.capture_count += 1
        self.last_capture_time = datetime.now()
        
        return experience
    
    def _store_tensor(self, experience_id: str, 
                     sensor_name: str, 
                     tensor: torch.Tensor) -> str:
        """Store tensor efficiently for later retrieval"""
        # Use FP4 quantization on Thor for storage efficiency
        if tensor.dtype == torch.float32 or tensor.dtype == torch.float16:
            tensor = tensor.to('cuda')  # Move to GPU for quantization
            # Thor-native FP4 storage
            tensor_fp4 = self._quantize_fp4(tensor)
            
        tensor_path = self.tensor_dir / f"{experience_id[:8]}_{sensor_name}.pt"
        torch.save(tensor_fp4, tensor_path)
        
        return str(tensor_path.relative_to(self.repo.working_dir))
    
    def _quantize_fp4(self, tensor: torch.Tensor) -> torch.Tensor:
        """Quantize tensor to FP4 for storage efficiency"""
        # Thor has native FP4 support
        # This is a simplified implementation
        return tensor.half()  # Placeholder - actual FP4 on Thor hardware
    
    def _compute_quality_score(self, experience: Experience) -> float:
        """
        Compute quality score for curation.
        Higher scores = more valuable for training.
        """
        score = 0.0
        
        # Success bonus
        if experience.success:
            score += 0.3
            
        # Human feedback bonus
        if experience.human_feedback == 'positive':
            score += 0.2
        elif experience.human_feedback == 'corrective':
            score += 0.15  # Corrections are valuable
            
        # Surprise bonus (novel situations)
        score += experience.surprise_metric * 0.2
        
        # Improvement opportunity bonus
        score += experience.improvement_opportunity * 0.15
        
        # Action complexity bonus (richer experiences)
        action_complexity = len(experience.action_sequence) / 20.0
        score += min(action_complexity, 0.15)
        
        return min(score, 1.0)
    
    def _compute_improvement_opportunity(self, experience: Experience) -> float:
        """Compute how much this experience could improve the agent"""
        if experience.success:
            return 0.0  # No improvement needed
            
        # Failed task = improvement opportunity
        base = 0.5
        
        # More override = more learning needed
        override_factor = len(experience.human_overrides) * 0.1
        
        # Unexpected events = edge cases to learn
        unexpected_factor = len(experience.unexpected_events) * 0.05
        
        return min(base + override_factor + unexpected_factor, 1.0)
    
    def _is_transferable(self, experience: Experience) -> bool:
        """Determine if experience is transferable to other domains"""
        # Experiences with human feedback are more transferable
        if experience.human_feedback:
            return True
            
        # High surprise = novel pattern worth transferring
        if experience.surprise_metric > 0.3:
            return True
            
        # Quality threshold
        if experience.quality_score > 0.7:
            return True
            
        return False
    
    def _infer_domains(self, experience: Experience) -> List[str]:
        """Infer which domains this experience applies to"""
        domains = ['maritime']  # Default domain
        
        # Check action types for domain hints
        action_types = [a['action_type'] for a in experience.action_sequence]
        
        if 'navigate' in action_types or 'path_plan' in action_types:
            domains.extend(['warehouse', 'aerial', 'ground'])
            
        if 'grip' in action_types or 'manipulate' in action_types:
            domains.extend(['industrial', 'medical'])
            
        if 'voice_command' in action_types or 'speak' in action_types:
            domains.extend(['home', 'commercial'])
            
        return list(set(domains))
    
    def _commit_experience(self, experience: Experience):
        """Commit experience to git repository"""
        # Write experience to file
        exp_file = self.experience_dir / f"{experience.experience_id[:8]}.json"
        exp_file.write_text(json.dumps(asdict(experience), indent=2))
        
        # Git add and commit
        self.repo.index.add([str(exp_file)])
        
        commit_message = f"""experience: {experience.experience_id[:8]}

Task: {experience.task_specification.get('type', 'unknown')}
Success: {experience.success}
Quality: {experience.quality_score:.2f}
Transferable: {experience.transferable}
Domains: {', '.join(experience.domains)}
Human Feedback: {experience.human_feedback or 'none'}
"""
        
        self.repo.index.commit(commit_message)
    
    def _get_experience(self, experience_id: str) -> Optional[Experience]:
        """Get experience from buffer by ID"""
        for exp in self.experience_buffer:
            if exp.experience_id == experience_id:
                return exp
        return None
    
    def _get_agent_version(self) -> str:
        """Get current agent version from git"""
        try:
            return self.repo.tags[-1].name
        except:
            return "dev"
```

### 1.4 Sensor Data Capture Patterns

```python
# sensor_capture.py
from typing import Dict, List, Any
import torch
from dataclasses import dataclass
from datetime import datetime
import threading
import queue

@dataclass
class SensorConfig:
    """Configuration for sensor capture"""
    sensor_id: str
    sensor_type: str
    capture_rate_hz: float
    buffer_size: int
    compression: str  # 'fp4', 'fp8', 'fp16', 'none'
    
class SensorCaptureManager:
    """
    Manages multi-sensor capture on Jetson Thor.
    Supports high-bandwidth sensors (cameras, lidar) with
    efficient tensor storage.
    """
    
    def __init__(self, experience_capture: ExperienceCaptureEngine):
        self.experience_capture = experience_capture
        self.sensors: Dict[str, SensorConfig] = {}
        self.capture_queues: Dict[str, queue.Queue] = {}
        self.capture_threads: Dict[str, threading.Thread] = {}
        self.running = False
        
    def register_sensor(self, config: SensorConfig):
        """Register a sensor for capture"""
        self.sensors[config.sensor_id] = config
        self.capture_queues[config.sensor_id] = queue.Queue(
            maxsize=config.buffer_size
        )
        
    def start_capture(self):
        """Start all sensor capture threads"""
        self.running = True
        for sensor_id, config in self.sensors.items():
            thread = threading.Thread(
                target=self._capture_loop,
                args=(sensor_id, config),
                daemon=True
            )
            thread.start()
            self.capture_threads[sensor_id] = thread
            
    def stop_capture(self):
        """Stop all sensor capture"""
        self.running = False
        for thread in self.capture_threads.values():
            thread.join(timeout=1.0)
            
    def _capture_loop(self, sensor_id: str, config: SensorConfig):
        """Capture loop for a single sensor"""
        while self.running:
            # Get sensor data (placeholder - actual implementation
            # would interface with Jetson sensor drivers)
            sensor_data = self._read_sensor(sensor_id, config)
            
            # Add timestamp
            timestamp = datetime.now().isoformat()
            
            # Queue for experience capture
            try:
                self.capture_queues[sensor_id].put(
                    (timestamp, sensor_data),
                    timeout=0.1
                )
            except queue.Full:
                # Drop oldest if buffer full
                try:
                    self.capture_queues[sensor_id].get_nowait()
                    self.capture_queues[sensor_id].put(
                        (timestamp, sensor_data),
                        timeout=0.1
                    )
                except:
                    pass
                    
    def get_snapshot(self, sensor_ids: List[str] = None) -> Dict[str, torch.Tensor]:
        """Get synchronized snapshot from specified sensors"""
        if sensor_ids is None:
            sensor_ids = list(self.sensors.keys())
            
        snapshot = {}
        for sensor_id in sensor_ids:
            if sensor_id in self.capture_queues:
                try:
                    timestamp, data = self.capture_queues[sensor_id].get_nowait()
                    snapshot[sensor_id] = data
                except queue.Empty:
                    pass
                    
        return snapshot
    
    def _read_sensor(self, sensor_id: str, config: SensorConfig) -> torch.Tensor:
        """Read from sensor - implementation depends on sensor type"""
        # Placeholder - actual implementation uses Isaac ROS
        # or Jetson sensor interfaces
        return torch.zeros((1,))  # Dummy tensor
```

---

## 2. Curation and Quality Filtering

### 2.1 Curation Architecture

```
+------------------------------------------------------------------+
|                    CURATION PIPELINE                              |
+------------------------------------------------------------------+
|                                                                   |
|  RAW EXPERIENCES                                                  |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Experience Buffer (1000 experiences)                       │ |
|  │  Quality: 0.0 - 1.0 (unfiltered)                           │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  QUALITY FILTER                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Filters:                                                    │ |
|  │  - Minimum quality threshold: 0.3                          │ |
|  │  - Deduplication (similar experiences)                      │ |
|  │  - Noise reduction (sensor anomalies)                       │ |
|  │  - Privacy filtering (remove PII)                           │ |
|  │                                                             │ |
|  │  Pass rate: ~60% of raw experiences                        │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  CURATION TIERS                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  TIER 3: HIGH VALUE (quality > 0.8)                        │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Always include in training                         │   │ |
|  │  │ - Priority for fleet sharing                         │   │ |
|  │  │ - Human review recommended                            │   │ |
|  │  │ - ~10% of filtered experiences                       │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  TIER 2: MEDIUM VALUE (quality 0.5 - 0.8)                  │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Include in local training                          │   │ |
|  │  │ - Share with fleet if transferable                    │   │ |
|  │  │ - No human review needed                              │   │ |
|  │  │ - ~30% of filtered experiences                       │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  TIER 1: LOW VALUE (quality 0.3 - 0.5)                     │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Local training only (not shared)                   │   │ |
|  │  │ - Keep for reference                                  │   │ |
|  │  │ - ~60% of filtered experiences                       │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  TRAINING DATASET                                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Curated experiences ready for fine-tuning                 │ |
|  │  - Indexed by task type                                    │ |
|  │  - Balanced across success/failure                         │ |
|  │  - Augmented with synthetic variations                     │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.2 Curation Implementation

```python
# curation.py
import torch
import json
from dataclasses import dataclass
from typing import List, Dict, Any, Optional, Tuple
from pathlib import Path
from datetime import datetime
import hashlib
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

@dataclass
class CurationConfig:
    """Configuration for curation pipeline"""
    min_quality_threshold: float = 0.3
    dedup_similarity_threshold: float = 0.95
    noise_zscore_threshold: float = 3.0
    tier1_threshold: float = 0.5
    tier2_threshold: float = 0.8
    max_experiences_per_tier: Dict[int, int] = None
    
    def __post_init__(self):
        if self.max_experiences_per_tier is None:
            self.max_experiences_per_tier = {
                1: 5000,  # Low value
                2: 2000,  # Medium value
                3: 500    # High value
            }


class ExperienceCurator:
    """
    Curates raw experiences into training-quality data.
    Implements quality filtering, deduplication, and tiering.
    """
    
    def __init__(self, repo_path: str, config: CurationConfig = None):
        self.repo_path = Path(repo_path)
        self.config = config or CurationConfig()
        self.experience_dir = self.repo_path / "experiences"
        
        # Embedding model for deduplication
        self.embedding_model = None  # Load on demand
        
    def curate_batch(self, experiences: List[Experience]) -> Dict[int, List[Experience]]:
        """
        Curate a batch of experiences into tiers.
        Returns dict mapping tier -> experiences.
        """
        # Step 1: Quality filter
        filtered = self._quality_filter(experiences)
        
        # Step 2: Deduplicate
        deduplicated = self._deduplicate(filtered)
        
        # Step 3: Noise reduction
        cleaned = self._noise_reduction(deduplicated)
        
        # Step 4: Privacy filter
        safe = self._privacy_filter(cleaned)
        
        # Step 5: Tier assignment
        tiered = self._assign_tiers(safe)
        
        # Step 6: Balance dataset
        balanced = self._balance_dataset(tiered)
        
        return balanced
    
    def _quality_filter(self, experiences: List[Experience]) -> List[Experience]:
        """Filter by minimum quality threshold"""
        return [
            exp for exp in experiences
            if exp.quality_score >= self.config.min_quality_threshold
        ]
    
    def _deduplicate(self, experiences: List[Experience]) -> List[Experience]:
        """Remove near-duplicate experiences"""
        if len(experiences) < 2:
            return experiences
            
        # Compute embeddings for experiences
        embeddings = self._compute_experience_embeddings(experiences)
        
        # Find duplicates using cosine similarity
        keep_indices = []
        for i, exp in enumerate(experiences):
            is_duplicate = False
            for j in keep_indices:
                similarity = cosine_similarity(
                    embeddings[i].reshape(1, -1),
                    embeddings[j].reshape(1, -1)
                )[0][0]
                
                if similarity > self.config.dedup_similarity_threshold:
                    is_duplicate = True
                    # Keep the higher quality one
                    if exp.quality_score > experiences[j].quality_score:
                        keep_indices.remove(j)
                        keep_indices.append(i)
                    break
                    
            if not is_duplicate:
                keep_indices.append(i)
                
        return [experiences[i] for i in keep_indices]
    
    def _compute_experience_embeddings(self, experiences: List[Experience]) -> np.ndarray:
        """Compute embeddings for experience deduplication"""
        # Lazy load embedding model
        if self.embedding_model is None:
            self._load_embedding_model()
            
        # Create text representation for embedding
        texts = [self._experience_to_text(exp) for exp in experiences]
        
        # Compute embeddings (simplified - actual implementation uses
        # sentence-transformers or similar)
        embeddings = []
        for text in texts:
            # Hash-based embedding for demonstration
            # Actual implementation would use neural embeddings
            embedding = np.random.randn(384)  # Placeholder
            embeddings.append(embedding)
            
        return np.array(embeddings)
    
    def _experience_to_text(self, experience: Experience) -> str:
        """Convert experience to text for embedding"""
        parts = [
            experience.task_specification.get('type', ''),
            ' '.join([a['action_type'] for a in experience.action_sequence]),
            'success' if experience.success else 'failure',
            str(experience.human_feedback or '')
        ]
        return ' '.join(parts)
    
    def _noise_reduction(self, experiences: List[Experience]) -> List[Experience]:
        """Remove experiences with anomalous sensor data"""
        if len(experiences) < 3:
            return experiences
            
        # Compute statistics on sensor data size
        sizes = [len(exp.sensor_data_refs) for exp in experiences]
        mean_size = np.mean(sizes)
        std_size = np.std(sizes)
        
        # Filter outliers
        filtered = []
        for exp in experiences:
            z_score = abs(len(exp.sensor_data_refs) - mean_size) / (std_size + 1e-6)
            if z_score < self.config.noise_zscore_threshold:
                filtered.append(exp)
                
        return filtered
    
    def _privacy_filter(self, experiences: List[Experience]) -> List[Experience]:
        """Remove personally identifiable information"""
        filtered = []
        for exp in experiences:
            # Remove human_intent if it contains PII
            if exp.human_intent:
                exp.human_intent = self._remove_pii(exp.human_intent)
                
            # Check environment state for PII
            exp.environment_state = self._scrub_pii(exp.environment_state)
            
            filtered.append(exp)
            
        return filtered
    
    def _remove_pii(self, text: str) -> str:
        """Remove PII from text (simplified)"""
        # Actual implementation would use NER or regex patterns
        # Placeholder - just return text
        return text
    
    def _scrub_pii(self, data: Dict[str, Any]) -> Dict[str, Any]:
        """Recursively scrub PII from data structure"""
        # Placeholder - actual implementation would check all strings
        return data
    
    def _assign_tiers(self, experiences: List[Experience]) -> Dict[int, List[Experience]]:
        """Assign experiences to quality tiers"""
        tiered = {1: [], 2: [], 3: []}
        
        for exp in experiences:
            if exp.quality_score >= self.config.tier2_threshold:
                tiered[3].append(exp)
            elif exp.quality_score >= self.config.tier1_threshold:
                tiered[2].append(exp)
            else:
                tiered[1].append(exp)
                
        return tiered
    
    def _balance_dataset(self, tiered: Dict[int, List[Experience]]) -> Dict[int, List[Experience]]:
        """Balance dataset within tiers"""
        balanced = {}
        
        for tier, experiences in tiered.items():
            max_count = self.config.max_experiences_per_tier.get(tier, len(experiences))
            
            if len(experiences) <= max_count:
                balanced[tier] = experiences
                continue
                
            # Stratified sampling by success/failure
            successes = [e for e in experiences if e.success]
            failures = [e for e in experiences if not e.success]
            
            # Target: 70% success, 30% failure (slightly optimistic)
            target_success = int(max_count * 0.7)
            target_failure = max_count - target_success
            
            # Sample (with replacement if needed)
            import random
            sampled_successes = random.choices(
                successes, k=min(target_success, len(successes))
            )
            sampled_failures = random.choices(
                failures, k=min(target_failure, len(failures))
            )
            
            balanced[tier] = sampled_successes + sampled_failures
            
        return balanced
    
    def _load_embedding_model(self):
        """Load embedding model for deduplication"""
        # Lazy load to save memory
        # from sentence_transformers import SentenceTransformer
        # self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        pass


class TrainingDatasetBuilder:
    """
    Builds training datasets from curated experiences.
    Supports multiple dataset formats for different training objectives.
    """
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self.dataset_dir = self.repo_path / "datasets"
        self.dataset_dir.mkdir(exist_ok=True)
        
    def build_dataset(self, 
                     experiences: List[Experience],
                     dataset_type: str = 'general',
                     format: str = 'jsonl') -> Path:
        """
        Build a training dataset from experiences.
        
        Args:
            experiences: List of curated experiences
            dataset_type: Type of dataset ('general', 'navigation', 'manipulation', 'voice')
            format: Output format ('jsonl', 'parquet', 'tfrecord')
            
        Returns:
            Path to created dataset
        """
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        dataset_name = f"{dataset_type}_{timestamp}.{format}"
        dataset_path = self.dataset_dir / dataset_name
        
        if format == 'jsonl':
            self._write_jsonl(experiences, dataset_path)
        elif format == 'parquet':
            self._write_parquet(experiences, dataset_path)
        else:
            raise ValueError(f"Unsupported format: {format}")
            
        # Write dataset manifest
        manifest = {
            'name': dataset_name,
            'created': timestamp,
            'num_experiences': len(experiences),
            'dataset_type': dataset_type,
            'format': format,
            'statistics': self._compute_statistics(experiences)
        }
        
        manifest_path = dataset_path.with_suffix('.manifest.json')
        manifest_path.write_text(json.dumps(manifest, indent=2))
        
        return dataset_path
    
    def _write_jsonl(self, experiences: List[Experience], path: Path):
        """Write experiences to JSONL format"""
        with open(path, 'w') as f:
            for exp in experiences:
                # Convert to training format
                training_example = self._to_training_format(exp)
                f.write(json.dumps(training_example) + '\n')
    
    def _write_parquet(self, experiences: List[Experience], path: Path):
        """Write experiences to Parquet format"""
        import pandas as pd
        
        records = [self._to_training_format(exp) for exp in experiences]
        df = pd.DataFrame(records)
        df.to_parquet(path)
    
    def _to_training_format(self, experience: Experience) -> Dict[str, Any]:
        """Convert experience to training format"""
        return {
            'id': experience.experience_id,
            'context': {
                'environment': experience.environment_state,
                'task': experience.task_specification,
                'constraints': experience.constraints
            },
            'observations': experience.perception_outputs,
            'actions': experience.action_sequence,
            'outcome': {
                'success': experience.success,
                'reward': experience.reward_signal,
                'human_feedback': experience.human_feedback
            },
            'learning_signals': {
                'confidence_before': experience.confidence_before,
                'confidence_after': experience.confidence_after,
                'surprise': experience.surprise_metric,
                'improvement_opportunity': experience.improvement_opportunity
            },
            'metadata': {
                'quality_score': experience.quality_score,
                'domains': experience.domains,
                'transferable': experience.transferable
            }
        }
    
    def _compute_statistics(self, experiences: List[Experience]) -> Dict[str, Any]:
        """Compute dataset statistics"""
        if not experiences:
            return {}
            
        return {
            'success_rate': sum(1 for e in experiences if e.success) / len(experiences),
            'avg_quality': sum(e.quality_score for e in experiences) / len(experiences),
            'avg_actions': sum(len(e.action_sequence) for e in experiences) / len(experiences),
            'domains': list(set(d for e in experiences for d in e.domains)),
            'human_feedback_rate': sum(1 for e in experiences if e.human_feedback) / len(experiences)
        }
```

---

## 3. Local Fine-Tuning on Jetson Thor

### 3.1 Fine-Tuning Architecture

```
+------------------------------------------------------------------+
|                LOCAL FINE-TUNING ON JETSON THOR                   |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    MIG PARTITION LAYOUT                      │ |
|  │                                                             │ |
|  │  ┌──────────────────────────────────────────────────────┐  │ |
|  │  │ GPU Memory: 128 GB Total                              │  │ |
|  │  │                                                        │  │ |
|  │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐ │  │ |
|  │  │  │ INFERENCE│ │ TRAINING │ │ VISION  │ │  RESERVE    │ │  │ |
|  │  │  │  48 GB   │ │  48 GB   │ │ 16 GB   │ │   16 GB     │ │  │ |
|  │  │  │ LLM/VLM  │ │ LoRA    │ │ Pipeline│ │  Safety     │ │  │ |
|  │  │  │ (Agent)  │ │ Fine-tune│ │         │ │  Monitor    │ │  │ |
|  │  │  └─────────┘ └─────────┘ └─────────┘ └─────────────┘ │  │ |
|  │  │                                                        │  │ |
|  │  └────────────────────────────────────────────────────────┘  │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TRAINING PIPELINE                                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  1. DATA PREPARATION                                        │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Curated Experiences → Training Format → FP4 Tensor  │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                           │                                 │ |
|  │                           ▼                                 │ |
|  │  2. LORA FINE-TUNING                                        │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Rank: 16-64 (adaptive based on data)              │   │ |
|  │  │ - Alpha: 32-128                                      │   │ |
|  │  │ - Target modules: attention, FFN                     │   │ |
|  │  │ - Learning rate: 1e-5 to 1e-4                        │   │ |
|  │  │ - Epochs: 1-3 (early stopping)                       │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                           │                                 │ |
|  │                           ▼                                 │ |
|  │  3. VALIDATION                                              │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Holdout set: 10% of experiences                   │   │ |
|  │  │ - Metrics: loss, accuracy, success rate             │   │ |
|  │  │ - Safety checks: constraint satisfaction            │   │ |
|  │  │ - Human approval for significant changes            │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                           │                                 │ |
|  │                           ▼                                 │ |
|  │  4. DEPLOYMENT                                              │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Merge LoRA weights into base model                │   │ |
|  │  │ - Update agent version                              │   │ |
|  │  │ - Commit to git with training metadata              │   │ |
|  │  │ - Notify fleet hub of improvement                   │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 3.2 LoRA Fine-Tuning Implementation

```python
# local_finetune.py
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
from dataclasses import dataclass, field
from typing import List, Dict, Any, Optional, Tuple
from pathlib import Path
import json
from datetime import datetime
import git

@dataclass
class LoRAConfig:
    """LoRA fine-tuning configuration optimized for Jetson Thor"""
    rank: int = 32
    alpha: int = 64
    dropout: float = 0.05
    
    # Target modules for LoRA
    target_modules: List[str] = field(default_factory=lambda: [
        'q_proj', 'k_proj', 'v_proj', 'o_proj',  # Attention
        'gate_proj', 'up_proj', 'down_proj'       # FFN
    ])
    
    # Training hyperparameters
    learning_rate: float = 2e-5
    weight_decay: float = 0.01
    warmup_steps: int = 100
    max_epochs: int = 3
    early_stopping_patience: int = 5
    
    # Memory optimization
    gradient_checkpointing: bool = True
    fp4_training: bool = True  # Thor native
    max_grad_norm: float = 1.0
    
    # Batch configuration
    per_device_train_batch_size: int = 4
    gradient_accumulation_steps: int = 4


class LocalFineTuner:
    """
    Implements LoRA fine-tuning on Jetson Thor.
    Optimized for edge deployment with minimal resource impact.
    """
    
    def __init__(self, 
                 repo_path: str,
                 base_model_path: str,
                 config: LoRAConfig = None):
        self.repo_path = Path(repo_path)
        self.repo = git.Repo(repo_path)
        self.base_model_path = Path(base_model_path)
        self.config = config or LoRAConfig()
        
        # Training state
        self.model = None
        self.lora_modules = {}
        self.training_history = []
        
    def prepare_training(self, 
                        experiences: List[Dict],
                        validation_split: float = 0.1) -> Tuple[Dataset, Dataset]:
        """Prepare training and validation datasets"""
        # Split experiences
        split_idx = int(len(experiences) * (1 - validation_split))
        train_data = experiences[:split_idx]
        val_data = experiences[split_idx:]
        
        # Create datasets
        train_dataset = ExperienceDataset(train_data)
        val_dataset = ExperienceDataset(val_data)
        
        return train_dataset, val_dataset
    
    def setup_lora(self, model: nn.Module):
        """Apply LoRA to target modules"""
        for name, module in model.named_modules():
            # Check if this module should have LoRA
            if any(target in name for target in self.config.target_modules):
                if isinstance(module, nn.Linear):
                    # Create LoRA wrapper
                    lora = LoRALinear(
                        in_features=module.in_features,
                        out_features=module.out_features,
                        rank=self.config.rank,
                        alpha=self.config.alpha,
                        dropout=self.config.dropout
                    )
                    
                    # Copy original weights
                    lora.original.weight.data = module.weight.data.clone()
                    if module.bias is not None:
                        lora.original.bias.data = module.bias.data.clone()
                    
                    # Replace module
                    self._replace_module(model, name, lora)
                    self.lora_modules[name] = lora
    
    def train(self, 
             train_dataset: Dataset,
             val_dataset: Dataset) -> Dict[str, Any]:
        """
        Execute LoRA fine-tuning.
        Returns training metrics.
        """
        # Load base model
        self.model = self._load_base_model()
        
        # Apply LoRA
        self.setup_lora(self.model)
        
        # Freeze base model, unfreeze LoRA
        for name, param in self.model.named_parameters():
            if 'lora_' in name:
                param.requires_grad = True
            else:
                param.requires_grad = False
        
        # Create data loaders
        train_loader = DataLoader(
            train_dataset,
            batch_size=self.config.per_device_train_batch_size,
            shuffle=True,
            collate_fn=self._collate_fn
        )
        
        val_loader = DataLoader(
            val_dataset,
            batch_size=self.config.per_device_train_batch_size,
            collate_fn=self._collate_fn
        )
        
        # Optimizer (only for LoRA parameters)
        lora_params = [p for n, p in self.model.named_parameters() 
                      if p.requires_grad]
        optimizer = torch.optim.AdamW(
            lora_params,
            lr=self.config.learning_rate,
            weight_decay=self.config.weight_decay
        )
        
        # Learning rate scheduler
        scheduler = torch.optim.lr_scheduler.LinearLR(
            optimizer,
            start_factor=0.1,
            total_iters=self.config.warmup_steps
        )
        
        # Training loop
        best_val_loss = float('inf')
        patience_counter = 0
        training_start = datetime.now()
        
        for epoch in range(self.config.max_epochs):
            epoch_metrics = self._train_epoch(
                train_loader, optimizer, scheduler, epoch
            )
            
            val_metrics = self._validate(val_loader)
            
            # Record history
            self.training_history.append({
                'epoch': epoch,
                'train_loss': epoch_metrics['loss'],
                'val_loss': val_metrics['loss'],
                'val_accuracy': val_metrics['accuracy'],
                'timestamp': datetime.now().isoformat()
            })
            
            # Early stopping check
            if val_metrics['loss'] < best_val_loss:
                best_val_loss = val_metrics['loss']
                patience_counter = 0
                self._save_checkpoint('best')
            else:
                patience_counter += 1
                
            if patience_counter >= self.config.early_stopping_patience:
                print(f"Early stopping at epoch {epoch}")
                break
        
        training_duration = (datetime.now() - training_start).total_seconds()
        
        # Load best checkpoint
        self._load_checkpoint('best')
        
        # Merge LoRA weights
        merged_model = self._merge_lora_weights()
        
        return {
            'best_val_loss': best_val_loss,
            'epochs_trained': epoch + 1,
            'training_duration_seconds': training_duration,
            'final_train_loss': self.training_history[-1]['train_loss'],
            'final_val_loss': self.training_history[-1]['val_loss'],
            'lora_rank': self.config.rank,
            'lora_alpha': self.config.alpha
        }
    
    def _train_epoch(self, 
                    train_loader: DataLoader,
                    optimizer: torch.optim.Optimizer,
                    scheduler: torch.optim.lr_scheduler._LRScheduler,
                    epoch: int) -> Dict[str, float]:
        """Train for one epoch"""
        self.model.train()
        total_loss = 0.0
        num_batches = 0
        
        for batch_idx, batch in enumerate(train_loader):
            # Move to GPU (Thor)
            batch = {k: v.to('cuda') if isinstance(v, torch.Tensor) else v 
                    for k, v in batch.items()}
            
            # Forward pass with FP4 on Thor
            with torch.cuda.amp.autocast():
                outputs = self.model(**batch)
                loss = outputs.loss
                
            # Backward pass
            loss.backward()
            
            # Gradient clipping
            torch.nn.utils.clip_grad_norm_(
                self.model.parameters(),
                self.config.max_grad_norm
            )
            
            # Optimizer step (with accumulation)
            if (batch_idx + 1) % self.config.gradient_accumulation_steps == 0:
                optimizer.step()
                optimizer.zero_grad()
                scheduler.step()
            
            total_loss += loss.item()
            num_batches += 1
            
        return {'loss': total_loss / num_batches}
    
    @torch.no_grad()
    def _validate(self, val_loader: DataLoader) -> Dict[str, float]:
        """Validate on holdout set"""
        self.model.eval()
        total_loss = 0.0
        total_correct = 0
        total_samples = 0
        
        for batch in val_loader:
            batch = {k: v.to('cuda') if isinstance(v, torch.Tensor) else v 
                    for k, v in batch.items()}
            
            outputs = self.model(**batch)
            total_loss += outputs.loss.item()
            
            # Compute accuracy
            if 'labels' in batch:
                predictions = outputs.logits.argmax(dim=-1)
                total_correct += (predictions == batch['labels']).sum().item()
                total_samples += batch['labels'].size(0)
        
        return {
            'loss': total_loss / len(val_loader),
            'accuracy': total_correct / total_samples if total_samples > 0 else 0.0
        }
    
    def commit_training(self, training_metrics: Dict[str, Any]):
        """Commit trained model to git repository"""
        # Save model
        model_path = self.repo_path / "models" / "latest"
        model_path.mkdir(parents=True, exist_ok=True)
        
        # Save merged model
        merged_model = self._merge_lora_weights()
        torch.save(merged_model.state_dict(), model_path / "model.pt")
        
        # Save training metadata
        metadata = {
            'timestamp': datetime.now().isoformat(),
            'base_model': str(self.base_model_path),
            'training_metrics': training_metrics,
            'config': {
                'rank': self.config.rank,
                'alpha': self.config.alpha,
                'learning_rate': self.config.learning_rate,
                'epochs': training_metrics['epochs_trained']
            },
            'history': self.training_history
        }
        
        metadata_path = model_path / "training_metadata.json"
        metadata_path.write_text(json.dumps(metadata, indent=2))
        
        # Git commit
        self.repo.index.add([str(model_path / "model.pt"),
                            str(metadata_path)])
        
        commit_message = f"""training: LoRA fine-tune complete

Metrics:
- Validation Loss: {training_metrics['best_val_loss']:.4f}
- Epochs: {training_metrics['epochs_trained']}
- Duration: {training_metrics['training_duration_seconds']:.1f}s

LoRA Config:
- Rank: {self.config.rank}
- Alpha: {self.config.alpha}
- LR: {self.config.learning_rate}
"""
        
        commit = self.repo.index.commit(commit_message)
        
        # Tag the commit
        tag_name = f"v{datetime.now().strftime('%Y%m%d.%H%M%S')}"
        self.repo.create_tag(tag_name, commit)
        
        return commit.hexsha
    
    def _load_base_model(self) -> nn.Module:
        """Load base model for fine-tuning"""
        # Placeholder - actual implementation loads from path
        # using transformers or similar library
        return nn.Module()
    
    def _replace_module(self, model: nn.Module, name: str, new_module: nn.Module):
        """Replace a module in the model hierarchy"""
        parts = name.split('.')
        parent = model
        for part in parts[:-1]:
            parent = getattr(parent, part)
        setattr(parent, parts[-1], new_module)
    
    def _merge_lora_weights(self) -> nn.Module:
        """Merge LoRA weights into base model"""
        # Create a copy of the model
        merged = self._load_base_model()
        
        # For each LoRA module, merge weights
        for name, module in merged.named_modules():
            if name in self.lora_modules:
                lora_module = self.lora_modules[name]
                # W_merged = W_original + (alpha/rank) * W_lora
                merged_weight = (
                    module.weight.data + 
                    (self.config.alpha / self.config.rank) * 
                    (lora_module.lora_A.weight @ lora_module.lora_B.weight)
                )
                module.weight.data = merged_weight
        
        return merged
    
    def _save_checkpoint(self, name: str):
        """Save training checkpoint"""
        checkpoint_dir = self.repo_path / "checkpoints" / name
        checkpoint_dir.mkdir(parents=True, exist_ok=True)
        
        # Save LoRA weights
        lora_state = {name: module.state_dict() 
                     for name, module in self.lora_modules.items()}
        torch.save(lora_state, checkpoint_dir / "lora_weights.pt")
    
    def _load_checkpoint(self, name: str):
        """Load training checkpoint"""
        checkpoint_dir = self.repo_path / "checkpoints" / name
        
        lora_state = torch.load(checkpoint_dir / "lora_weights.pt")
        for name, state in lora_state.items():
            self.lora_modules[name].load_state_dict(state)
    
    def _collate_fn(self, batch: List[Dict]) -> Dict[str, torch.Tensor]:
        """Collate batch for training"""
        # Placeholder - actual implementation depends on model
        return {}


class LoRALinear(nn.Module):
    """LoRA adapter for linear layers"""
    
    def __init__(self, 
                 in_features: int, 
                 out_features: int,
                 rank: int, 
                 alpha: int,
                 dropout: float = 0.0):
        super().__init__()
        
        self.original = nn.Linear(in_features, out_features, bias=True)
        self.original.weight.requires_grad = False
        
        # LoRA matrices
        self.lora_A = nn.Linear(in_features, rank, bias=False)
        self.lora_B = nn.Linear(rank, out_features, bias=False)
        
        # Scaling
        self.scaling = alpha / rank
        
        # Dropout
        self.dropout = nn.Dropout(dropout) if dropout > 0 else nn.Identity()
        
        # Initialize
        nn.init.kaiming_uniform_(self.lora_A.weight, a=5**0.5)
        nn.init.zeros_(self.lora_B.weight)
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # Original output
        result = self.original(x)
        
        # LoRA contribution
        lora_out = self.dropout(self.lora_A(x))
        lora_out = self.lora_B(lora_out)
        
        return result + lora_out * self.scaling


class ExperienceDataset(Dataset):
    """Dataset wrapper for experiences"""
    
    def __init__(self, experiences: List[Dict]):
        self.experiences = experiences
        
    def __len__(self) -> int:
        return len(self.experiences)
    
    def __getitem__(self, idx: int) -> Dict[str, Any]:
        return self.experiences[idx]
```

---

## 4. Fleet-Level Aggregation

### 4.1 Aggregation Architecture

```
+------------------------------------------------------------------+
|                    FLEET-LEVEL AGGREGATION                        |
+------------------------------------------------------------------+
|                                                                   |
|  VESSEL LEVEL (Edge)                                              |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Vessel A               Vessel B               Vessel C     │ |
|  │  ┌─────────┐           ┌─────────┐           ┌─────────┐   │ |
|  │  │Exp 1    │           │Exp 4    │           │Exp 7    │   │ |
|  │  │Exp 2    │           │Exp 5    │           │Exp 8    │   │ |
|  │  │Exp 3    │           │Exp 6    │           │Exp 9    │   │ |
|  │  └─────────┘           └─────────┘           └─────────┘   │ |
|  │       │                     │                     │         │ |
|  │       └─────────────────────┼─────────────────────┘         │ |
|  │                             ▼                               │ |
|  │                    ┌─────────────────┐                      │ |
|  │                    │ Local Curated   │                      │ |
|  │                    │ Training Set    │                      │ |
|  │                    └─────────────────┘                      │ |
|  │                             │                               │ |
|  └─────────────────────────────┼───────────────────────────────┘ |
|                                │                                  |
|                                ▼                                  |
|  FLEET HUB LEVEL                                                  |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │              FEDERATED AGGREGATOR                    │   │ |
|  │  │                                                     │   │ |
|  │  │  1. Receive curated experiences from vessels        │   │ |
|  │  │  2. Byzantine-robust aggregation                    │   │ |
|  │  │  3. Cross-domain pattern extraction                 │   │ |
|  │  │  4. Fleet model update                              │   │ |
|  │  │  5. Distribute updated model                        │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │              KNOWLEDGE GRAPH                         │   │ |
|  │  │                                                     │   │ |
|  │  │  Patterns:                                          │   │ |
|  │  │  - Navigation patterns (transferable)               │   │ |
|  │  │  - Manipulation patterns (domain-specific)          │   │ |
|  │  │  - Communication patterns (universal)               │   │ |
|  │  │  - Safety patterns (critical)                       │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                │                                  |
|                                ▼                                  |
|  GLOBAL NETWORK LEVEL                                            |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Fleet A (Maritime) ←→ Fleet B (Warehouse) ←→ Fleet C...  │ |
|  │                                                             │ |
|  │  Cross-domain pattern exchange via A2A-R protocol          │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 4.2 Federated Aggregation Implementation

```python
# fleet_aggregation.py
import torch
import torch.nn as nn
import numpy as np
from dataclasses import dataclass
from typing import List, Dict, Any, Optional, Tuple
from datetime import datetime
import hashlib
import json
from pathlib import Path

@dataclass
class FleetNode:
    """Represents a vessel/node in the fleet"""
    node_id: str
    public_key: str
    reputation: float
    last_seen: datetime
    model_version: str
    num_contributions: int


class ByzantineRobustAggregator:
    """
    Implements Byzantine-robust federated learning.
    Based on Krum and Multi-Krum algorithms.
    """
    
    def __init__(self, 
                 num_byzantine: int = 2,
                 aggregation_method: str = 'krum'):
        """
        Args:
            num_byzantine: Expected number of malicious nodes
            aggregation_method: 'krum', 'multi_krum', or 'trimmed_mean'
        """
        self.num_byzantine = num_byzantine
        self.aggregation_method = aggregation_method
        
    def aggregate(self, 
                 updates: List[Dict[str, torch.Tensor]],
                 reputations: List[float]) -> Dict[str, torch.Tensor]:
        """
        Aggregate model updates with Byzantine robustness.
        
        Args:
            updates: List of model updates (LoRA deltas) from vessels
            reputations: Reputation scores for each vessel
            
        Returns:
            Aggregated model update
        """
        if len(updates) == 0:
            raise ValueError("No updates to aggregate")
            
        if self.aggregation_method == 'krum':
            return self._krum_aggregate(updates, reputations)
        elif self.aggregation_method == 'multi_krum':
            return self._multi_krum_aggregate(updates, reputations)
        elif self.aggregation_method == 'trimmed_mean':
            return self._trimmed_mean_aggregate(updates, reputations)
        else:
            raise ValueError(f"Unknown aggregation method: {self.aggregation_method}")
    
    def _krum_aggregate(self, 
                        updates: List[Dict[str, torch.Tensor]],
                        reputations: List[float]) -> Dict[str, torch.Tensor]:
        """
        Krum algorithm: select the most central update.
        Robust to up to f Byzantine nodes among n = 2f + 3.
        """
        n = len(updates)
        f = self.num_byzantine
        
        if n < 2 * f + 3:
            # Not enough nodes for Krum, fall back to weighted average
            return self._reputation_weighted_average(updates, reputations)
        
        # Compute pairwise distances
        distances = self._compute_pairwise_distances(updates)
        
        # For each update, find sum of distances to n-f-2 nearest neighbors
        scores = []
        for i in range(n):
            # Sort distances from this update
            sorted_dists = sorted(distances[i])
            # Sum of n-f-2 smallest (excluding self)
            score = sum(sorted_dists[1:n-f-1])
            scores.append(score)
        
        # Weight by reputation
        weighted_scores = [
            scores[i] / (reputations[i] + 0.1)
            for i in range(n)
        ]
        
        # Select the update with lowest weighted score
        best_idx = np.argmin(weighted_scores)
        
        return updates[best_idx]
    
    def _multi_krum_aggregate(self,
                              updates: List[Dict[str, torch.Tensor]],
                              reputations: List[float]) -> Dict[str, torch.Tensor]:
        """
        Multi-Krum: average the m best updates selected by Krum.
        """
        n = len(updates)
        f = self.num_byzantine
        m = n - f  # Number of updates to average
        
        # Compute pairwise distances
        distances = self._compute_pairwise_distances(updates)
        
        # Score each update
        scores = []
        for i in range(n):
            sorted_dists = sorted(distances[i])
            score = sum(sorted_dists[1:n-f-1])
            scores.append(score)
        
        # Weight by reputation
        weighted_scores = [
            scores[i] / (reputations[i] + 0.1)
            for i in range(n)
        ]
        
        # Select m best updates
        best_indices = np.argsort(weighted_scores)[:m]
        
        # Average selected updates
        return self._average_updates([updates[i] for i in best_indices])
    
    def _trimmed_mean_aggregate(self,
                                updates: List[Dict[str, torch.Tensor]],
                                reputations: List[float]) -> Dict[str, torch.Tensor]:
        """
        Trimmed mean: remove extreme values and average the rest.
        """
        n = len(updates)
        trim = self.num_byzantine
        
        if n <= 2 * trim:
            return self._reputation_weighted_average(updates, reputations)
        
        aggregated = {}
        
        # For each parameter, compute trimmed mean
        for key in updates[0].keys():
            # Stack all updates for this parameter
            stacked = torch.stack([u[key] for u in updates])
            
            # Sort along the update dimension
            sorted_updates, _ = torch.sort(stacked, dim=0)
            
            # Trim top and bottom trim updates
            trimmed = sorted_updates[trim:-trim]
            
            # Average the remaining
            aggregated[key] = trimmed.mean(dim=0)
        
        return aggregated
    
    def _compute_pairwise_distances(self, 
                                   updates: List[Dict[str, torch.Tensor]]) -> np.ndarray:
        """Compute pairwise Euclidean distances between updates"""
        n = len(updates)
        
        # Flatten updates to vectors
        vectors = []
        for update in updates:
            flat = torch.cat([v.flatten() for v in update.values()])
            vectors.append(flat)
        
        # Compute pairwise distances
        distances = np.zeros((n, n))
        for i in range(n):
            for j in range(i + 1, n):
                dist = torch.norm(vectors[i] - vectors[j]).item()
                distances[i, j] = dist
                distances[j, i] = dist
        
        return distances
    
    def _reputation_weighted_average(self,
                                     updates: List[Dict[str, torch.Tensor]],
                                     reputations: List[float]) -> Dict[str, torch.Tensor]:
        """Compute reputation-weighted average of updates"""
        total_reputation = sum(reputations)
        weights = [r / total_reputation for r in reputations]
        
        aggregated = {}
        for key in updates[0].keys():
            weighted_sum = sum(
                w * u[key] for w, u in zip(weights, updates)
            )
            aggregated[key] = weighted_sum
        
        return aggregated
    
    def _average_updates(self, 
                        updates: List[Dict[str, torch.Tensor]]) -> Dict[str, torch.Tensor]:
        """Simple average of updates"""
        aggregated = {}
        for key in updates[0].keys():
            stacked = torch.stack([u[key] for u in updates])
            aggregated[key] = stacked.mean(dim=0)
        return aggregated


class FleetLearningCoordinator:
    """
    Coordinates fleet-level learning aggregation.
    Runs on Fleet Hub (cloud/edge server).
    """
    
    def __init__(self, 
                 repo_path: str,
                 aggregator: ByzantineRobustAggregator = None):
        self.repo_path = Path(repo_path)
        self.aggregator = aggregator or ByzantineRobustAggregator()
        
        # Fleet state
        self.nodes: Dict[str, FleetNode] = {}
        self.pending_updates: List[Dict] = []
        self.aggregation_round = 0
        
    def register_vessel(self, 
                       node_id: str, 
                       public_key: str) -> bool:
        """Register a new vessel in the fleet"""
        self.nodes[node_id] = FleetNode(
            node_id=node_id,
            public_key=public_key,
            reputation=1.0,  # Start with neutral reputation
            last_seen=datetime.now(),
            model_version='initial',
            num_contributions=0
        )
        return True
    
    def receive_update(self, 
                      vessel_id: str,
                      model_update: Dict[str, torch.Tensor],
                      experience_summary: Dict[str, Any],
                      signature: str) -> bool:
        """
        Receive a model update from a vessel.
        Validates signature and queues for aggregation.
        """
        if vessel_id not in self.nodes:
            return False
        
        # Verify signature
        if not self._verify_signature(vessel_id, model_update, signature):
            # Potential Byzantine behavior - reduce reputation
            self.nodes[vessel_id].reputation *= 0.8
            return False
        
        # Queue update for aggregation
        self.pending_updates.append({
            'vessel_id': vessel_id,
            'update': model_update,
            'summary': experience_summary,
            'timestamp': datetime.now().isoformat()
        })
        
        # Update vessel stats
        self.nodes[vessel_id].last_seen = datetime.now()
        self.nodes[vessel_id].num_contributions += 1
        
        return True
    
    def aggregate_updates(self, 
                         min_updates: int = 5) -> Optional[Dict[str, torch.Tensor]]:
        """
        Aggregate pending updates using Byzantine-robust method.
        Returns aggregated model update if enough updates available.
        """
        if len(self.pending_updates) < min_updates:
            return None
        
        # Extract updates and reputations
        updates = [u['update'] for u in self.pending_updates]
        reputations = [
            self.nodes[u['vessel_id']].reputation 
            for u in self.pending_updates
        ]
        
        # Perform aggregation
        aggregated = self.aggregator.aggregate(updates, reputations)
        
        # Update reputations based on contribution quality
        self._update_reputations(aggregated, updates)
        
        # Clear pending updates
        self.pending_updates = []
        self.aggregation_round += 1
        
        return aggregated
    
    def distribute_update(self, 
                         model_update: Dict[str, torch.Tensor]) -> Dict[str, str]:
        """
        Prepare model update for distribution to vessels.
        Returns update manifest for each vessel.
        """
        # Create update package
        update_package = {
            'round': self.aggregation_round,
            'timestamp': datetime.now().isoformat(),
            'update': model_update
        }
        
        # Generate manifest for each vessel
        manifests = {}
        for vessel_id, node in self.nodes.items():
            manifest = {
                'vessel_id': vessel_id,
                'round': self.aggregation_round,
                'update_hash': self._hash_update(model_update),
                'model_version': f"fleet_v{self.aggregation_round}",
                'signature': self._sign_update(vessel_id, model_update)
            }
            manifests[vessel_id] = manifest
        
        return manifests
    
    def _verify_signature(self, 
                         vessel_id: str, 
                         update: Dict, 
                         signature: str) -> bool:
        """Verify update signature from vessel"""
        # Placeholder - actual implementation uses cryptographic signatures
        return True
    
    def _update_reputations(self, 
                           aggregated: Dict[str, torch.Tensor],
                           updates: List[Dict[str, torch.Tensor]]):
        """Update vessel reputations based on contribution quality"""
        for i, update in enumerate(updates):
            # Compute similarity to aggregated
            similarity = self._compute_similarity(update, aggregated)
            
            # Update reputation
            vessel_id = self.pending_updates[i]['vessel_id']
            self.nodes[vessel_id].reputation = (
                0.9 * self.nodes[vessel_id].reputation + 
                0.1 * similarity
            )
    
    def _compute_similarity(self, 
                           update1: Dict[str, torch.Tensor],
                           update2: Dict[str, torch.Tensor]) -> float:
        """Compute cosine similarity between updates"""
        flat1 = torch.cat([v.flatten() for v in update1.values()])
        flat2 = torch.cat([v.flatten() for v in update2.values()])
        
        cos_sim = torch.nn.functional.cosine_similarity(
            flat1.unsqueeze(0), flat2.unsqueeze(0)
        )
        return cos_sim.item()
    
    def _hash_update(self, update: Dict[str, torch.Tensor]) -> str:
        """Generate hash of update for integrity"""
        flat = torch.cat([v.flatten() for v in update.values()])
        return hashlib.sha256(flat.numpy().tobytes()).hexdigest()[:16]
    
    def _sign_update(self, vessel_id: str, update: Dict) -> str:
        """Sign update for vessel verification"""
        # Placeholder - actual implementation uses private key
        return f"signed_{vessel_id}"
```

---

## 5. Human-in-the-Loop Feedback

### 5.1 Feedback Architecture

```
+------------------------------------------------------------------+
|                    HUMAN-IN-THE-LOOP FEEDBACK                     |
+------------------------------------------------------------------+
|                                                                   |
|  FEEDBACK CHANNELS                                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │ |
|  │  │  IMPLICIT   │  │  EXPLICIT   │  │  CORRECTIVE │        │ |
|  │  │             │  │             │  │             │        │ |
|  │  │ - Override  │  │ - Thumbs    │  │ - Edit      │        │ |
|  │  │ - Ignore    │  │   up/down   │  │   action    │        │ |
|  │  │ - Repeat    │  │ - Rating    │  │ - Correct   │        │ |
|  │  │ - Timeout   │  │ - Comment   │  │   speech    │        │ |
|  │  │             │  │             │  │             │        │ |
|  │  │ Auto-captured│ │ User-action │ │ High-value  │        │ |
|  │  │             │  │             │  │ training    │        │ |
|  │  └─────────────┘  └─────────────┘  └─────────────┘        │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  FEEDBACK PROCESSING                                              |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │              FEEDBACK CLASSIFIER                     │   │ |
|  │  │                                                     │   │ |
|  │  │  Input: Raw feedback event                          │   │ |
|  │  │  Output: Feedback type, sentiment, urgency          │   │ |
|  │  │                                                     │   │ |
|  │  │  Types:                                             │   │ |
|  │  │  - POSITIVE: Agent did well                        │   │ |
|  │  │  - NEGATIVE: Agent made mistake                    │   │ |
|  │  │  - CORRECTIVE: Human showed better way             │   │ |
|  │  │  - UNCERTAIN: Need clarification                    │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │              TRAINING VALUE ASSESSOR                 │   │ |
|  │  │                                                     │   │ |
|  │  │  Factors:                                           │   │ |
|  │  │  - Novelty: Is this a new pattern?                 │   │ |
|  │  │  - Impact: How much does this affect operations?   │   │ |
|  │  │  - Clarity: Is the feedback unambiguous?           │   │ |
|  │  │  - Reproducibility: Can we generalize this?        │   │ |
|  │  │                                                     │   │ |
|  │  │  Training Value = Novelty × Impact × Clarity       │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  TRAINING INTEGRATION                                             |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │ |
|  │  │ HIGH VALUE  │  │ MEDIUM      │  │ LOW VALUE   │        │ |
|  │  │             │  │ VALUE       │  │             │        │ |
|  │  │ Immediate   │  │ Queue for   │  │ Log only    │        │ |
|  │  │ training    │  │ next batch  │  │             │        │ |
|  │  │ Fleet share │  │ Local train │  │             │        │ |
|  │  └─────────────┘  └─────────────┘  └─────────────┘        │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 5.2 Feedback Collection Implementation

```python
# human_feedback.py
import torch
from dataclasses import dataclass, field
from typing import List, Dict, Any, Optional, Enum
from datetime import datetime
import json
from pathlib import Path

class FeedbackType(Enum):
    POSITIVE = 'positive'
    NEGATIVE = 'negative'
    CORRECTIVE = 'corrective'
    UNCERTAIN = 'uncertain'
    NEUTRAL = 'neutral'

class FeedbackChannel(Enum):
    IMPLICIT = 'implicit'
    EXPLICIT = 'explicit'
    CORRECTIVE = 'corrective'

@dataclass
class FeedbackEvent:
    """Single feedback event from human"""
    event_id: str
    timestamp: str
    vessel_id: str
    agent_version: str
    
    # Context
    task_id: str
    action_id: str
    experience_id: str
    
    # Feedback
    channel: FeedbackChannel
    type: FeedbackType
    raw_data: Dict[str, Any]  # Original feedback data
    
    # Computed
    sentiment: float = 0.0  # -1 to 1
    urgency: float = 0.0    # 0 to 1
    training_value: float = 0.0
    processed: bool = False

@dataclass
class ImplicitFeedback:
    """Implicit feedback detected from user behavior"""
    feedback_type: str
    trigger: str
    context: Dict[str, Any]
    confidence: float

class ImplicitFeedbackDetector:
    """
    Detects implicit feedback from user behavior.
    Runs continuously on the vessel.
    """
    
    def __init__(self):
        self.behavior_history: List[Dict] = []
        self.window_size = 10  # Number of actions to consider
        
    def analyze_action(self, 
                      action: Dict[str, Any],
                      context: Dict[str, Any]) -> Optional[ImplicitFeedback]:
        """
        Analyze user action for implicit feedback.
        Returns detected feedback if any.
        """
        self.behavior_history.append({
            'action': action,
            'context': context,
            'timestamp': datetime.now().isoformat()
        })
        
        # Keep only recent history
        if len(self.behavior_history) > self.window_size:
            self.behavior_history.pop(0)
        
        # Detect patterns
        feedback = None
        
        # Override detection
        if action.get('type') == 'human_override':
            feedback = ImplicitFeedback(
                feedback_type='NEGATIVE',
                trigger='human_override',
                context={
                    'agent_action': action.get('agent_action'),
                    'human_action': action.get('human_action'),
                    'difference': self._compute_action_difference(
                        action.get('agent_action'),
                        action.get('human_action')
                    )
                },
                confidence=0.8
            )
        
        # Ignore detection
        elif action.get('type') == 'ignore_suggestion':
            feedback = ImplicitFeedback(
                feedback_type='NEGATIVE',
                trigger='ignore_suggestion',
                context={
                    'suggestion': action.get('suggestion'),
                    'alternatives': action.get('alternatives', [])
                },
                confidence=0.6
            )
        
        # Repeat detection
        elif action.get('type') == 'repeat_request':
            feedback = ImplicitFeedback(
                feedback_type='UNCERTAIN',
                trigger='repeat_request',
                context={
                    'original_request': action.get('original'),
                    'repeat_count': action.get('repeat_count', 1)
                },
                confidence=0.5
            )
        
        # Timeout detection
        elif action.get('type') == 'action_timeout':
            feedback = ImplicitFeedback(
                feedback_type='NEGATIVE',
                trigger='action_timeout',
                context={
                    'pending_action': action.get('pending_action'),
                    'timeout_duration': action.get('timeout_duration')
                },
                confidence=0.4
            )
        
        # Accept detection (implicit positive)
        elif action.get('type') == 'accept_suggestion':
            feedback = ImplicitFeedback(
                feedback_type='POSITIVE',
                trigger='accept_suggestion',
                context={
                    'suggestion': action.get('suggestion')
                },
                confidence=0.7
            )
        
        return feedback
    
    def _compute_action_difference(self, 
                                  agent_action: Dict,
                                  human_action: Dict) -> Dict:
        """Compute the difference between agent and human actions"""
        if not agent_action or not human_action:
            return {}
        
        difference = {}
        
        # Compare action types
        if agent_action.get('type') != human_action.get('type'):
            difference['type_changed'] = True
            difference['agent_type'] = agent_action.get('type')
            difference['human_type'] = human_action.get('type')
        
        # Compare parameters
        agent_params = agent_action.get('parameters', {})
        human_params = human_action.get('parameters', {})
        
        param_diff = {}
        for key in set(list(agent_params.keys()) + list(human_params.keys())):
            if agent_params.get(key) != human_params.get(key):
                param_diff[key] = {
                    'agent': agent_params.get(key),
                    'human': human_params.get(key)
                }
        
        if param_diff:
            difference['parameter_differences'] = param_diff
        
        return difference


class FeedbackProcessor:
    """
    Processes human feedback for training integration.
    """
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self.feedback_dir = self.repo_path / "feedback"
        self.feedback_dir.mkdir(exist_ok=True)
        
        self.pending_feedback: List[FeedbackEvent] = []
        
    def process_feedback(self, 
                        feedback: FeedbackEvent) -> Dict[str, Any]:
        """
        Process a feedback event for training.
        Returns processed feedback with training value assessment.
        """
        # Classify feedback type
        feedback.type = self._classify_feedback(feedback)
        
        # Compute sentiment
        feedback.sentiment = self._compute_sentiment(feedback)
        
        # Compute urgency
        feedback.urgency = self._compute_urgency(feedback)
        
        # Compute training value
        feedback.training_value = self._compute_training_value(feedback)
        
        # Mark as processed
        feedback.processed = True
        
        # Store feedback
        self._store_feedback(feedback)
        
        # Queue for training if high value
        if feedback.training_value > 0.7:
            self.pending_feedback.append(feedback)
        
        return {
            'type': feedback.type.value,
            'sentiment': feedback.sentiment,
            'urgency': feedback.urgency,
            'training_value': feedback.training_value,
            'queued_for_training': feedback.training_value > 0.7
        }
    
    def _classify_feedback(self, feedback: FeedbackEvent) -> FeedbackType:
        """Classify feedback into type"""
        raw = feedback.raw_data
        
        # Explicit feedback
        if 'rating' in raw:
            if raw['rating'] >= 4:
                return FeedbackType.POSITIVE
            elif raw['rating'] <= 2:
                return FeedbackType.NEGATIVE
            return FeedbackType.NEUTRAL
        
        if 'thumbs' in raw:
            return FeedbackType.POSITIVE if raw['thumbs'] == 'up' else FeedbackType.NEGATIVE
        
        # Corrective feedback
        if 'correction' in raw or 'edited_action' in raw:
            return FeedbackType.CORRECTIVE
        
        # Implicit feedback
        if 'trigger' in raw:
            trigger_type = raw['trigger']
            if trigger_type == 'human_override':
                return FeedbackType.CORRECTIVE
            elif trigger_type == 'ignore_suggestion':
                return FeedbackType.NEGATIVE
            elif trigger_type == 'accept_suggestion':
                return FeedbackType.POSITIVE
            elif trigger_type == 'repeat_request':
                return FeedbackType.UNCERTAIN
        
        return FeedbackType.NEUTRAL
    
    def _compute_sentiment(self, feedback: FeedbackEvent) -> float:
        """Compute sentiment score (-1 to 1)"""
        type_sentiment = {
            FeedbackType.POSITIVE: 0.8,
            FeedbackType.NEGATIVE: -0.6,
            FeedbackType.CORRECTIVE: -0.3,
            FeedbackType.UNCERTAIN: 0.0,
            FeedbackType.NEUTRAL: 0.0
        }
        
        base_sentiment = type_sentiment.get(feedback.type, 0.0)
        
        # Adjust based on raw data
        raw = feedback.raw_data
        
        if 'comment' in raw:
            # Simple sentiment analysis on comment
            comment_sentiment = self._analyze_text_sentiment(raw['comment'])
            base_sentiment = 0.7 * base_sentiment + 0.3 * comment_sentiment
        
        return max(-1.0, min(1.0, base_sentiment))
    
    def _compute_urgency(self, feedback: FeedbackEvent) -> float:
        """Compute urgency score (0 to 1)"""
        raw = feedback.raw_data
        
        urgency = 0.0
        
        # Safety-related feedback is urgent
        if raw.get('safety_related', False):
            urgency += 0.5
        
        # Corrective feedback on active task is urgent
        if feedback.type == FeedbackType.CORRECTIVE:
            urgency += 0.3
        
        # Multiple similar feedbacks indicate urgent issue
        similar_count = self._count_similar_recent(feedback)
        if similar_count > 2:
            urgency += 0.2
        
        return min(1.0, urgency)
    
    def _compute_training_value(self, feedback: FeedbackEvent) -> float:
        """
        Compute training value of this feedback.
        Higher = more valuable for training.
        """
        value = 0.0
        
        # Novelty: Is this a new pattern?
        novelty = self._compute_novelty(feedback)
        value += novelty * 0.3
        
        # Impact: How much does this affect operations?
        impact = self._compute_impact(feedback)
        value += impact * 0.3
        
        # Clarity: Is the feedback unambiguous?
        clarity = self._compute_clarity(feedback)
        value += clarity * 0.2
        
        # Reproducibility: Can we generalize this?
        reproducibility = self._compute_reproducibility(feedback)
        value += reproducibility * 0.2
        
        return min(1.0, value)
    
    def _compute_novelty(self, feedback: FeedbackEvent) -> float:
        """Compute how novel this feedback pattern is"""
        # Check recent history for similar feedback
        similar = self._count_similar_recent(feedback)
        
        # Higher similarity = lower novelty
        novelty = 1.0 / (1.0 + similar * 0.2)
        
        return novelty
    
    def _compute_impact(self, feedback: FeedbackEvent) -> float:
        """Compute operational impact of this feedback"""
        raw = feedback.raw_data
        
        # Corrective feedback on successful task has high impact
        if feedback.type == FeedbackType.CORRECTIVE:
            if raw.get('task_completed', False):
                return 0.9
        
        # Override during active operation has medium-high impact
        if raw.get('trigger') == 'human_override':
            if raw.get('during_operation', False):
                return 0.7
        
        # Positive feedback on novel task has medium impact
        if feedback.type == FeedbackType.POSITIVE:
            if raw.get('task_type') in ['novel', 'complex']:
                return 0.5
        
        return 0.3
    
    def _compute_clarity(self, feedback: FeedbackEvent) -> float:
        """Compute how clear/unambiguous the feedback is"""
        raw = feedback.raw_data
        
        clarity = 0.5  # Base clarity
        
        # Explicit feedback is clearer
        if feedback.channel == FeedbackChannel.EXPLICIT:
            clarity += 0.3
        
        # Has comment/explanation is clearer
        if 'comment' in raw and len(raw['comment']) > 10:
            clarity += 0.2
        
        # Corrective with clear difference is clearer
        if 'difference' in raw:
            clarity += 0.2
        
        return min(1.0, clarity)
    
    def _compute_reproducibility(self, feedback: FeedbackEvent) -> float:
        """Compute how reproducible/generalizable this feedback is"""
        raw = feedback.raw_data
        
        # Domain-general tasks are more reproducible
        domains = raw.get('domains', ['specific'])
        if 'general' in domains or len(domains) > 2:
            return 0.8
        
        # Common task types are more reproducible
        task_type = raw.get('task_type', '')
        common_tasks = ['navigation', 'voice_command', 'perception']
        if task_type in common_tasks:
            return 0.7
        
        return 0.4
    
    def _count_similar_recent(self, feedback: FeedbackEvent) -> int:
        """Count similar feedback in recent history"""
        # Check recent feedback files
        recent_files = list(self.feedback_dir.glob("*.json"))[-100:]
        
        count = 0
        for f in recent_files:
            try:
                data = json.loads(f.read_text())
                if (data.get('type') == feedback.type.value and
                    data.get('task_id') == feedback.task_id):
                    count += 1
            except:
                continue
        
        return count
    
    def _analyze_text_sentiment(self, text: str) -> float:
        """Simple text sentiment analysis"""
        # Placeholder - actual implementation would use NLP
        positive_words = ['good', 'great', 'excellent', 'helpful', 'correct']
        negative_words = ['bad', 'wrong', 'error', 'mistake', 'incorrect']
        
        text_lower = text.lower()
        pos_count = sum(1 for w in positive_words if w in text_lower)
        neg_count = sum(1 for w in negative_words if w in text_lower)
        
        if pos_count + neg_count == 0:
            return 0.0
        
        return (pos_count - neg_count) / (pos_count + neg_count)
    
    def _store_feedback(self, feedback: FeedbackEvent):
        """Store feedback to file"""
        feedback_file = self.feedback_dir / f"{feedback.event_id[:8]}.json"
        
        data = {
            'event_id': feedback.event_id,
            'timestamp': feedback.timestamp,
            'vessel_id': feedback.vessel_id,
            'task_id': feedback.task_id,
            'action_id': feedback.action_id,
            'experience_id': feedback.experience_id,
            'channel': feedback.channel.value,
            'type': feedback.type.value,
            'sentiment': feedback.sentiment,
            'urgency': feedback.urgency,
            'training_value': feedback.training_value,
            'processed': feedback.processed,
            'raw_data': feedback.raw_data
        }
        
        feedback_file.write_text(json.dumps(data, indent=2))
    
    def get_training_batch(self, 
                          min_value: float = 0.5,
                          max_size: int = 100) -> List[FeedbackEvent]:
        """Get batch of high-value feedback for training"""
        batch = [
            f for f in self.pending_feedback
            if f.training_value >= min_value
        ][:max_size]
        
        # Remove from pending
        for f in batch:
            self.pending_feedback.remove(f)
        
        return batch
```

---

## 6. Implementation Specifications

### 6.1 Resource Requirements

```
+------------------------------------------------------------------+
|                    RESOURCE REQUIREMENTS                           |
+------------------------------------------------------------------+
|                                                                   |
|  EXPERIENCE CAPTURE                                               |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  CPU: ~5% of one core for capture loop                      │ |
|  │  Memory: 50-100 MB for buffer (1000 experiences)            │ |
|  │  Storage: 1-5 MB per experience (depends on sensor data)    │ |
|  │  GPU: None for capture                                       │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  CURATION                                                         |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  CPU: ~20% of one core for batch curation                   │ |
|  │  Memory: 200-500 MB for embedding model                     │ |
|  │  Storage: Minimal (just metadata changes)                   │ |
|  │  GPU: Optional for embedding computation                     │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  LOCAL FINE-TUNING (LoRA)                                         |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  GPU: 32-48 GB VRAM (MIG partition)                         │ |
|  │  CPU: 1-2 cores for data loading                            │ |
|  │  Memory: 8-16 GB system RAM                                 │ |
|  │  Storage: 5-10 GB for checkpoints                           │ |
|  │  Duration: 10-30 minutes per training round                 │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  FLEET AGGREGATION                                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Fleet Hub Requirements:                                     │ |
|  │  - CPU: 4+ cores                                             │ |
|  │  - Memory: 16+ GB                                            │ |
|  │  - Storage: 100+ GB for model versions                       │ |
|  │  - Network: Low-latency connection to vessels               │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 6.2 Training Schedule

```
+------------------------------------------------------------------+
|                    TRAINING SCHEDULE                               |
+------------------------------------------------------------------+
|                                                                   |
|  CONTINUOUS (Always Running)                                      |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - Experience capture (every interaction)                    │ |
|  │  - Implicit feedback detection                               │ |
|  │  - Quality scoring                                           │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  PERIODIC (Local on Vessel)                                       |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Every 6 hours:                                              │ |
|  │  - Curation of recent experiences                            │ |
|  │  - Quality filtering                                         │ |
|  │  - Dataset building                                          │ |
|  │                                                             │ |
|  │  Every 24 hours (overnight):                                 │ |
|  │  - Local LoRA fine-tuning                                    │ |
|  │  - Validation and safety checks                              │ |
|  │  - Model update if improved                                  │ |
|  │  - Sync to fleet hub                                         │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  FLEET LEVEL (Centralized)                                        |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Every 48 hours:                                              │ |
|  │  - Aggregate vessel updates                                  │ |
|  │  - Byzantine-robust aggregation                              │ |
|  │  - Cross-domain pattern extraction                           │ |
|  │  - Fleet model distribution                                  │ |
|  │                                                             │ |
|  │  Weekly:                                                     │ |
|  │  - Full fleet knowledge graph update                         │ |
|  │  - Domain translation model refresh                          │ |
|  │  - Reputation recalculation                                  │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  HUMAN-IN-THE-LOOP                                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  Immediate:                                                   │ |
|  │  - Safety-critical corrective feedback                       │ |
|  │  - High-urgency negative feedback                            │ |
|  │                                                             │ |
|  │  Batch (daily):                                               │ |
|  │  - Review of high-value feedback                             │ |
|  │  - Portfolio capture from human innovations                  │ |
|  │  - Quality assurance for fleet model updates                 │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

---

## 7. Integration with Git-Native Architecture

### 7.1 Git Commit Semantics for Training

```
+------------------------------------------------------------------+
|                    GIT COMMIT PATTERNS                            |
+------------------------------------------------------------------+
|                                                                   |
|  EXPERIENCE COMMITS                                               |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  commit abc123...                                             │ |
|  │  Author: vessel-alpha-7 <agent@fleet.io>                    │ |
|  │  Date: 2025-01-15 14:32:00 UTC                               │ |
|  │                                                             │ |
|  │  experience: navigation/docking/failed                      │ |
|  │                                                             │ |
|  │  Task: autonomous_docking                                    │ |
|  │  Success: false                                              │ |
|  │  Quality: 0.72                                               │ |
|  │  Human Feedback: corrective                                  │ |
|  │                                                             │ |
|  │  Details:                                                    │ |
|  │  - Wind speed: 15 knots                                     │ |
|  │  - Confidence before: 0.85                                  │ |
|  │  - Confidence after: 0.62                                   │ |
|  │  - Override: human took control at 2m from dock            │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TRAINING COMMITS                                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  commit def456...                                             │ |
|  │  Author: training-pipeline <pipeline@fleet.io>              │ |
|  │  Date: 2025-01-16 02:15:00 UTC                               │ |
|  │                                                             │ |
|  │  training: LoRA fine-tune v2025.01.16.0215                   │ |
|  │                                                             │ |
|  │  Metrics:                                                    │ |
|  │  - Validation Loss: 0.0342                                  │ |
|  │  - Epochs: 2                                                 │ |
|  │  - Experiences used: 347                                     │ |
|  │  - Duration: 18m 32s                                         │ |
|  │                                                             │ |
|  │  Improvements:                                               │ |
|  │  - Docking success rate: 87% → 92%                          │ |
|  │  - Voice command accuracy: 94% → 96%                        │ |
|  │                                                             │ |
|  │  Config:                                                     │ |
|  │  - LoRA rank: 32                                             │ |
|  │  - Learning rate: 2e-5                                       │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  FLEET AGGREGATION COMMITS                                        |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  commit ghi789...                                             │ |
|  │  Author: fleet-hub <hub@fleet.io>                           │ |
|  │  Date: 2025-01-17 00:00:00 UTC                               │ |
|  │                                                             │ |
|  │  fleet: Aggregation round #47                                │ |
|  │                                                             │ |
|  │  Contributors:                                               │ |
|  │  - 12 vessels submitted updates                             │ |
|  │  - 3 vessels flagged (Byzantine detection)                  │ |
|  │  - Total experiences aggregated: 2,847                      │ |
|  │                                                             │ |
|  │  Patterns Extracted:                                         │ |
|  │  - navigation/wave_response → obstacle_avoidance            │ |
|  │  - voice/noise_handling → general/audio                     │ |
|  │                                                             │ |
|  │  Model Version: fleet_v47.2025.01.17                        │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 7.2 Repository Structure

```
field-captain/
├── .git/
│   └── (git history = agent memory)
├── experiences/
│   ├── daily/
│   │   └── 2025-01-15/
│   │       ├── abc12345.json
│   │       └── ...
│   ├── curated/
│   │   ├── tier1/
│   │   ├── tier2/
│   │   └── tier3/
│   └── archive/
├── tensors/
│   ├── abc12345_camera.pt
│   ├── abc12345_lidar.pt
│   └── ...
├── datasets/
│   ├── navigation_20250115.jsonl
│   ├── voice_20250115.jsonl
│   └── ...
├── models/
│   ├── base/
│   │   └── model.pt
│   ├── latest/
│   │   ├── model.pt
│   │   └── training_metadata.json
│   └── fleet/
│       └── v47/
│           ├── model.pt
│           └── aggregation_metadata.json
├── checkpoints/
│   ├── best/
│   │   └── lora_weights.pt
│   └── latest/
│       └── lora_weights.pt
├── feedback/
│   ├── pending/
│   └── processed/
├── config/
│   ├── capture.yaml
│   ├── curation.yaml
│   ├── training.yaml
│   └── fleet.yaml
└── pipeline/
    ├── capture.py
    ├── curation.py
    ├── finetune.py
    ├── fleet_sync.py
    └── feedback.py
```

---

## Summary

This document specifies the complete training data pipeline for the Jetson Robotics Ecosystem:

1. **Experience Capture**: Every interaction is captured as a structured experience primitive with context, observations, actions, outcomes, and learning signals.

2. **Curation and Quality Filtering**: Raw experiences are filtered, deduplicated, and tiered by quality score. Only high-quality, novel experiences are used for training.

3. **Local Fine-Tuning on Jetson Thor**: LoRA-based fine-tuning runs on MIG-partitioned GPU, using FP4 quantization for memory efficiency. Training happens overnight when the vessel is idle.

4. **Fleet-Level Aggregation**: Byzantine-robust federated learning aggregates improvements across vessels while detecting and filtering malicious or erroneous updates.

5. **Human-in-the-Loop Feedback**: Implicit and explicit feedback is captured, classified, and processed for training. Corrective feedback from humans is especially valuable for improving agent behavior.

The pipeline integrates seamlessly with the git-native agent architecture, using commits as the persistence mechanism and git history as the training dataset.

---

*Document prepared for Iteration 4 of 6*  
*Jetson Robotics Ecosystem Grand Design*  
*Date: January 2025*
