# 04 — Sensors: RPLidar + Orbbec Astra

Build and verify the 2D lidar and depth camera drivers. Prereq: [03 — Kobuki workspace](03-kobuki.md) finished — you already have `~/ws/src` symlinked to `external/`.

The driver sources are already in `external/` from the submodule init in 03:

| Path | Upstream | Branch |
|---|---|---|
| `external/rplidar_ros` | `Slamtec/rplidar_ros` | `ros2` |
| `external/OrbbecSDK_ROS2` | `orbbec/OrbbecSDK_ROS2` | `v2-main` |

If you skipped 03 and only want sensors, run the submodule init from there now.

## Build

```bash
cd ~/ws
source /opt/ros/jazzy/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --packages-select rplidar_ros orbbec_camera
source install/setup.bash
```

`OrbbecSDK_ROS2` ships prebuilt Orbbec SDK shared libraries (~130 MB total). The build copies them into `install/orbbec_camera/lib/`. First build is slow — closer to 5 min.

## RPLidar — udev + smoke test

```bash
sudo cp ~/ws/src/rplidar_ros/rplidar.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Unplug + replug the lidar USB; you should see `/dev/rplidar`.

Pick the launch file that matches your model (C1, A1, A2, A3, S1, S2, S3). For the C1:

```bash
ros2 launch rplidar_ros rplidar_c1_launch.py
```

In another shell:

```bash
ros2 topic hz /scan
```

Should report ~10 Hz. If you see 0 Hz or device-not-found errors, double-check that `/dev/rplidar` exists and the lidar's blue LED is on.

## Orbbec Astra — udev + smoke test

```bash
sudo cp ~/ws/src/OrbbecSDK_ROS2/orbbec_camera/scripts/99-obsensor-libusb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Replug the Astra USB. Then:

```bash
ros2 launch orbbec_camera astra.launch.py
```

Verify the depth stream is publishing:

```bash
ros2 topic hz /camera/depth/image_raw
ros2 topic hz /camera/color/image_raw
```

Both should be ≥ 15 Hz. If only color publishes, the device probably isn't getting enough power — that's the powered USB hub story from [01](01-hardware.md). The Astra USB plug is mandatory on the *powered* hub, not directly into the Pi.

## Visualizing on the workstation

On the workstation (not the Pi), with ROS 2 Jazzy installed and `ROS_DOMAIN_ID` matching:

```bash
rviz2
```

Add a `LaserScan` display on `/scan` (Fixed Frame: `laser`) and a `PointCloud2` on `/camera/depth/points`. You should see the lidar sweep and a live depth point cloud.

## Next

→ [05 — First drive: teleop and basic verification](05-first-drive.md) _(to be updated)_
