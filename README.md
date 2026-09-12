# RGB-D SLAM with Intel RealSense D435i in Gazebo

A ROS 2-based RGB-D SLAM project using a custom quadruped robot equipped with an Intel RealSense D435i RGB-D camera. The robot is simulated in Gazebo and uses RTAB-Map for real-time 3D mapping and 2D occupancy-grid generation.

The simulation is designed as the first stage of a larger robotics pipeline, with Nav2-based autonomous navigation planned as a future extension.

---

## 📌 Overview

This project implements an RGB-D SLAM pipeline for a simulated quadruped robot operating in a custom maze environment.

The robot is equipped with:

- Intel RealSense D435i RGB-D camera
- IMU
- Encoder-based odometry
- Joint controllers
- ROS 2 control interfaces

RGB and depth data from the simulated D435i camera, along with robot odometry, are provided to RTAB-Map for Simultaneous Localization and Mapping (SLAM).

RTAB-Map generates both:

- 3D map / point-cloud representation
- 2D occupancy grid

The generated map can be visualized using RViz2 and RTAB-Map's visualization interface.

---

## 🎯 Objectives

The main objectives of this project are:

- Simulate a quadruped robot in Gazebo
- Integrate an Intel RealSense D435i RGB-D camera
- Generate RGB-D sensor data
- Perform RGB-D SLAM using RTAB-Map
- Generate a 3D representation of the environment
- Generate a 2D occupancy grid
- Visualize mapping results in RViz2
- Provide a foundation for future autonomous navigation using Nav2

---

## 🏗️ System Architecture

```text
                 ┌─────────────────────┐
                 │   Gazebo Simulation │
                 │                     │
                 │  Custom Quadruped   │
                 │         +           │
                 │  RealSense D435i    │
                 │         +           │
                 │       IMU           │
                 └──────────┬──────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
             RGB + Depth            /odom
                 │                     │
                 └──────────┬──────────┘
                            │
                            ▼
                    ┌──────────────┐
                    │   RTAB-Map   │
                    │   RGB-D SLAM │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          3D Map / PointCloud     2D Grid Map
                 │                   │
                 └─────────┬─────────┘
                           ▼
                       ┌───────┐
                       │ RViz2 │
                       └───────┘
```

---

## 🤖 Robot & Simulation

The project uses a custom quadruped robot model defined using URDF.

The robot model includes:

- Four-legged structure
- Joint controllers
- Differential-drive plugin
- Encoder-based odometry
- Intel RealSense D435i RGB-D camera
- IMU sensor
- ROS 2 control integration

The robot is spawned inside a custom maze environment in Gazebo.

---

## 📷 Intel RealSense D435i

The robot uses an Intel RealSense D435i RGB-D camera.

For simulation, the D435i sensor configuration is integrated directly into the robot URDF and simulated using Gazebo sensor plugins.

The camera provides:

- RGB Image
- Depth Image
- Camera Info
- Point Cloud

The relevant topics used by RTAB-Map are:

```
/camera/camera/image_raw
/camera/camera/depth/image_raw
/camera/camera/camera_info
```

---

## 🗺️ RTAB-Map

RTAB-Map is used as the RGB-D SLAM system.

RTAB-Map receives:

```
RGB Image
   +
Depth Image
   +
Camera Info
   +
Robot Odometry
```

and incrementally constructs a map while the robot moves through the environment.

### Odometry

The simulated robot provides encoder-based odometry through the Gazebo differential-drive plugin:

```
/odom
```

RTAB-Map currently uses this odometry rather than camera-based visual odometry.

### RGB-D Synchronization

Approximate synchronization is enabled to tolerate small timestamp differences between RGB and depth streams.

```
approx_sync = true
approx_sync_max_interval = 0.1
queue_size = 30
```

### Grid Mapping

Depth-based occupancy-grid generation is enabled:

```
Grid/FromDepth = true
Grid/RayTracing = true
Grid/3D = false
```

The 2D occupancy grid is currently generated for mapping purposes but is not yet being used for autonomous navigation.

### Loop Closure

The current configuration enables:

```
RGBD/NeighborLinkRefining = true
RGBD/ProximityBySpace = true
```

with:

```
RGBD/LinearUpdate = 0.05
RGBD/AngularUpdate = 0.05
```

### Fresh Mapping Session

RTAB-Map is launched with:

```
--delete_db_on_start
```

so that every new run starts with a fresh mapping database.

---

## 📁 Project Structure

```
quad2/
│
├── config/
│   ├── controllers.yaml
│   ├── my_map.pgm
│   ├── my_map.yaml
│   └── nav2_params.yaml
│
├── launch/
│   ├── 2_gazebo.launch.py
│   ├── display.launch.py
│   ├── navigation.launch.py
│   └── rtabmap.launch.py
│
├── meshes/
│   ├── body-v1.stl
│   ├── leg11.stl
│   ├── leg12.stl
│   ├── leg13.stl
│   ├── leg21.stl
│   ├── leg22.stl
│   ├── leg23.stl
│   ├── leg31.stl
│   ├── leg32.stl
│   ├── leg33.stl
│   ├── leg41.stl
│   ├── leg42.stl
│   └── leg43.stl
│
├── quad2/
│   ├── cal_servo.py
│   ├── esp.py
│   └── fwd.py
│
├── resource/
│   └── quad2
│
├── urdf/
│   ├── model.config
│   ├── rover.urdf
│   ├── urdf.sdf
│   └── urdf.xacro
│
├── worlds/
│   ├── A1.world
│   ├── maze.world
│   ├── model.config
│   └── model.sdf
│
├── CMakeLists.txt
├── package.xml
└── setup.py
```

---

## 💻 Software Requirements

The project has been tested with:

