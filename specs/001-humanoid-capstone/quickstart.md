# Humanoid Capstone: Developer Quickstart

**Version**: 1.0.0  
**Date**: 2025-12-05

This guide walks you through setting up your development environment and running the humanoid capstone system for the first time.

---

## Prerequisites

- **OS**: Ubuntu 22.04 LTS (recommended; Windows/macOS use WSL2 or Docker)
- **Python**: 3.10+
- **GPU** (optional but recommended): NVIDIA RTX 4070 Ti or equivalent for real-time inference
- **Disk Space**: ~20 GB (ROS 2, Gazebo, models, datasets)
- **Time**: ~45 minutes for full setup

---

## Phase 0: Environment Setup

### 1.1 Install System Dependencies

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install Python and build tools
sudo apt install -y python3.10 python3.10-venv python3-pip build-essential \
  git curl wget gnupg lsb-release ca-certificates

# Verify Python installation
python3 --version  # Should output 3.10.x or higher

# Set Python 3.10 as default (optional)
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
```

### 1.2 Install ROS 2 Humble

```bash
# Add ROS 2 repository
sudo curl -sSL https://repo.ros2.org/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://repo.ros2.org/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Update and install ROS 2 Humble
sudo apt update
sudo apt install -y ros-humble-desktop

# Source ROS 2 setup script
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc

# Verify ROS 2 installation
ros2 --version  # Should output ROS 2 humble
```

### 1.3 Install Gazebo 11

```bash
# Install Gazebo and common plugins
sudo apt install -y gazebo11 libgazebo11-dev

# Install ROS 2 Gazebo integration
sudo apt install -y ros-humble-gazebo-ros ros-humble-gazebo-plugins

# Verify Gazebo installation
gazebo --version  # Should output Gazebo 11.x
```

### 1.4 Install Nav2 Stack

```bash
# Install Nav2 and dependencies
sudo apt install -y ros-humble-navigation2 ros-humble-nav2-bringup \
  ros-humble-cartographer ros-humble-cartographer-ros

# Verify Nav2 installation
ros2 pkg list | grep nav2  # Should list nav2-* packages
```

### 1.5 Create Python Virtual Environment

```bash
# Create venv in project root
cd ~/humanoid-capstone  # or wherever you cloned the repo
python3 -m venv .venv

# Activate virtual environment
source .venv/bin/activate

# Upgrade pip and install wheel
pip install --upgrade pip setuptools wheel
```

---

## Phase 1: Clone and Install Capstone Repository

### 1.6 Clone Repository

```bash
# Clone the main repository
git clone https://github.com/<your-org>/humanoid-capstone.git
cd humanoid-capstone

# Checkout feature branch (created during /sp.specify)
git checkout 001-humanoid-capstone
```

### 1.7 Install Python Dependencies

```bash
# Activate virtual environment (if not already activated)
source .venv/bin/activate

# Install required Python packages
pip install -r requirements.txt

# Key packages installed:
# - OpenAI Whisper: speech-to-text
# - OpenAI Python SDK: GPT-4 API calls
# - YOLOv8 (ultralytics): object detection
# - scipy: inverse kinematics
# - pytest: testing framework
# - opencv-python: image processing
```

### 1.8 Install ROS 2 Packages

```bash
# Navigate to workspace
cd ~/humanoid-capstone

# Build all ROS 2 packages (colcon)
colcon build --symlink-install

# Source the overlay workspace
source install/setup.bash

# Add to ~/.bashrc for convenience
echo "source ~/humanoid-capstone/install/setup.bash" >> ~/.bashrc
```

### 1.9 Configure API Keys

**Create `.env` file** in project root (NEVER commit to git):

```bash
# Copy template
cp .env.example .env

# Edit with your API keys
nano .env
```

**Required Environment Variables**:

```bash
# OpenAI API
OPENAI_API_KEY=sk-...your-key-here...
OPENAI_ORG_ID=org-...optional...

# Model selection (use smaller models for testing)
INTENT_PARSER_MODEL=gpt-4-1106-preview  # or gpt-3.5-turbo for cost savings

# ROS 2 Domain (for multi-robot experiments)
ROS_DOMAIN_ID=0  # Change to 1, 2, etc. for different instances

# Gazebo simulation
GAZEBO_MODEL_PATH=~/humanoid-capstone/simulation/models:$GAZEBO_MODEL_PATH
GAZEBO_PLUGIN_PATH=~/humanoid-capstone/build/humanoid_control/lib:$GAZEBO_PLUGIN_PATH
```

**Load environment in active shell**:

```bash
source .env
```

---

## Phase 2: First Run - Simple Navigation

### 2.1 Launch Gazebo Simulation

```bash
# Terminal 1: Start Gazebo with robot model
ros2 launch humanoid_control gazebo_world.launch.py

# Expected output:
# - Gazebo window opens with empty world
# - Robot (default Franka Emika Panda) spawns at origin
# - TF tree shows: world → base_link → arm_base → joint0 → ... → gripper
```

### 2.2 Launch SLAM Node

```bash
# Terminal 2: Start Cartographer SLAM
ros2 launch humanoid_perception slam.launch.py use_sim:=true

