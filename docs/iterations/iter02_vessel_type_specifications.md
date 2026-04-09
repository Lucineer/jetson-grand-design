# Iteration 2: Vessel Type Specifications

## Domain-Specific Configurations for the Jetson Robotics Ecosystem

**Document Version**: 1.0  
**Iteration**: 02  
**Date**: January 2025  
**Status**: Architectural Specification

---

## Executive Summary

This iteration defines the five primary vessel types in the Jetson Robotics Ecosystem. Each vessel type represents a domain-specific configuration of the ROM-emulation architecture, with tailored capabilities, sensors, and operational patterns.

### Vessel Type Taxonomy

```
+------------------------------------------------------------------+
|                    VESSEL TYPE HIERARCHY                           |
+------------------------------------------------------------------+
|                                                                   |
|                         Software Vessel                           |
|                              │                                    |
|           ┌──────────────────┼──────────────────┐                |
|           │                  │                  │                |
|      ┌────▼────┐       ┌────▼────┐       ┌────▼────┐            |
|      │ MARINE  │       │ AERIAL  │       │INDUSTRIAL│            |
|      │ VESSEL  │       │ VESSEL  │       │ VESSEL  │            |
|      └────┬────┘       └────┬────┘       └────┬────┘            |
|           │                  │                  │                |
|      ┌────┴────┐       ┌────┴────┐       ┌────┴────┐            |
|      │ Boats   │       │ Drones  │       │Warehouse│            |
|      │ Ships   │       │ UAVs    │       │Factory  │            |
|      │ ROVs    │       │ eVTOLs  │       │Logistics│            |
|      └─────────┘       └─────────┘       └─────────┘            |
|                                                                   |
|           ┌──────────────────┼──────────────────┐                |
|           │                  │                  │                |
|      ┌────▼────┐       ┌────▼────┐                             |
|      │  HOME   │       │ MEDICAL │                             |
|      │ VESSEL  │       │ VESSEL  │                             |
|      └────┬────┘       └────┬────┘                             |
|           │                  │                                   |
|      ┌────┴────┐       ┌────┴────┐                             |
|      │Assistants│       │Procedure│                             |
|      │ Chefs   │       │Diagnostic│                            |
|      │Cleaners │       │Rehab    │                             |
|      └─────────┘       └─────────┘                             |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Part 1: Marine Vessels

### 1.1 Overview

Marine vessels represent the primary initial domain for the Jetson Robotics Ecosystem. The challenging maritime environment—saltwater, weather, connectivity limitations—makes it an ideal proving ground for the offline-first, git-native architecture.

```
+------------------------------------------------------------------+
|                    MARINE VESSEL ARCHITECTURE                      |
+------------------------------------------------------------------+
|                                                                   |
|  Physical Vessel (Boat/Ship/ROV)                                  |
|  ────────────────────────────────                                 |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    SENSORS & ACTUATORS                       │ |
|  │                                                              │ |
|  │  Navigation              Perception              Control      │ |
|  │  ┌───────────┐          ┌───────────┐          ┌──────────┐ │ |
|  │  │ GPS      │          │ Cameras   │          │ Autopilot│ │ |
|  │  │ Compass  │          │ (360°)    │          │ Throttle │ │ |
|  │  │ AIS      │          │ Sonar     │          │ Rudder   │ │ |
|  │  │ Radar    │          │ Lidar     │          │ Winches  │ │ |
|  │  │ Depth    │          │ Thermal   │          │ Lighting │ │ |
|  │  └─────┬─────┘          └─────┬─────┘          └────┬─────┘ │ |
|  │        │                      │                      │       │ |
|  └────────┼──────────────────────┼──────────────────────┼───────┘ |
|           │                      │                      │         |
|           └──────────────────────┴──────────────────────┘         |
|                                  │                                |
|  Software Vessel (Jetson Thor)   │                                |
|  ─────────────────────────────   │                                |
|                                  ▼                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    ROM BASE LAYER                            │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │ Navigation Agent │ Species Agent │ Safety Agent         ││ |
|  │  │ (Path planning)  │ (ID & log)    │ (Collision avoid)    ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                                                              │ |
|  │                    APPLICATION LAYER                         │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │ capitaine (memory) + field-captain (voice/vision)      ││ |
|  │  │ + nexus-runtime (fleet coordination)                    ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 Marine Vessel Types

