# ROS 1 Turtlesim Controller

ROS 1 package in C++ that lets the user drive two turtles in Turtlesim from the terminal, while a second node monitors their distance and stops them before they collide with each other or with the walls.

> First assignment of the Research Track 1 course (1st year, 1st semester), MSc Robotics Engineering, University of Genoa.

## Package structure

```
assignment1_rt/
├── src/
│   ├── UI.cpp          # ass1_node1: user interface and turtle commands
│   └── distance.cpp    # ass1_node2: distance monitoring and collision stop
├── CMakeLists.txt
└── package.xml
```

## Nodes

### `ass1_node1` (`UI.cpp`): user interface

1. Spawns a second turtle, `turtle2`, at (2.5, 2.5) through the `/spawn` service.
2. Asks the user, in the terminal:
   - the turtle to control (`turtle1` or `turtle2`);
   - the linear velocity along x and y;
   - the angular velocity.
3. Publishes the command on `/<turtle>/cmd_vel` for a fixed time (`duration_movement`), then stops the turtle and asks for a new command.

| Type | Name | Message |
|---|---|---|
| Service client | `/spawn` | `turtlesim/Spawn` |
| Publisher | `/turtle1/cmd_vel`, `/turtle2/cmd_vel` | `geometry_msgs/Twist` |
| Publisher | `/selected_turtle` | `std_msgs/String` |

### `ass1_node2` (`distance.cpp`): distance monitoring

At 10 Hz, the node:
1. computes the Euclidean distance between the two turtles from their poses and publishes it on `/turtle_distance`;
2. stops both turtles if the distance drops below **1** and is decreasing;
3. stops a turtle if it gets closer than the border thresholds (x or y ≥ 9 or ≤ 2) while moving towards the wall.

Each stop is reported in the terminal with the reason (collision between turtles or with the border).

| Type | Name | Message |
|---|---|---|
| Subscriber | `/turtle1/pose`, `/turtle2/pose` | `turtlesim/Pose` |
| Publisher | `/turtle_distance` | `std_msgs/Float32` |
| Publisher | `/turtle1/cmd_vel`, `/turtle2/cmd_vel` | `geometry_msgs/Twist` |

## Requirements

- ROS Noetic
- `turtlesim` package

## Installation

Clone the repository in the `src/` folder of a catkin workspace and build it:

```bash
cd ~/catkin_ws/src
git clone https://github.com/mrtelisa/ros1_turtlesim_controller.git
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

## Usage

Run each command in a separate terminal:

```bash
roscore
rosrun turtlesim turtlesim_node
rosrun assignment1_rt ass1_node1     # user interface
rosrun assignment1_rt ass1_node2     # distance monitoring
```

In the terminal of `ass1_node1`, enter the turtle name and the three velocities, pressing Enter after each value. To close the node, press `Ctrl+C`, then `q` and Enter.

To monitor the distance between the turtles:

```bash
rostopic echo /turtle_distance
```

## Branches

| Branch | Content |
|---|---|
| `main` | The assignment |
| `exams` | Exam exercise: when a turtle collides with the other one or with a wall, `ass1_node2` respawns it at a random safe position (inside the area and at least 2 units from both turtles) instead of stopping it |
