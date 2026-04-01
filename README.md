# Lab: The Transform System (tf2)
### RA305 Robotics Programming — ROS 2 Jazzy

> Based on *"The Transform System (tf2)"* by **Josh Newans** — [Articulated Robotics](https://articulatedrobotics.xyz)

---

## What is tf2?

Robots have many parts — a camera, a gripper, a lidar, wheels — each with its own coordinate frame. **tf2** is ROS 2's system for tracking how all those frames relate to each other so you can convert a point from one frame to any other automatically.

**Key terms:**
- **Frame** — a coordinate system attached to a part of the robot or environment
- **Transform** — the translation + rotation between two frames
- **Transform tree** — all frames linked in a parent → child tree structure; every frame has exactly one parent
- **Broadcast** — publishing a transform to `/tf` or `/tf_static`
- **Listen** — subscribing to those topics to look up transforms
- **Static transform** — fixed, doesn't change (camera bolted to chassis)
- **Dynamic transform** — changes over time (a spinning joint)

The data flow: nodes *broadcast* to `/tf` or `/tf_static` → other nodes *listen* and convert coordinates as needed.

---

## Prerequisites

```bash
# RViz and tf2 tools are included with ROS 2 Jazzy desktop install
# Verify tf2_tools is available:
ros2 pkg list | grep tf2_tools
```

---

## Part 1 — Static Frames

Manually broadcast two static transforms to build a small tree:

```
world
  └── robot_1   (2m in X, 1m in Y, 45° yaw)
        └── robot_2   (1m ahead of robot_1)
```

### Step 1: Broadcast world → robot_1

The `static_transform_publisher` tool broadcasts a fixed transform from a parent frame to a child frame. The full argument pattern is:

```bash
ros2 run tf2_ros static_transform_publisher x y z yaw pitch roll parent_frame child_frame
```

- `x y z` — translation in meters along each axis
- `yaw pitch roll` — rotation in **radians**, applied after translation, relative to the local coordinate system
- `parent_frame` → `child_frame` — defines the direction of the relationship in the tree

For this step, we want `robot_1` to be 2 m in X, 1 m in Y from `world`, rotated 45° (0.785 rad) in yaw. Open a terminal:

```bash
ros2 run tf2_ros static_transform_publisher 2 1 0 0.785 0 0 world robot_1
```

Leave this terminal running — stopping it removes the broadcast.

### Step 2: Broadcast robot_1 → robot_2

Now add a second robot that always sits 1 m ahead of `robot_1` — like a sidecar. Because this transform is defined *relative to* `robot_1` (not `world`), whenever `robot_1` moves or rotates, `robot_2` moves with it automatically.

Open a second terminal:

```bash
ros2 run tf2_ros static_transform_publisher 1 0 0 0 0 0 robot_1 robot_2
```

Leave this terminal running as well.

### Step 3: Visualize in RViz

**RViz** (ROS Visualization) is ROS 2's built-in 3D visualization tool. It can display TF frames, robot models, sensor data, and more. It is part of the `rviz2` package and can be launched two ways — both do the same thing:

```bash
ros2 run rviz2 rviz2
```

```bash
rviz2
```

Open a third terminal and launch RViz, then:
1. Top-left **Fixed Frame** → set to `world`
2. Click **Add** (bottom-left) → **TF** → OK
3. Expand the TF panel → enable **Show Names**

You should see all three frames with axes and labels.

### Step 4: View the transform tree

Open a fourth terminal and run:

```bash
ros2 run tf2_tools view_frames
```

This listens to `/tf` and `/tf_static` for a few seconds, then generates a PDF of the tree structure in your current directory. The filename includes a timestamp, so it will look something like:

```
frames_2025_03_15_14.32.07.pdf
```

Open it with:

```bash
xdg-open frames_*.pdf
```

> The `*` wildcard matches the timestamp so you don't have to type the full filename.

### Step 5: Change the rotation and observe

Go back to terminal 1, stop it (Ctrl-C), and re-run with 90°:

```bash
ros2 run tf2_ros static_transform_publisher 2 1 0 1.57 0 0 world robot_1
```

Watch what happens to `robot_2` in RViz — it moves even though you only changed `robot_1`.

### Answer questions in Canvas - Transform System (tf2) in ROS2


## Part 2 — Dynamic Transforms with robot_state_publisher
 
**Stop all terminals from Part 1 before continuing.**
 
For a robot with moving joints, you describe the robot in a **URDF** file and let `robot_state_publisher` handle all the TF broadcasting automatically.
 
Data flow:
 
```
URDF file → robot_state_publisher → /tf  /tf_static  /robot_description
                                              ↑
joint_state_publisher_gui → /joint_states ───┘
```
 
### Step 1: Install required packages
 
```bash
sudo apt install ros-jazzy-joint-state-publisher-gui ros-jazzy-xacro
```
 
### Step 2: Get the example URDF
 
Download the example URDF from Josh Newans' gist:
 
```bash
wget -O /tmp/simple_arm.urdf.xacro \
  https://gist.githubusercontent.com/joshnewans/69cb8a049fb4606b0a6bdecd6933164e/raw/simple_arm.urdf.xacro
```
 
### Step 3: Launch robot_state_publisher
 
```bash
ros2 run robot_state_publisher robot_state_publisher \
  --ros-args -p robot_description:="$(xacro /tmp/simple_arm.urdf.xacro)"
```
 
> `xacro` preprocesses the file; `$(...)` passes the full URDF text (not a path) as the parameter.
 
### Step 4: Launch joint_state_publisher_gui
 
Open a new terminal:
 
```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```
 
A slider panel will appear for each moveable joint.
 
### Step 5: Open RViz
 
Open a new terminal:
 
```bash
rviz2
```
 
In RViz:
1. Fixed Frame → `base_link`
2. Add → **TF** → OK
3. Add → **RobotModel** → OK → set **Description Topic** to `/robot_description`
 
Move the sliders and watch the frames and model update in real time.
 
### Step 6: Inspect the system
 
```bash
ros2 node list
ros2 topic list
ros2 topic echo /joint_states --once
ros2 run tf2_tools view_frames && xdg-open frames.pdf
```
### Answer questions in Canvas - Transform System (tf2) in ROS2
