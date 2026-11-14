# ASIX AX88179 USB Ethernet driver on recent macOS

Short, practical guide to making a USB-Ethernet adapter with the **ASIX AX88179** chip (USB ID `0b95:1790`) work on recent macOS, Apple Silicon.

Deutsche Version: [ANLEITUNG.de.md](ANLEITUNG.de.md)
For AI coding agents: [AGENTS.md](AGENTS.md)

> This repository does **not** include the ASIX driver. The driver belongs to ASIX Electronics. This repo is not affiliated with ASIX or Apple.

## Problem

Adapters with the ASIX AX88179 chip show up in System Information > USB, but macOS creates no network interface. Apple's built-in driver covers only the older AX88178 and AX88772. The newer AX88179A and AX88179B use macOS's built-in CDC-NCM driver and need no install.

## Do you need this? (check)

1. Find the product name:
   ```
   system_profiler SPUSBHostDataType | grep -B2 -A8 AX88179
   ```
   If nothing is shown, the adapter is not connected or does not use this chip.
2. Check for a network port:
   ```
   networksetup -listallhardwareports
   ```
   If no "AX88179" port is listed, the driver is not active. Continue with Install.

Adapters that already work with a built-in driver (AX88179A/B) do not need this guide.

## Install

Download the official ASIX installer (v2.5.0, macOS 12 and above):

- Direct file: https://www.asix.com.tw/en/support/download/file/1951
- File name: `ASIX_USB_Device_Installer_macOS_12_above_Driver_v2.5.0.zip`
- Size: 2,720,776 bytes
- SHA-256: `616fc388b5a10462f9aa5b1780fb86bc7230a40431da4a287cffd5fbcacc47b0`

If `curl` returns an empty file, download it in a browser instead.

Terminal install:

```
curl -L -o asix.zip https://www.asix.com.tw/en/support/download/file/1951
shasum -a 256 asix.zip
unzip asix.zip -d asix
hdiutil attach -nobrowse -readonly -mountpoint /tmp/asix-dmg asix/ASIX_USB_Device_Installer_v2.5.0.dmg
pkgutil --check-signature /tmp/asix-dmg/ASIX_USB_Device_Installer_v2.5.0.pkg
sudo installer -pkg /tmp/asix-dmg/ASIX_USB_Device_Installer_v2.5.0.pkg -target /
hdiutil detach /tmp/asix-dmg
open /Applications/ASIX_USB_Device_App.app
```

Check that the SHA-256 matches and that the signature names `ASIX ELECTRONICS CORPORATION (5RHFAZ9D4P)` before you continue.

After the install, a dialog says "Please run ASIX_USB_DEVICE_APP in Launchpad after reboot." Click OK. In our test no reboot was needed and SIP stayed enabled.

## Approve the driver

1. In the ASIX app, click **Activate**.
2. Open System Settings > General > Login Items & Extensions. (German macOS: Allgemein > Anmeldeobjekte.)
3. Scroll to the very bottom to **Extensions** (Erweiterungen). Switch to the **By Category** (Nach Kategorie) tab.
4. Click the ⓘ next to **Driver Extensions** (Treibererweiterungen).
5. Enable **ASIX_USB_Device_App** (`com.asix.dext.usbdevice`) and authenticate.

Direct link to the pane:

```
open "x-apple.systempreferences:com.apple.LoginItems-Settings.extension"
```

## Verify

```
systemextensionsctl list
```

Expected line:

```
5RHFAZ9D4P  com.asix.dext.usbdevice (2.4.0/2.4.0)  ...  [activated waiting for user]
```

The state stays `waiting for user` until the toggle is on. Then it changes to `[activated enabled]`. The version shows 2.4.0 even with the 2.5.0 installer. This is normal.

Unplug and replug the adapter, then:

```
networksetup -listallhardwareports
```

You should see `Hardware Port: AX88179` with a device such as `en6`, and a network service named "AX88179".

## Troubleshooting

- **Gatekeeper warns on double-click.** The installer package is signed but not notarized. Right-click > Open, or install from Terminal as shown above.
- **Interface does not appear after approval.** Unplug and replug the adapter. If it still does not appear, reboot.
- **State stays `waiting for user`.** Open the Extensions pane again and check the toggle for ASIX_USB_Device_App.
- **Homebrew cask fails.** The Homebrew cask `asix-ax88179` was disabled on 2026-09-01 because it fails the Gatekeeper check. Use the official download.

## Uninstall

1. Unplug the adapter.
2. In the ASIX app, click **Deactivate**.
3. Open the DMG and run `ASIX_USB_Device_Uninstaller_v1.0.0.pkg`. It removes only the app.
4. Reboot.

## Security notes

- Always check the SHA-256 hash and the Team ID `5RHFAZ9D4P` before installing.
- The installer runs with administrator rights (`sudo`). Only install from the official ASIX link above.
- The DMG is unsigned. The installed app `/Applications/ASIX_USB_Device_App.app` is notarized.
- Do not disable SIP. The install did not require it.
- The approval step is intentional. Do not bypass it.

## Tested on

- macOS 27.0.1 (build 26A434), Apple Silicon
- ASIX_USB_Device_Installer v2.5.0 (driver extension reports 2.4.0)
- Adapter with ASIX AX88179 (USB ID `0b95:1790`)

## License

MIT, see [LICENSE](LICENSE). Copyright (c) the contributors.