```yaml
# vessel-types/marine.yaml
# Marine vessel type specifications

marine:
  
  # ═══════════════════════════════════════════════════════════════
  # COMMERCIAL FISHING VESSEL
  # ═══════════════════════════════════════════════════════════════
  fishing_vessel:
    display_name: "Commercial Fishing Vessel"
    description: |
      Primary workhorse for commercial fishing operations.
      Emphasis on species identification, catch logging, and navigation.
      
    hardware:
      sensors:
        - type: "camera_array"
          count: 4
          specs: "4K IP67, 360° coverage"
          
        - type: "sonar"
          model: "garmin-panoptix"
          purpose: "fish detection, depth"
          
        - type: "gps"
          model: "garmin-gpsmap-8612"
          features: ["ais", "charts"]
          
        - type: "compass"
          model: "fluxgate"
          interface: "nmea_2000"
          
        - type: "weather_station"
          model: "raymarine-aetemp"
          measurements: ["wind", "temp", "pressure"]
          
      actuators:
        - type: "autopilot"
          model: "raymarine-ev100"
          interface: "nmea_2000"
          
        - type: "winch_control"
          interface: "modbus"
          
    software:
      agents:
        - name: "navigation_agent"
          capabilities:
            - "route_optimization"
            - "weather_aware_routing"
            - "collision_avoidance"
            - "ais_tracking"
            
        - name: "species_agent"
          capabilities:
            - "fish_identification"
            - "catch_logging"
            - "regulation_compliance"
            - "market_price_tracking"
            
        - name: "safety_agent"
          capabilities:
            - "man_overboard_detection"
            - "weather_alerts"
            - "emergency_coordination"
            
      models:
        llm: "Qwen2.5-32B-Instruct-FP4"
        vision: "fishinglog-ai-v2"
        stt: "whisper-large-v3"
        
      features:
        - "voice_interface"
        - "offline_navigation"
        - "catch_report_generation"
        - "fleet_communication"
        
    operational:
      typical_offline_duration: "14 days"
      update_window: "port_visits"
      human_oversight: "critical_decisions"
      
  # ═══════════════════════════════════════════════════════════════
  # RESEARCH VESSEL
  # ═══════════════════════════════════════════════════════════════
  research_vessel:
    display_name: "Research Vessel"
    description: |
      Scientific research platform with emphasis on data collection,
      sample tracking, and collaboration with research institutions.
      
    hardware:
      sensors:
        - type: "camera_array"
          count: 8
          specs: "4K IP68, stereo depth pairs"
          
        - type: "ctd_sensor"
          model: "seabird-sbe19plus"
          measurements: ["conductivity", "temperature", "depth"]
          
        - type: "plankton_imager"
          model: "flowcytobot"
          
        - type: "water_sampler"
          model: "niskin_bottles"
          interface: "modbus"
          
    software:
      agents:
        - name: "science_agent"
          capabilities:
            - "sample_tracking"
            - "data_quality_control"
            - "protocol_guidance"
            - "collaboration_sync"
            
      models:
        llm: "Qwen2.5-72B-Instruct-FP4"  # Larger for science
        vision: "marine-species-v3"
        
  # ═══════════════════════════════════════════════════════════════
  # AUTONOMOUS SURFACE VEHICLE (ASV)
  # ═══════════════════════════════════════════════════════════════
  asv:
    display_name: "Autonomous Surface Vehicle"
    description: |
      Unmanned surface vessel for surveying, monitoring, and
      light payload delivery. Extended autonomous operation.
      
    hardware:
      sensors:
        - type: "lidar"
          model: "velodyne-vlp16"
          
        - type: "camera"
          count: 2
          specs: "Stereo pair, 1080p"
          
        - type: "imu"
          model: "xsens-mti-680g"
          
    software:
      agents:
        - name: "autonomy_agent"
          capabilities:
            - "fully_autonomous_navigation"
            - "mission_planning"
            - "obstacle_avoidance"
            - "energy_optimization"
            
      autonomy_level: 4  # L4: High autonomy, human supervises
      
  # ═══════════════════════════════════════════════════════════════
  # REMOTELY OPERATED VEHICLE (ROV)
  # ═══════════════════════════════════════════════════════════════
  rov:
    display_name: "Remotely Operated Vehicle"
    description: |
      Underwater robot for inspection, maintenance, and
      deep-water operations. Tethered to support vessel.
      
    hardware:
      sensors:
        - type: "camera"
          count: 4
          specs: "4K, depth rated 6000m"
          
        - type: "sonar"
          model: "imaging_sonar"
          
        - type: "manipulator"
          model: "schilling-orion"
          interface: "rs485"
          
    software:
      agents:
        - name: "pilot_agent"
          capabilities:
            - "station_keeping"
            - "obstacle_avoidance"
            - "manipulator_assist"
            - "task_guidance"
```

### 1.3 Marine-Specific Agent Behaviors

