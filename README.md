# Autonomous Deep Learning Robot


---


## Bill of materials

- Kobuki mobile base (Yujin Robot)
- Kobuki docking station + power supply
- Turtlebot 2 wooden discs and struts
- USB type A → type B cable (Kobuki control + power passthrough)
- Raspberry Pi 5 (8 GB recommended)
- Raspberry Pi 27 W USB-C PD power supply (for desk use; on-robot power comes from Kobuki)
- Raspberry Pi Active Cooler or equivalent
- microSD card, 32 GB+ class A2 (or NVMe via M.2 HAT)
- Slamtec RPLidar (C1 / A1 / A2 — any 2D USB lidar)
- Orbbec Astra depth camera (or replace with Intel RealSense / Orbbec Gemini if Astra unavailable)
- Powered USB hub (USB 3.0, externally powered — Pi 5's USB ports do not deliver enough sustained current for camera + lidar + Kobuki together)
- USB-C step-down regulator: 12 V (Kobuki aux power output) → 5 V / 5 A USB-C PD trigger, to power the Pi from the Kobuki
- Ethernet cable (initial provisioning only)

---

## System architecture

```
[Kobuki base] ──USB-A↔B──┐
                          │
[RPLidar]    ──USB-A─────┤
                          ├──→ [Powered USB hub] ──USB-C──→ [Raspberry Pi 5]
[Orbbec Astra] ──USB-A───┘                                       │
                                                                  │ WiFi
                                                                  ▼
                                                          [Workstation w/ rviz2]
```

Kobuki 12 V aux out → step-down → Pi 5 USB-C. Do not draw Pi power from the Kobuki USB-B port; it cannot supply 5 A.

---

## Getting started

Work through the docs in order:

1. [Hardware assembly and wiring](docs/01-hardware.md)
2. [Pi 5: Ubuntu 24.04 install + ROS 2 Jazzy](docs/02-pi5-setup.md)
3. [Kobuki: build the ROS 2 workspace](docs/03-kobuki.md)
4. [Sensors: RPLidar + Orbbec Astra](docs/04-sensors.md)
5. [First drive: teleop and basic verification](docs/05-first-drive.md)
6. [Deep learning on Pi 5: realistic options](docs/06-deep-learning.md)
7. [Troubleshooting](docs/07-troubleshooting.md)
8. [Remote access (optional): Tailscale + NoMachine](docs/08-remote-access.md)

---

## Status

Tested on: Raspberry Pi 5 (8 GB), Ubuntu 24.04.3 LTS arm64, ROS 2 Jazzy Jalopy, Kobuki (firmware 1.2), Slamtec RPLidar C1, Orbbec Astra (serial 15102210050).



## Credits

- kobuki-base maintainers — keeping the Kobuki stack alive on ROS 2
- Slamtec, Orbbec — driver SDKs