# Expected output:
# - `/map` topic publishing 2D occupancy grid
# - `/tf` publishing world → base_link transform
# - Console: "SLAM node running at 10 Hz"
```

### 2.3 Launch Navigation Stack

```bash
# Terminal 3: Start Nav2 path planning
ros2 launch humanoid_planning nav2.launch.py use_sim:=true

# Expected output:
# - "/cost_map" topic created
# - "/plan" service available
# - "/follow_path" action available
```

### 2.4 Send Navigation Goal

```bash
# Terminal 4: Send a simple navigation goal
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  '{pose: {header: {frame_id: world}, pose: {position: {x: 2.0, y: 1.0, z: 0}, orientation: {x: 0, y: 0, z: 0.707, w: 0.707}}}}'

# Expected behavior:
# - Robot drives towards (2.0, 1.0) in simulation
# - Gazebo shows robot motion
# - Terminal shows "Goal accepted" → navigation feedback → "Goal succeeded"
# - Duration: ~5-10 seconds in simulation
```

### 2.5 Check Node Status

```bash
# Terminal 5: View ROS 2 graph
rqt_graph

# Expected graph:
# - gazebo_node → [/robot_state] → motor_controller_node
# - motor_controller_node → [/tf] → slam_node
# - slam_node → [/map] → path_planner_node
# - path_planner_node → [/plan] ← user input

# Alternative: command-line node list
ros2 node list  # Shows all active nodes
ros2 topic list  # Shows all active topics
ros2 service list  # Shows all available services
```

---

## Phase 3: First Run - Voice Command (Optional)

### 3.1 Launch Voice Listener

```bash
# Terminal 6: Start voice command listener (requires microphone)
ros2 run humanoid_perception voice_listener_node

# Expected output:
# - "Listening for voice commands..."
# - Whisper processes audio in real-time
# - Transcribed text published to `/voice_command` topic
```

### 3.2 Launch Intent Parser

```bash
# Terminal 7: Start intent parser (GPT-4 API)
ros2 run humanoid_perception intent_parser_node

# Expected output:
# - "Intent parser ready"
# - Subscribes to `/voice_command`
# - Publishes parsed intent to `/parsed_intent`
```

### 3.3 Send Voice Command

```bash
# Speak into microphone (loud, clear):
# "Navigate to the table"

# Expected behavior:
# Terminal 1 (voice_listener): "Transcribed: navigate to the table"
# Terminal 2 (intent_parser): "Intent: action=navigate, target_location=table, confidence=0.95"
# Terminal 3 (nav2): "Received goal: table location, starting navigation"
# Gazebo: Robot moves towards table
```

---

## Phase 4: Troubleshooting

### Issue: "ROS 2 not found" after `colcon build`

**Solution**: Ensure you've sourced the ROS 2 setup:

```bash
source /opt/ros/humble/setup.bash
source install/setup.bash
```

### Issue: Gazebo crashes on startup

**Solution**: Update GPU drivers and verify OpenGL support:

```bash
# Check GPU drivers
nvidia-smi  # Should show your GPU info

# Test OpenGL
glxinfo | grep "OpenGL version"

# If using RTX 4070 Ti, ensure CUDA 11.8+ is installed
nvcc --version
```

### Issue: Nav2 not planning paths

**Solution**: Check if SLAM is publishing map:

```bash
# Terminal (separate):
ros2 topic echo /map --max-count=1

# If empty, restart SLAM node:
# Terminal 2: Ctrl+C
# ros2 launch humanoid_perception slam.launch.py use_sim:=true
```

### Issue: Voice recognition always fails

**Solution**: Check Whisper model size and audio:

```bash
# Test audio input directly
arecord -f cd -d 5 test.wav  # Record 5 seconds
ffplay test.wav  # Playback to verify

# If audio is too quiet/noisy, adjust microphone in system settings

# Test Whisper model separately (download base model):
python3 -c "import whisper; model = whisper.load_model('base'); print('Model loaded successfully')"
```

### Issue: GPT-4 API key errors

**Solution**: Verify API key and usage:

```bash
# Check .env is loaded
echo $OPENAI_API_KEY  # Should print your key (not empty)

# Test API directly:
python3 -c "
import openai
openai.api_key = '${OPENAI_API_KEY}'
response = openai.ChatCompletion.create(
    model='gpt-4-1106-preview',
    messages=[{'role': 'user', 'content': 'Hello'}]
)
print(response.choices[0].message.content)
"

# If error, check OpenAI account:
# - Verify API key at https://platform.openai.com/api-keys
# - Ensure account has sufficient credits (check usage dashboard)
# - For free tier: GPT-4 not available (use gpt-3.5-turbo)
```

---

## Running Tests

### Unit Tests (Python & C++)

```bash
# Run all tests
colcon test

# Run specific test
colcon test --packages-select humanoid_perception --ctest-args -V

# View test results
cat log/latest_test/<package>/stdout
```

### Integration Tests (ROS 2 Nodes)

```bash
# Launch test environment (pre-loaded world)
ros2 launch humanoid_control test_world.launch.py

