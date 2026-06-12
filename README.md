# Razer Laptop Control — Revived

Control fan speed, power profiles, CPU/GPU boost, battery charge limits, and keyboard RGB on Razer Blade laptops under Linux. No kernel modules, no DKMS — userspace HID over a systemd user service.

[![License: GPL-2.0](https://img.shields.io/badge/License-GPL%202.0-blue.svg)](LICENSE)
[![Release](https://img.shields.io/github/v/release/encomjp/razer-control-revived)](https://github.com/encomjp/razer-control-revived/releases/latest)

![Main Window](https://github.com/user-attachments/assets/48b10737-76ed-45a0-886b-df5ba6bea30d)

## Install

Grab the latest package from the [releases page](https://github.com/encomjp/razer-control-revived/releases/latest), then:

```bash
# Debian / Ubuntu
sudo apt install ./razercontrol-revived_*_amd64.deb

# Fedora / RHEL
sudo dnf install ./razercontrol-*.x86_64.rpm

# Any distro (tarball)
tar -xzf razer-control-*-x86_64.tar.gz && cd razer-control-*/ && sudo ./install.sh
```

Each package installs the daemon, CLI, GUI, systemd service, udev rules, and desktop entry. **Log out and back in** (or reboot) after install for udev rules to take effect.

<details>
<summary>NixOS</summary>

```nix
# flake.nix
inputs.razerdaemon = {
  url = "github:encomjp/razer-control-revived";
  inputs.nixpkgs.follows = "nixpkgs";
};

# configuration.nix
imports = [ inputs.razerdaemon.nixosModules.default ];
services.razer-laptop-control.enable = true;
```
</details>

<details>
<summary>Build from source</summary>

```bash
# Arch
sudo pacman -S rust cargo dbus libusb hidapi pkgconf systemd gtk4 libadwaita git

# Ubuntu/Debian
sudo apt install -y rustc cargo libdbus-1-dev libusb-1.0-0-dev libhidapi-dev \
    pkg-config libsystemd-dev libgtk-4-dev libadwaita-1-dev git

# Fedora
sudo dnf install -y rust cargo dbus-devel libusb1-devel hidapi-devel \
    pkgconf systemd-devel gtk4-devel libadwaita-devel git

# Then:
git clone https://github.com/encomjp/razer-control-revived
cd razer-control-revived/razer_control_gui
./install.sh install
```
</details>

## Usage

### GUI

Launch from your app menu ("Razer Settings") or:

```bash
razer-settings
```

### CLI

```bash
# Read
razer-cli read fan ac
razer-cli read power ac
razer-cli read brightness ac
razer-cli read logo ac
razer-cli read bho

# Write
razer-cli write fan ac 0              # auto
razer-cli write fan ac 4000           # manual RPM
razer-cli write power ac 1 0 0        # Gaming mode
razer-cli write power ac 4 2 2        # Custom, CPU=High GPU=High
razer-cli write brightness ac 75
razer-cli write logo ac 1             # 0=Off 1=On 2=Breathing
razer-cli write bho on 80             # charge limit 80%
razer-cli write bho off

# RGB effects
razer-cli standard-effect static 0 255 0
razer-cli standard-effect wave 1
razer-cli standard-effect breathing 1 255 0 0
razer-cli standard-effect spectrum
razer-cli standard-effect reactive 2 255 255 0
razer-cli standard-effect off
```

Power modes: `0` Balanced, `1` Gaming, `2` Creator, `3` Silent, `4` Custom. Use `ac` or `bat` for the power source.

### Daemon

Runs as a systemd user service (no root):

```bash
systemctl --user status razercontrol
systemctl --user restart razercontrol
journalctl --user -u razercontrol -f
```

### KDE Plasma Widget

A native Plasma 6 widget with live sensors, battery status, and quick settings. See [kde-widget/README.md](razer_control_gui/kde-widget/README.md).

```bash
cd razer_control_gui/kde-widget && ./install-plasmoid.sh
```

## Architecture

```
razer-cli / razer-settings / KDE widget
        │          │              │
        │ bincode  │ bincode      │ JSON
        └────┬─────┘              │
             ▼                    ▼
         daemon (systemd user service)
           │  Unix socket: $XDG_RUNTIME_DIR/razercontrol-socket
           │  Config: ~/.local/share/razercontrol/daemon.json
           ▼
         HID API → Razer hardware
```

## Supported Devices

49 Razer Blade models from 2015–2025. Check yours:

```bash
lsusb | grep -i razer
# Match the PID (after 1532:) against data/devices/laptops.json
```

<details>
<summary>Full list</summary>

| Model | PID |
|-------|-----|
| Blade Stealth 2015 | 0205 |
| Blade Stealth Late 2016 | 0220 |
| Blade Stealth 2017 | 022D |
| Blade Stealth Late 2017 | 0232 |
| Blade Stealth 2019 | 0239 |
| Blade Stealth 2019 GTX | 024A |
| Blade Stealth 2020 | 0252 |
| Blade Stealth Late 2020 | 0259 |
| Blade QHD | 020F |
| Blade 15 2016 | 0224 |
| Blade 15 2018 Base | 023B |
| Blade 15 2018 Advanced | 0233 |
| Blade 15 2018 Mercury | 0240 |
| Blade 15 2019 Base | 0246 |
| Blade 15 2019 Advanced | 023A |
| Blade 15 2019 Mercury | 0245 |
| Blade 15 Late 2019 Advanced | 024B |
| Blade 15 Studio Edition 2019 | 024D |
| Blade 15 2020 Base | 0255 |
| Blade 15 2020 Advanced | 0253 |
| Blade 15 Late 2020 Base | 0268 |
| Blade 15 2021 Base | 026F |
| Blade 15 2021 Advanced | 0276 |
| Blade 15 Late 2021 Base | 027A |
| Blade 15 Late 2021 Advanced | 026D |
| Blade 15 Early 2022 Advanced | 028A |
| Blade 15 2023 | 029E |
| Blade 2022 Advanced | 028A |
| Blade 14 2021 | 0270 |
| Blade 14 2022 | 028C |
| Blade 14 2023 | 029D |
| Blade 14 2024 | 02B6 |
| Blade 14 2025 | 02C5 |
| Blade 16 2023 | 029F |
| Blade 16 2024 | 02B7 |
| Blade 16 2025 | 02C6 |
| Blade 17 2022 | 028B |
| Blade 18 2023 | 02A0 |
| Blade 18 2024 | 02B8 |
| Blade 18 2025 | 02C7 |
| Blade Pro 2017 | 0225 |
| Blade Pro 2017 v2 | 0210 |
| Blade Pro 2018 FHD | 022F |
| Blade Pro 2019 | 0234 |
| Blade Pro Late 2019 | 024C |
| Blade Pro 2020 | 0256 |
| Blade Pro 17 Early 2021 | 026E |
| Blade Pro 17 Mid 2021 | 0279 |
| Razer Book 13 2020 | 026A |

</details>

## Adding a New Device

1. Find your PID: `lsusb | grep -i razer`
2. Add an entry to `razer_control_gui/data/devices/laptops.json`
3. Add the PID to `razer_control_gui/data/udev/99-hidraw-permissions.rules`
4. `./install.sh install`
5. Open a PR.

## Troubleshooting

**"No supported device found"** — Your PID isn't in `laptops.json`. See [Adding a New Device](#adding-a-new-device).

**"Permission denied" on hidraw** — Udev rules not loaded. Run `sudo udevadm control --reload-rules && sudo udevadm trigger`, then log out/in.

**Daemon not starting** — Check for conflicting services: `systemctl --user list-units | grep -i razer`. Check logs: `journalctl --user -u razercontrol -n 50`.

**GUI can't connect** — Start the daemon: `systemctl --user start razercontrol`.

## Uninstall

```bash
cd razer_control_gui && ./install.sh uninstall
```

## Credits

- Original project: [Razer-Linux/razer-laptop-control-no-dkms](https://github.com/Razer-Linux/razer-laptop-control-no-dkms)
- HID modifications, GTK4 implementation & packaging: [@encomjp](https://github.com/encomjp)
- UI rework (native libadwaita widgets, CSS cleanup): [Claude](https://claude.ai/) by Anthropic
- [@johva1312](https://github.com/johva1312) — HID init fallbacks, partial socket read fix (PR #8)
- [@sini](https://github.com/sini) — NixOS flake fixes

## License

[GPL-2.0](LICENSE)

> This is community software, not affiliated with Razer Inc. No warranty. Use at your own risk.