| Component | Version |
|---|---|
| Ubuntu | 22.04.5 LTS |
| ROS 2 | Humble |
| Gazebo | 11.10.2 |
| RTAB-Map | 0.22.1 |

---

## 📦 Dependencies

The main ROS 2 dependencies are:

- ROS 2 Humble
- Gazebo 11
- gazebo_ros
- rtabmap_ros
- rtabmap_launch
- robot_state_publisher
- rviz2
- ros2_control
- controller_manager
- teleop_twist_keyboard

> **Note:** `librealsense2` is not required for the simulation because the D435i is simulated directly inside Gazebo. It will be required for the physical D435i implementation.

---

## ⚙️ Installation

### 1. Create a ROS 2 workspace

```bash
mkdir -p ~/Quadraf/src
cd ~/Quadraf/src
```

### 2. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
```

### 3. Install ROS dependencies

```bash
sudo apt update

sudo apt install \
    ros-humble-rtabmap-ros \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-robot-state-publisher \
    ros-humble-rviz2 \
    ros-humble-ros2-control \
    ros-humble-ros2-controllers \
    ros-humble-teleop-twist-keyboard
```

### 4. Build the workspace

```bash
cd ~/Quadraf
colcon build --symlink-install
```

### 5. Source the workspace

```bash
source install/setup.bash
```

---

## ▶️ Running the Simulation

The simulation currently uses three terminals.

### Terminal 1 — Gazebo

```bash
cd ~/Quadraf

source install/setup.bash

ros2 launch quad2 2_gazebo.launch.py
```

This starts:

- Gazebo
- Robot State Publisher
- RViz2
- Quadruped robot
- Joint State Broadcaster
- Joint Group Controller
- Robot control node

The robot will spawn inside the maze environment.

### Terminal 2 — RTAB-Map

```bash
cd ~/Quadraf

source install/setup.bash

ros2 launch quad2 rtabmap.launch.py
```

This launches RTAB-Map and connects it to the simulated RGB-D camera and robot odometry.

### Terminal 3 — Teleoperation

```bash
source /opt/ros/humble/setup.bash

ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Use the keyboard controls displayed in the terminal to move the robot through the maze. As the robot moves, RTAB-Map continuously processes the RGB-D data and updates the map.

---

## 👀 Visualization

### Gazebo
Gazebo provides the simulated environment and robot.

![Gazebo simulation](screenshots/simulation/gazebo.png)

### RViz2
RViz2 is used to visualize the robot, map, point cloud and other ROS data.

![RViz2 visualization](screenshots/simulation/rviz.png)

### RTAB-Map 3D Visualization
RTAB-Map provides a 3D visualization of the reconstructed environment.

![RTAB-Map visualization](screenshots/simulation/rtabmap.png)

---

## 🔄 Mapping Workflow

```text
Start Gazebo
     │
     ▼
Spawn Quadruped
     │
     ▼
Start RTAB-Map
     │
     ▼
Start Keyboard Teleoperation
     │
     ▼
Move Robot Through Maze
     │
     ▼
RGB + Depth + Odometry
     │
     ▼
RTAB-Map RGB-D SLAM
     │
     ├───────────────┐
     ▼               ▼
  3D Map          2D Grid
     │               │
     └───────┬───────┘
             ▼
          RViz2
```

---

## 📊 Current Results

The current implementation successfully demonstrates:

- Simulation of a custom quadruped robot
- D435i RGB-D camera integration
- RGB image acquisition
- Depth image acquisition
- IMU simulation
- Encoder-based odometry
- RGB-D SLAM using RTAB-Map
- 3D environment reconstruction
- 2D occupancy-grid generation
- Real-time visualization in RViz2
- Keyboard-based robot control

---

## 🚧 Current Status

| Component | Status |
|---|---|
| Quadruped simulation | ✅ Complete |
| Custom maze environment | ✅ Complete |
| Robot URDF | ✅ Complete |
| D435i RGB-D camera | ✅ Complete |
| RGB data | ✅ Working |
| Depth data | ✅ Working |
| IMU | ✅ Working |
| Odometry | ✅ Working |
| Teleoperation | ✅ Working |
| RTAB-Map | ✅ Working |
| 3D mapping | ✅ Working |
| 2D occupancy grid | ✅ Working |
| RViz2 visualization | ✅ Working |
| Autonomous navigation | 🔜 Planned |

---

## 🔮 Future Work

The next stage of the project is to integrate autonomous navigation.

Planned improvements include:

- Nav2 integration
- Localization using the generated map
- Autonomous waypoint navigation
- Obstacle avoidance
- Navigation through the simulated maze
- Testing the navigation pipeline on physical hardware
- Integration with the physical Intel RealSense D435i

The long-term goal is to develop a perception, SLAM and navigation pipeline that can first be validated in simulation and then transferred to a physical robot.

---

## 🧪 Troubleshooting

**Check ROS topics**
```bash
ros2 topic list
```

**Check RGB camera data**
```bash
ros2 topic hz /camera/camera/image_raw
```

**Check depth data**
```bash
ros2 topic hz /camera/camera/depth/image_raw
```

**Check odometry**
```bash
ros2 topic echo /odom
```

**Check TF tree**
```bash
ros2 run tf2_tools view_frames
```

**Check RTAB-Map topics**
```bash
ros2 topic list | grep rtabmap
```

---

## 📌 Notes

- The simulated D435i is integrated into the robot URDF.
- RTAB-Map uses RGB-D data and encoder-based odometry.
- Camera-based visual odometry is currently disabled.
- A 2D occupancy grid is generated but is not yet connected to autonomous navigation.
- Each RTAB-Map session starts with a fresh database.
- The physical D435i implementation will be maintained separately from the simulation setup.

---

## 📜 License

This project is licensed under the Apache 2.0 License.