# Run integration test
python3 tests/integration/test_navigation.py

# Expected output:
# - Robot navigates 3 waypoints ✓
# - Collision detection triggers ✓
# - SLAM map updates ✓
# "All integration tests passed: 3/3"
```

### Manual Smoke Test

```bash
# Quick verification that all nodes start without errors
ros2 launch humanoid_control full_system.launch.py

# Wait 5 seconds, then:
# Ctrl+C to stop

# Expected: All nodes started, no crash logs, graceful shutdown
```

---

## Next Steps

### Immediate (Week 1-2, Phase 0 Research):

1. **Validate Technology Choices** (from `research.md`):
   - Verify Whisper WER <4% on lab microphone
   - Test GPT-4 intent parsing (accuracy >85% on sample commands)
   - Benchmark YOLOv8 object detection (FPS on Jetson vs RTX 4070 Ti)

2. **Prototype Data Model** (from `data-model.md`):
   - Create custom ROS 2 message types (Intent, ObjectDetection, TaskGraph)
   - Test serialization/deserialization
   - Verify latency <100ms per message round-trip

3. **Set Up Continuous Integration**:
   - Configure GitHub Actions for `colcon build + test` on every PR
   - Ensure all tests pass before merging to `main`

### Medium-Term (Week 3-8, Phase 1 MVP):

1. **Implement Core Nodes**:
   - `voice_listener_node`: Whisper → `/voice_command` topic
   - `intent_parser_node`: GPT-4 → `/parsed_intent` topic
   - `perception_node`: YOLO + 6D pose → `/detected_objects` topic

2. **Integrate Navigation Stack**:
   - Launch Nav2 with Gazebo
   - Achieve <30s simple navigation tasks (e.g., "go to kitchen")

3. **Implement Safety Monitors**:
   - Collision detection (stop robot if obstacle)
   - Watchdog timer (kill commands >30s old)
   - Faculty sign-off checkpoint (code review before Jetson deployment)

### Long-Term (Week 9-13, Phase 2-3 Advanced):

1. **Implement Manipulation**:
   - Grasp planning node (scipy inverse kinematics)
   - Grasp execution with force feedback
   - Multi-step task execution (pick + place)

2. **Deploy to Hardware**:
   - Quantize models (INT8) for Jetson Orin Nano
   - Profile latency on real hardware
   - Stress-test on 50+ real objects (not just simulation)

3. **Final Capstone Demo**:
   - Execute 3 complex tasks (multi-step, real-time obstacle avoidance)
   - Live voice commands
   - Faculty evaluation

---

## Key Files & Directories

```
humanoid-capstone/
├── specs/001-humanoid-capstone/
│   ├── spec.md              # Feature requirements
│   ├── plan.md              # Implementation architecture
│   ├── research.md          # Technology validation
│   ├── data-model.md        # Entity schemas & workflows
│   └── contracts/           # ROS 2 interface definitions
├── src/
│   ├── humanoid_perception/ # Perception nodes (voice, detection, SLAM)
│   ├── humanoid_planning/   # Planning nodes (navigation, grasping, tasks)
│   ├── humanoid_control/    # Control nodes (motor commands, gripper)
│   └── humanoid_msgs/       # Custom ROS 2 message types
├── launch/
│   ├── gazebo_world.launch.py
│   ├── slam.launch.py
│   ├── nav2.launch.py
│   └── full_system.launch.py
├── tests/
│   ├── unit/                # Python & C++ unit tests
│   ├── integration/         # ROS 2 node integration tests
│   └── integration_tests.py # Smoke tests for all nodes
├── config/
│   ├── nav2_params.yaml     # Nav2 configuration
│   ├── slam_params.yaml     # Cartographer config
│   ├── yolo_params.yaml     # YOLOv8 config
│   └── ik_params.yaml       # Inverse kinematics config
├── simulation/
│   ├── models/              # URDF robot models, objects
│   ├── worlds/              # Gazebo world files
│   └── plugins/             # Gazebo plugins
├── requirements.txt         # Python dependencies
├── .env.example             # Environment variables template
└── README.md                # Project overview
```

---

## Constitution Alignment

This quickstart follows the program **Constitution** principles:

✅ **Embodied Excellence**: Hands-on setup ensures direct interaction with hardware simulation; developers understand the full stack.

✅ **Collaborative Intelligence**: Distributed ROS 2 architecture enables team parallelization (perception team, planning team, control team).

✅ **Simulation-First Safety**: All first runs use Gazebo; no real hardware until Phase 2; collision detection always active.

✅ **Open Standards**: ROS 2 Humble (LTS), standard message types, reproducible environment (requirements.txt, .env template).

✅ **Ethical Design**: Failsafes (watchdog timer, emergency stop action), transparency (all logs to console), bias testing checklist in requirements.

✅ **Continuous Learning**: Setup validates prerequisite technologies; troubleshooting section fosters debugging skills.

---

**Status**: ✅ Complete; Ready for Phase 1 development

**Last Updated**: 2025-12-05
