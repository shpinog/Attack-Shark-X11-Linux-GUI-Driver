# Attack Shark X11 — Linux GUI Driver

> ⚠️ **This project was developed with the assistance of AI (Claude by Anthropic)**

A native Linux GUI application for configuring the **Attack Shark X11** gaming mouse.  
The official software is Windows-only — this project reverse-engineers the USB HID protocol to provide Linux support.

---

## Screenshot

> _(add screenshot here)_

---

## What works

| Feature | Status |
|---|---|
| DPI configuration (6 stages, custom values 50–22000) | ✅ |
| Active DPI stage selection | ✅ |
| Angle Snap | ✅ |
| Ripple Control | ✅ |
| Polling rate (125 / 250 / 500 / 1000 Hz) | ✅ |
| Lighting mode (Off, Static, Breathing, Neon, Color Breathing, Static DPI, Breathing DPI) | ✅ |
| RGB color | ✅ |
| LED speed | ✅ |
| Key response time (4–50 ms) | ✅ |
| Sleep timer | ✅ |
| Deep sleep timer | ✅ |
| Battery level (2.4GHz wireless mode) | ✅ |
| Reset to factory defaults | ✅ |
| Save / restore settings between sessions | ✅ |
| Auto-detect wired / 2.4GHz wireless mode | ✅ |
| Mouse returns to system after apply (no replug needed) | ✅ |

---

## Supported hardware

| Device | Mode | Status |
|---|---|---|
| Attack Shark X11 | Wired | ✅ Supported |
| Attack Shark X11 | 2.4GHz wireless | ✅ Supported |
| Attack Shark X11 | Bluetooth | ❓ Not tested |
| Attack Shark R1 | Any | ❓ Possibly compatible |

---

## Requirements

- Linux (NixOS, Arch, Ubuntu, Fedora, etc.)
- libusb 1.0
- Access to USB device (udev rule or root)

---

## Installation on NixOS

### 1. Clone the repository

```bash
git clone https://github.com/yourname/go-attack-shark-x11-driver
cd go-attack-shark-x11-driver
```

### 2. Enter the dev shell

The project includes a `shell.nix` file that sets up a fully reproducible development environment.

```bash
nix-shell
```

**What `shell.nix` does automatically:**

- Installs all required dependencies into an isolated environment:
  - `go` — compiler
  - `libusb1` + `libusb1.dev` — USB communication + C headers for cgo
  - `systemd.dev` — `libudev.so.1` required by native USB bindings
  - `stdenv.cc.cc.lib` — `libstdc++.so.6` for prebuilt native modules
  - `pkg-config` — for cgo to find library paths
  - Fyne GUI dependencies: `gtk4`, `wayland`, `libxkbcommon`, `libGL`, `glfw`, X11 libs
- Sets `CGO_ENABLED=1`, `PKG_CONFIG_PATH`, `LD_LIBRARY_PATH` so cgo and libusb work correctly on NixOS
- **Creates a temporary udev rule** at `/run/udev/rules.d/99-attack-shark-x11.rules` so the mouse is accessible without `sudo` (rule is removed automatically on shell exit)
- Nothing is installed system-wide — everything is sandboxed to the shell session

> For a permanent udev rule (survives reboot), add to your NixOS configuration:
> ```nix
> services.udev.extraRules = ''
>   SUBSYSTEM=="usb", ATTR{idVendor}=="1d57", ATTR{idProduct}=="fa60", MODE="0666", TAG+="uaccess"
>   SUBSYSTEM=="usb", ATTR{idVendor}=="1d57", ATTR{idProduct}=="fa55", MODE="0666", TAG+="uaccess"
> '';
> users.users.YOUR_USER.extraGroups = [ "plugdev" ];
> ```

### 3. Build

```bash
go build -o x11-mouse .
```

First build takes 1–3 minutes (Fyne compiles OpenGL bindings). Subsequent builds are fast due to Go's build cache.

### 4. Run

```bash
./x11-mouse
```

---

## Installation on other distros

### Dependencies

**Debian/Ubuntu:**
```bash
sudo apt install golang libusb-1.0-0-dev pkg-config \
  libgl1-mesa-dev libxcursor-dev libxrandr-dev libxinerama-dev \
  libxi-dev libxxf86vm-dev libwayland-dev libxkbcommon-dev
```

**Arch Linux:**
```bash
sudo pacman -S go libusb pkg-config mesa libxcursor libxrandr \
  libxinerama libxi libxxf86vm wayland libxkbcommon
```

### udev rule (required for non-root access)

```bash
sudo tee /etc/udev/rules.d/99-attack-shark-x11.rules << 'EOF'
SUBSYSTEM=="usb", ATTR{idVendor}=="1d57", ATTR{idProduct}=="fa60", MODE="0666", TAG+="uaccess"
SUBSYSTEM=="usb", ATTR{idVendor}=="1d57", ATTR{idProduct}=="fa55", MODE="0666", TAG+="uaccess"
EOF

sudo udevadm control --reload-rules
sudo udevadm trigger
```

### Build and run

```bash
go build -o x11-mouse .
./x11-mouse
```

---

## Configuration file

Settings are saved automatically to `~/.config/x11-mouse.json` after each **Apply**.  
On next launch the app restores the last used configuration.

To reset to defaults, either use the **Reset to factory** button in the app, or delete the file:

```bash
rm ~/.config/x11-mouse.json
```

---

## How it works

The USB HID protocol was reverse-engineered from:
- The [HarukaYamamoto0/attack-shark-x11-driver](https://github.com/HarukaYamamoto0/attack-shark-x11-driver) TypeScript library
- The official web configurator at [szslxd-tech.com](https://szslxd-tech.com) (JavaScript source analysis)
- USB traffic capture with Wireshark + usbmon

The app communicates directly with the mouse via **libusb** (cgo), performing `detach → claim → transfer → release → reattach` of the kernel HID driver for each operation, so the mouse continues to function normally between configuration changes.

---

## Tech stack

- **Go** — main language
- **Fyne v2** — GUI framework (works on Wayland and X11)
- **libusb 1.0** — direct USB communication via cgo
- **NixOS** — primary development environment

---

## Known issues

- Lighting mode changes may not apply correctly in all firmware versions (under investigation)
- Bluetooth mode is untested
- The app does not read current settings from the mouse (device has no read command) — it uses the last saved configuration file instead

---

## Contributing

Contributions welcome. Useful areas:
- Testing on other Attack Shark / Inphic models
- Bluetooth mode investigation
- Lighting protocol improvements

---

## License

MIT

---

## Acknowledgements

- [HarukaYamamoto0](https://github.com/HarukaYamamoto0) for the original TypeScript reverse engineering work
- **Claude (Anthropic)** — AI assistant used throughout development of this project
