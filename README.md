# k8ctl

Backlight control for **stock-firmware Keychron keyboards on Linux** — the
ones no other tool supports.

If you have a Keychron K8 (the original, not the V2/Pro QMK versions), you may
have noticed that nothing on Linux can change its lighting: OpenRGB doesn't
list its USB ID, and Keychron Launcher only speaks to QMK boards. `k8ctl`
fills that gap: a single-file, dependency-free Python script with a CLI and a
small local web GUI.

## Supported devices

| Device | USB ID | Status |
|---|---|---|
| Keychron K8 (stock firmware) | `05ac:0250` | ✅ confirmed working |
| Keychron K3 v2 | `05ac:024f` | should work — same protocol, untested with this tool |

Other stock-firmware Keychron boards (K4, K10, …) likely speak the same
protocol; if you try one, open an issue with your USB ID and result.

Works over the **USB cable only** — not Bluetooth.

## Install

```sh
curl -o ~/.local/bin/k8ctl https://raw.githubusercontent.com/Crash0verr1d3/k8ctl/main/k8ctl
chmod +x ~/.local/bin/k8ctl
```

Requires Python 3.8+ (standard library only, no pip packages).

### Device permissions (skip the sudo)

```sh
sudo cp 70-keychron.rules /etc/udev/rules.d/
sudo udevadm control --reload
```

then unplug/replug the keyboard once. Without the rule, run `k8ctl` with
`sudo`.

## Usage

```sh
k8ctl gui                                  # web interface at http://127.0.0.1:8688
k8ctl probe                                # safe connectivity test (read-only)
k8ctl set static --color ff2a00 --brightness 15
k8ctl set breathing --color 00ff88 --speed 10
k8ctl set spectrum-cycle
k8ctl set rain --random --direction ud
k8ctl modes                                # list all 21 modes
k8ctl off
```

`brightness` and `speed` are 0–15. Directions: `lr`, `rl`, `du`, `ud`.

The settings are written to all mode slots, so cycling effects with the
keyboard's lightbulb key keeps your color and brightness.

## How it works

The keyboard is driven over 64-byte HID feature reports on its hidraw
interface, using the stock Keychron protocol: a `04 13` handshake, 18
effect pages (mode, RGB, brightness, speed, direction, `AA 55` checksum),
a 144-LED color buffer, and a `04 F0` apply command.

The protocol was documented for the K3 v2 by the
[OpenRGB KeychronKeyboardController](https://gitlab.com/CalcProgrammer1/OpenRGB/-/merge_requests/1090)
and [mishamyrt/keychron-go](https://github.com/mishamyrt/keychron-go)
(see [OpenRGB issue #1790](https://gitlab.com/CalcProgrammer1/OpenRGB/-/issues/1790)).
This project confirmed the K8 (`05ac:0250`) speaks the same protocol.

## Caveats

- Each `set` writes the keyboard's settings flash — don't hammer it in a
  tight loop.
- On the white-backlight (non-RGB) variants, color values are ignored by the
  hardware; mode, brightness and speed still work.
- If the keyboard ever stops responding, unplugging and replugging it
  recovers it.

## License

GPL-3.0 — the protocol implementation derives from the GPL projects credited
above.
