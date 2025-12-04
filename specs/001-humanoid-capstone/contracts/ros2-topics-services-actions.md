# ROS 2 Perception Topics & Services

**Contract Version**: 1.0.0  
**Date**: 2025-12-05

## Topics (Pub/Sub)

### `/voice_command` (Publisher: voice_listener_node)

**Message Type**: `std_msgs/String` (for simplicity) or `humanoid_msgs/VoiceCommand`

**Description**: Raw transcribed voice command from user

**Frequency**: On-demand (when user speaks)

**Example**:
```json
{
  "data": "pick up the cup on the table"
}
```

---

### `/parsed_intent` (Publisher: intent_parser_node)

**Message Type**: `humanoid_msgs/Intent`

**Description**: Structured intent after LLM parsing

**Frequency**: Following `/voice_command` (typically 2-3s latency due to GPT-4 call)

**Example**:
```json
{
  "action": "grasp",
  "target_object": "cup",
  "target_location": "table",
  "constraints": ["careful"],
  "confidence": 0.95,
  "task_graph": null
}
```

---

### `/detected_objects` (Publisher: object_detection_node)

**Message Type**: `humanoid_msgs/ObjectDetectionArray`

**Description**: All objects detected in current scene with 6D poses

**Frequency**: 2-10 Hz (depending on camera and Jetson load)

**Example**:
```json
{
  "header": {"seq": 42, "stamp": 1701800400.5, "frame_id": "camera_color_optical_frame"},
  "objects": [
    {
      "object_class": "cup",
      "confidence": 0.92,
      "pose": {"x": 0.5, "y": 0.2, "z": 1.0, "qx": 0, "qy": 0, "qz": 0, "qw": 1},
      "grasp_points": [{"x": 0.45, "y": 0.15, "z": 1.05}]
    },
    {
      "object_class": "plate",
      "confidence": 0.88,
      "pose": {"x": 0.8, "y": -0.1, "z": 0.95, ...},
      ...
    }
  ]
}
```

---

### `/robot_state` (Publisher: motor_controller_node)

**Message Type**: `humanoid_msgs/RobotState` or `sensor_msgs/JointState`

**Description**: Current robot pose, joint angles, gripper force

**Frequency**: 10 Hz

**Example**:
```json
{
  "header": {"seq": 1234, "stamp": 1701800400.5},
  "base_pose": {"position": {"x": 0.0, "y": 0.0, "z": 0.0}, "orientation": {"x": 0, "y": 0, "z": 0, "w": 1}},
  "joint_angles": {
    "arm_shoulder_pan": 0.5,
    "arm_shoulder_lift": 1.2,
    "arm_elbow_flex": -1.5,
    "arm_wrist_1": -0.8,
    "arm_wrist_2": 1.57,
    "arm_wrist_3": 0.0,
    "gripper_left": 0.0,
    "gripper_right": 0.0
  },
  "gripper_force": 0.0,
  "collision_detected": false
}
```

---

### `/map` (Publisher: slam_node)

**Message Type**: `nav_msgs/OccupancyGrid`

**Description**: 2D occupancy map of the environment

**Frequency**: 1 Hz (updates every second or less frequently)

**Format**: Standard ROS 2 occupancy grid (0 = free, 100 = occupied, -1 = unknown)

---

### `/tf` & `/tf_static` (Publisher: slam_node, motor_controller_node)

**Message Type**: `tf2_msgs/TFMessage`

**Description**: Coordinate frame transforms (world → base_link → arm_link0 → ... → gripper)

**Frequency**: 10+ Hz for moving transforms (base_link, gripper); static for fixed offsets

**Standard Frames**:
- `world`: origin of SLAM map
- `base_link`: center of robot body
- `arm_base`: origin of arm (on torso)
- `gripper`: end-effector frame

---

## Services (Request/Response)

### `/perceive_scene` (Service: object_detection_node)

**Request**: `humanoid_msgs/PerceiveSceneRequest`

**Response**: `humanoid_msgs/PerceiveSceneResponse`

**Description**: Capture and process current scene on-demand (returns RGB, depth, detected objects, robot pose)

**Example Request**:
```json
{
  "request_id": 1,
  "capture_depth": true,
  "capture_rgb": true
}
```

**Example Response**:
```json
{
  "success": true,
  "perception": {
    "rgb_image": <binary PNG>,
    "depth_image": <binary PNG>,
    "detected_objects": [... ObjectDetectionArray ...],
    "robot_pose": {...},
    "occupancy_grid": {...}
  },
  "error_message": ""
}
```

---

### `/parse_intent` (Service: intent_parser_node)

**Request**: `humanoid_msgs/ParseIntentRequest`

**Response**: `humanoid_msgs/ParseIntentResponse`

**Description**: Parse natural language text to structured intent (alternative to async topic-based flow)

**Example Request**:
```json
{
  "text": "pick up the cup and place it in the sink"
}
```

