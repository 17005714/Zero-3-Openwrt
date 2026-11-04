# Zero-3-Openwrt

Clean GitHub Actions firmware builds for the Orange Pi Zero 3. This repository starts from an empty history and uses the official ImmortalWrt source snapshot below; it does not import the old `Orange-Pi-Zero-3-Openwrt` repository.

## Build source and profiles

- ImmortalWrt source: `immortalwrt/immortalwrt` at `89f1ef18c888f841633f19f92655ee4b7f7716a4`.
- `network-test` is the default and intentionally has no third-party feeds or proxy applications. It builds a network diagnostic image first.
- `full` is protected by the `network_test_passed` workflow input. Set it to `true` only after the diagnostic image is stable on the physical board.

The full profile locks these third-party feeds after download:

| Feed | Commit |
| --- | --- |
| `kenzok8/openwrt-packages` | `8b73238a7eaf0e862d1d5528c6c67e090e7b6e1a` |
| `kenzok8/small` | `0e5a86fe10af5668ad3d333494b53be9d4d23b6b` |
| `EasyTier/luci-app-easytier` | `9c51d3f8d2bfcefd039f2ff2354c6a7674293a06` |

## Required secret

Before dispatching either workflow profile, add an Actions repository secret named `INITIAL_WIFI_PSK`. The workflow injects it into the image only at build time and never logs it. It is deliberately not present in this repository.

## First boot network settings

- LAN IP: `192.168.50.1/24`
- DHCP range: `192.168.50.100` through `192.168.50.249`
- Wi-Fi SSID: `Zero3`
- Wi-Fi: WPA2-PSK, country `CN`, channel `1`, HT20, bridged to LAN

## Validation order

1. Run **Actions -> Build Zero 3 OpenWrt -> Run workflow**, select `network-test`.
2. Flash the resulting `sysupgrade` or `sdcard` image appropriate for the boot medium.
3. Verify wired link LEDs, ten minutes of continuous ping to `192.168.50.1`, DHCP, Wi-Fi association, and LuCI access.
4. Only after that succeeds, run `full` with **network_test_passed** selected.

If the diagnostic image still has PHY resets, link loss, or Wi-Fi association failures, preserve the serial log and investigate the device tree, PHY, board power, or hardware before adding packages.
