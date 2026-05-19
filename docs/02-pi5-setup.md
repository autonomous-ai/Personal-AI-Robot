# 02 — Raspberry Pi 5: Ubuntu 24.04 + ROS 2 Jazzy

## Flash Ubuntu 24.04 to the microSD card

1. Install Raspberry Pi Imager on the workstation: <https://www.raspberrypi.com/software/>
2. Download Ubuntu Desktop 24.04 LTS for arm64 (Raspberry Pi): <https://ubuntu.com/download/raspberry-pi>
   - Server image is fine if you do not need a local desktop; it boots faster and uses less RAM.
3. Insert the microSD into the workstation.
4. In Imager: **Choose device → Raspberry Pi 5**, **Choose OS → Use custom**, pick the downloaded `.img.xz`, **Choose storage → the microSD**.
5. Click the gear / "Edit settings" before writing. Set:
   - Hostname (e.g. `hacky-pi`)
   - Username + password (do not use `ubuntu` / `ubuntu`)
   - WiFi SSID + password + country
   - SSH enabled, password auth (or paste your public key)
6. Write. Eject.

## First boot

Insert the SD into the Pi 5. If you have HDMI + keyboard, plug them in; otherwise the Pi will boot headless and join WiFi from the credentials you baked in.

Find its IP from the router DHCP table, or scan the network:

```bash
sudo nmap -sn 192.168.1.0/24 | grep -i raspberry
```

SSH in:

```bash
ssh <username>@<pi-ip>
```

Update:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

## ROS 2 Jazzy install

Jazzy Jalopy is the current LTS for Ubuntu 24.04 (Noble) and supports arm64 binaries. Reference: <https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html>

```bash
# locale
sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# enable universe
sudo apt install -y software-properties-common
sudo add-apt-repository universe -y

# add ROS 2 apt source
sudo apt update && sudo apt install -y curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# install
sudo apt update
sudo apt install -y ros-jazzy-desktop ros-dev-tools
```

`desktop` includes rviz2 and demo nodes. On a server install, swap for `ros-jazzy-ros-base` and skip rviz2 on the robot itself (run it on the workstation instead).

## Shell setup

Add this to `~/.bashrc`:

```bash
source /opt/ros/jazzy/setup.bash
export ROS_DOMAIN_ID=42         # any int 0–101; pick one and reuse on the workstation
export ROS_LOCALHOST_ONLY=0
```

Then `source ~/.bashrc`.

Verify:

```bash
ros2 doctor
ros2 run demo_nodes_cpp talker
```

In a second SSH session:

```bash
ros2 run demo_nodes_cpp listener
```

Both should connect and the listener should print `Hello World: N`. If they do not, ROS_DOMAIN_ID is the first thing to check.

## Pi 5–specific tuning

**Active cooling.** Without a fan the Pi 5 thermal-throttles at sustained 80% CPU. ROS 2 + lidar + depth stream will hit that. Active Cooler is mandatory, not optional.

**Swap.** Default Ubuntu 24.04 arm64 ships with 2 GB swap. Building Kobuki packages from source (next doc) eats memory. If you have 4 GB RAM, bump swap to 4 GB:

```bash
sudo systemctl disable --now dphys-swapfile 2>/dev/null || true
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

With 8 GB+ RAM, skip this.

**USB current cap.** Pi 5 by default limits total downstream USB current. The Astra depth camera and Slamtec lidar together can trip this. The proper fix is the powered USB hub recommended in [01-hardware](01-hardware.md). The lazy fix — only valid with a 5 A PD-capable PSU — is to remove the cap by adding `usb_max_current_enable=1` to `/boot/firmware/config.txt` and rebooting. Use a hub. Do not rely on the cap removal alone.

## Save the SD

Before going further, image the SD card to your workstation:

```bash
# on the workstation, with the SD inserted (Linux example)
sudo dd if=/dev/sdX of=dlr-pi5-base-ros2-jazzy.img bs=4M status=progress
```

You will rebuild this at least twice. Cache the working baseline.

## Next

→ [03 — Kobuki workspace build](03-kobuki.md)
