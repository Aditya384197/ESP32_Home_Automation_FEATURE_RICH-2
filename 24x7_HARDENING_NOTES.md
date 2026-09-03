# FEATURE_RICH 24/7 Hardening + OTA

This revision keeps the FEATURE_RICH feature set and hardens the runtime paths that matter for continuous operation.

## Reliability fixes
- Cloud HTTP body/response buffers are heap allocated instead of consuming a large local task-stack footprint.
- Cloud ACK IDs are removed only after the server has received them; newly returned command IDs remain pending for the next poll.
- Re-issued command IDs are retained as pending acknowledgements instead of being lost by the same response cycle.
- Remote relay commands are acknowledged only when the requested relay is actually accepted; disabled relays are not falsely reported as executed.
- Legacy schedule validation no longer compares unsigned fields against impossible negative values.

## OTA
- Dual OTA app slots (`ota_0`, `ota_1`) plus `otadata`.
- Image size is checked against the inactive slot before the upload starts.
- `esp_ota_end()` validates the downloaded application before boot selection.
- `esp_ota_set_boot_partition()` changes the next boot only after successful validation.
- Application rollback is enabled in the bootloader.
- The new application is marked valid only after the core system has initialized, so a startup crash can still trigger rollback.
- Cloud polling and MQTT are paused while an OTA upload is active.

## First migration
Devices running the previous factory-only layout need one wired/serial migration of bootloader + partition table + application. After that, application updates can be performed locally from the web UI.

## Validation boundary
Static source checks, JavaScript syntax checks, server syntax checks, partition-layout checks and protocol-behavior simulations can be run in the development environment. Actual RF performance, flash wear, brownout recovery, relay hardware behavior and multi-day hardware burn-in require the physical ESP32 system.