**Example Response**:
```json
{
  "success": true,
  "intent": {
    "action": "grasp",
    "target_object": "cup",
    "task_graph": [
      {"action": "navigate", "target_location": "table"},
      {"action": "grasp", "object": "cup"},
      {"action": "navigate", "target_location": "sink"},
      {"action": "place", "object": "cup", "target_location": "sink"}
    ]
  },
  "confidence": 0.93
}
```

---

### `/get_robot_state` (Service: motor_controller_node)

**Request**: `humanoid_msgs/GetRobotStateRequest` (empty or request specific fields)

**Response**: `humanoid_msgs/GetRobotStateResponse`

**Description**: Query current robot state synchronously

**Example Response**:
```json
{
  "state": {...RobotState...}
}
```

---

## Actions (Goal/Feedback/Result)

### `NavigateTo` (Action: path_planner_node)

**Goal**: `humanoid_msgs/NavigateToGoal`

**Feedback**: `humanoid_msgs/NavigateToFeedback`

**Result**: `humanoid_msgs/NavigateToResult`

**Description**: Navigate robot to a target pose; provides progress feedback

**Example Goal**:
```json
{
  "target_pose": {"x": 2.0, "y": 1.0, "z": 0.0, "qx": 0, "qy": 0, "qz": 0.707, "qw": 0.707},
  "max_velocity": 0.5
}
```

**Example Feedback** (published every 0.5s during navigation):
```json
{
  "current_pose": {"x": 0.5, "y": 0.25, "z": 0.0, ...},
  "distance_remaining": 2.0,
  "time_remaining": 4.0,
  "status": "navigating"
}
```

**Example Result**:
```json
{
  "success": true,
  "final_pose": {"x": 2.0, "y": 1.0, "z": 0.0, ...},
  "total_time": 4.1,
  "error_message": ""
}
```

---

### `GraspObject` (Action: grasp_planner_node)

**Goal**: `humanoid_msgs/GraspObjectGoal`

**Feedback**: `humanoid_msgs/GraspObjectFeedback`

**Result**: `humanoid_msgs/GraspObjectResult`

**Description**: Grasp a detected object; includes planning and execution

**Example Goal**:
```json
{
  "target_object": {
    "object_id": 1,
    "object_class": "cup",
    "pose": {"x": 0.5, "y": 0.2, "z": 1.0, ...}
  },
  "desired_grip_force": 2.5
}
```

**Example Feedback**:
```json
{
  "status": "planning",
  "message": "Generated 5 grasp candidates"
}
```

then

```json
{
  "status": "executing",
  "message": "Reaching to object...",
  "progress": 0.3
}
```

then

```json
{
  "status": "closing_gripper",
  "current_force": 1.2,
  "message": "Gripper force ramping..."
}
```

**Example Result**:
```json
{
  "success": true,
  "final_grip_force": 2.5,
  "grasp_stability": 0.85,
  "error_message": ""
}
```

---

### `ExecuteTask` (Action: task_executor_node)

**Goal**: `humanoid_msgs/ExecuteTaskGoal`

**Feedback**: `humanoid_msgs/ExecuteTaskFeedback`

**Result**: `humanoid_msgs/ExecuteTaskResult`

**Description**: Execute a high-level task (e.g., "pick up and place"), including all subtasks

**Example Goal**:
```json
{
  "task_description": "pick up the cup and place it in the sink",
  "retry_failed_subtasks": true,
  "max_retries": 3
}
```

**Example Feedback** (published for each subtask):
```json
{
  "current_subtask_index": 0,
  "current_subtask_description": "navigate_to(table)",
  "subtasks_completed": 0,
  "total_subtasks": 4,
  "overall_progress": 0.0
}
```

**Example Result**:
```json
{
  "success": true,
  "total_time": 12.5,
  "subtask_results": [
    {"subtask": 0, "success": true, "time": 2.3},
    {"subtask": 1, "success": true, "time": 3.2},
    {"subtask": 2, "success": true, "time": 4.1},
    {"subtask": 3, "success": true, "time": 2.9}
  ]
}
```

---

## QoS Policies

**Safety-Critical Topics** (e.g., `/robot_state`, `/collision_detected`):
- Reliability: RELIABLE
- Durability: VOLATILE (no need for historical data)
- History: KEEP_LAST (1 message)
- Latency Budget: <100ms

**Non-Critical Topics** (e.g., `/detected_objects`, `/map`):
- Reliability: BEST_EFFORT (can tolerate dropped messages)
- Durability: VOLATILE
- History: KEEP_LAST (1 message)
- Latency Budget: <500ms

**Service Calls** (all):
- Timeout: 5s for short calls (e.g., `/perceive_scene`), 10s for long calls (e.g., `/execute_task`)

---

**Contract Status**: ✅ Complete; defines all message interfaces for inter-node communication