```python
# marine_agents.py
"""
Marine-specific agent behaviors
Extends base capitaine agent framework
"""

from capitaine import GitNativeAgent, Heartbeat
from typing import Dict, Any, List
from dataclasses import dataclass
import numpy as np
from datetime import datetime

@dataclass
class NavigationContext:
    """Current navigation state"""
    position: tuple  # (lat, lon)
    heading: float
    speed: float
    destination: tuple
    eta: datetime
    
@dataclass
class WeatherConditions:
    """Current weather state"""
    wind_speed: float
    wind_direction: float
    wave_height: float
    visibility: float
    barometric_pressure: float

class NavigationAgent(GitNativeAgent):
    """
    Marine navigation agent
    Handles route planning, collision avoidance, weather routing
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        self.nav_context = None
        self.weather = None
        self.ais_targets = []
        self.route_history = []
        
    def perceive(self) -> Dict[str, Any]:
        """Gather navigation-related observations"""
        return {
            "timestamp": datetime.now().isoformat(),
            "position": self._get_gps_position(),
            "heading": self._get_heading(),
            "speed": self._get_speed(),
            "ais_targets": self._get_ais_targets(),
            "weather": self._get_weather(),
            "depth": self._get_depth(),
            "route_progress": self._calculate_route_progress(),
        }
        
    def reason(self, observations: Dict) -> Dict[str, Any]:
        """
        Process observations using local LLM
        Focus on safety and optimization
        """
        # Build context for LLM
        context = self._build_navigation_context(observations)
        
        # Generate reasoning
        reasoning = self.llm.generate(
            prompt=self._format_navigation_prompt(context),
            max_tokens=1024,
            temperature=0.3  # Lower temp for navigation decisions
        )
        
        # Parse into structured decisions
        decisions = {
            "heading_adjustment": self._extract_heading_adjustment(reasoning),
            "speed_recommendation": self._extract_speed_recommendation(reasoning),
            "collision_risks": self._assess_collision_risks(observations),
            "weather_concerns": self._assess_weather(observations),
            "route_deviation": self._check_route_deviation(observations),
        }
        
        return decisions
        
    def act(self, decisions: Dict) -> List[Dict[str, Any]]:
        """Execute navigation actions"""
        actions = []
        
        # Critical: Collision avoidance
        if decisions["collision_risks"]:
            for risk in decisions["collision_risks"]:
                if risk["severity"] > 0.7:
                    actions.append({
                        "type": "collision_avoidance",
                        "target": risk["target_mmsi"],
                        "maneuver": risk["recommended_maneuver"],
                        "priority": "critical"
                    })
                    
        # Weather routing
        if decisions["weather_concerns"]:
            actions.append({
                "type": "weather_routing",
                "concerns": decisions["weather_concerns"],
                "alternative_route": self._calculate_alternate_route(decisions),
                "priority": "high"
            })
            
        # Standard navigation
        if decisions["heading_adjustment"]:
            actions.append({
                "type": "heading_adjustment",
                "new_heading": decisions["heading_adjustment"],
                "reason": decisions["heading_adjustment_reason"],
                "priority": "normal"
            })
            
        return actions
        
    def commit_navigation_experience(self, experience: Dict):
        """
        Commit navigation experience to git memory
        Enables learning from past voyages
        """
        # Create memory entry
        memory = {
            "type": "navigation_experience",
            "timestamp": datetime.now().isoformat(),
            "position": experience["position"],
            "conditions": experience["conditions"],
            "actions": experience["actions"],
            "outcome": experience["outcome"],
            "human_feedback": experience.get("human_feedback")
        }
        
        # Store in git
        self.memory.store_memory(
            content=json.dumps(memory),
            metadata={"type": "navigation", "importance": experience.get("importance", "normal")}
        )


class SpeciesAgent(GitNativeAgent):
    """
    Marine species identification and logging agent
    Supports fishing, research, and conservation operations
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        self.vision_model = self._load_vision_model()
        self.species_database = self._load_species_database()
        self.catch_log = []
        
    def identify_species(self, image: np.ndarray) -> Dict[str, Any]:
        """
        Identify species from image
        Returns identification with confidence
        """
        # Run vision model
        detections = self.vision_model.detect(image)
        
        results = []
        for detection in detections:
            species_id = detection["class_id"]
            confidence = detection["confidence"]
            
            # Look up species info
            species_info = self.species_database.get(species_id, {})
            
            results.append({
                "species_id": species_id,
                "common_name": species_info.get("common_name", "Unknown"),
                "scientific_name": species_info.get("scientific_name", ""),
                "confidence": confidence,
                "regulations": species_info.get("regulations", {}),
                "market_price": species_info.get("market_price", None),
                "bbox": detection["bbox"]
            })
            
        # Sort by confidence
        results.sort(key=lambda x: x["confidence"], reverse=True)
        
        return {
            "identifications": results,
            "primary": results[0] if results else None,
            "timestamp": datetime.now().isoformat()
        }
        
    def log_catch(self, identification: Dict, weight: float, 
                  location: tuple, human_verified: bool = False):
        """
        Log catch to git memory and export for regulations
        """
        catch_entry = {
            "catch_id": self._generate_catch_id(),
            "timestamp": datetime.now().isoformat(),
            "species": identification["primary"],
            "weight_kg": weight,
            "location": location,
            "verified": human_verified,
            "agent_version": self._get_version()
        }
        
        # Add to local catch log
        self.catch_log.append(catch_entry)
        
        # Commit to git memory
        self.memory.store_memory(
            content=json.dumps(catch_entry),
            metadata={
                "type": "catch_log",
                "species": identification["primary"]["common_name"],
                "weight": weight
            }
        )
        
        return catch_entry
        
    def generate_catch_report(self, format: str = "csv") -> str:
        """
        Generate catch report for regulatory compliance
        """
        if format == "csv":
            lines = ["timestamp,species,weight_kg,lat,lon,verified"]
            for entry in self.catch_log:
                lines.append(
                    f"{entry['timestamp']},{entry['species']['common_name']},"
                    f"{entry['weight_kg']},{entry['location'][0]},{entry['location'][1]},"
                    f"{entry['verified']}"
                )
            return "\n".join(lines)
        # Other formats...


class SafetyAgent(GitNativeAgent):
    """
    Marine safety agent
    Handles emergency detection, coordination, and training
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        self.emergency_state = "normal"
        self.crew_manifest = []
        self.safety_protocols = self._load_safety_protocols()
        
    def perceive(self) -> Dict[str, Any]:
        """Monitor safety-related sensors"""
        return {
            "man_overboard_beacons": self._check_mob_beacons(),
            "life_jacket_status": self._check_life_jackets(),
            "fire_sensors": self._check_fire_sensors(),
            "bilge_sensors": self._check_bilge(),
            "weather_alerts": self._check_weather_alerts(),
            "crew_locations": self._get_crew_locations(),
        }
        
    def detect_emergency(self, observations: Dict) -> Dict[str, Any]:
        """
        Detect emergency conditions
        Returns emergency type and severity
        """
        emergencies = []
        
        # Man overboard
        mob = observations.get("man_overboard_beacons", [])
        for beacon in mob:
            if beacon["activated"]:
                emergencies.append({
                    "type": "man_overboard",
                    "severity": "critical",
                    "location": beacon["location"],
                    "person_id": beacon["id"]
                })
                
        # Fire
        fire_sensors = observations.get("fire_sensors", [])
        for sensor in fire_sensors:
            if sensor["triggered"]:
                emergencies.append({
                    "type": "fire",
                    "severity": "critical",
                    "location": sensor["location"]
                })
                
        # Weather
        weather_alerts = observations.get("weather_alerts", [])
        for alert in weather_alerts:
            if alert["severity"] >= 0.8:
                emergencies.append({
                    "type": "weather",
                    "severity": "high",
                    "alert": alert
                })
                
        return {
            "has_emergency": len(emergencies) > 0,
            "emergencies": emergencies,
            "recommendations": self._generate_recommendations(emergencies)
        }
        
    def coordinate_response(self, emergency: Dict) -> List[Dict]:
        """
        Coordinate emergency response
        Returns action sequence
        """
        actions = []
        
        if emergency["type"] == "man_overboard":
            actions.extend([
                {"type": "alert_crew", "message": "MAN OVERBOARD"},
                {"type": "mark_position", "location": emergency["location"]},
                {"type": "turn_vessel", "maneuver": "williamson_turn"},
                {"type": "launch_rescue", "equipment": "life_ring"},
                {"type": "contact_coast_guard", "situation": emergency}
            ])
            
        elif emergency["type"] == "fire":
            actions.extend([
                {"type": "alert_crew", "message": "FIRE EMERGENCY"},
                {"type": "activate_suppression", "zone": emergency["location"]},
                {"type": "evacuate_zone", "zone": emergency["location"]},
                {"type": "contact_coast_guard", "situation": emergency}
            ])
            
        # Commit emergency to memory
        self.memory.store_memory(
            content=json.dumps({
                "emergency": emergency,
                "response": actions,
                "timestamp": datetime.now().isoformat()
            }),
            metadata={"type": "emergency", "severity": emergency["severity"]}
        )
        
        return actions
```

---

