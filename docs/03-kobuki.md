# 03 — Kobuki: build the ROS 2 workspace

By the end of this doc the Pi 5 will have a `~/ws` colcon workspace that compiles `kobuki_ros` and its dependencies from source, and you'll have verified the base talks over USB.

Prereq: [02 — Pi 5 setup](02-pi5-setup.md) completed (Ubuntu 24.04, ROS 2 Jazzy, `ros-jazzy-desktop` or `-ros-base` installed).

## Pull the source via submodules

The upstream Kobuki + ECL repos live under [`external/`](../external/) as pinned git submodules. If you cloned this repo without `--recurse-submodules`, fetch them now:

```bash
cd ~/hacky-robotics-devkit
git submodule update --init --recursive
```

Six packages will land in `external/`:

| Path | Upstream | Branch |
|---|---|---|
| `external/ecl_lite` | `stonier/ecl_lite` | `devel` |
| `external/ecl_tools` | `stonier/ecl_tools` | `devel` |
| `external/ecl_core` | `stonier/ecl_core` | `devel` |
| `external/kobuki_core` | `kobuki-base/kobuki_core` | `devel` |
| `external/kobuki_ros_interfaces` | `kobuki-base/kobuki_ros_interfaces` | `devel` |
| `external/kobuki_ros` | `kobuki-base/kobuki_ros` | `devel` |

## Wire the colcon workspace

Point `~/ws/src` at the `external/` directory using a symlink. This keeps source ownership inside this repo and avoids copies:

```bash
mkdir -p ~/ws
ln -s ~/hacky-robotics-devkit/external ~/ws/src
```

Install rosdep keys and resolve system dependencies:

```bash
sudo apt install -y python3-rosdep python3-colcon-common-extensions
sudo rosdep init 2>/dev/null || true
rosdep update
cd ~/ws
rosdep install --from-paths src --ignore-src -r -y
```

## Build

```bash
cd ~/ws
source /opt/ros/jazzy/setup.bash
colcon build --symlink-install
```

First build takes ~10–20 min on a Pi 5 with active cooling. If you hit OOM, bump swap as described in [02 — Swap](02-pi5-setup.md#pi-5specific-tuning).

Source the workspace overlay:

```bash
echo 'source ~/ws/install/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

## udev rule for the Kobuki USB device

Without this, every reboot the Kobuki shows up at a different `/dev/ttyUSB*` and `kobuki_ros` refuses to start.

```bash
sudo cp ~/ws/src/kobuki_ros/60-kobuki.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Unplug + replug the Kobuki USB. There should now be a `/dev/kobuki` symlink.

```bash
ls -l /dev/kobuki
```

## Smoke test

Power the Kobuki on (push button on the base). Then:

```bash
ros2 launch kobuki_node kobuki_node-launch.py
```

In another SSH session:

```bash
ros2 topic echo /events/wheel_drop --once
ros2 topic echo /battery_state --once
```

You should see a wheel-drop message and a battery voltage around 16.7 V (fully charged). If `ros2 topic list` is empty, check that `ROS_DOMAIN_ID` matches what you set in [02](02-pi5-setup.md#shell-setup).

## References & lineage

The Kobuki driver stack has changed hands a few times. The submodules under `external/` track the [kobuki-base](https://github.com/kobuki-base) org (Daniel Stonier et al.), which is the actively maintained ROS 2 line and what this repo builds against. The other forks are useful when chasing history, firmware quirks, or alternative ROS 2 ports:

| Repo | Role | When to look here |
|---|---|---|
| [yujinrobot/kobuki](https://github.com/yujinrobot/kobuki) | Original Yujin Robot upstream (ROS 1) | Firmware-level questions, original hardware docs, legacy ROS 1 driver |
| [kobuki-base](https://github.com/kobuki-base) | ROS 2 port — what we build from | Source of `kobuki_core`, `kobuki_ros`, `kobuki_ros_interfaces` submodules |
| [IntelligentRoboticsLabs/kobuki](https://github.com/IntelligentRoboticsLabs/kobuki) | Alternative ROS 2 fork (Univ. Rey Juan Carlos) | Cross-checking patches, alt launch files, classroom-tested configs |

## Next

→ [04 — Sensors: RPLidar + Orbbec Astra](04-sensors.md)
