# AGENTS.md: ASIX AX88179 driver install on macOS

Guidance for AI coding agents. Human-readable version: [README.md](README.md).

## Scope

Installs the official ASIX_USB_Device_Installer v2.5.0 for an AX88179 USB-Ethernet adapter (USB ID `0b95:1790`) on recent macOS (Apple Silicon verified on macOS 27.0.1, build 26A434). Not included: the driver itself. It belongs to ASIX Electronics. Not affiliated with ASIX or Apple.

Do not install this driver for AX88179A or AX88179B. Those use macOS's built-in CDC-NCM driver.

## What can be automated

- Detect the adapter and driver state (read-only commands).
- Download, verify, mount, check signature, install (with admin rights), unmount.
- Launch the ASIX app.
- Poll the extension state until it is enabled.

## What needs a human (3 steps)

1. **Admin password.** `sudo` or the GUI prompt needs the user's password.
2. **Activate click.** In the ASIX app, click **Activate**.
3. **Settings toggle.** Enable `ASIX_USB_Device_App` (`com.asix.dext.usbdevice`) under Login Items & Extensions > Extensions > By Category > Driver Extensions, and authenticate.

Do not script, bypass, or automate the approval. Do not disable SIP. Poll instead.

## Commands

Detect the adapter (read-only):

```
system_profiler SPUSBHostDataType | grep -B2 -A8 AX88179
networksetup -listallhardwareports
```

Download and verify (must match before install):

```
curl -L -o asix.zip https://www.asix.com.tw/en/support/download/file/1951
shasum -a 256 asix.zip
# expected: 616fc388b5a10462f9aa5b1780fb86bc7230a40431da4a287cffd5fbcacc47b0
# expected size: 2720776 bytes
```

If the file is empty, the ASIX product pages return empty bodies to scripted clients. The direct file URL above works. If it still fails, ask the user to download it in a browser.

Unpack, mount, check signature, install:

```
unzip asix.zip -d asix
hdiutil attach -nobrowse -readonly -mountpoint /tmp/asix-dmg asix/ASIX_USB_Device_Installer_v2.5.0.dmg
pkgutil --check-signature /tmp/asix-dmg/ASIX_USB_Device_Installer_v2.5.0.pkg
# expected: Developer ID Installer: ASIX ELECTRONICS CORPORATION (5RHFAZ9D4P)
sudo installer -pkg /tmp/asix-dmg/ASIX_USB_Device_Installer_v2.5.0.pkg -target /
hdiutil detach /tmp/asix-dmg
open /Applications/ASIX_USB_Device_App.app
```

Without a terminal for sudo, use a GUI password prompt:

```
osascript -e 'do shell script "/usr/sbin/installer -pkg <path> -target /" with administrator privileges'
```

Replace `<path>` with the absolute path to the `.pkg`.

Deep link to the approval pane (the human clicks the toggle):

```
open "x-apple.systempreferences:com.apple.LoginItems-Settings.extension"
```

## States to poll

Poll the extension state:

```
systemextensionsctl list
```

- `5RHFAZ9D4P  com.asix.dext.usbdevice (2.4.0/2.4.0)  [activated waiting for user]`: the toggle is not on yet. Ask the human to finish the Settings toggle.
- `... [activated enabled]`: the driver is active. Continue.

The version shows 2.4.0 even with the 2.5.0 installer. This is normal. Do not treat it as an error.

After the state is `[activated enabled]`, ask the human to unplug and replug the adapter, then check:

```
networksetup -listallhardwareports
```

Success: a `Hardware Port: AX88179` entry with a device such as `en6`, and a network service named "AX88179". If it is missing, reboot is the next step (ask the human).

## Notes

- Installer package: signed, not notarized. Gatekeeper warns on double-click. Install from Terminal as above.
- The installed app `/Applications/ASIX_USB_Device_App.app` is notarized.
- The DMG is unsigned. Verify the SHA-256 hash and the Team ID `5RHFAZ9D4P` before installing.
- The Homebrew cask `asix-ax88179` was disabled on 2026-09-01 because it fails the Gatekeeper check. Do not use it.
- The installer picks its payload by OS version: v2.5.0 on macOS 15.4+, v2.4.0 on macOS 12.4 to 15.3.
- Uninstall: the human unplugs the adapter and clicks **Deactivate** in the ASIX app. Then run `ASIX_USB_Device_Uninstaller_v1.0.0.pkg` from the DMG (removes only the app) and reboot.
