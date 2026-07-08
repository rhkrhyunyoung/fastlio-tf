# 🚀 Navigation2 Configuration & Integration
To achieve seamless autonomous navigation with FAST-LIO Localization, the Nav2 parameter configuration was optimized to align with the restored map -> odom -> base_link TF hierarchy.

1. TF Tree Evolution
Previously, the odom frame was often omitted or disconnected in standard FAST-LIO implementations, which caused failures in Nav2's local planners. By implementing the inverse transform logic, the TF tree was successfully restored to the ROS 2 standard.

Before: (Incomplete TF): map -> base_link (Missing odom)

After: (Correct Nav2 Standard): map -> odom -> base_link (Connected)
<img width="1580" height="690" alt="Screenshot from 2026-07-08 16-32-51" src="https://github.com/user-attachments/assets/9d552d7d-b2e0-4da5-b582-626f8eaa36fd" />

2. Why Modify Nav2 Parameters?
Since system-level configuration files (in /opt/ros/...) are read-only and reset upon updates, a local copy of the Nav2 parameters must be used.
- Frame Alignment: Ensured global_frame is set to map and local_frame is set to odom.
- Topic Remapping: The odometry source is redirected to /Odometry (published by our modified FAST-LIO node).
- Stability: The restored odom frame provides the continuous motion required by the Controller Server (Local Planner).

3. Modify Local YAML
- global_frame: map
- odom_frame: odom
- robot_base_frame: base_link (or body)
- odom_topic: /Odometry

4. Results
The screenshot below shows the robot's localization being maintained by FAST-LIO while Nav2 successfully generates costmaps and paths based on the restored TF tree.

5. Run
```
ros2 launch nav2_bringup bringup_launch.py \
    use_sim_time:=false \
    autostart:=true \
    map:=/path/to/your/map.yaml \
    params_file:=/home/$USER/nav2_config/my_nav2_params.yaml
```
