# ESP32 Smart Home — FEATURE_RICH + 24/7 + Safe OTA

FEATURE_RICH is the project baseline for the classic ESP32 (4 MB flash): offline-first local relay control, AP+STA networking, local schedules, optional cloud access, optional MQTT/Home Assistant, diagnostics, NVS persistence and safe dual-slot OTA.

## Core behavior
- The local AP remains available when STA or Internet is unavailable.
- Local relay control and cached schedules do not depend on Internet access.
- Relay state/configuration and schedules are persisted in NVS.
- STA reconnect uses bounded backoff.
- Cloud polling uses bounded backoff, HTTPS and explicit command acknowledgement.
- MQTT/Home Assistant remains optional and separate from the local relay control path.

## OTA
Firmware updates are available locally at **Settings → Firmware Update**. The ESP32 writes the image only to the inactive OTA slot, validates it with the ESP-IDF OTA APIs, selects it for the next boot and reboots. Application rollback is enabled, so an unconfirmed new image can fall back to the previous application.

Each OTA application slot is `0x1F0000` bytes (about 1.94 MiB). The CI workflow checks that `build/smart_home.bin` fits inside this limit.

### First installation / migration
The previous revision used a factory-only application partition. Existing devices on that old layout must first receive the OTA-capable bootloader, partition table and application using a normal wired/serial flash. Future firmware updates can then use the web UI. The NVS partition remains at `0x9000` and `phy_init` remains at `0xF000`; OTA metadata and the two OTA app slots are added after them.

The first migration therefore changes the application layout and must be treated as a full firmware migration. Do not mix the old factory-only partition table with this OTA-enabled source.

### OTA safety
- Images larger than the inactive slot are rejected before writing.
- Incomplete uploads are aborted.
- Invalid images are rejected by `esp_ota_end()`.
- The running application slot is never erased by the OTA handler.
- Cloud polling and MQTT activity are paused during the update.
- Relay/configuration POST endpoints are temporarily blocked while the OTA write is active.

## Build
Use ESP-IDF v5.1.2 and target `esp32`.

```text
idf.py set-target esp32
idf.py build
```

The CI workflow also verifies the bootloader, partition table, application size and SHA-256 checksums.

## Flash outputs
For the initial OTA migration, flash the generated bootloader, partition table and application at the offsets reported by ESP-IDF/esptool. After the migration, normal firmware upgrades use **Settings → Firmware Update** and do not require reflashing the partition table.

## Hardware note
`RELAY_ACTIVE_LEVEL` is intentionally preserved from the supplied source. Verify the actual low-voltage relay module logic before changing it.
