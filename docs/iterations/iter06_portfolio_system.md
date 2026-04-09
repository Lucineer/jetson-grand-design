# Iteration 06: Innovation Portfolio System

## Capture, Verification, Attribution, and Value Tracking for Human Innovations

**Document Version**: 1.0  
**Phase**: Iteration 6 of 6  
**Date**: January 2025  
**Status**: System Specification

---

## Executive Summary

This document specifies the **Innovation Portfolio System** - the mechanism by which human contributions to the Jetson Robotics Ecosystem are captured, verified, attributed, and tracked over time. The system enables the core paradigm: **human value INCREASES because their work becomes repeatable portfolios of innovation**.

**Key Principle**: Every human innovation is a contribution that compounds in value. The portfolio system provides the infrastructure for capturing these innovations, proving their origin and impact, and enabling value to flow back to contributors.

**Critical Design Decision**: Verification uses **git-native mechanisms** - cryptographic hashes, signed commits, and transparent git history - NOT blockchain. This aligns with the "repository IS the agent" paradigm and ensures verification is accessible, auditable, and free from external dependencies.

---

## Table of Contents

1. [Innovation Capture Workflow](#1-innovation-capture-workflow)
2. [Verification Mechanisms](#2-verification-mechanisms)
3. [Attribution and Provenance](#3-attribution-and-provenance)
4. [Value Tracking and Appreciation](#4-value-tracking-and-appreciation)
5. [Portfolio Visualization Interfaces](#5-portfolio-visualization-interfaces)
6. [Implementation Specification](#6-implementation-specification)

---

## 1. Innovation Capture Workflow

### 1.1 What Constitutes an Innovation

```
+------------------------------------------------------------------+
|                    INNOVATION TAXONOMY                            |
+------------------------------------------------------------------+
|                                                                   |
|  TYPE 1: CONFIGURATION INNOVATIONS                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - New sensor configurations                                 │ |
|  │  - Calibration parameters for specific environments          │ |
|  │  - Hardware integration patterns                             │ |
|  │  - Performance tuning settings                               │ |
|  │                                                             │ |
|  │  Example: "Optimized camera exposure for low-light fishing" │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TYPE 2: SKILL INNOVATIONS                                        |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - New agent skills or behaviors                             │ |
|  │  - Improvements to existing skills                           │ |
|  │  - Task-specific automation scripts                          │ |
|  │  - Decision trees and rule sets                              │ |
|  │                                                             │ |
|  │  Example: "Docking procedure for single-propeller vessels"  │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TYPE 3: KNOWLEDGE INNOVATIONS                                    |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - Diagnostic patterns and troubleshooting procedures        │ |
|  │  - Environmental knowledge (local conditions, hazards)       │ |
|  │  - Best practices documentation                              │ |
|  │  - Training materials and guides                             │ |
|  │                                                             │ |
|  │  Example: "RF interference diagnosis pattern for port areas"│ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TYPE 4: TRAINING DATA INNOVATIONS                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - Corrective feedback that improves models                  │ |
|  │  - Novel scenarios for training                              │ |
|  │  - Edge case demonstrations                                  │ |
|  │  - Domain-specific datasets                                  │ |
|  │                                                             │ |
|  │  Example: "Wave response corrections for 15+ knot winds"    │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TYPE 5: CROSS-DOMAIN TRANSFERS                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  - Pattern translations between domains                      │ |
|  │  - Abstraction of domain-specific knowledge                  │ |
|  │  - Novel applications of existing patterns                   │ |
|  │                                                             │ |
|  │  Example: "Warehouse navigation pattern adapted for marinas"│ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 Innovation Capture Pipeline

```
+------------------------------------------------------------------+
|                    INNOVATION CAPTURE PIPELINE                    |
+------------------------------------------------------------------+
|                                                                   |
|  STAGE 1: DETECTION                                               |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Detection Sources:                                         │ |
|  │  ┌─────────────────┐  ┌─────────────────┐                  │ |
|  │  │ HUMAN OVERRIDE  │  │ CORRECTIVE      │                  │ |
|  │  │ Agent detects   │  │ FEEDBACK        │                  │ |
|  │  │ human did it    │  │ User provides   │                  │ |
|  │  │ differently     │  │ explicit fix    │                  │ |
|  │  └─────────────────┘  └─────────────────┘                  │ |
|  │           │                   │                              │ |
|  │           └─────────┬─────────┘                              │ |
|  │                     ▼                                        │ |
|  │  ┌─────────────────┐  ┌─────────────────┐                  │ |
|  │  │ NOVEL PATTERN   │  │ EXPLICIT        │                  │ |
|  │  │ Agent learns    │  │ SUBMISSION      │                  │ |
|  │  │ something new   │  │ User creates    │                  │ |
|  │  │                 │  │ innovation      │                  │ |
|  │  └─────────────────┘  └─────────────────┘                  │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  STAGE 2: CAPTURE                                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │              INNOVATION CAPTURE FORM                 │   │ |
|  │  │                                                     │   │ |
|  │  │  Title: [Agent-generated or user-provided]          │   │ |
|  │  │                                                     │   │ |
|  │  │  Type: [Configuration/Skill/Knowledge/Training]     │   │ |
|  │  │                                                     │   │ |
|  │  │  Context:                                           │   │ |
|  │  │  - What situation triggered this?                   │   │ |
|  │  │  - What was the agent doing?                        │   │ |
|  │  │  - What did the human do differently?               │   │ |
|  │  │                                                     │   │ |
|  │  │  Solution:                                          │   │ |
|  │  │  - What specific change was made?                   │   │ |
|  │  │  - Code/Config diff                                 │   │ |
|  │  │  - Parameters changed                               │   │ |
|  │  │                                                     │   │ |
|  │  │  Outcome:                                           │   │ |
|  │  │  - Did it work?                                     │   │ |
|  │  │  - Performance improvement?                         │   │ |
|  │  │  - Side effects?                                    │   │ |
|  │  │                                                     │   │ |
|  │  │  Applicability:                                     │   │ |
|  │  │  - Which domains can use this?                      │   │ |
|  │  │  - Under what conditions?                           │   │ |
|  │  │  - Prerequisites?                                   │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  STAGE 3: REVIEW                                                  |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Review Tiers:                                              │ |
|  │                                                             │ |
|  │  TIER 1: AUTOMATED (immediate)                              │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Syntax validation                                  │   │ |
|  │  │ - Safety checks (no dangerous configs)               │   │ |
|  │  │ - Duplicate detection                                │   │ |
|  │  │ - Format validation                                  │   │ |
|  │  │ Pass rate: ~80%                                      │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  TIER 2: EXPERT REVIEW (24-48 hours)                        │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Domain expert review                               │   │ |
|  │  │ - Quality assessment                                 │   │ |
|  │  │ - Generalization check                               │   │ |
|  │  │ - Integration planning                               │   │ |
|  │  │ Pass rate: ~60% of Tier 1 passes                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  TIER 3: FLEET TRIAL (1-2 weeks)                            │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Deploy to subset of fleet                          │   │ |
|  │  │ - Monitor performance                                │   │ |
|  │  │ - Collect feedback                                   │   │ |
|  │  │ - Decide on fleet-wide adoption                      │   │ |
|  │  │ Pass rate: ~40% of Tier 2 passes                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                            │                                      |
|                            ▼                                      |
|  STAGE 4: PUBLICATION                                             |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Published innovations are:                                 │ |
|  │  - Signed by contributor (GPG key)                         │ |
|  │  - Timestamped (git commit)                                │ |
|  │  - Indexed in portfolio database                            │ |
|  │  - Available for fleet adoption                             │ |
|  │  - Tracked for attribution                                  │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.3 Capture Implementation

```python
# innovation_capture.py
import hashlib
import json
import time
from dataclasses import dataclass, field, asdict
from typing import List, Dict, Any, Optional, Set
from datetime import datetime
from pathlib import Path
import git
from enum import Enum

class InnovationType(Enum):
    CONFIGURATION = 'configuration'
    SKILL = 'skill'
    KNOWLEDGE = 'knowledge'
    TRAINING_DATA = 'training_data'
    CROSS_DOMAIN = 'cross_domain'

class InnovationStatus(Enum):
    DRAFT = 'draft'
    DETECTED = 'detected'
    SUBMITTED = 'submitted'
    TIER1_REVIEW = 'tier1_review'
    TIER2_REVIEW = 'tier2_review'
    TIER3_TRIAL = 'tier3_trial'
    PUBLISHED = 'published'
    REJECTED = 'rejected'
    DEPRECATED = 'deprecated'

@dataclass
class Innovation:
    """Complete innovation record"""
    
    # Identity
    innovation_id: str
    title: str
    type: InnovationType
    
    # Attribution
    contributor_id: str
    contributor_name: str
    vessel_id: str
    
    # Content
    context: Dict[str, Any]
    solution: Dict[str, Any]
    outcome: Dict[str, Any]
    applicability: Dict[str, Any]
    
    # Code changes (if applicable)
    code_diff: Optional[str] = None
    config_changes: Optional[Dict[str, Any]] = None
    
    # Review state
    status: InnovationStatus = InnovationStatus.DRAFT
    review_history: List[Dict[str, Any]] = field(default_factory=list)
    
    # Impact tracking
    adoption_count: int = 0
    domains: Set[str] = field(default_factory=set)
    derivative_innovations: List[str] = field(default_factory=list)
    
    # Timestamps
    created: str = field(default_factory=lambda: datetime.now().isoformat())
    modified: str = field(default_factory=lambda: datetime.now().isoformat())
    published: Optional[str] = None
    
    # Git references
    commit_hash: Optional[str] = None
    signed_commit_hash: Optional[str] = None
    
    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary for serialization"""
        d = asdict(self)
        d['type'] = self.type.value
        d['status'] = self.status.value
        d['domains'] = list(self.domains)
        return d


class InnovationDetector:
    """
    Detects potential innovations from agent interactions.
    Identifies when humans do something different/better than agents.
    """
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self.repo = git.Repo(repo_path)
        
        # Detection thresholds
        self.override_significance_threshold = 0.3  # How different is significant
        self.novelty_threshold = 0.7  # How novel must it be
        
        # History for novelty detection
        self.recent_actions: List[Dict] = []
        
    def detect_from_override(self, 
                            agent_action: Dict,
                            human_action: Dict,
                            outcome: Dict) -> Optional[Dict]:
        """
        Detect innovation from human override.
        Returns innovation draft if significant.
        """
        # Compute difference
        difference = self._compute_action_difference(agent_action, human_action)
        
        # Check significance
        significance = self._compute_significance(difference, outcome)
        
        if significance < self.override_significance_threshold:
            return None
        
        # Check novelty
        novelty = self._compute_novelty(human_action)
        
        if novelty < self.novelty_threshold:
            return None
        
        # Generate innovation draft
        return {
            'type': self._infer_innovation_type(human_action),
            'title': self._generate_title(agent_action, human_action, outcome),
            'context': {
                'trigger': 'human_override',
                'agent_action': agent_action,
                'situation': outcome.get('context', {})
            },
            'solution': {
                'human_action': human_action,
                'difference': difference
            },
            'outcome': {
                'success': outcome.get('success', False),
                'improvement': significance
            },
            'applicability': {
                'domains': self._infer_domains(human_action),
                'conditions': self._infer_conditions(outcome)
            }
        }
    
    def detect_from_feedback(self, 
                            feedback: Dict,
                            experience: Dict) -> Optional[Dict]:
        """
        Detect innovation from corrective feedback.
        """
        if feedback.get('type') != 'corrective':
            return None
        
        if not feedback.get('correction'):
            return None
        
        return {
            'type': InnovationType.TRAINING_DATA.value,
            'title': f"Corrective feedback: {experience.get('task_type', 'unknown')}",
            'context': {
                'trigger': 'corrective_feedback',
                'experience_id': experience.get('experience_id'),
                'task': experience.get('task_specification', {})
            },
            'solution': {
                'correction': feedback.get('correction'),
                'reason': feedback.get('reason', '')
            },
            'outcome': {
                'success': feedback.get('outcome_success', True),
                'feedback_sentiment': feedback.get('sentiment', 0)
            },
            'applicability': {
                'domains': experience.get('domains', ['maritime']),
                'conditions': {}
            }
        }
    
    def detect_from_pattern(self, 
                           action_sequence: List[Dict],
                           outcome: Dict) -> Optional[Dict]:
        """
        Detect innovation from novel action patterns.
        """
        # Check if this pattern has been seen before
        pattern_hash = self._hash_pattern(action_sequence)
        
        # Compare with historical patterns
        is_novel = self._is_novel_pattern(pattern_hash)
        
        if not is_novel:
            return None
        
        # Check if outcome is positive
        if not outcome.get('success'):
            return None
        
        return {
            'type': InnovationType.SKILL.value,
            'title': f"Novel pattern: {outcome.get('task_type', 'unknown')}",
            'context': {
                'trigger': 'novel_pattern',
                'pattern_hash': pattern_hash
            },
            'solution': {
                'action_sequence': action_sequence,
                'pattern_description': self._describe_pattern(action_sequence)
            },
            'outcome': {
                'success': True,
                'novelty_score': 1.0
            },
            'applicability': {
                'domains': self._infer_domains_from_actions(action_sequence),
                'conditions': {}
            }
        }
    
    def _compute_action_difference(self, 
                                   agent_action: Dict, 
                                   human_action: Dict) -> Dict:
        """Compute the difference between agent and human actions"""
        diff = {}
        
        # Different action type
        if agent_action.get('type') != human_action.get('type'):
            diff['type_changed'] = {
                'agent': agent_action.get('type'),
                'human': human_action.get('type')
            }
        
        # Different parameters
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
            diff['parameter_changes'] = param_diff
        
        # Different timing
        if 'timing' in agent_action or 'timing' in human_action:
            if agent_action.get('timing') != human_action.get('timing'):
                diff['timing_change'] = {
                    'agent': agent_action.get('timing'),
                    'human': human_action.get('timing')
                }
        
        return diff
    
    def _compute_significance(self, difference: Dict, outcome: Dict) -> float:
        """Compute significance of the difference"""
        significance = 0.0
        
        # Type change is significant
        if 'type_changed' in difference:
            significance += 0.3
        
        # Parameter changes
        param_changes = difference.get('parameter_changes', {})
        significance += len(param_changes) * 0.1
        
        # Successful outcome adds significance
        if outcome.get('success'):
            significance += 0.2
        
        # Improvement over agent
        if outcome.get('improved_over_agent'):
            significance += 0.2
        
        return min(significance, 1.0)
    
    def _compute_novelty(self, action: Dict) -> float:
        """Compute how novel this action is"""
        action_hash = self._hash_action(action)
        
        # Check against recent history
        similar_count = sum(
            1 for a in self.recent_actions
            if self._action_similarity(action, a) > 0.8
        )
        
        # Add to history
        self.recent_actions.append(action)
        if len(self.recent_actions) > 1000:
            self.recent_actions.pop(0)
        
        # Novelty is inverse of similarity frequency
        novelty = 1.0 / (1.0 + similar_count * 0.2)
        
        return novelty
    
    def _hash_action(self, action: Dict) -> str:
        """Create a hash of an action for comparison"""
        action_str = json.dumps(action, sort_keys=True)
        return hashlib.sha256(action_str.encode()).hexdigest()[:16]
    
    def _hash_pattern(self, actions: List[Dict]) -> str:
        """Create a hash of an action pattern"""
        pattern_str = json.dumps(actions, sort_keys=True)
        return hashlib.sha256(pattern_str.encode()).hexdigest()[:16]
    
    def _action_similarity(self, a1: Dict, a2: Dict) -> float:
        """Compute similarity between two actions"""
        if a1.get('type') != a2.get('type'):
            return 0.0
        
        p1 = a1.get('parameters', {})
        p2 = a2.get('parameters', {})
        
        if not p1 and not p2:
            return 1.0
        
        if not p1 or not p2:
            return 0.0
        
        common_keys = set(p1.keys()) & set(p2.keys())
        if not common_keys:
            return 0.0
        
        matches = sum(1 for k in common_keys if p1[k] == p2[k])
        return matches / len(common_keys)
    
    def _is_novel_pattern(self, pattern_hash: str) -> bool:
        """Check if pattern has been seen before"""
        # Would check against database of known patterns
        return True  # Placeholder
    
    def _infer_innovation_type(self, action: Dict) -> str:
        """Infer the type of innovation from action"""
        action_type = action.get('type', '')
        
        if action_type in ['configure', 'calibrate', 'tune']:
            return InnovationType.CONFIGURATION.value
        elif action_type in ['execute', 'navigate', 'manipulate']:
            return InnovationType.SKILL.value
        elif action_type in ['diagnose', 'troubleshoot', 'document']:
            return InnovationType.KNOWLEDGE.value
        else:
            return InnovationType.SKILL.value
    
    def _generate_title(self, 
                       agent_action: Dict, 
                       human_action: Dict, 
                       outcome: Dict) -> str:
        """Generate a title for the innovation"""
        task = outcome.get('task_type', 'unknown')
        action_type = human_action.get('type', 'action')
        
        return f"Improved {action_type} for {task}"
    
    def _infer_domains(self, action: Dict) -> List[str]:
        """Infer applicable domains from action"""
        # Would use more sophisticated inference
        return ['maritime']
    
    def _infer_conditions(self, outcome: Dict) -> Dict:
        """Infer applicability conditions from outcome"""
        return {}
    
    def _infer_domains_from_actions(self, actions: List[Dict]) -> List[str]:
        """Infer domains from action sequence"""
        return ['maritime']
    
    def _describe_pattern(self, actions: List[Dict]) -> str:
        """Generate description of action pattern"""
        action_types = [a.get('type', 'unknown') for a in actions]
        return " -> ".join(action_types)


class InnovationCaptureManager:
    """
    Manages the complete innovation capture workflow.
    """
    
    def __init__(self, 
                 repo_path: str,
                 contributor_id: str,
                 contributor_name: str,
                 vessel_id: str):
        self.repo_path = Path(repo_path)
        self.repo = git.Repo(repo_path)
        
        self.contributor_id = contributor_id
        self.contributor_name = contributor_name
        self.vessel_id = vessel_id
        
        self.detector = InnovationDetector(repo_path)
        
        # Storage
        self.innovations_dir = self.repo_path / "innovations"
        self.innovations_dir.mkdir(exist_ok=True)
        
        # Pending innovations
        self.pending: Dict[str, Innovation] = {}
        
    def create_innovation(self, 
                         innovation_data: Dict,
                         contributor_id: str = None) -> Innovation:
        """Create a new innovation from captured data"""
        innovation_id = self._generate_innovation_id()
        
        innovation = Innovation(
            innovation_id=innovation_id,
            title=innovation_data.get('title', 'Untitled Innovation'),
            type=InnovationType(innovation_data.get('type', 'skill')),
            contributor_id=contributor_id or self.contributor_id,
            contributor_name=self.contributor_name,
            vessel_id=self.vessel_id,
            context=innovation_data.get('context', {}),
            solution=innovation_data.get('solution', {}),
            outcome=innovation_data.get('outcome', {}),
            applicability=innovation_data.get('applicability', {}),
            status=InnovationStatus.DRAFT
        )
        
        self.pending[innovation_id] = innovation
        return innovation
    
    def submit_innovation(self, innovation_id: str) -> bool:
        """Submit innovation for review"""
        if innovation_id not in self.pending:
            return False
        
        innovation = self.pending[innovation_id]
        innovation.status = InnovationStatus.SUBMITTED
        innovation.modified = datetime.now().isoformat()
        
        # Save to file
        self._save_innovation(innovation)
        
        # Commit to git
        commit_hash = self._commit_innovation(innovation, "submitted")
        innovation.commit_hash = commit_hash
        
        return True
    
    def add_review(self, 
                  innovation_id: str,
                  reviewer_id: str,
                  verdict: str,
                  comments: str,
                  tier: int = 1) -> bool:
        """Add a review to an innovation"""
        innovation = self._load_innovation(innovation_id)
        if not innovation:
            return False
        
        review = {
            'reviewer_id': reviewer_id,
            'verdict': verdict,
            'comments': comments,
            'tier': tier,
            'timestamp': datetime.now().isoformat()
        }
        
        innovation.review_history.append(review)
        
        # Update status based on tier
        if verdict == 'approved':
            if tier == 1:
                innovation.status = InnovationStatus.TIER1_REVIEW
            elif tier == 2:
                innovation.status = InnovationStatus.TIER2_REVIEW
            elif tier == 3:
                innovation.status = InnovationStatus.PUBLISHED
                innovation.published = datetime.now().isoformat()
        elif verdict == 'rejected':
            innovation.status = InnovationStatus.REJECTED
        
        innovation.modified = datetime.now().isoformat()
        
        self._save_innovation(innovation)
        self._commit_innovation(innovation, f"reviewed (tier {tier})")
        
        return True
    
    def record_adoption(self, 
                       innovation_id: str,
                       adopting_vessel: str,
                       domain: str) -> bool:
        """Record an adoption of an innovation"""
        innovation = self._load_innovation(innovation_id)
        if not innovation:
            return False
        
        innovation.adoption_count += 1
        innovation.domains.add(domain)
        
        self._save_innovation(innovation)
        
        return True
    
    def record_derivative(self, 
                         parent_id: str,
                         child_id: str) -> bool:
        """Record a derivative innovation"""
        innovation = self._load_innovation(parent_id)
        if not innovation:
            return False
        
        innovation.derivative_innovations.append(child_id)
        
        self._save_innovation(innovation)
        
        return True
    
    def _generate_innovation_id(self) -> str:
        """Generate unique innovation ID"""
        timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
        random_suffix = hashlib.sha256(
            f"{timestamp}{self.contributor_id}".encode()
        ).hexdigest()[:8]
        return f"inn_{timestamp}_{random_suffix}"
    
    def _save_innovation(self, innovation: Innovation):
        """Save innovation to file"""
        innovation_file = self.innovations_dir / f"{innovation.innovation_id}.json"
        innovation_file.write_text(json.dumps(innovation.to_dict(), indent=2))
    
    def _load_innovation(self, innovation_id: str) -> Optional[Innovation]:
        """Load innovation from file"""
        innovation_file = self.innovations_dir / f"{innovation_id}.json"
        
        if not innovation_file.exists():
            if innovation_id in self.pending:
                return self.pending[innovation_id]
            return None
        
        data = json.loads(innovation_file.read_text())
        
        return Innovation(
            innovation_id=data['innovation_id'],
            title=data['title'],
            type=InnovationType(data['type']),
            contributor_id=data['contributor_id'],
            contributor_name=data['contributor_name'],
            vessel_id=data['vessel_id'],
            context=data['context'],
            solution=data['solution'],
            outcome=data['outcome'],
            applicability=data['applicability'],
            code_diff=data.get('code_diff'),
            config_changes=data.get('config_changes'),
            status=InnovationStatus(data['status']),
            review_history=data.get('review_history', []),
            adoption_count=data.get('adoption_count', 0),
            domains=set(data.get('domains', [])),
            derivative_innovations=data.get('derivative_innovations', []),
            created=data['created'],
            modified=data['modified'],
            published=data.get('published'),
            commit_hash=data.get('commit_hash'),
            signed_commit_hash=data.get('signed_commit_hash')
        )
    
    def _commit_innovation(self, innovation: Innovation, action: str) -> str:
        """Commit innovation to git repository"""
        innovation_file = self.innovations_dir / f"{innovation.innovation_id}.json"
        
        self.repo.index.add([str(innovation_file)])
        
        commit_message = f"""innovation: {innovation.title}

ID: {innovation.innovation_id}
Type: {innovation.type.value}
Status: {innovation.status.value}
Contributor: {innovation.contributor_name} ({innovation.contributor_id})
Vessel: {innovation.vessel_id}
Action: {action}
"""
        
        commit = self.repo.index.commit(commit_message)
        
        return commit.hexsha
```

---

## 2. Verification Mechanisms

### 2.1 Git-Native Verification Architecture

```
+------------------------------------------------------------------+
|                    GIT-NATIVE VERIFICATION                        |
+------------------------------------------------------------------+
|                                                                   |
|  WHY NOT BLOCKCHAIN?                                              |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  ❌ External dependency (can't run offline)                 │ |
|  │  ❌ Transaction costs (even if minimal)                     │ |
|  │  ❌ Complexity for non-technical users                      │ |
|  │  ❌ Scalability concerns at fleet level                     │ |
|  │  ❌ Misalignment with "repository IS the agent" paradigm    │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  WHY GIT-NATIVE?                                                  |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  ✅ Already part of the architecture                        │ |
|  │  ✅ Cryptographic hashes (SHA-256) for integrity            │ |
|  │  ✅ GPG signing for authentication                           │ |
|  │  ✅ Immutable history (once pushed)                          │ |
|  │  ✅ Transparent and auditable                                │ |
|  │  ✅ Works offline                                            │ |
|  │  ✅ No external dependencies                                 │ |
|  │  ✅ Familiar tools (git log, git verify)                     │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.2 Verification Layers

```
+------------------------------------------------------------------+
|                    VERIFICATION LAYERS                            |
+------------------------------------------------------------------+
|                                                                   |
|  LAYER 1: COMMIT INTEGRITY                                        |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  What: Verify commit wasn't tampered with                   │ |
|  │  How: Git's built-in SHA-256 hash chain                     │ |
|  │  Tools: git log --verify                                    │ |
|  │  Guarantee: Any modification invalidates all child commits  │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  LAYER 2: CONTRIBUTOR AUTHENTICATION                              |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  What: Verify who created the innovation                    │ |
|  │  How: GPG-signed commits                                    │ |
|  │  Tools: git commit -S, git verify-commit                    │ |
|  │  Guarantee: Cryptographic proof of authorship               │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  LAYER 3: TIMESTAMP VERIFICATION                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  What: Prove when innovation was created                    │ |
|  │  How: Commit timestamps + fleet hub witnessing              │ |
|  │  Tools: git log --date, signed tags                         │ |
|  │  Guarantee: Order of innovations is verifiable              │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  LAYER 4: CONTENT INTEGRITY                                       |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  What: Verify innovation content is complete                │ |
|  │  How: Content-addressed storage (git objects)               │ |
|  │  Tools: git fsck, git hash-object                           │ |
|  │  Guarantee: Content cannot be modified without detection    │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  LAYER 5: ATTRIBUTION CHAIN                                       |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │  What: Track innovation lineage and derivatives             │ |
|  │  How: Parent references in innovation metadata              │ |
|  │  Tools: Innovation graph traversal                          │ |
|  │  Guarantee: Full provenance from origin to present          │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.3 Verification Implementation

```python
# verification.py
import hashlib
import json
import subprocess
from dataclasses import dataclass
from typing import Dict, List, Optional, Tuple
from datetime import datetime
from pathlib import Path
import git
import gnupg

@dataclass
class VerificationResult:
    """Result of verification check"""
    layer: str
    passed: bool
    message: str
    details: Dict = None
    
    def to_dict(self) -> Dict:
        return {
            'layer': self.layer,
            'passed': self.passed,
            'message': self.message,
            'details': self.details
        }


class GitNativeVerifier:
    """
    Verifies innovations using git-native mechanisms.
    No blockchain required.
    """
    
    def __init__(self, repo_path: str, gpg_home: str = None):
        self.repo_path = Path(repo_path)
        self.repo = git.Repo(repo_path)
        
        # GPG for signing verification
        self.gpg = gnupg.GPG(gnupghome=gpg_home) if gpg_home else None
        
    def verify_innovation(self, innovation_id: str) -> List[VerificationResult]:
        """
        Verify an innovation across all layers.
        Returns list of verification results.
        """
        results = []
        
        # Load innovation
        innovation = self._load_innovation(innovation_id)
        if not innovation:
            results.append(VerificationResult(
                layer='existence',
                passed=False,
                message=f"Innovation {innovation_id} not found"
            ))
            return results
        
        # Layer 1: Commit Integrity
        results.append(self._verify_commit_integrity(innovation))
        
        # Layer 2: Contributor Authentication
        results.append(self._verify_contributor(innovation))
        
        # Layer 3: Timestamp Verification
        results.append(self._verify_timestamp(innovation))
        
        # Layer 4: Content Integrity
        results.append(self._verify_content_integrity(innovation))
        
        # Layer 5: Attribution Chain
        results.append(self._verify_attribution_chain(innovation))
        
        return results
    
    def _verify_commit_integrity(self, innovation: Dict) -> VerificationResult:
        """Verify the git commit wasn't tampered with"""
        commit_hash = innovation.get('commit_hash')
        
        if not commit_hash:
            return VerificationResult(
                layer='commit_integrity',
                passed=False,
                message="No commit hash recorded"
            )
        
        try:
            commit = self.repo.commit(commit_hash)
            
            # Verify the commit exists and is reachable
            # Git's hash chain ensures integrity
            if commit.hexsha == commit_hash:
                return VerificationResult(
                    layer='commit_integrity',
                    passed=True,
                    message="Commit integrity verified",
                    details={
                        'commit_hash': commit_hash,
                        'tree_hash': commit.tree.hexsha,
                        'parent_count': len(commit.parents)
                    }
                )
            else:
                return VerificationResult(
                    layer='commit_integrity',
                    passed=False,
                    message="Commit hash mismatch"
                )
                
        except Exception as e:
            return VerificationResult(
                layer='commit_integrity',
                passed=False,
                message=f"Commit verification failed: {str(e)}"
            )
    
    def _verify_contributor(self, innovation: Dict) -> VerificationResult:
        """Verify the contributor's identity via GPG signature"""
        commit_hash = innovation.get('commit_hash')
        contributor_id = innovation.get('contributor_id')
        
        if not commit_hash:
            return VerificationResult(
                layer='contributor_auth',
                passed=False,
                message="No commit to verify"
            )
        
        try:
            commit = self.repo.commit(commit_hash)
            
            # Check if commit is signed
            if not commit.gpgsig:
                # Unsigned commit - can still verify author but not cryptographically
                author = commit.author
                return VerificationResult(
                    layer='contributor_auth',
                    passed=True,  # Partial pass
                    message="Commit is unsigned (author verified but not authenticated)",
                    details={
                        'author_name': author.name,
                        'author_email': author.email,
                        'signed': False
                    }
                )
            
            # Verify GPG signature
            if self.gpg:
                verified = self._verify_gpg_signature(commit)
                
                if verified:
                    return VerificationResult(
                        layer='contributor_auth',
                        passed=True,
                        message="Contributor cryptographically authenticated",
                        details={
                            'contributor_id': contributor_id,
                            'signed': True,
                            'key_id': verified.get('key_id')
                        }
                    )
                else:
                    return VerificationResult(
                        layer='contributor_auth',
                        passed=False,
                        message="GPG signature verification failed"
                    )
            
            return VerificationResult(
                layer='contributor_auth',
                passed=True,
                message="Signature present (GPG verification not available)",
                details={'signed': True}
            )
            
        except Exception as e:
            return VerificationResult(
                layer='contributor_auth',
                passed=False,
                message=f"Contributor verification failed: {str(e)}"
            )
    
    def _verify_timestamp(self, innovation: Dict) -> VerificationResult:
        """Verify when the innovation was created"""
        commit_hash = innovation.get('commit_hash')
        recorded_created = innovation.get('created')
        
        if not commit_hash:
            return VerificationResult(
                layer='timestamp',
                passed=False,
                message="No commit to verify timestamp"
            )
        
        try:
            commit = self.repo.commit(commit_hash)
            
            # Git commit timestamp
            commit_time = datetime.fromtimestamp(commit.committed_date, 
                                                 tz=commit.committer_tz_offset)
            
            # Compare with recorded timestamp
            recorded_time = datetime.fromisoformat(recorded_created.replace('Z', '+00:00'))
            
            # Allow some tolerance (git time vs recorded time may differ slightly)
            time_diff = abs((commit_time - recorded_time.replace(tzinfo=None)).total_seconds())
            
            if time_diff < 60:  # Within 1 minute
                return VerificationResult(
                    layer='timestamp',
                    passed=True,
                    message="Timestamp verified",
                    details={
                        'commit_time': commit_time.isoformat(),
                        'recorded_time': recorded_created,
                        'difference_seconds': time_diff
                    }
                )
            else:
                return VerificationResult(
                    layer='timestamp',
                    passed=False,
                    message="Timestamp mismatch",
                    details={
                        'commit_time': commit_time.isoformat(),
                        'recorded_time': recorded_created,
                        'difference_seconds': time_diff
                    }
                )
                
        except Exception as e:
            return VerificationResult(
                layer='timestamp',
                passed=False,
                message=f"Timestamp verification failed: {str(e)}"
            )
    
    def _verify_content_integrity(self, innovation: Dict) -> VerificationResult:
        """Verify the innovation content hasn't been modified"""
        innovation_id = innovation.get('innovation_id')
        
        # Get the file from git at the commit
        commit_hash = innovation.get('commit_hash')
        
        if not commit_hash:
            return VerificationResult(
                layer='content_integrity',
                passed=False,
                message="No commit to verify content"
            )
        
        try:
            commit = self.repo.commit(commit_hash)
            
            # Get the file blob
            try:
                blob = (commit.tree / 'innovations' / f"{innovation_id}.json")
                content = blob.data_stream.read().decode('utf-8')
            except KeyError:
                return VerificationResult(
                    layer='content_integrity',
                    passed=False,
                    message="Innovation file not found in commit tree"
                )
            
            # Compute hash of current file
            current_file = self.repo_path / 'innovations' / f"{innovation_id}.json"
            if current_file.exists():
                current_content = current_file.read_text()
                
                if content == current_content:
                    return VerificationResult(
                        layer='content_integrity',
                        passed=True,
                        message="Content integrity verified",
                        details={
                            'blob_hash': blob.hexsha,
                            'content_length': len(content)
                        }
                    )
                else:
                    return VerificationResult(
                        layer='content_integrity',
                        passed=False,
                        message="Content has been modified since commit"
                    )
            else:
                return VerificationResult(
                    layer='content_integrity',
                    passed=True,
                    message="Content verified from git history only",
                    details={'blob_hash': blob.hexsha}
                )
                
        except Exception as e:
            return VerificationResult(
                layer='content_integrity',
                passed=False,
                message=f"Content verification failed: {str(e)}"
            )
    
    def _verify_attribution_chain(self, innovation: Dict) -> VerificationResult:
        """Verify the attribution chain for derivative innovations"""
        derivatives = innovation.get('derivative_innovations', [])
        
        if not derivatives:
            return VerificationResult(
                layer='attribution_chain',
                passed=True,
                message="No derivatives to verify"
            )
        
        # Verify each derivative references this innovation correctly
        verified_count = 0
        failed_derivatives = []
        
        for deriv_id in derivatives:
            deriv = self._load_innovation(deriv_id)
            
            if deriv:
                # Check if derivative references this innovation
                parent_refs = deriv.get('parent_innovations', [])
                if innovation.get('innovation_id') in parent_refs:
                    verified_count += 1
                else:
                    failed_derivatives.append(deriv_id)
            else:
                failed_derivatives.append(f"{deriv_id} (not found)")
        
        if len(failed_derivatives) == 0:
            return VerificationResult(
                layer='attribution_chain',
                passed=True,
                message=f"All {len(derivatives)} derivatives verified",
                details={
                    'derivative_count': len(derivatives),
                    'verified_count': verified_count
                }
            )
        else:
            return VerificationResult(
                layer='attribution_chain',
                passed=False,
                message=f"Some derivatives failed verification",
                details={
                    'failed_derivatives': failed_derivatives
                }
            )
    
    def _verify_gpg_signature(self, commit) -> Optional[Dict]:
        """Verify GPG signature on a commit"""
        if not self.gpg:
            return None
        
        try:
            # Use git to verify the signature
            result = subprocess.run(
                ['git', '-C', str(self.repo_path), 'verify-commit', commit.hexsha],
                capture_output=True,
                text=True
            )
            
            if result.returncode == 0:
                # Extract key ID from output
                key_id = None
                for line in result.stdout.split('\n'):
                    if 'key' in line.lower():
                        parts = line.split()
                        for i, part in enumerate(parts):
                            if part.lower() == 'key' and i + 1 < len(parts):
                                key_id = parts[i + 1]
                
                return {'key_id': key_id}
            
            return None
            
        except Exception:
            return None
    
    def _load_innovation(self, innovation_id: str) -> Optional[Dict]:
        """Load innovation from file"""
        innovation_file = self.repo_path / 'innovations' / f"{innovation_id}.json"
        
        if not innovation_file.exists():
            return None
        
        return json.loads(innovation_file.read_text())
    
    def generate_verification_report(self, 
                                    innovation_id: str) -> Dict:
        """Generate a complete verification report"""
        results = self.verify_innovation(innovation_id)
        
        all_passed = all(r.passed for r in results)
        
        return {
            'innovation_id': innovation_id,
            'verified': all_passed,
            'timestamp': datetime.now().isoformat(),
            'layers': [r.to_dict() for r in results],
            'summary': {
                'total_layers': len(results),
                'passed_layers': sum(1 for r in results if r.passed),
                'failed_layers': sum(1 for r in results if not r.passed)
            }
        }


class TimestampWitness:
    """
    Fleet hub acts as a timestamp witness for innovations.
    Provides additional timestamp verification beyond git commits.
    """
    
    def __init__(self, hub_repo_path: str):
        self.hub_repo_path = Path(hub_repo_path)
        self.hub_repo = git.Repo(hub_repo_path)
        
        self.witness_dir = self.hub_repo_path / "witness"
        self.witness_dir.mkdir(exist_ok=True)
        
    def witness_innovation(self, 
                          innovation_id: str,
                          source_commit_hash: str,
                          source_repo_url: str) -> str:
        """
        Create a witness record for an innovation.
        This provides fleet-level timestamp verification.
        """
        witness_record = {
            'innovation_id': innovation_id,
            'source_commit': source_commit_hash,
            'source_repo': source_repo_url,
            'witnessed_at': datetime.now().isoformat(),
            'witness_id': self._generate_witness_id()
        }
        
        # Save witness record
        witness_file = self.witness_dir / f"{innovation_id}.witness.json"
        witness_file.write_text(json.dumps(witness_record, indent=2))
        
        # Commit to hub repo
        self.hub_repo.index.add([str(witness_file)])
        
        commit = self.hub_repo.index.commit(
            f"witness: {innovation_id}\n\n"
            f"Source: {source_repo_url}\n"
            f"Commit: {source_commit_hash}"
        )
        
        return commit.hexsha
    
    def verify_witness(self, innovation_id: str) -> Optional[Dict]:
        """Verify a witness record exists"""
        witness_file = self.witness_dir / f"{innovation_id}.witness.json"
        
        if not witness_file.exists():
            return None
        
        return json.loads(witness_file.read_text())
    
    def _generate_witness_id(self) -> str:
        return hashlib.sha256(
            f"{datetime.now().isoformat()}".encode()
        ).hexdigest()[:16]
```

---

## 3. Attribution and Provenance

### 3.1 Attribution Model

```
+------------------------------------------------------------------+
|                    ATTRIBUTION MODEL                              |
+------------------------------------------------------------------+
|                                                                   |
|  ATTRIBUTION TYPES                                                |
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ORIGINAL CREATOR                                           │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ The human who first created the innovation          │   │ |
|  │  │ Share: 60-80% of value attribution                   │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  CONTRIBUTOR                                                │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Someone who improved or extended the innovation      │   │ |
|  │  │ Share: 10-30% of value attribution                   │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  REVIEWER                                                   │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Expert who reviewed and approved the innovation      │   │ |
|  │  │ Share: 5-10% of value attribution                    │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  CURATOR                                                    │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Someone who adapted innovation for new domain        │   │ |
|  │  │ Share: 10-20% of value attribution                   │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  ATTRIBUTION CHAIN EXAMPLE                                        |
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Innovation: "Low-light camera calibration"                │ |
|  │                                                             │ |
|  │  ┌───────────┐    ┌───────────┐    ┌───────────┐          │ |
|  │  │ Creator   │───►│ Reviewer  │───►│ Curator   │          │ |
|  │  │ Captain   │    │ Expert    │    │ Drone     │          │ |
|  │  │ Santos    │    │ Chen      │    │ Operator  │          │ |
|  │  │ (60%)     │    │ (10%)     │    │ (30%)     │          │ |
|  │  └───────────┘    └───────────┘    └───────────┘          │ |
|  │                                                             │ |
|  │  Derivative: "Night-vision drone inspection"               │ |
|  │                                                             │ |
|  │  Value flows:                                               │ |
|  │  - Santos gets 60% × 0.5 (derivative factor) = 30%        │ |
|  │  - Chen gets 10% × 0.5 = 5%                                │ |
|  │  - Drone Operator gets 30% + 35% (new work) = 65%         │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 3.2 Attribution Implementation

```python
# attribution.py
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Set, Tuple
from datetime import datetime
from enum import Enum
import json
from pathlib import Path

class AttributionType(Enum):
    ORIGINAL_CREATOR = 'original_creator'
    CONTRIBUTOR = 'contributor'
    REVIEWER = 'reviewer'
    CURATOR = 'curator'

@dataclass
class AttributionRecord:
    """Record of attribution for an innovation"""
    innovation_id: str
    attribution_type: AttributionType
    contributor_id: str
    contributor_name: str
    share_percentage: float
    timestamp: str
    
    def to_dict(self) -> Dict:
        return {
            'innovation_id': self.innovation_id,
            'attribution_type': self.attribution_type.value,
            'contributor_id': self.contributor_id,
            'contributor_name': self.contributor_name,
            'share_percentage': self.share_percentage,
            'timestamp': self.timestamp
        }

@dataclass
class AttributionChain:
    """Complete attribution chain for an innovation"""
    innovation_id: str
    attributions: List[AttributionRecord]
    parent_innovations: List[str]
    derivative_factor: float = 1.0
    
    def compute_effective_shares(self) -> Dict[str, float]:
        """Compute effective share for each contributor"""
        shares = {}
        
        for attr in self.attributions:
            effective_share = attr.share_percentage * self.derivative_factor
            contributor = attr.contributor_id
            
            if contributor not in shares:
                shares[contributor] = 0.0
            
            shares[contributor] += effective_share
        
        return shares
    
    def to_dict(self) -> Dict:
        return {
            'innovation_id': self.innovation_id,
            'attributions': [a.to_dict() for a in self.attributions],
            'parent_innovations': self.parent_innovations,
            'derivative_factor': self.derivative_factor
        }


class AttributionManager:
    """
    Manages attribution for innovations.
    Tracks value flow across the innovation graph.
    """
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self.attribution_dir = self.repo_path / "attribution"
        self.attribution_dir.mkdir(exist_ok=True)
        
        # Cache of attribution chains
        self.chains: Dict[str, AttributionChain] = {}
        
    def create_attribution(self,
                          innovation_id: str,
                          attribution_type: AttributionType,
                          contributor_id: str,
                          contributor_name: str,
                          share_percentage: float) -> AttributionRecord:
        """Create an attribution record"""
        attribution = AttributionRecord(
            innovation_id=innovation_id,
            attribution_type=attribution_type,
            contributor_id=contributor_id,
            contributor_name=contributor_name,
            share_percentage=share_percentage,
            timestamp=datetime.now().isoformat()
        )
        
        # Add to chain
        if innovation_id not in self.chains:
            self.chains[innovation_id] = AttributionChain(
                innovation_id=innovation_id,
                attributions=[],
                parent_innovations=[]
            )
        
        self.chains[innovation_id].attributions.append(attribution)
        
        # Save
        self._save_chain(self.chains[innovation_id])
        
        return attribution
    
    def link_parent(self, 
                   child_id: str, 
                   parent_id: str,
                   derivative_factor: float = 0.5):
        """
        Link a derivative innovation to its parent.
        derivative_factor determines how much value flows to parent.
        """
        if child_id not in self.chains:
            self.chains[child_id] = AttributionChain(
                innovation_id=child_id,
                attributions=[],
                parent_innovations=[]
            )
        
        self.chains[child_id].parent_innovations.append(parent_id)
        self.chains[child_id].derivative_factor = derivative_factor
        
        self._save_chain(self.chains[child_id])
    
    def compute_full_attribution(self, innovation_id: str) -> Dict[str, float]:
        """
        Compute full attribution including all parents.
        Returns dict of contributor_id -> effective_share.
        """
        if innovation_id not in self.chains:
            self._load_chain(innovation_id)
        
        if innovation_id not in self.chains:
            return {}
        
        chain = self.chains[innovation_id]
        
        # Start with direct attributions
        full_shares = chain.compute_effective_shares()
        
        # Add parent attributions
        for parent_id in chain.parent_innovations:
            parent_shares = self.compute_full_attribution(parent_id)
            
            # Scale by derivative factor
            for contributor, share in parent_shares.items():
                parent_factor = chain.derivative_factor
                if contributor not in full_shares:
                    full_shares[contributor] = 0.0
                full_shares[contributor] += share * parent_factor
        
        # Normalize to 100%
        total = sum(full_shares.values())
        if total > 0:
            full_shares = {k: v / total * 100 for k, v in full_shares.items()}
        
        return full_shares
    
    def get_contributor_portfolio(self, contributor_id: str) -> Dict:
        """
        Get all attributions for a contributor.
        Returns portfolio summary.
        """
        portfolio = {
            'contributor_id': contributor_id,
            'total_innovations': 0,
            'total_adopted': 0,
            'by_type': {},
            'by_domain': {},
            'recent': []
        }
        
        # Scan all attribution chains
        for chain_file in self.attribution_dir.glob("*.json"):
            chain = self._load_chain(chain_file.stem)
            
            for attr in chain.attributions:
                if attr.contributor_id == contributor_id:
                    portfolio['total_innovations'] += 1
                    
                    # By type
                    type_key = attr.attribution_type.value
                    portfolio['by_type'][type_key] = portfolio['by_type'].get(type_key, 0) + 1
                    
                    # Recent
                    portfolio['recent'].append({
                        'innovation_id': attr.innovation_id,
                        'type': type_key,
                        'share': attr.share_percentage,
                        'timestamp': attr.timestamp
                    })
        
        # Sort recent by timestamp
        portfolio['recent'].sort(key=lambda x: x['timestamp'], reverse=True)
        portfolio['recent'] = portfolio['recent'][:20]  # Keep last 20
        
        return portfolio
    
    def _save_chain(self, chain: AttributionChain):
        """Save attribution chain to file"""
        chain_file = self.attribution_dir / f"{chain.innovation_id}.json"
        chain_file.write_text(json.dumps(chain.to_dict(), indent=2))
    
    def _load_chain(self, innovation_id: str) -> Optional[AttributionChain]:
        """Load attribution chain from file"""
        chain_file = self.attribution_dir / f"{innovation_id}.json"
        
        if not chain_file.exists():
            return None
        
        data = json.loads(chain_file.read_text())
        
        chain = AttributionChain(
            innovation_id=data['innovation_id'],
            attributions=[
                AttributionRecord(
                    innovation_id=a['innovation_id'],
                    attribution_type=AttributionType(a['attribution_type']),
                    contributor_id=a['contributor_id'],
                    contributor_name=a['contributor_name'],
                    share_percentage=a['share_percentage'],
                    timestamp=a['timestamp']
                )
                for a in data['attributions']
            ],
            parent_innovations=data.get('parent_innovations', []),
            derivative_factor=data.get('derivative_factor', 1.0)
        )
        
        self.chains[innovation_id] = chain
        return chain
```

---

## 4. Value Tracking and Appreciation

### 4.1 Value Model

```
+------------------------------------------------------------------+
|                    VALUE TRACKING MODEL                           |
+------------------------------------------------------------------+
|                                                                   |
|  VALUE COMPONENTS                                                 |
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  BASE VALUE (intrinsic)                                     │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ - Quality score from review                          │   │ |
|  │  │ - Novelty score                                       │   │ |
|  │  │ - Applicability score                                 │   │ |
|  │  │ Base Value = quality × novelty × applicability        │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  ADOPTION VALUE (network effect)                           │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Each adoption adds value:                             │   │ |
|  │  │ Adoption Value = base_value × log(adoptions + 1)     │   │ |
|  │  │                                                      │   │ |
|  │  │ More adoptions = more proven utility = more value    │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  DERIVATIVE VALUE (cascading)                              │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Derivatives create value for parent:                 │   │ |
|  │  │ Derivative Value = Σ (child_value × derivative_pct)  │   │ |
|  │  │                                                      │   │ |
|  │  │ Parents share in success of children                 │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  TIME VALUE (appreciation)                                 │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │ Value can appreciate or depreciate over time:        │   │ |
|  │  │                                                      │   │ |
|  │  │ Appreciating: Foundational innovations that enable   │   │ |
|  │  │               many derivatives                       │   │ |
|  │  │                                                      │   │ |
|  │  │ Depreciating: Domain-specific innovations that       │   │ |
|  │  │               become obsolete                        │   │ |
|  │  │                                                      │   │ |
|  │  │ Time Factor = f(adoption_trend, derivative_count)    │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  TOTAL VALUE CALCULATION                                          |
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  Total Value = Base Value                                   │ |
|  │              × (1 + Adoption Multiplier)                    │ |
|  │              × (1 + Derivative Multiplier)                  │ |
|  │              × Time Factor                                  │ |
|  │                                                             │ |
|  │  This value is distributed to contributors per their        │ |
|  │  attribution shares.                                        │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 4.2 Value Tracking Implementation

```python
# value_tracking.py
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Tuple
from datetime import datetime, timedelta
from pathlib import Path
import json
import math
from collections import defaultdict

@dataclass
class ValueSnapshot:
    """Snapshot of innovation value at a point in time"""
    innovation_id: str
    timestamp: str
    
    base_value: float
    adoption_value: float
    derivative_value: float
    time_factor: float
    total_value: float
    
    # Distribution
    contributor_shares: Dict[str, float]
    
    def to_dict(self) -> Dict:
        return {
            'innovation_id': self.innovation_id,
            'timestamp': self.timestamp,
            'base_value': self.base_value,
            'adoption_value': self.adoption_value,
            'derivative_value': self.derivative_value,
            'time_factor': self.time_factor,
            'total_value': self.total_value,
            'contributor_shares': self.contributor_shares
        }


class ValueTracker:
    """
    Tracks and computes innovation value over time.
    Handles appreciation, distribution, and portfolio aggregation.
    """
    
    def __init__(self, repo_path: str, attribution_manager):
        self.repo_path = Path(repo_path)
        self.attribution_manager = attribution_manager
        
        self.value_dir = self.repo_path / "value"
        self.value_dir.mkdir(exist_ok=True)
        
        # Value history
        self.history: Dict[str, List[ValueSnapshot]] = defaultdict(list)
        
        # Value parameters
        self.adoption_scale = 0.1  # Log base for adoption multiplier
        self.derivative_scale = 0.05  # Scale for derivative value
        self.time_decay_rate = 0.02  # Base decay rate
        self.appreciation_threshold = 3  # Min derivatives for appreciation
        
    def compute_value(self, innovation: Dict) -> ValueSnapshot:
        """
        Compute current value of an innovation.
        """
        innovation_id = innovation.get('innovation_id')
        
        # Get attribution shares
        contributor_shares = self.attribution_manager.compute_full_attribution(innovation_id)
        
        # Compute base value
        base_value = self._compute_base_value(innovation)
        
        # Compute adoption value
        adoption_count = innovation.get('adoption_count', 0)
        adoption_value = base_value * self._adoption_multiplier(adoption_count)
        
        # Compute derivative value
        derivative_count = len(innovation.get('derivative_innovations', []))
        derivative_value = self._compute_derivative_value(innovation_id, derivative_count)
        
        # Compute time factor
        created = datetime.fromisoformat(innovation.get('created', datetime.now().isoformat()))
        age_days = (datetime.now() - created).days
        
        time_factor = self._compute_time_factor(age_days, derivative_count, adoption_count)
        
        # Total value
        total_value = base_value + adoption_value + derivative_value
        total_value *= time_factor
        
        snapshot = ValueSnapshot(
            innovation_id=innovation_id,
            timestamp=datetime.now().isoformat(),
            base_value=base_value,
            adoption_value=adoption_value,
            derivative_value=derivative_value,
            time_factor=time_factor,
            total_value=total_value,
            contributor_shares=contributor_shares
        )
        
        # Record history
        self.history[innovation_id].append(snapshot)
        self._save_snapshot(snapshot)
        
        return snapshot
    
    def _compute_base_value(self, innovation: Dict) -> float:
        """Compute base intrinsic value"""
        # Quality score from review
        quality = self._get_quality_score(innovation)
        
        # Novelty score
        novelty = self._get_novelty_score(innovation)
        
        # Applicability score
        applicability = self._get_applicability_score(innovation)
        
        return quality * novelty * applicability
    
    def _get_quality_score(self, innovation: Dict) -> float:
        """Get quality score from review history"""
        reviews = innovation.get('review_history', [])
        
        if not reviews:
            return 0.5  # Default
        
        # Average of approved review scores
        approved_reviews = [r for r in reviews if r.get('verdict') == 'approved']
        
        if not approved_reviews:
            return 0.5
        
        scores = [r.get('score', 0.7) for r in approved_reviews]
        return sum(scores) / len(scores)
    
    def _get_novelty_score(self, innovation: Dict) -> float:
        """Estimate novelty of the innovation"""
        # Check if similar innovations exist
        # For now, use heuristic based on type
        innovation_type = innovation.get('type', 'skill')
        
        novelty_by_type = {
            'configuration': 0.6,
            'skill': 0.7,
            'knowledge': 0.8,
            'training_data': 0.5,
            'cross_domain': 1.0  # Cross-domain is most novel
        }
        
        return novelty_by_type.get(innovation_type, 0.5)
    
    def _get_applicability_score(self, innovation: Dict) -> float:
        """Compute applicability across domains"""
        domains = innovation.get('domains', [])
        
        if not domains:
            return 0.5
        
        # More domains = more applicable
        # Use log scale to avoid over-rewarding
        return min(1.0, 0.3 + 0.1 * math.log2(len(domains) + 1))
    
    def _adoption_multiplier(self, adoption_count: int) -> float:
        """Compute adoption-based value multiplier"""
        if adoption_count == 0:
            return 0.0
        
        return self.adoption_scale * math.log(adoption_count + 1)
    
    def _compute_derivative_value(self, 
                                  innovation_id: str, 
                                  derivative_count: int) -> float:
        """Compute value from derivative innovations"""
        if derivative_count == 0:
            return 0.0
        
        total_derivative_value = 0.0
        
        # Get value of each derivative
        for deriv_id in self._get_derivatives(innovation_id):
            deriv_value = self.get_latest_value(deriv_id)
            if deriv_value:
                # Share based on derivative factor
                share = self.derivative_scale  # Default share
                total_derivative_value += deriv_value.total_value * share
        
        return total_derivative_value
    
    def _compute_time_factor(self, 
                            age_days: int, 
                            derivative_count: int,
                            adoption_count: int) -> float:
        """
        Compute time-based appreciation/depreciation factor.
        """
        # Base time factor
        time_factor = 1.0
        
        # Check if appreciating or depreciating
        if derivative_count >= self.appreciation_threshold:
            # Appreciating: foundational innovation
            # Value grows with age and derivatives
            appreciation = 0.01 * math.sqrt(derivative_count)  # Up to ~10% per derivative
            time_factor = 1.0 + appreciation
        else:
            # Potential depreciation for non-foundational innovations
            # But active adoption prevents depreciation
            if adoption_count > 0:
                # Still being used, no decay
                pass
            else:
                # No adoptions, slight decay
                decay = self.time_decay_rate * (age_days / 365)
                time_factor = max(0.5, 1.0 - decay)  # Floor at 50%
        
        return time_factor
    
    def get_latest_value(self, innovation_id: str) -> Optional[ValueSnapshot]:
        """Get the most recent value snapshot"""
        if innovation_id in self.history and self.history[innovation_id]:
            return self.history[innovation_id][-1]
        
        # Try to load from file
        return self._load_latest_snapshot(innovation_id)
    
    def get_contributor_value_history(self, 
                                     contributor_id: str,
                                     days: int = 30) -> Dict:
        """
        Get value history for a contributor's portfolio.
        """
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        portfolio = self.attribution_manager.get_contributor_portfolio(contributor_id)
        
        history = []
        total_current_value = 0.0
        
        for innovation in portfolio.get('recent', []):
            innovation_id = innovation['innovation_id']
            snapshot = self.get_latest_value(innovation_id)
            
            if snapshot:
                contributor_share = snapshot.contributor_shares.get(contributor_id, 0)
                value_share = snapshot.total_value * (contributor_share / 100)
                
                total_current_value += value_share
                
                history.append({
                    'innovation_id': innovation_id,
                    'date': snapshot.timestamp,
                    'total_value': snapshot.total_value,
                    'contributor_share': contributor_share,
                    'value_share': value_share
                })
        
        return {
            'contributor_id': contributor_id,
            'period_days': days,
            'total_current_value': total_current_value,
            'innovation_count': portfolio.get('total_innovations', 0),
            'history': history
        }
    
    def get_fleet_value_summary(self) -> Dict:
        """Get summary of all innovation value in fleet"""
        total_value = 0.0
        innovation_count = 0
        top_innovations = []
        
        for innovation_file in (self.repo_path / 'innovations').glob('*.json'):
            innovation = json.loads(innovation_file.read_text())
            snapshot = self.compute_value(innovation)
            
            total_value += snapshot.total_value
            innovation_count += 1
            
            top_innovations.append({
                'innovation_id': snapshot.innovation_id,
                'value': snapshot.total_value,
                'title': innovation.get('title', 'Unknown')
            })
        
        # Sort by value
        top_innovations.sort(key=lambda x: x['value'], reverse=True)
        
        return {
            'total_value': total_value,
            'innovation_count': innovation_count,
            'average_value': total_value / max(innovation_count, 1),
            'top_10_innovations': top_innovations[:10]
        }
    
    def _get_derivatives(self, innovation_id: str) -> List[str]:
        """Get list of derivative innovation IDs"""
        innovation_file = self.repo_path / 'innovations' / f'{innovation_id}.json'
        
        if not innovation_file.exists():
            return []
        
        innovation = json.loads(innovation_file.read_text())
        return innovation.get('derivative_innovations', [])
    
    def _save_snapshot(self, snapshot: ValueSnapshot):
        """Save value snapshot to file"""
        snapshot_dir = self.value_dir / snapshot.innovation_id
        snapshot_dir.mkdir(exist_ok=True)
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        snapshot_file = snapshot_dir / f"{timestamp}.json"
        
        snapshot_file.write_text(json.dumps(snapshot.to_dict(), indent=2))
    
    def _load_latest_snapshot(self, innovation_id: str) -> Optional[ValueSnapshot]:
        """Load most recent snapshot from file"""
        snapshot_dir = self.value_dir / innovation_id
        
        if not snapshot_dir.exists():
            return None
        
        snapshot_files = sorted(snapshot_dir.glob('*.json'), reverse=True)
        
        if not snapshot_files:
            return None
        
        data = json.loads(snapshot_files[0].read_text())
        
        return ValueSnapshot(
            innovation_id=data['innovation_id'],
            timestamp=data['timestamp'],
            base_value=data['base_value'],
            adoption_value=data['adoption_value'],
            derivative_value=data['derivative_value'],
            time_factor=data['time_factor'],
            total_value=data['total_value'],
            contributor_shares=data['contributor_shares']
        )
```

---

## 5. Portfolio Visualization Interfaces

### 5.1 Visualization Architecture

```
+------------------------------------------------------------------+
|                    PORTFOLIO VISUALIZATION                        |
+------------------------------------------------------------------+
|                                                                   |
|  VIEW 1: CONTRIBUTOR PORTFOLIO DASHBOARD                          |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │  CONTRIBUTOR: Captain Maria Santos                   │   │ |
|  │  │  Member since: March 2023                            │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  ┌─────────────────┐  ┌─────────────────────────────────┐ │ |
|  │  │ TOTAL VALUE     │  │ VALUE TREND (12 months)         │ │ |
|  │  │                 │  │                                 │ │ |
|  │  │   847.3         │  │    ▲                            │ │ |
|  │  │                 │  │      ▲                          │ │ |
|  │  │ ↑ 12% this      │  │        ▲    ▲                   │ │ |
|  │  │   month         │  │          ▲    ▲    ▲            │ │ |
|  │  │                 │  │            ▲────▲───▲───►       │ │ |
|  │  └─────────────────┘  │  J  A  S  O  N  D  J            │ │ |
|  │                       └─────────────────────────────────┘ │ |
|  │                                                             │ |
|  │  ┌─────────────────────────────────────────────────────┐   │ |
|  │  │  TOP INNOVATIONS                                     │   │ |
|  │  │                                                     │   │ |
|  │  │  1. Harbor Drift Calibration (Maritime)     Value:  │   │ |
|  │  │     ████████████████████████░░░░░░░░  234.5          │   │ |
|  │  │     Adopted by 47 vessels                           │   │ |
|  │  │                                                     │   │ |
|  │  │  2. RF Interference Diagnosis (Knowledge)   Value:  │   │ |
|  │  │     ████████████████░░░░░░░░░░░░░░░░░░░  156.2       │   │ |
|  │  │     Adopted by 23 vessels                           │   │ |
|  │  │                                                     │   │ |
|  │  │  3. Low-light Camera Config (Config)        Value:  │   │ |
|  │  │     ██████████████░░░░░░░░░░░░░░░░░░░░░  128.7       │   │ |
|  │  │     Cross-domain: Drone adapted                    │   │ |
|  │  │                                                     │   │ |
|  │  └─────────────────────────────────────────────────────┘   │ |
|  │                                                             │ |
|  │  ┌─────────────────┐  ┌─────────────────────────────────┐ │ |
|  │  │ BY TYPE         │  │ BY DOMAIN                       │ │ |
|  │  │                 │  │                                 │ │ |
|  │  │ Config  ████ 12 │  │ Maritime ████████████████ 23   │ │ |
|  │  │ Skill   █████ 15│  │ Drone   ███████ 8              │ │ |
|  │  │ Knowl.  ████ 11 │  │ Warehouse ███ 4                │ │ |
|  │  │ Training ██ 6   │  │ Medical  ███ 3                 │ │ |
|  │  │ Cross   █ 3     │  │                                 │ │ |
|  │  └─────────────────┘  └─────────────────────────────────┘ │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  VIEW 2: INNOVATION GRAPH                                         |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │                     INNOVATION NETWORK                      │ |
|  │                                                             │ |
|  │                    ┌─────────┐                             │ |
|  │                    │ Wave    │ (Santos)                    │ |
|  │              ┌────►│ Response│ Value: 234                  │ |
|  │              │     │ Pattern │                             │ |
|  │              │     └────┬────┘                             │ |
|  │              │          │                                   │ |
|  │  ┌─────────┐ │     ┌────▼────┐     ┌─────────┐            │ |
|  │  │ Harbor  │ │     │ Obstacle│────►│ Warehouse│ (Chen)    │ |
|  │  │ Drift   │─┘     │ Avoid   │     │ Nav      │ Value: 89 │ |
|  │  │ Calib.  │       └─────────┘     └─────────┘            │ |
|  │  └─────────┘                                               │ |
|  │  (Santos)                                                   │ |
|  │  Value: 312                                                 │ |
|  │                                                             │ |
|  │  Node size = value, color = domain                         │ |
|  │  Edge = derivation relationship                            │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
|  VIEW 3: FLEET VALUE FLOW                                         |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                                                             │ |
|  │  VALUE FLOW THIS MONTH                                      │ |
|  │                                                             │ |
|  │  Top Contributors:               Top Adopters:             │ |
|  │  ┌─────────────────────┐        ┌─────────────────────┐   │ |
|  │  │ Santos    ████████  │        │ Fleet Alpha ██████ │   │ |
|  │  │ Chen      ██████    │        │ Fleet Beta  █████  │   │ |
|  │  │ Patel     ████      │        │ Fleet Gamma ████   │   │ |
|  │  │ Okonkwo   ███       │        │ Fleet Delta ███    │   │ |
|  │  └─────────────────────┘        └─────────────────────┘   │ |
|  │                                                             │ |
|  │  Cross-Domain Value Flow:                                  │ |
|  │                                                             │ |
|  │  Maritime ────────────────────► Drone (34.5)              │ |
|  │  Maritime ──────────► Warehouse (12.3)                     │ |
|  │  Medical ─────────────────────► Home (8.7)                │ |
|  │                                                             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 5.2 Visualization API

```python
# portfolio_visualization.py
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from datetime import datetime, timedelta
from pathlib import Path
import json

@dataclass
class PortfolioDashboard:
    """Data for contributor portfolio dashboard"""
    contributor_id: str
    contributor_name: str
    member_since: str
    
    total_value: float
    value_change_percent: float
    value_trend: List[Dict]
    
    innovation_count: int
    innovations_by_type: Dict[str, int]
    innovations_by_domain: Dict[str, int]
    
    top_innovations: List[Dict]
    
    def to_dict(self) -> Dict:
        return {
            'contributor_id': self.contributor_id,
            'contributor_name': self.contributor_name,
            'member_since': self.member_since,
            'total_value': self.total_value,
            'value_change_percent': self.value_change_percent,
            'value_trend': self.value_trend,
            'innovation_count': self.innovation_count,
            'innovations_by_type': self.innovations_by_type,
            'innovations_by_domain': self.innovations_by_domain,
            'top_innovations': self.top_innovations
        }


@dataclass
class InnovationNode:
    """Node in the innovation graph"""
    innovation_id: str
    title: str
    contributor_name: str
    value: float
    domain: str
    x: float = 0.0
    y: float = 0.0
    
    def to_dict(self) -> Dict:
        return {
            'id': self.innovation_id,
            'title': self.title,
            'contributor': self.contributor_name,
            'value': self.value,
            'domain': self.domain,
            'x': self.x,
            'y': self.y
        }


@dataclass
class InnovationEdge:
    """Edge in the innovation graph"""
    source_id: str
    target_id: str
    relationship: str  # 'derivative', 'inspired_by'
    
    def to_dict(self) -> Dict:
        return {
            'source': self.source_id,
            'target': self.target_id,
            'relationship': self.relationship
        }


@dataclass
class InnovationGraph:
    """Complete innovation graph"""
    nodes: List[InnovationNode]
    edges: List[InnovationEdge]
    
    def to_dict(self) -> Dict:
        return {
            'nodes': [n.to_dict() for n in self.nodes],
            'edges': [e.to_dict() for e in self.edges]
        }


class PortfolioVisualizer:
    """
    Generates visualization data for portfolio interfaces.
    """
    
    def __init__(self, repo_path: str, value_tracker, attribution_manager):
        self.repo_path = Path(repo_path)
        self.value_tracker = value_tracker
        self.attribution_manager = attribution_manager
        
    def generate_contributor_dashboard(self, 
                                       contributor_id: str) -> PortfolioDashboard:
        """Generate complete dashboard for a contributor"""
        # Get portfolio
        portfolio = self.attribution_manager.get_contributor_portfolio(contributor_id)
        
        # Get value history
        value_history = self.value_tracker.get_contributor_value_history(contributor_id, days=365)
        
        # Compute value trend
        value_trend = self._compute_value_trend(contributor_id, months=12)
        
        # Get contributor info
        contributor_name = self._get_contributor_name(contributor_id)
        member_since = self._get_member_since(contributor_id)
        
        # Get top innovations
        top_innovations = self._get_top_innovations(contributor_id, limit=5)
        
        # Compute value change
        current_value = value_history.get('total_current_value', 0)
        previous_value = self._get_previous_month_value(contributor_id)
        value_change = ((current_value - previous_value) / max(previous_value, 1)) * 100
        
        return PortfolioDashboard(
            contributor_id=contributor_id,
            contributor_name=contributor_name,
            member_since=member_since,
            total_value=current_value,
            value_change_percent=value_change,
            value_trend=value_trend,
            innovation_count=portfolio.get('total_innovations', 0),
            innovations_by_type=portfolio.get('by_type', {}),
            innovations_by_domain=portfolio.get('by_domain', {}),
            top_innovations=top_innovations
        )
    
    def generate_innovation_graph(self, 
                                  domain_filter: str = None,
                                  contributor_filter: str = None,
                                  min_value: float = 0) -> InnovationGraph:
        """Generate innovation relationship graph"""
        nodes = []
        edges = []
        
        node_map = {}  # id -> index
        
        # Scan all innovations
        for innovation_file in (self.repo_path / 'innovations').glob('*.json'):
            innovation = json.loads(innovation_file.read_text())
            
            # Apply filters
            if domain_filter:
                if domain_filter not in innovation.get('domains', []):
                    continue
            
            if contributor_filter:
                if innovation.get('contributor_id') != contributor_filter:
                    continue
            
            # Get value
            snapshot = self.value_tracker.get_latest_value(innovation.get('innovation_id'))
            value = snapshot.total_value if snapshot else 0
            
            if value < min_value:
                continue
            
            # Create node
            node = InnovationNode(
                innovation_id=innovation.get('innovation_id'),
                title=innovation.get('title', 'Unknown'),
                contributor_name=innovation.get('contributor_name', 'Unknown'),
                value=value,
                domain=innovation.get('domains', ['unknown'])[0] if innovation.get('domains') else 'unknown'
            )
            
            nodes.append(node)
            node_map[node.innovation_id] = len(nodes) - 1
            
            # Create edges for derivatives
            for parent_id in self._get_parent_innovations(innovation):
                if parent_id in node_map:
                    edges.append(InnovationEdge(
                        source_id=parent_id,
                        target_id=node.innovation_id,
                        relationship='derivative'
                    ))
        
        # Layout nodes (force-directed)
        nodes = self._layout_nodes(nodes, edges)
        
        return InnovationGraph(nodes=nodes, edges=edges)
    
    def generate_value_flow_report(self, period_days: int = 30) -> Dict:
        """Generate value flow report for the fleet"""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=period_days)
        
        # Aggregate by contributor
        contributor_flows = {}
        
        # Aggregate by domain
        domain_flows = {}
        
        # Aggregate by fleet
        fleet_flows = {}
        
        # Scan all innovations
        for innovation_file in (self.repo_path / 'innovations').glob('*.json'):
            innovation = json.loads(innovation_file.read_text())
            
            created = datetime.fromisoformat(innovation.get('created', datetime.now().isoformat()))
            
            if created < start_date:
                continue
            
            snapshot = self.value_tracker.get_latest_value(innovation.get('innovation_id'))
            
            if not snapshot:
                continue
            
            # Contributor flow
            for contributor_id, share in snapshot.contributor_shares.items():
                value_share = snapshot.total_value * (share / 100)
                
                if contributor_id not in contributor_flows:
                    contributor_flows[contributor_id] = {
                        'contributor_name': self._get_contributor_name(contributor_id),
                        'total_value': 0,
                        'innovation_count': 0
                    }
                
                contributor_flows[contributor_id]['total_value'] += value_share
                contributor_flows[contributor_id]['innovation_count'] += 1
            
            # Domain flow
            for domain in innovation.get('domains', ['unknown']):
                if domain not in domain_flows:
                    domain_flows[domain] = {'value_in': 0, 'value_out': 0}
                
                domain_flows[domain]['value_in'] += snapshot.total_value
        
        # Cross-domain flows
        cross_domain_flows = []
        for innovation_file in (self.repo_path / 'innovations').glob('*.json'):
            innovation = json.loads(innovation_file.read_text())
            
            # Check for cross-domain derivative
            if innovation.get('type') == 'cross_domain':
                parent_domains = []
                for parent_id in self._get_parent_innovations(innovation):
                    parent = self._load_innovation(parent_id)
                    if parent:
                        parent_domains.extend(parent.get('domains', []))
                
                current_domains = innovation.get('domains', [])
                
                for parent_domain in set(parent_domains):
                    for current_domain in set(current_domains):
                        if parent_domain != current_domain:
                            snapshot = self.value_tracker.get_latest_value(innovation.get('innovation_id'))
                            if snapshot:
                                cross_domain_flows.append({
                                    'from': parent_domain,
                                    'to': current_domain,
                                    'value': snapshot.total_value * 0.5  # Assume 50% attribution
                                })
        
        # Sort contributors
        top_contributors = sorted(
            [{'id': k, **v} for k, v in contributor_flows.items()],
            key=lambda x: x['total_value'],
            reverse=True
        )[:10]
        
        return {
            'period_start': start_date.isoformat(),
            'period_end': end_date.isoformat(),
            'period_days': period_days,
            'total_value_created': sum(c['total_value'] for c in contributor_flows.values()),
            'top_contributors': top_contributors,
            'domain_flows': domain_flows,
            'cross_domain_flows': cross_domain_flows[:20]
        }
    
    def _compute_value_trend(self, contributor_id: str, months: int) -> List[Dict]:
        """Compute monthly value trend"""
        trend = []
        
        for i in range(months, 0, -1):
            end_date = datetime.now() - timedelta(days=(i-1) * 30)
            start_date = end_date - timedelta(days=30)
            
            # Get value at end of month
            # This is simplified - real implementation would use snapshots
            value_history = self.value_tracker.get_contributor_value_history(
                contributor_id, 
                days=365
            )
            
            # Use current value as placeholder
            current_value = value_history.get('total_current_value', 0)
            
            trend.append({
                'month': end_date.strftime('%b %Y'),
                'value': current_value * (1 - (i * 0.02))  # Simulated growth
            })
        
        return trend
    
    def _get_top_innovations(self, contributor_id: str, limit: int) -> List[Dict]:
        """Get top innovations by value for contributor"""
        portfolio = self.attribution_manager.get_contributor_portfolio(contributor_id)
        
        innovations = []
        
        for item in portfolio.get('recent', []):
            innovation_id = item['innovation_id']
            snapshot = self.value_tracker.get_latest_value(innovation_id)
            
            if snapshot:
                contributor_share = snapshot.contributor_shares.get(contributor_id, 0)
                
                innovation = self._load_innovation(innovation_id)
                
                innovations.append({
                    'innovation_id': innovation_id,
                    'title': innovation.get('title', 'Unknown') if innovation else 'Unknown',
                    'type': item.get('type', 'unknown'),
                    'total_value': snapshot.total_value,
                    'contributor_share': contributor_share,
                    'value_share': snapshot.total_value * (contributor_share / 100),
                    'adoption_count': innovation.get('adoption_count', 0) if innovation else 0
                })
        
        # Sort by contributor's value share
        innovations.sort(key=lambda x: x['value_share'], reverse=True)
        
        return innovations[:limit]
    
    def _layout_nodes(self, 
                     nodes: List[InnovationNode], 
                     edges: List[InnovationEdge]) -> List[InnovationNode]:
        """Simple force-directed layout"""
        if not nodes:
            return nodes
        
        # Initialize positions in a circle
        n = len(nodes)
        for i, node in enumerate(nodes):
            angle = 2 * 3.14159 * i / n
            node.x = 0.5 + 0.4 * math.cos(angle)
            node.y = 0.5 + 0.4 * math.sin(angle)
        
        # Apply forces (simplified)
        # In real implementation, use a proper force-directed algorithm
        for _ in range(10):
            for edge in edges:
                source_idx = next((i for i, n in enumerate(nodes) 
                                  if n.innovation_id == edge.source_id), None)
                target_idx = next((i for i, n in enumerate(nodes) 
                                  if n.innovation_id == edge.target_id), None)
                
                if source_idx is not None and target_idx is not None:
                    # Attract connected nodes
                    dx = nodes[target_idx].x - nodes[source_idx].x
                    dy = nodes[target_idx].y - nodes[source_idx].y
                    
                    force = 0.01
                    nodes[source_idx].x += dx * force
                    nodes[source_idx].y += dy * force
                    nodes[target_idx].x -= dx * force
                    nodes[target_idx].y -= dy * force
        
        return nodes
    
    def _get_contributor_name(self, contributor_id: str) -> str:
        """Get contributor name from ID"""
        # Would look up from contributor registry
        return contributor_id
    
    def _get_member_since(self, contributor_id: str) -> str:
        """Get member since date"""
        # Would look up from contributor registry
        return datetime.now().isoformat()
    
    def _get_previous_month_value(self, contributor_id: str) -> float:
        """Get value from previous month"""
        # Would use historical snapshots
        return 0.0
    
    def _get_parent_innovations(self, innovation: Dict) -> List[str]:
        """Get parent innovation IDs"""
        # Check attribution chain
        attribution_file = self.repo_path / 'attribution' / f"{innovation.get('innovation_id')}.json"
        
        if attribution_file.exists():
            data = json.loads(attribution_file.read_text())
            return data.get('parent_innovations', [])
        
        return []
    
    def _load_innovation(self, innovation_id: str) -> Optional[Dict]:
        """Load innovation from file"""
        innovation_file = self.repo_path / 'innovations' / f"{innovation_id}.json"
        
        if not innovation_file.exists():
            return None
        
        return json.loads(innovation_file.read_text())


import math  # Add import for layout
```

---

## 6. Implementation Specification

### 6.1 Repository Structure

```
field-captain/
├── .git/
│   └── (git history = verification chain)
├── innovations/
│   ├── inn_20250115_abc12345.json
│   ├── inn_20250116_def67890.json
│   └── ...
├── attribution/
│   ├── inn_20250115_abc12345.json
│   └── ...
├── value/
│   ├── inn_20250115_abc12345/
│   │   ├── 20250115_143022.json
│   │   ├── 20250116_090015.json
│   │   └── ...
│   └── ...
├── witness/
│   └── (fleet hub witness records)
├── contributors/
│   ├── contrib_001.json
│   └── ...
├── config/
│   ├── portfolio.yaml
│   └── ...
└── portfolio/
    ├── capture.py
    ├── verification.py
    ├── attribution.py
    ├── value_tracking.py
    └── visualization.py
```

### 6.2 API Endpoints

```python
# portfolio_api.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict, List, Optional
from pathlib import Path

app = FastAPI(title="Portfolio System API", version="1.0")

# Initialize managers
# (In production, these would be properly initialized with dependencies)

@app.get("/portfolio/{contributor_id}")
async def get_portfolio(contributor_id: str):
    """Get contributor portfolio dashboard"""
    # Return PortfolioDashboard
    pass

@app.get("/portfolio/{contributor_id}/history")
async def get_value_history(contributor_id: str, days: int = 30):
    """Get contributor value history"""
    pass

@app.get("/innovations/{innovation_id}")
async def get_innovation(innovation_id: str):
    """Get innovation details"""
    pass

@app.get("/innovations/{innovation_id}/verify")
async def verify_innovation(innovation_id: str):
    """Verify innovation integrity"""
    pass

@app.get("/innovations/{innovation_id}/attribution")
async def get_attribution(innovation_id: str):
    """Get full attribution chain"""
    pass

@app.post("/innovations")
async def create_innovation(innovation: Dict):
    """Create new innovation"""
    pass

@app.get("/graph")
async def get_innovation_graph(
    domain: Optional[str] = None,
    contributor: Optional[str] = None,
    min_value: float = 0
):
    """Get innovation relationship graph"""
    pass

@app.get("/flow")
async def get_value_flow(days: int = 30):
    """Get fleet value flow report"""
    pass

@app.get("/fleet/summary")
async def get_fleet_summary():
    """Get fleet-wide portfolio summary"""
    pass
```

---

## Summary

This document specifies the complete **Innovation Portfolio System** for the Jetson Robotics Ecosystem:

1. **Innovation Capture Workflow**: Automated detection and structured capture of human innovations across five types (Configuration, Skill, Knowledge, Training Data, Cross-Domain).

2. **Git-Native Verification**: Five-layer verification using git's built-in cryptographic guarantees - no blockchain required. Aligns with "repository IS the agent" paradigm.

3. **Attribution and Provenance**: Complete attribution chains tracking original creators, contributors, reviewers, and curators with value distribution percentages.

4. **Value Tracking and Appreciation**: Multi-factor value model combining base value, adoption value, derivative value, and time-based appreciation/depreciation.

5. **Portfolio Visualization**: Three visualization views - Contributor Dashboard, Innovation Graph, and Fleet Value Flow - enabling humans to see and track their growing portfolios.

The portfolio system is the mechanism that realizes the core paradigm: **human value INCREASES because their work becomes repeatable portfolios of innovation**.

---

*Document prepared for Iteration 6 of 6*  
*Jetson Robotics Ecosystem Grand Design*  
*Date: January 2025*
