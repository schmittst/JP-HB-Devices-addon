# Fork notes — schmittst/JP-HB-Devices-addon

Dieser Fork von [`jp112sdl/JP-HB-Devices-addon`](https://github.com/jp112sdl/JP-HB-Devices-addon) erweitert das Addon um ein eigenes Custom-Device:

- **`HB-REM-VIVALDI`** (Model-ID `0xF400`) — Fernsteuerung für eine La-Spaziale-Vivaldi-II-Espressomaschine: zwei `SWITCH`-Channels (Power + Boiler), STATE wird aus den Vivaldi-Bedienpanel-LEDs gelesen, SET wird als Tastendruck emuliert. Die zugehörige Bluepill-Firmware liegt in einem separaten lokalen Sketchbook (`HB-REM-VIVALDI/VivaldiRemote_V1/`), nicht in diesem Repo.

## Branch-Strategie

| Branch | Zweck |
|---|---|
| `master` | Sauberer Mirror des Upstream (`jp112sdl/JP-HB-Devices-addon`). **Keine eigenen Commits** — wird per `git fetch upstream` aktualisiert und dann `git push origin master`. So bleiben Upstream-Sync-Operationen ein Fast-Forward, kein Merge. |
| `vivaldi-remote` | Working-Branch dieses Forks. Hier liegen alle Vivaldi-Device-Erweiterungen (XML-Definition, devdb-Eintrag, webui.js-Patch, Custom-Espresso-Icon, VERSION-Bumps). Dieser Branch wird auch zur Generierung des CCU2-Addon-tgz verwendet. |

### Upstream-Updates einpflegen

```bash
git checkout master
git fetch upstream
git merge --ff-only upstream/master
git push origin master

git checkout vivaldi-remote
git rebase master                # oder git merge master, wenn kein force-push
git push --force-with-lease origin vivaldi-remote   # nur bei rebase
```

## Was wurde gegenüber Upstream geändert?

Alle Commits liegen ausschließlich auf `vivaldi-remote`. `git log master..vivaldi-remote --oneline` listet sie auf. Inhaltlich:

- `src/addon/firmware/rftypes/hb-rem-vivaldi.xml` — Device-Definition (2 SWITCH-Channels + MAINTENANCE)
- `src/addon/firmware/devdescr/` — devdb-Eintrag für Model-ID `0xF400`
- `src/addon/www/config/easymodes/HM-LC-Sw1Bba-CCU3.htm.fn` etc. — falls für UI-Modi nötig
- `src/addon/www/webui/webui.js.patch` — registriert das Device in der WebUI (Latin-1 byte-mode beachten!)
- Custom-Icon (250×250 + 50×50 RGBA, erzeugt aus SVG via `resvg-py`)
- `src/addon/VERSION` — Bumps für jeden Release des Forks
- Diverse Audit-Fixes (`X4`-Reihe etc.) — siehe `git log`

## Addon-tgz bauen

```bash
cd src/addon
bash build.sh
# erzeugt jp-hb-devices-addon.tgz in src/addon/build/
```

Hochladen via CCU2-WebUI → `Einstellungen` → `Systemsteuerung` → `Zusatzsoftware`. Nach Reboot Install-Logs prüfen:

```bash
ssh root@<ccu>
cat /tmp/jp-hb-devices-addon-inst.log
cat /tmp/jp-hb-devices-addon-inst.err
```

## Pull-Request an Upstream?

Aktuell nicht eingereicht. Würde Update-Survival verbessern (jp112sdl-Releases würden Vivaldi automatisch beinhalten), bedeutet aber Maintenance-Verantwortung. Kann jederzeit erfolgen — der `vivaldi-remote`-Branch ist auf Upstream rebased und PR-bereit.

## Hardware-/Firmware-Kontext

Die Vivaldi-Remote besteht aus:
- **Bluepill STM32F103C8** + CC1101 (868 MHz) + AT24C32 (auf DS3231-Modul I²C 0x57)
- Direkte Anzapfung am Vivaldi-PIC16F916 (gemeinsamer GND dank Trafo-Netztrennung)
- 2× PC817-Optokoppler für die Taster-Outputs
- Versorgung via LM2596 ab Vivaldi-Header M9 (12 V)
- Firmware in AskSinPP, deployed Status: stabil seit Audit-Round 3 (2026-06-06)

Code dazu liegt im privaten Sketchbook `Homematic/HB-REM-VIVALDI/VivaldiRemote_V1/`, nicht in diesem Repo.
