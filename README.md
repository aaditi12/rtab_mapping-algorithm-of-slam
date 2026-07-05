# rtab_mapping-algorithm-of-slam
This is a **ROS 2 Humble + Gazebo Classic + RTAB-Map (3D SLAM) + Nav2 + BehaviorTree.CPP** intelligent inspection robot — a full autonomy stack: mapping, obstacle avoidance, battery-aware docking, and exploration, all driven by a behavior tree.

## Setup + Build (one command)

```bash
cd inspection_robot_ws
bash build.sh
```
This installs all apt deps (BehaviorTree.CPP, Nav2, RTAB-Map, Gazebo, etc.) and runs `colcon build`.

Or manually:
```bash
source /opt/ros/humble/setup.bash
sudo apt-get install -y --no-install-recommends \
  ros-humble-behaviortree-cpp \
  ros-humble-nav2-bringup \
  ros-humble-nav2-msgs \
  ros-humble-rtabmap-ros \
  ros-humble-gazebo-ros-pkgs \
  ros-humble-robot-state-publisher \
  ros-humble-joint-state-publisher \
  ros-humble-xacro \
  ros-humble-teleop-twist-keyboard \
  ros-humble-nav2-regulated-pure-pursuit-controller \
  espeak-ng tmux

cd inspection_robot_ws
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

## Run — full system (recommended, one command)

```bash
bash run.sh
```
This builds (if needed), sources everything, and opens a tmux session with 5 windows:
- `launch` — the full simulation (Gazebo + RTAB-Map + Nav2 + behavior tree executor)
- `bt_log` — live voice/TTS text announcements (`/voice_text`)
- `battery` — battery level watch
- `estop` — press Enter here to trigger an emergency stop
- `teleop` — manual keyboard override

```
Ctrl+B 0-4  → switch tmux windows
Ctrl+B D    → detach
```

## Run manually (without tmux)

```bash
source setup_env.sh    # sources ROS + workspace, sets LIBGL_ALWAYS_SOFTWARE=1
ros2 launch inspection_robot_bringup inspection_robot.launch.py
```

## Runtime interaction

```bash
# Trigger emergency stop (halts the whole behavior tree)
ros2 topic pub --once /emergency_stop std_msgs/msg/Bool "{data: true}"

# Manual teleop override
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# Monitor key topics
ros2 topic echo /voice_text          # TTS announcements
ros2 topic echo /battery_level       # 0–100%
ros2 topic echo /rtabmap/loop_closures
```

## Customization (edit `bt_trees/inspection_bt.xml`)

```xml
<CheckObstacle min_range_threshold="0.5"/>          <!-- metres -->
<CheckBatteryLow low_threshold="20.0"/>             <!-- % -->
<CheckMappingComplete required_closures="10"/>
<ExploreAction explore_radius="6.0" nav_timeout="30.0"/>
<DockAction dock_pose_x="-6.0" dock_pose_y="-6.0"/>  <!-- must match the world's charging_station pose -->
```

Note: the zip bundles a pre-built `build/`, `install/`, and `log/` directory with absolute paths baked in from someone else's machine (`/home/aa/Downloads/...`) — `build.sh`/`run.sh` rebuild fresh anyway, so just run those scripts rather than reusing the bundled artifacts directly. Needs a real Ubuntu 22.04 + ROS 2 Humble + Gazebo machine; won't run in this sandbox.