## Part 2: Aerial Vessels

### 2.1 Overview

Aerial vessels encompass drones, UAVs, and eVTOLs operating in three-dimensional space. The key challenges are real-time control latency, flight safety, and regulatory compliance.

```
+------------------------------------------------------------------+
|                    AERIAL VESSEL ARCHITECTURE                      |
+------------------------------------------------------------------+
|                                                                   |
|  Physical Vessel (Drone/UAV/eVTOL)                                |
|  ─────────────────────────────────                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    SENSORS & ACTUATORS                       │ |
|  │                                                              │ |
|  │  Flight Control          Perception             Payload      │ |
|  │  ┌───────────┐          ┌───────────┐          ┌──────────┐ │ |
|  │  │ IMU       │          │ Cameras   │          │ Gimbal   │ │ |
|  │  │ GPS       │          │ (Stereo)  │          │ Cargo    │ │ |
|  │  │ Barometer │          │ Lidar     │          │ Sensors  │ │ |
|  │  │ Compass   │          │ Thermal   │          │ Comms    │ │ |
|  │  │ Airspeed  │          │ Ultrasonic│          │          │ │ |
|  │  └─────┬─────┘          └─────┬─────┘          └────┬─────┘ │ |
|  └────────┼──────────────────────┼──────────────────────┼───────┘ |
|           │                      │                      │         |
|           └──────────────────────┴──────────────────────┘         |
|                                  │                                |
|  Software Vessel (Jetson Thor)   │                                |
|  ─────────────────────────────   │                                |
|                                  ▼                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    ROM BASE LAYER                            │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │ Flight Agent │ Perception Agent │ Safety Agent          ││ |
|  │  │ (PX4 bridge) │ (Obstacle det.)  │ (Geofence, Emerg.)    ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                                                              │ |
|  │  Real-time constraint: Control loop < 10ms                   │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 2.2 Aerial Vessel Types

```yaml
# vessel-types/aerial.yaml
# Aerial vessel type specifications

aerial:
  
  # ═══════════════════════════════════════════════════════════════
  # COMMERCIAL DRONE
  # ═══════════════════════════════════════════════════════════════
  commercial_drone:
    display_name: "Commercial Drone"
    description: |
      Multi-rotor drone for inspection, survey, and light delivery.
      Emphasis on flight safety, obstacle avoidance, and battery management.
      
    hardware:
      platform: "quadcopter"  # or hexacopter, octocopter
      
      sensors:
        - type: "camera"
          count: 2
          specs: "Stereo, 4K, global shutter"
          purpose: "obstacle_detection"
          
        - type: "lidar"
          model: "livox-mid360"
          range_m: 200
          
        - type: "imu"
          model: "vectornav-vn100"
          rate_hz: 400
          
        - type: "gps"
          model: "u-blox-f9p"
          accuracy: "rtk"
          
        - type: "barometer"
          model: "te-ms5611"
          
      actuators:
        - type: "motors"
          count: 4
          interface: "dshot"
          
        - type: "gimbal"
          axes: 3
          payload_kg: 2
          
    software:
      flight_controller:
        firmware: "px4"
        version: "1.14+"
        
      agents:
        - name: "flight_agent"
          capabilities:
            - "position_control"
            - "velocity_control"
            - "trajectory_tracking"
            - "auto_tune"
            
        - name: "perception_agent"
          capabilities:
            - "obstacle_detection"
            - "terrain_mapping"
            - "object_tracking"
            - "landing_zone_detection"
            
        - name: "mission_agent"
          capabilities:
            - "waypoint_navigation"
            - "survey_patterns"
            - "return_to_home"
            - "fail_safe"
            
      models:
        llm: "Qwen2.5-7B-Instruct-FP4"  # Smaller for weight
        vision: "obstacle-avoidance-v2"
        
      realtime:
        control_loop_hz: 250
        perception_hz: 30
        safety_check_hz: 100
        
    operational:
      max_flight_time_min: 45
      max_range_km: 15
      max_speed_ms: 15
      max_altitude_m: 120
      
  # ═══════════════════════════════════════════════════════════════
  # FIXED-WING UAV
  # ═══════════════════════════════════════════════════════════════
  fixed_wing_uav:
    display_name: "Fixed-Wing UAV"
    description: |
      Long-endurance fixed-wing aircraft for survey, mapping,
      and surveillance missions. Extended range and flight time.
      
    hardware:
      platform: "fixed_wing"
      wingspan_m: 2.5
      
      sensors:
        - type: "camera"
          model: "sony-a7riv"
          resolution_mp: 61
          purpose: "mapping"
          
        - type: "lidar"
          model: "yellowscan"
          purpose: "terrain_mapping"
          
    software:
      agents:
        - name: "survey_agent"
          capabilities:
            - "area_survey"
            - "grid_pattern"
            - "terrain_following"
            - "photo_overlap_control"
            
    operational:
      max_flight_time_min: 120
      max_range_km: 100
      cruise_speed_ms: 25
      max_altitude_m: 3000
      
  # ═══════════════════════════════════════════════════════════════
  # eVTOL
  # ═══════════════════════════════════════════════════════════════
  evtol:
    display_name: "Electric Vertical Take-Off and Landing"
    description: |
      Passenger or cargo eVTOL for urban air mobility.
      Highest safety requirements, human-rated systems.
      
    hardware:
      platform: "vtol"
      passenger_capacity: 4
      
    software:
      autonomy_level: 3  # L3: Conditional automation
      
      agents:
        - name: "pilot_agent"
          capabilities:
            - "full_flight_automation"
            - "passenger_communication"
            - "emergency_landing"
            - "air_traffic_coordination"
            
      safety:
        redundancy: "triple_modular"
        human_override: "always_available"
        certification: "faa_part_23"
```

### 2.3 Aerial-Specific Agent Behaviors

```python
# aerial_agents.py
"""
Aerial vessel agent behaviors
Real-time flight control and safety
"""

import numpy as np
from typing import Dict, Any, List, Tuple
from dataclasses import dataclass
from enum import Enum
import time

