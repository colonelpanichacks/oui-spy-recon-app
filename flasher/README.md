# OUI SPY web flasher

The browser installer for the OUI SPY firmware and the remote mesh nodes, served by GitHub
Pages from this folder (`/flasher/`). It runs [ESP Web Tools](https://esphome.github.io/esp-web-tools/)
in Chrome or Edge on a computer and writes the images below to an OUI SPY (Seeed XIAO
ESP32-S3, 8 MB flash) over USB.

## Layout

| Path | What it is |
|---|---|
| `index.html` | The page: get the app, pick what the board will be (OUI SPY for the app, or a remote node), flash it, set a remote node up on the fleet's mesh over Web Serial, after install, troubleshooting, the Recon teaser, thanks. |
| `manifest.json` | ESP Web Tools manifest of the OUI SPY for the app: chip `ESP32-S3`, the four parts and their offsets, the erase prompt, the version. |
| `node-detector.json`, `node-skyspy.json`, `node-wardrive.json` | The remote nodes' manifests: the same first three parts, then the node's image. |
| `bootloader.bin` | Second-stage bootloader. |
| `partitions.bin` | Partition table (8 MB, two app slots). |
| `boot_app0.bin` | OTA data that boots the first app slot. |
| `ouispyblue.bin` | The OUI SPY firmware, byte for byte the image the app bundles. |
| `node-detector.bin`, `node-skyspy.bin`, `node-wardrive.bin` | The remote nodes (the OUI-SPY Detector family), byte for byte the images the app bundles as `detector.bin`, `skyspy.bin`, `wardrive.bin` to update them over the mesh. |
| `art/` | The app's header art, the board's animals and contour lines, and `recon.jpg`, a render of the OUI SPY Recon. |
| `.nojekyll` | Pages serves the folder as is. |

All paths in the page and the manifest are relative, so the folder works under any URL.

## Parts (build Nov 25 2026)

| Offset | File | Bytes | sha256 |
|---|---|---|---|
| `0x0` | `bootloader.bin` | 15,104 | `1776e4dd896a69d0a5c2e79957b0e2a88aa4129b1381d6478683515a1f6af343` |
| `0x8000` | `partitions.bin` | 3,072 | `1d9cca96de0fe07ad7fc0648b9878ddecd9ce565e38b589ad20fea698ed4c80c` |
| `0xe000` | `boot_app0.bin` | 8,192 | `f94c5d786a7a8fab06ac5d10e33bf37711a6697636dc037559ea19cc410a17f0` |
| `0x10000` | `ouispyblue.bin` | 1,662,976 | `90242c0ee8b85d7a7b5afb44d255b77eef229511e6612e46bd27a5a8044f2225` |

The firmware is build `Nov 25 2026 09:00:26`. The remote nodes write the same first three
parts, then their own image at `0x10000`:

| Node | File | Build | Bytes | sha256 |
|---|---|---|---|---|
| Detector | `node-detector.bin` | `Nov 5 2026 20:55:34` | 1,407,472 | `f88562a7fc7b51fb7868e267b5a2e05224e093df6ee725a92a99c11a49293b2f` |
| Sky-Spy | `node-skyspy.bin` | `Nov 17 2026 21:05:37` | 1,407,776 | `ce2241ac58663d6d9ad4e5ea822c43a79b58578b72607e6363f98c191e4138cb` |
| Wardriver | `node-wardrive.bin` | `Nov 5 2026 20:55:53` | 1,156,384 | `56630e5d7d4db6829e24d9934e519865de94ede96cd9ca22c515739396752e19` |
 The partition table is Arduino-ESP32's
`default_8MB.csv`: nvs `0x9000`, otadata `0xe000`, app0 `0x10000` and app1 `0x340000`
(3 MB each), spiffs `0x670000`, coredump `0x7f0000`. USB is only for the first install;
after that the app updates the firmware over the air into the other slot.

## Updating for a new release

1. Copy the new release's `app/assets/firmware/ouispyblue.bin` (the image the app bundles)
   over `ouispyblue.bin`, and its `detector.bin`, `skyspy.bin`, `wardrive.bin` over the
   `node-*.bin` files when they changed. Do not rebuild it: the flasher must write the same bytes the app
   ships.
2. Name the new image by its build date, the stamp the app's Firmware page shows for it:
   `version` in `manifest.json`, and the subtitle and the line under "Flash your OUI SPY" in
   `index.html`. One image ships in several app versions, so the flasher never names an app
   version.
3. Leave `bootloader.bin`, `partitions.bin` and `boot_app0.bin` alone unless the partition
   table, the flash size or the Arduino framework version changes. If one does, take
   `bootloader.bin` and `partitions.bin` from the PlatformIO build that produced the new
   firmware (`.pio/build/<env>/`) and `boot_app0.bin` from the framework package
   (`framework-arduinoespressif32/tools/partitions/boot_app0.bin`).
4. Check the hashes with `shasum -a 256 *.bin` and update the table above.
5. Test locally: run `python3 -m http.server 8000` in this folder and open
   `http://localhost:8000/` in Chrome. Web Serial works on localhost.

Never put an APK or other app build in this folder. The page's download buttons ask the
GitHub API for the latest release and link its signed files (`.apk`, `-macos.zip`); when
the API does not answer they link the releases page.

## ESP Web Tools

The page loads ESP Web Tools 10.4.0 from unpkg. The script tag carries the sha384 of
`install-button.js`, and the import map above it carries the sha384 of every other file in
the package's `dist/web/`, so the browser refuses any file that is not the published one.
To move to another version, list its files
(`https://data.jsdelivr.com/v1/packages/npm/esp-web-tools@<version>?structure=flat`), hash
each one under `dist/web/` with

    curl -s https://unpkg.com/esp-web-tools@<version>/dist/web/<file> | openssl dgst -sha384 -binary | openssl base64 -A

and replace the version and every hash. A wrong hash stops the installer from loading.
