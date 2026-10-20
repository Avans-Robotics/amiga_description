# amiga_description

ROS 2 robot model (URDF/xacro) of the [farm-ng Amiga](https://amiga.farm-ng.com/): 60 in track, 31 in wheelbase, without Intelligence Kit (bare chassis with dash). Only the Amiga itself: arms, cameras and other sensors belong in the repo of the robot that carries them, mounted on `base_link`.

Tested with ROS 2 Jazzy.

## Use it in your workspace

Add it to your `.repos` file and pin a tag:

```yaml
repositories:
  amiga_description:
    type: git
    url: https://github.com/Avans-Robotics/amiga_description.git
    version: v0.5.1
```

```bash
vcs import src < my.repos
colcon build --packages-select amiga_description
```

Add `<exec_depend>amiga_description</exec_depend>` to your `package.xml`.

## Include it in your robot

```xml
<robot name="my_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <xacro:include filename="$(find amiga_description)/urdf/amiga.xacro"/>
  <xacro:amiga prefix=""/>

  <!-- your own parts, mounted on the Amiga -->
  <joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.5 0 0.6" rpy="0 0 0"/>
  </joint>
  <link name="camera_link"/>
</robot>
```

`prefix` is prepended to every link and joint name (for example `amiga_` gives `amiga_base_link`), for more than one Amiga in one scene.

## View it

```bash
ros2 launch amiga_description display.launch.py           # RViz with wheel sliders
ros2 launch amiga_description display.launch.py gui:=false
```

## Frames and dimensions

REP-103: X forward, Y left, Z up. Units: metres.

| Frame | Where |
|---|---|
| `base_footprint` | Ground plane under the centre of the four wheel axles |
| `base_link` | Centre of the four wheel axles, at axle height (0.216 m above `base_footprint`) |
| `front_left_wheel_link`, `front_right_wheel_link`, `rear_left_wheel_link`, `rear_right_wheel_link` | Wheel centres; continuous joints `*_wheel_joint` about Y |

| Dimension | Value |
|---|---|
| Wheelbase (front to rear axle) | 0.787 m (31 in) |
| Track (left to right wheel centre) | 1.524 m (60 in) |
| Wheel radius | 0.216 m |
| Top of the cross tubes above `base_link` | 0.435 m |

The chassis is modelled with boxes (side beams, cross tubes, wheel forks, dash mast), used for both visual and collision. The wheels have a mesh visual and a cylinder collision. Masses are estimates.

## Gazebo

[`urdf/amiga.gazebo.xacro`](urdf/amiga.gazebo.xacro) adds a skid-steer drive (gz sim DiffDrive system) and wheel joint states for Gazebo Harmonic. Include it after the Amiga:

```xml
<xacro:include filename="$(find amiga_description)/urdf/amiga.gazebo.xacro"/>
<xacro:amiga_gazebo prefix=""/>
```

or use the standalone model with `gazebo:=true`. Gazebo topics, to bridge with `ros_gz_bridge`:

| Gazebo topic | Type | ROS type | Direction |
|---|---|---|---|
| `/cmd_vel` | `gz.msgs.Twist` | `geometry_msgs/msg/Twist` | ROS to Gazebo: velocity command, the same topic and type as on the real robot |
| `/wheel/odometry` | `gz.msgs.Odometry` | `nav_msgs/msg/Odometry` | Gazebo to ROS: wheel odometry, `odom` to `base_footprint` |
| `/tf` | `gz.msgs.Pose_V` | `tf2_msgs/msg/TFMessage` | Gazebo to ROS: the odometry as TF; skip it if something else (an EKF) publishes `odom` |
| `/wheel_joint_states` | `gz.msgs.Model` | `sensor_msgs/msg/JointState` | Gazebo to ROS: wheel joints, bridge to `/joint_states` |

With a prefix, every topic starts with it. Wheel friction is set by `amiga_wheel_mu_side` and `amiga_wheel_mu_roll` in that file: tune them against the real robot.

Example bridge:

```bash
ros2 run ros_gz_bridge parameter_bridge \
  /cmd_vel@geometry_msgs/msg/Twist]gz.msgs.Twist \
  /wheel/odometry@nav_msgs/msg/Odometry[gz.msgs.Odometry \
  /tf@tf2_msgs/msg/TFMessage[gz.msgs.Pose_V \
  /wheel_joint_states@sensor_msgs/msg/JointState[gz.msgs.Model --ros-args -r /wheel_joint_states:=/joint_states
```

No `ros2_control` tags: the drive uses the Gazebo system directly, so `/cmd_vel` stays a plain `Twist`.

## Credits

Model by Andries van Dongen, from a FreeCAD model of the Amiga. Wheel meshes from [ucmercedrobotics/amiga-ros2-bridge](https://github.com/ucmercedrobotics/amiga-ros2-bridge) (Apache-2.0), see [meshes/ATTRIBUTION.md](meshes/ATTRIBUTION.md).

## License

Apache-2.0, see [LICENSE](LICENSE).