class FlightMode(Enum):
    MANUAL = "manual"
    STABILIZE = "stabilize"
    ALTITUDE = "altitude"
    POSITION = "position"
    AUTO = "auto"
    HOLD = "hold"
    RTL = "rtl"  # Return to Launch
    LAND = "land"

@dataclass
class FlightState:
    """Current flight state"""
    position: np.ndarray  # [x, y, z] in NED frame
    velocity: np.ndarray  # [vx, vy, vz]
    attitude: np.ndarray  # [roll, pitch, yaw]
    altitude_agl: float   # Above ground level
    battery_percent: float
    flight_mode: FlightMode

class FlightAgent(GitNativeAgent):
    """
    Aerial flight control agent
    Real-time control with safety guarantees
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        # Connect to flight controller (PX4)
        self.mavlink = self._init_mavlink()
        
        # State estimation
        self.ekf = self._init_ekf()
        
        # Control loops
        self.position_controller = self._init_position_controller()
        self.attitude_controller = self._init_attitude_controller()
        
        # Safety
        self.geofence = self._load_geofence()
        self.failsafe_triggers = []
        
        # Timing
        self.control_rate_hz = 250
        self.last_control_time = time.time()
        
    def run_control_loop(self):
        """
        Main control loop - must run at 250Hz
        Real-time constraint: < 4ms per iteration
        """
        now = time.time()
        dt = now - self.last_control_time
        
        # 1. Read sensors
        imu_data = self._read_imu()
        gps_data = self._read_gps()
        
        # 2. State estimation
        state = self.ekf.update(imu_data, gps_data, dt)
        
        # 3. Safety checks
        if self._check_safety(state):
            # 4. Compute control
            if state.flight_mode in [FlightMode.POSITION, FlightMode.AUTO]:
                control = self.position_controller.compute(state, self.target_position)
            elif state.flight_mode == FlightMode.STABILIZE:
                control = self.attitude_controller.compute(state)
            else:
                control = None  # Manual pass-through
                
            # 5. Send to actuators
            if control:
                self._send_control(control)
                
        # 6. Log to memory (async)
        if now - self.last_log_time > 0.1:  # 10Hz logging
            self._log_state(state)
            self.last_log_time = now
            
        self.last_control_time = now
        
        # Enforce timing
        elapsed = time.time() - now
        if elapsed > 0.004:
            self._log_timing_violation(elapsed)
            
    def _check_safety(self, state: FlightState) -> bool:
        """
        Safety checks that can disable control
        Returns True if safe to continue
        """
        # Battery
        if state.battery_percent < 20:
            self._trigger_failsafe("low_battery")
            return False
            
        # Geofence
        if not self.geofence.contains(state.position):
            self._trigger_failsafe("geofence_violation")
            return False
            
        # Altitude
        if state.altitude_agl > 120:  # Legal limit
            self._trigger_failsafe("altitude_limit")
            return False
            
        return True
        
    def _trigger_failsafe(self, reason: str):
        """
        Trigger failsafe action
        """
        self.failsafe_triggers.append({
            "reason": reason,
            "timestamp": time.time()
        })
        
        if reason == "low_battery":
            self._set_flight_mode(FlightMode.RTL)
        elif reason == "geofence_violation":
            self._set_flight_mode(FlightMode.HOLD)
        elif reason == "altitude_limit":
            self._descend_to_safe_altitude()
            

class PerceptionAgent(GitNativeAgent):
    """
    Aerial perception agent
    Obstacle detection, terrain mapping, landing detection
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        self.depth_model = self._load_depth_model()
        self.obstacle_detector = self._load_obstacle_detector()
        self.terrain_mapper = self._init_terrain_mapper()
        
    def process_frame(self, stereo_images: Tuple[np.ndarray, np.ndarray]) -> Dict:
        """
        Process stereo frame for perception
        Runs at 30Hz
        """
        # Compute depth
        depth_map = self.depth_model.compute(stereo_images)
        
        # Detect obstacles
        obstacles = self.obstacle_detector.detect(depth_map)
        
        # Update terrain map
        self.terrain_mapper.update(depth_map)
        
        # Find safe landing zones
        landing_zones = self._find_landing_zones(depth_map)
        
        return {
            "depth_map": depth_map,
            "obstacles": obstacles,
            "landing_zones": landing_zones,
            "terrain_elevation": self.terrain_mapper.get_elevation_map()
        }
        
    def get_avoidance_vector(self, obstacles: List[Dict]) -> np.ndarray:
        """
        Compute velocity vector to avoid obstacles
        Used by flight agent for collision avoidance
        """
        avoidance = np.zeros(3)
        
        for obstacle in obstacles:
            # Direction away from obstacle
            direction = obstacle["position"] - self.position
            distance = np.linalg.norm(direction)
            
            if distance < obstacle["radius"] + self.safety_margin:
                # Repulsion force
                strength = 1.0 / max(distance, 0.1)
                avoidance -= direction / distance * strength
                
        return avoidance
```

---

## Part 3: Industrial Vessels

### 3.1 Overview

Industrial vessels operate in controlled environments like warehouses, factories, and logistics centers. Key requirements are precision, reliability, and integration with enterprise systems.

```
+------------------------------------------------------------------+
|                  INDUSTRIAL VESSEL ARCHITECTURE                    |
+------------------------------------------------------------------+
|                                                                   |
|  Physical Vessel (Robot/AGV/Manipulator)                          |
|  ──────────────────────────────────────                           |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    SENSORS & ACTUATORS                       │ |
|  │                                                              │ |
|  │  Navigation             Manipulation            Safety       │ |
|  │  ┌───────────┐          ┌───────────┐          ┌──────────┐ │ |
|  │  │ Lidar     │          │ Gripper   │          │ E-stop   │ │ |
|  │  │ Cameras   │          │ Arm joints│          │ Proximity│ │ |
|  │  │ Encoders  │          │ Force/torque│        │ Light    │ │ |
|  │  │ IMU       │          │ Tool changers│       │          │ │ |
|  │  └─────┬─────┘          └─────┬─────┘          └────┬─────┘ │ |
|  └────────┼──────────────────────┼──────────────────────┼───────┘ |
|           │                      │                      │         |
|           └──────────────────────┴──────────────────────┘         |
|                                  │                                |
|  Software Vessel (Jetson Thor)   │                                |
|  ─────────────────────────────   │                                |
|                                  ▼                                |
|  ┌─────────────────────────────────────────────────────────────┐ |
|  │                    ROM BASE LAYER                            │ |
|  │  ┌─────────────────────────────────────────────────────────┐│ |
|  │  │ Nav Agent │ Manipulation Agent │ Safety Agent           ││ |
|  │  │ (SLAM)    │ (Motion planning)  │ (Human detection)      ││ |
|  │  └─────────────────────────────────────────────────────────┘│ |
|  │                                                              │ |
|  │  Enterprise Integration: REST API, MQTT, OPC-UA             │ |
|  └─────────────────────────────────────────────────────────────┘ |
|                                                                   |
+------------------------------------------------------------------+
```

### 3.2 Industrial Vessel Types

```yaml
# vessel-types/industrial.yaml
# Industrial vessel type specifications

