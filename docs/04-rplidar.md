# 04 — RPLidar (Slamtec) driver

Build and verify the 2D lidar driver. Prereq: [03 — Kobuki workspace](03-kobuki.md) finished — `~/ws/src` is symlinked to `external/` and the base build succeeded.

## Driver choice

Slamtec maintains two ROS 2 driver repos. We use the one pinned in `external/`:

| Repo | Status | Used here |
|---|---|---|
| [`Slamtec/rplidar_ros`](https://github.com/Slamtec/rplidar_ros) (`ros2` branch) | Long-standing driver, all RPLidar models | ✅ (`external/rplidar_ros`) |
| [`Slamtec/sllidar_ros2`](https://github.com/Slamtec/sllidar_ros2) (`main`) | Newer driver for the same hardware | Alternative — see bottom of page |

Both work on the C1. If you need the newer launch files (`sllidar_c1_launch.py`, configurable serial port at launch), see the [sllidar_ros2 alternative](#alternative-sllidar_ros2-driver) below.

## Build

```bash
cd ~/ws
source /opt/ros/jazzy/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --packages-select rplidar_ros
source install/setup.bash
```

## udev rule

```bash
sudo cp ~/ws/src/rplidar_ros/rplidar.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Unplug + replug the lidar USB; you should see `/dev/rplidar`.

## Smoke test

Pick the launch file that matches your model (C1, A1, A2, A3, S1, S2, S3). For the C1:

```bash
ros2 launch rplidar_ros rplidar_c1_launch.py
```

In another shell:

```bash
ros2 topic hz /scan          # should report ~10 Hz
ros2 topic echo /scan --once # one full sweep of ranges
ros2 topic info /scan        # publisher / type sanity
```

If you see 0 Hz or device-not-found errors, double-check that `/dev/rplidar` exists and the lidar's blue LED is on.

## Alternative: `sllidar_ros2` driver

If you prefer the newer driver, build it side-by-side without removing `rplidar_ros`:

```bash
cd ~/ws/src
git clone https://github.com/Slamtec/sllidar_ros2.git -b main
cd ~/ws
colcon build --symlink-install --packages-select sllidar_ros2
source install/setup.bash

ros2 launch sllidar_ros2 sllidar_c1_launch.py serial_port:=/dev/rplidar
# Or, for A2 / A3:
# ros2 launch sllidar_ros2 sllidar_a2_launch.py serial_port:=/dev/rplidar
# ros2 launch sllidar_ros2 sllidar_a3_launch.py serial_port:=/dev/rplidar
```

This is a `git clone` rather than a submodule because it is an alternative, not the canonical driver for this build.

## Next

→ [05 — Orbbec Astra depth camera](05-orbbec-astra.md)
