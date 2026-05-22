# 03 — Kobuki: build the ROS 2 workspace

By the end of this doc the Pi 5 will have a `~/ws` colcon workspace that compiles `kobuki_ros` and its dependencies from source, and you will have driven the base with a keyboard teleop.

Prereq: [02 — Pi 5 setup](02-pi5-setup.md) completed (Ubuntu 24.04, ROS 2 Jazzy, `ros-jazzy-desktop` or `-ros-base` installed).

## Why build from source

`kobuki_ros` is not published as an apt binary for Jazzy. The kobuki-base org targets Foxy (Ubuntu 20.04) for binaries, but their `devel` branches use `ament_cmake` and build cleanly on Jazzy. We pin to specific upstream branches via git submodules in [`external/`](../external/) so the build is reproducible.

## Pull the source via submodules

If you cloned this repo without `--recurse-submodules`, fetch them now:

```bash
cd ~/hacky-robotics-devkit
git submodule update --init --recursive
```

What lands in `external/` and what each package contributes to the build:

| Path | Upstream | Branch | Role |
|---|---|---|---|
| `external/ecl_lite` | `stonier/ecl_lite` | `devel` | low-level C++ utility lib |
| `external/ecl_tools` | `stonier/ecl_tools` | `devel` | ament tooling |
| `external/ecl_core` | `stonier/ecl_core` | `devel` | C++ utility lib |
| `external/sophus` | `stonier/sophus` | `release/1.2.x` | Lie-group geometry (kobuki_core dep) |
| `external/kobuki_core` | `kobuki-base/kobuki_core` | `devel` | C++ driver for Kobuki hardware |
| `external/kobuki_ros_interfaces` | `kobuki-base/kobuki_ros_interfaces` | `devel` | ROS 2 message definitions |
| `external/cmd_vel_mux` | `kobuki-base/cmd_vel_mux` | `devel` | priority mux for `cmd_vel` topics |
| `external/kobuki_velocity_smoother` | `kobuki-base/kobuki_velocity_smoother` | `devel` | accel-limited smoother for cmd_vel |
| `external/kobuki_ros` | `kobuki-base/kobuki_ros` | `devel` | ROS 2 nodes, launchers, keyop |

## Wire the colcon workspace

Point `~/ws/src` at `external/` with a symlink. This keeps source ownership inside this repo and avoids copies:

```bash
mkdir -p ~/ws
ln -s ~/hacky-robotics-devkit/external ~/ws/src
```

Install rosdep keys and resolve system dependencies:

```bash
sudo apt install -y python3-rosdep python3-colcon-common-extensions python3-vcstool
sudo rosdep init 2>/dev/null || true
rosdep update
cd ~/ws
source /opt/ros/jazzy/setup.bash
rosdep install --from-paths src --ignore-src -r -y
```

## Build

```bash
cd ~/ws
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
sudo cp ~/ws/src/kobuki_core/60-kobuki.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Unplug + replug the Kobuki USB. There should now be a `/dev/kobuki` symlink:

```bash
ls -l /dev/kobuki
```

## Match the serial device in the launch params

`kobuki_node` defaults to `/dev/kobuki` via the udev rule above. If you skipped the udev step or you have multiple USB-serial devices, point the param file at the actual device (commonly `/dev/ttyUSB0` when nothing else is plugged in):

```bash
sed -i 's|/dev/kobuki|/dev/ttyUSB0|' \
  ~/ws/install/kobuki_node/share/kobuki_node/config/kobuki_node_params.yaml
```

Prefer the udev path (`/dev/kobuki`) for anything permanent — `ttyUSB*` numbering changes whenever you replug.

## Smoke test

Power the Kobuki on (push button on the base). Then:

```bash
ros2 launch kobuki_node kobuki_node-launch.py
```

In another SSH session, confirm the base is publishing:

```bash
ros2 topic echo /events/wheel_drop --once
ros2 topic echo /battery_state --once
```

You should see a wheel-drop message and a battery voltage around 16.7 V (fully charged). If `ros2 topic list` is empty, check that `ROS_DOMAIN_ID` matches what you set in [02](02-pi5-setup.md#shell-setup).

## Drive it with the keyboard

`kobuki_keyop` publishes to `/commands/velocity` by default. Remap it onto `/cmd_vel` so the base picks it up:

```bash
ros2 run kobuki_keyop kobuki_keyop_node --ros-args -r /cmd_vel:=/commands/velocity
```

Arrow keys drive, space stops, `q` quits. The base should beep on enable. Put it on blocks or in an open area before you press anything — there is no soft-start; you go from 0 to commanded velocity in one tick.

## References & lineage

The Kobuki driver stack has changed hands a few times. The submodules under `external/` track the [kobuki-base](https://github.com/kobuki-base) org (Daniel Stonier et al.), which is the actively maintained ROS 2 line and what this repo builds against. The other forks are useful when chasing history, firmware quirks, or alternative ROS 2 ports:

| Repo | Role | When to look here |
|---|---|---|
| [yujinrobot/kobuki](https://github.com/yujinrobot/kobuki) | Original Yujin Robot upstream (ROS 1) | Firmware-level questions, original hardware docs, legacy ROS 1 driver |
| [kobuki-base](https://github.com/kobuki-base) | ROS 2 port — what we build from | Source of the `kobuki_*` and `cmd_vel_mux` submodules |
| [IntelligentRoboticsLabs/kobuki](https://github.com/IntelligentRoboticsLabs/kobuki) | Alternative ROS 2 fork (Univ. Rey Juan Carlos) | Cross-checking patches, alt launch files, classroom-tested configs |

## Next

→ [04 — RPLidar driver](04-rplidar.md)