industrial:
  
  # ═══════════════════════════════════════════════════════════════
  # WAREHOUSE ROBOT (AMR)
  # ═══════════════════════════════════════════════════════════════
  warehouse_amr:
    display_name: "Autonomous Mobile Robot (Warehouse)"
    description: |
      Mobile robot for warehouse logistics, picking, and transport.
      Navigation in dynamic environments with human coworkers.
      
    hardware:
      platform: "differential_drive"
      
      sensors:
        - type: "lidar"
          model: "sick-tim781"
          range_m: 25
          angular_resolution: 0.1
          
        - type: "camera"
          count: 4
          specs: "1080p, wide angle"
          purpose: "object_detection, human_detection"
          
        - type: "depth_camera"
          model: "intel-realsense-d455"
          purpose: "manipulation"
          
        - type: "imu"
          model: "bno055"
          
      actuators:
        - type: "drive_motors"
          count: 2
          max_speed_ms: 2.0
          
        - type: "lift"
          capacity_kg: 1000
          
        - type: "conveyor"
          bidirectional: true
          
    software:
      agents:
        - name: "navigation_agent"
          capabilities:
            - "slam"
            - "dynamic_path_planning"
            - "human_aware_navigation"
            - "fleet_coordination"
            
        - name: "picking_agent"
          capabilities:
            - "object_recognition"
            - "barcode_reading"
            - "slot_detection"
            - "precision_placement"
            
        - name: "safety_agent"
          capabilities:
            - "human_detection"
            - "collision_prediction"
            - "emergency_stop"
            - "speed_limiting"
            
      models:
        llm: "Qwen2.5-14B-Instruct-FP4"
        vision: "warehouse-objects-v3"
        
      integration:
        wms_api: "rest"
        fleet_manager: "mqtt"
        safety_systems: "plc"
        
    operational:
      max_payload_kg: 1000
      runtime_hours: 8
      charging_time_hours: 1
      
  # ═══════════════════════════════════════════════════════════════
  # MANIPULATOR ARM
  # ═══════════════════════════════════════════════════════════════
  manipulator_arm:
    display_name: "Industrial Manipulator Arm"
    description: |
      6-DOF robot arm for assembly, welding, painting, or material handling.
      Precision control with safety-critical operation.
      
    hardware:
      platform: "6_dof_arm"
      reach_m: 1.8
      payload_kg: 25
      
      sensors:
        - type: "joint_encoders"
          resolution: 19  # bit
          
        - type: "force_torque"
          location: "wrist"
          
        - type: "camera"
          location: "end_effector"
          
    software:
      agents:
        - name: "motion_agent"
          capabilities:
            - "trajectory_planning"
            - "collision_avoidance"
            - "force_control"
            - "visual_servoing"
            
  # ═══════════════════════════════════════════════════════════════
  # AUTOMATED GUIDED VEHICLE (AGV)
  # ═══════════════════════════════════════════════════════════════
  agv:
    display_name: "Automated Guided Vehicle"
    description: |
      Fixed-path transport vehicle for factory and logistics.
      Simpler navigation than AMR, higher reliability.
      
    hardware:
      platform: "differential_drive"
      guidance: ["magnetic_tape", "wire", "natural_feature"]
      
    software:
      agents:
        - name: "transport_agent"
          capabilities:
            - "route_following"
            - "load_handling"
            - "traffic_coordination"
```

### 3.3 Industrial-Specific Agent Behaviors

```python
# industrial_agents.py
"""
Industrial vessel agent behaviors
Precision, reliability, enterprise integration
"""

import numpy as np
from typing import Dict, Any, List, Tuple
from dataclasses import dataclass
import json
import requests

@dataclass
class WarehouseTask:
    """A warehouse task"""
    task_id: str
    task_type: str  # "pick", "place", "transport", "charge"
    source: Tuple[float, float]
    destination: Tuple[float, float]
    priority: int
    deadline: str

