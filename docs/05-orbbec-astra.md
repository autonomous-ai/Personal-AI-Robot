# 05 — Orbbec Astra depth camera

Build the ROS 2 driver for the Astra and verify a depth + color stream. Prereq: [03 — Kobuki workspace](03-kobuki.md) — `~/ws/src` is symlinked to `external/`.

## Driver: `OrbbecSDK_ROS2`

We use Orbbec's official ROS 2 wrapper (pinned at `external/OrbbecSDK_ROS2`, branch `v2-main`). It bundles the prebuilt Orbbec SDK shared libraries (~130 MB total). The build copies them into `install/orbbec_camera/lib/`.

```bash
cd ~/ws
source /opt/ros/jazzy/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --packages-select orbbec_camera
source install/setup.bash
```

First build is slow — closer to 5 min.

## udev rule

```bash
sudo cp ~/ws/src/OrbbecSDK_ROS2/orbbec_camera/scripts/99-obsensor-libusb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Replug the Astra USB.

## Smoke test

```bash
ros2 launch orbbec_camera astra.launch.py
```

Verify the depth stream is publishing:

```bash
ros2 topic hz /camera/depth/image_raw
ros2 topic hz /camera/color/image_raw
```

Both should be ≥ 15 Hz. If only color publishes, the device probably isn't getting enough power — see the powered-USB-hub note in [02 § USB current cap](02-pi5-setup.md#pi-5specific-tuning). The Astra USB plug is mandatory on the *powered* hub, not directly into the Pi.

## Sanity check with the standalone Astra SDK (optional)

If the ROS driver behaves oddly and you want to confirm the camera itself is fine, the standalone Astra SDK has a tiny C++ binary that polls depth frames directly. This bypasses ROS entirely — useful for isolating whether the issue is in the driver wrapper or in the hardware/USB chain.

Download the Linux aarch64 build from <https://www.orbbec.com/developers/astra-sdk/> (current direct link: <https://dl.orbbec3d.com/dist/astra/v2.1.3/AstraSDK-v2.1.3-Linux-arm.zip>), then:

```bash
cd AstraSDK-v2.1.3-Linux-aarch64/install
sudo ./install.sh                          # installs orbbec-usb.rules

cd ../bin
./DepthReaderPoll
```

Expected output starts with:

```
depth sensor -- hFov: 1.022600 radians vFov: 0.796616 radians
depth sensor -- serial number: 15102210050
Chip ID: MX400
depth frameIndex 0 value 0
depth frameIndex 1 value 0
...
```

A serial number and increasing `frameIndex` mean the hardware + USB path is healthy; any remaining issue is in the ROS layer.

## Visualizing on the workstation

On the workstation (not the Pi), with ROS 2 Jazzy installed and `ROS_DOMAIN_ID` matching:

```bash
rviz2
```

Add a `LaserScan` display on `/scan` (Fixed Frame: `laser`) and a `PointCloud2` on `/camera/depth/points`. You should see the lidar sweep and a live depth point cloud.

## Next

→ [06 — First drive: teleop and basic verification](06-first-drive.md) _(to be updated)_
