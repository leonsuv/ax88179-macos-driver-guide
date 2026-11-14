# ASIX AX88179 USB-Ethernet-Treiber auf aktuellem macOS

Kurze, praktische Anleitung, damit ein USB-Ethernet-Adapter mit dem **ASIX AX88179**-Chip (USB-ID `0b95:1790`) unter aktuellem macOS mit Apple Silicon funktioniert.

English version: [README.md](README.md)
Für KI-Coding-Agenten: [AGENTS.md](AGENTS.md)

> Dieses Repository enthält den ASIX-Treiber **nicht**. Der Treiber gehört ASIX Electronics. Dieses Repo ist nicht mit ASIX oder Apple verbunden.

## Problem

Adapter mit dem ASIX-AX88179-Chip erscheinen unter Systeminformationen > USB, aber macOS legt keine Netzwerkschnittstelle an. Apples eingebauter Treiber unterstützt nur die älteren Chips AX88178 und AX88772. Die neueren AX88179A und AX88179B nutzen den eingebauten CDC-NCM-Treiber von macOS und brauchen keine Installation.

## Brauchen Sie das? (prüfen)

1. Produktnamen anzeigen:
   ```
   system_profiler SPUSBHostDataType | grep -B2 -A8 AX88179
   ```
   Wird nichts angezeigt, ist der Adapter nicht angeschlossen oder nutzt diesen Chip nicht.
2. Netzwerkanschluss prüfen:
   ```
   networksetup -listallhardwareports
   ```
   Wird kein Anschluss „AX88179“ aufgelistet, ist der Treiber nicht aktiv. Weiter mit „Installieren“.

Adapter, die mit dem eingebauten Treiber schon funktionieren (AX88179A/B), brauchen diese Anleitung nicht.

## Installieren

Den offiziellen ASIX-Installer herunterladen (v2.5.0, macOS 12 und neuer):

- Direkte Datei: https://www.asix.com.tw/en/support/download/file/1951
- Dateiname: `ASIX_USB_Device_Installer_macOS_12_above_Driver_v2.5.0.zip`
- Größe: 2.720.776 Bytes
- SHA-256: `616fc388b5a10462f9aa5b1780fb86bc7230a40431da4a287cffd5fbcacc47b0`

Liefert `curl` eine leere Datei, laden Sie die Datei stattdessen im Browser herunter.

Installation im Terminal:

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

Bevor Sie weitermachen, prüfen Sie, ob der SHA-256-Wert übereinstimmt und die Signatur `ASIX ELECTRONICS CORPORATION (5RHFAZ9D4P)` nennt.

Nach der Installation erscheint ein Dialog: „Please run ASIX_USB_DEVICE_APP in Launchpad after reboot.“ Klicken Sie auf OK. Bei unserem Test war kein Neustart nötig, und SIP blieb aktiviert.

## Treiber freigeben

1. In der ASIX-App auf **Activate** klicken.
2. Öffnen Sie Systemeinstellungen > Allgemein > Anmeldeobjekte. (Englisches macOS: Login Items & Extensions.)
3. Ganz nach unten zu **Erweiterungen** scrollen. Auf den Tab **Nach Kategorie** wechseln.
4. Auf das ⓘ neben **Treibererweiterungen** klicken.
5. **ASIX_USB_Device_App** (`com.asix.dext.usbdevice`) aktivieren und mit Ihrem Passwort bestätigen.

Direkter Link zu dieser Ansicht:

```
open "x-apple.systempreferences:com.apple.LoginItems-Settings.extension"
```

## Prüfen

```
systemextensionsctl list
```

Erwartete Zeile:

```
5RHFAZ9D4P  com.asix.dext.usbdevice (2.4.0/2.4.0)  ...  [activated waiting for user]
```

Der Status bleibt `waiting for user`, bis der Schalter eingeschaltet ist. Danach wechselt er zu `[activated enabled]`. Die Version zeigt 2.4.0, auch mit dem 2.5.0-Installer. Das ist normal.

Adapter ab- und wieder anstecken, dann:

```
networksetup -listallhardwareports
```

Sie sollten `Hardware Port: AX88179` mit einem Gerät wie `en6` sehen sowie einen Netzwerkdienst namens „AX88179“.

## Fehlerbehebung

- **Gatekeeper warnt beim Doppelklick.** Das Installationspaket ist signiert, aber nicht notarisiert. Rechtsklick > Öffnen, oder Installation im Terminal wie oben.
- **Schnittstelle erscheint nach der Freigabe nicht.** Adapter ab- und wieder anstecken. Erscheint sie dann immer noch nicht, starten Sie den Mac neu.
- **Status bleibt bei `waiting for user`.** Öffnen Sie die Erweiterungen-Ansicht erneut und prüfen Sie den Schalter für ASIX_USB_Device_App.
- **Homebrew-Cask schlägt fehl.** Der Homebrew-Cask `asix-ax88179` wurde am 01.09.2026 deaktiviert, weil er die Gatekeeper-Prüfung nicht besteht. Nutzen Sie den offiziellen Download.

## Deinstallieren

1. Adapter abziehen.
2. In der ASIX-App auf **Deactivate** klicken.
3. Im DMG `ASIX_USB_Device_Uninstaller_v1.0.0.pkg` ausführen. Es entfernt nur die App.
4. Mac neu starten.

## Sicherheitshinweise

- Prüfen Sie vor der Installation immer den SHA-256-Wert und die Team-ID `5RHFAZ9D4P`.
- Der Installer läuft mit Administratorrechten (`sudo`). Installieren Sie nur von dem oben genannten offiziellen ASIX-Link.
- Das DMG ist nicht signiert. Die installierte App `/Applications/ASIX_USB_Device_App.app` ist notarisiert.
- SIP nicht deaktivieren. Die Installation hat es nicht erfordert.
- Der Freigabeschritt ist Absicht. Umgehen Sie ihn nicht.

## Getestet auf

- macOS 27.0.1 (Build 26A434), Apple Silicon
- ASIX_USB_Device_Installer v2.5.0 (die Treibererweiterung meldet 2.4.0)
- Adapter mit ASIX AX88179 (USB-ID `0b95:1790`)

## Lizenz

MIT, siehe [LICENSE](LICENSE). Copyright (c) the contributors.