class WarehouseNavigationAgent(GitNativeAgent):
    """
    Warehouse navigation agent
    Human-aware, dynamic environment
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        # SLAM
        self.slam = self._init_slam()
        
        # Path planning
        self.planner = self._init_planner()
        
        # Human detection
        self.human_detector = self._load_human_detector()
        
        # Fleet coordination
        self.fleet_client = self._init_fleet_client()
        
    def plan_path(self, start: np.ndarray, goal: np.ndarray,
                  dynamic_obstacles: List[np.ndarray]) -> np.ndarray:
        """
        Plan collision-free path through warehouse
        Considers dynamic obstacles including humans
        """
        # Get current map from SLAM
        occupancy_map = self.slam.get_map()
        
        # Inflate obstacles for safety margin
        inflated_map = self._inflate_obstacles(occupancy_map)
        
        # Add dynamic obstacles
        for obs in dynamic_obstacles:
            inflated_map = self._add_dynamic_obstacle(inflated_map, obs)
            
        # Plan path
        path = self.planner.plan(
            start=start,
            goal=goal,
            map=inflated_map,
            cost_function=self._create_cost_function(dynamic_obstacles)
        )
        
        return path
        
    def _create_cost_function(self, dynamic_obstacles):
        """
        Create cost function that penalizes:
        - Proximity to humans (high penalty)
        - Proximity to other obstacles
        - Distance from preferred paths
        """
        def cost(pos):
            c = 0
            
            for obs in dynamic_obstacles:
                dist = np.linalg.norm(pos - obs.position)
                if obs.type == "human":
                    c += 100.0 / max(dist, 0.5)  # High cost near humans
                else:
                    c += 10.0 / max(dist, 0.5)
                    
            return c
            
        return cost
        
    def coordinate_with_fleet(self, task: WarehouseTask) -> Dict:
        """
        Coordinate task execution with fleet manager
        """
        # Request path reservation
        reservation = self.fleet_client.request_path(
            path=self.planned_path,
            task_id=task.task_id,
            priority=task.priority
        )
        
        if reservation["granted"]:
            return {"status": "proceed", "reservation": reservation}
        else:
            # Wait for clearance
            return {"status": "wait", "wait_time": reservation["wait_time"]}


class PickingAgent(GitNativeAgent):
    """
    Warehouse picking agent
    Object recognition and precision handling
    """
    
    def __init__(self, repo_path: str, config: Dict):
        super().__init__(repo_path)
        
        self.object_detector = self._load_object_detector()
        self.barcode_reader = self._init_barcode_reader()
        
    def identify_pick_target(self, image: np.ndarray,
                             slot_info: Dict) -> Dict:
        """
        Identify target object in shelf slot
        """
        # Detect all objects
        detections = self.object_detector.detect(image)
        
        # Match to expected object
        for det in detections:
            if self._matches_expected(det, slot_info):
                # Read barcode for verification
                barcode = self.barcode_reader.read(
                    image, det["bbox"]
                )
                
                if barcode == slot_info["expected_barcode"]:
                    return {
                        "found": True,
                        "position": det["position_3d"],
                        "orientation": det["orientation"],
                        "confidence": det["confidence"],
                        "verified": True
                    }
                    
        return {"found": False, "reason": "object_not_found"}
```

---

## Part 4: Home Vessels

### 4.1 Overview

Home vessels are consumer-facing robots for domestic assistance. Key requirements are safety, ease of use, and privacy protection.

```yaml
# vessel-types/home.yaml
# Home vessel type specifications

home:
  
  # ═══════════════════════════════════════════════════════════════
  # HOME ASSISTANT ROBOT
  # ═══════════════════════════════════════════════════════════════
  assistant_robot:
    display_name: "Home Assistant Robot"
    description: |
      General-purpose home robot for assistance, companionship,
      and household tasks. Privacy-first, user-friendly.
      
    hardware:
      platform: "mobile_manipulator"
      height_m: 1.2
      
      sensors:
        - type: "camera"
          count: 2
          specs: "RGB-D, privacy shutter"
          privacy: "user_controllable"
          
        - type: "microphone"
          specs: "array, beamforming"
          privacy: "hardware_mute"
          
        - type: "lidar"
          model: "rplidar-a3"
          
        - type: "imu"
          
      actuators:
        - type: "drive_base"
          max_speed_ms: 0.5
          
        - type: "arm"
          dofs: 7
          reach_m: 0.8
          
        - type: "gripper"
          
        - type: "speaker"
          
    software:
      privacy:
        camera_shutter: "hardware"
        microphone_mute: "hardware"
        local_processing: true
        no_cloud_upload: true
        
      agents:
        - name: "assistant_agent"
          capabilities:
            - "natural_conversation"
            - "task_execution"
            - "learning_preferences"
            - "home_automation"
            
        - name: "navigation_agent"
          capabilities:
            - "home_mapping"
            - "object_finding"
            - "fetch_and_carry"
            
        - name: "safety_agent"
          capabilities:
            - "fall_detection"
            - "intrusion_alert"
            - "emergency_call"
            
      models:
        llm: "Qwen2.5-7B-Instruct-FP4"  # Smaller for cost
        vision: "home-objects-v2"
        stt: "whisper-small"
        
    operational:
      runtime_hours: 8
      charging_time_hours: 2
      
  # ═══════════════════════════════════════════════════════════════
  # KITCHEN ROBOT (CHEF)
  # ═══════════════════════════════════════════════════════════════
  kitchen_robot:
    display_name: "Kitchen Robot (Chef)"
    description: |
      Specialized robot for food preparation and cooking.
      Food safety compliance, recipe following.
      
    hardware:
      platform: "stationary_manipulator"
      
      sensors:
        - type: "camera"
          purpose: "ingredient_recognition"
          
        - type: "temperature"
          purpose: "cooking_monitoring"
          
        - type: "scale"
          purpose: "portion_control"
          
      actuators:
        - type: "arms"
          count: 2
          
        - type: "grippers"
          purpose: "utensil_handling"
          
    software:
      agents:
        - name: "chef_agent"
          capabilities:
            - "recipe_interpretation"
            - "ingredient_prep"
            - "cooking_execution"
            - "food_safety"
```

---

## Part 5: Medical Vessels

### 5.1 Overview

Medical vessels have the highest safety and regulatory requirements. All operations must be explainable, auditable, and have human oversight.

```yaml
# vessel-types/medical.yaml
# Medical vessel type specifications

medical:
  
  # ═══════════════════════════════════════════════════════════════
  # SURGICAL ASSISTANT ROBOT
  # ═══════════════════════════════════════════════════════════════
  surgical_assistant:
    display_name: "Surgical Assistant Robot"
    description: |
      Robot-assisted surgery platform with surgeon oversight.
      Highest precision and safety requirements.
      
    hardware:
      platform: "surgical_manipulator"
      
      sensors:
        - type: "camera"
          specs: "4K, 3D stereo, medical grade"
          
        - type: "force_torque"
          sensitivity: "0.1N"
          
        - type: "tracking"
          purpose: "instrument_tracking"
          
      actuators:
        - type: "arms"
          count: 3
          precision: "sub-millimeter"
          
    software:
      regulatory:
        certification: "fda_510k"
        audit_trail: true
        explainability: "required"
        
      agents:
        - name: "surgical_agent"
          capabilities:
            - "procedure_guidance"
            - "tissue_recognition"
            - "instrument_tracking"
            - "complication_detection"
            
        - name: "safety_agent"
          capabilities:
            - "vital_signs_monitoring"
            - "force_limiting"
            - "emergency_stop"
            
      models:
        llm: "medical-specialized-v1"
        vision: "tissue-classification-v2"
        
      human_oversight:
        mode: "full"  # All actions require surgeon approval
        latency_max_ms: 50
        
  # ═══════════════════════════════════════════════════════════════
  # DIAGNOSTIC ROBOT
  # ═══════════════════════════════════════════════════════════════
  diagnostic_robot:
    display_name: "Diagnostic Robot"
    description: |
      Robot for medical diagnostics, patient interviews,
      and specimen collection.
      
    hardware:
      platform: "mobile_manipulator"
      
      sensors:
        - type: "camera"
          purpose: "dermatology, ophthalmology"
          
        - type: "microphone"
          purpose: "auscultation"
          
        - type: "thermal"
          purpose: "temperature_mapping"
          
    software:
      agents:
        - name: "diagnostic_agent"
          capabilities:
            - "patient_interview"
            - "symptom_analysis"
            - "specimen_collection_guidance"
            - "referral_recommendation"
```

### 5.2 Medical-Specific Safety Patterns

```python
# medical_safety.py
"""
Medical vessel safety patterns
Highest safety requirements with full auditability
"""

from typing import Dict, Any, List, Optional
from dataclasses import dataclass
from enum import Enum
import hashlib
import json
from datetime import datetime

class SafetyLevel(Enum):
    OBSERVATION = "observation"      # AI observes, human acts
    ASSISTANCE = "assistance"        # AI suggests, human approves
    SUPERVISION = "supervision"     # AI acts, human supervises
    AUTONOMOUS = "autonomous"       # AI acts independently

@dataclass
class MedicalAction:
    """A medical action proposed or taken by the agent"""
    action_id: str
    action_type: str
    parameters: Dict[str, Any]
    proposed_by: str  # "agent" or "human"
    approved_by: Optional[str]
    timestamp: str
    reasoning: str  # Why this action was proposed
    confidence: float

class MedicalSafetyAgent:
    """
    Safety agent for medical vessels
    Enforces human oversight and auditability
    """
    
    def __init__(self, safety_level: SafetyLevel):
        self.safety_level = safety_level
        self.audit_log = []
        self.pending_approvals = []
        
    def propose_action(self, action: MedicalAction) -> Dict:
        """
        Propose a medical action
        Requires approval based on safety level
        """
        # Always require human approval for medical
        if self.safety_level in [SafetyLevel.OBSERVATION, SafetyLevel.ASSISTANCE]:
            # Queue for human approval
            approval_request = {
                "action": action,
                "status": "pending_approval",
                "requested_at": datetime.now().isoformat()
            }
            self.pending_approvals.append(approval_request)
            return approval_request
            
        else:
            # Still log for audit
            self._log_action(action)
            return {"action": action, "status": "executed"}
            
    def approve_action(self, action_id: str, approver: str) -> Dict:
        """
        Human approves a pending action
        """
        for pending in self.pending_approvals:
            if pending["action"].action_id == action_id:
                pending["action"].approved_by = approver
                pending["status"] = "approved"
                self._log_action(pending["action"])
                return pending
                
        return {"error": "action_not_found"}
        
    def _log_action(self, action: MedicalAction):
        """
        Create immutable audit log entry
        """
        entry = {
            "action_id": action.action_id,
            "action_type": action.action_type,
            "parameters": action.parameters,
            "proposed_by": action.proposed_by,
            "approved_by": action.approved_by,
            "timestamp": action.timestamp,
            "reasoning": action.reasoning,
            "confidence": action.confidence,
            "hash": self._compute_hash(action)
        }
        
        self.audit_log.append(entry)
        
    def _compute_hash(self, action: MedicalAction) -> str:
        """Compute hash for audit integrity"""
        data = json.dumps({
            "action_id": action.action_id,
            "action_type": action.action_type,
            "timestamp": action.timestamp
        }, sort_keys=True)
        return hashlib.sha256(data.encode()).hexdigest()
        
    def get_audit_trail(self) -> List[Dict]:
        """Get complete audit trail for review"""
        return self.audit_log
```

---

## Summary: Cross-Domain Transfer Potential

```
+------------------------------------------------------------------+
|                CROSS-DOMAIN KNOWLEDGE TRANSFER                     |
+------------------------------------------------------------------+
|                                                                   |
|  Marine          Aerial          Industrial      Home    Medical  │
|  ──────          ──────          ──────────      ────    ───────  │
|                                                                   |
|  Navigation ─────────────────────────────────────────────────►   │
|  (Route opt.)           (Path planning)      (Home nav)          │
|                                                                   |
|  Obstacle Avoid ─────────────────────────────────────────────►   │
|  (Collision)    (In-flight)    (Human-aware)  (Pet-aware)        │
|                                                                   |
|  Safety Systems ─────────────────────────────────────────────►   │
|  (Emergency)    (Failsafe)     (E-stop)       (Fall det)  (Med)  │
|                                                                   |
|  Voice Interface ────────────────────────────────────────────►   │
|  (Hands-free)   (Ground control) (Operator)   (Natural)   (Pat.) │
|                                                                   |
|  Vision Pipeline ─────────────────────────────────────────────►  │
|  (Species ID)   (Landing zone)  (Object rec)  (Person)    (Tiss.)│
|                                                                   |
+------------------------------------------------------------------+
```

### Key Transfer Opportunities

| Source Domain | Target Domain | Transferable Capability |
|---------------|---------------|------------------------|
| Marine (weather routing) | Aerial (flight planning) | Environmental adaptation |
| Marine (AIS tracking) | Industrial (fleet coord) | Multi-agent coordination |
| Aerial (obstacle avoid) | Industrial (human-aware) | Dynamic obstacle handling |
| Industrial (precision) | Medical (surgical) | Sub-millimeter control |
| Home (voice UI) | All domains | Natural language interface |
| Medical (safety) | All domains | Human oversight patterns |

---

*Document prepared for Jetson Robotics Ecosystem Grand Design*  
*Iteration 02 - Vessel Type Specifications*  
*Date: January 2025*
