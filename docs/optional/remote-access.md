# Optional — Remote access (Tailscale + NoMachine)

These are not required to run the robot. They are quality-of-life tools for working with the Pi 5 from another network or seeing the Ubuntu desktop without HDMI.

| Tool | What it does | When you want it |
|---|---|---|
| [Tailscale](https://tailscale.com) | WireGuard-based mesh VPN — gives the Pi a stable address reachable from anywhere | SSHing into the robot off your home LAN |
| [NoMachine](https://www.nomachine.com) | Low-latency remote desktop | Driving rviz2 / GUI tools running *on* the Pi |

If you only need SSH + rviz2 on the workstation, you can skip both.

## Tailscale

Reference: <https://tailscale.com/download/linux>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

The CLI prints a URL — open it on the workstation to log into the same Tailscale account, and the Pi shows up in your tailnet. From then on, `ssh <user>@<pi-hostname>` works over Tailscale's mesh.

Pick the hostname you set with `rpi-imager` (or `sudo hostnamectl set-hostname hacky-pi`) — Tailscale's MagicDNS uses it.

## NoMachine

Reference: <https://download.nomachine.com/download/?id=30&platform=linux&distro=arm>

Pick the `arm64` build matching your Ubuntu (24.04 = `arm64`, not `armhf`). Example for v9.3.7:

```bash
# on the Pi
wget https://download.nomachine.com/download/9.3/Arm/nomachine_9.3.7_1_arm64.deb
sudo dpkg -i nomachine_9.3.7_1_arm64.deb
```

The installer drops `nxserver` into `/etc/init.d/` and starts it. On the workstation, install the matching NoMachine client and connect to the Pi's Tailscale address (or LAN IP) on port 4000.

You'll need a real desktop session for NoMachine to attach to — if you installed `ros-jazzy-ros-base` instead of `ros-jazzy-desktop`, also install `ubuntu-desktop-minimal` first.

## Security notes

- Tailscale ACLs default to "all my devices can talk to each other." If you share a tailnet with other people, lock the Pi down with [tags + ACLs](https://tailscale.com/kb/1068/acl-tags).
- NoMachine listens on port 4000 with password auth by default. If you expose it on a LAN with untrusted devices, switch to key auth in `/usr/NX/etc/server.cfg` and restart `nxserver`.
