# BIOS Firmware Updater 0.1.0

BIOS Firmware Updater is a Rust + Qt 6 desktop application for inspecting and safely staging vendor firmware packages that are distributed as Windows-only archives even when the machine exposes a stand

The project is intentionally conservative. It is **not** a generic raw BIOS flasher and does not use `flashrom` or direct SPI writes. Firmware is handed to the platform through `fwupd`/UEFI capsule mec

# Use only at your own risk!

If you’ve used it and it works, please let me know your machine model so I can expand the list of supported devices.

## Validated status

The first `Validated` hardware profile is:

- Acer Predator PHN18-71
- Insyde firmware platform JH63T
- system firmware ESRT GUID `5dd9a515-7d6a-41c5-ab68-93fd1d08dcc6`
- Acer/Insyde H2OFFT package containing `platform.ini` and `abobios.bin`
- `SecureUpdate.viaESP=1`
- fwupd >= 2.1.1
- real-world validated transition: V1.11 -> V1.17

## Components

- `lfb-core` — pure Rust domain logic, package/profile parsing, safety engine, UEFI capsule validation, system probe and recovery snapshot support.
- `lfb-helper` — separate privileged CLI. The GUI invokes it with `pkexec`. It is the only component allowed to stage firmware.
- `lfb-gui` — unprivileged Qt 6 / QML application with a Rust `QObject` implemented through CXX-Qt.

## Safety model

Firmware staging is refused unless all applicable gates pass. Important safety rules:

- recovery snapshot is mandatory before any ESP write;
- the GUI never runs as root;
- `lfb-helper stage` requires `--execute` and the exact phrase `I UNDERSTAND FIRMWARE FLASH RISK`;
- no automatic reboot, shutdown, forced reset or reboot button exists;
- no `--force`, `--no-safety-check`, downgrade or power-ignore fwupd options are used;
- raw writes to `/sys/firmware/efi/efivars` are forbidden;
- a non-empty unknown `EFI/UpdateCapsule` directory blocks staging;
- staged UEFI capsule GUID/header/image-size and payload SHA-256 are verified after fwupd returns;
- BootCurrent/BootOrder and all non-capsule ESP files must remain unchanged;
- post-stage failures are marked rollback-required and must not be rebooted.

A recovery snapshot cannot guarantee recovery from a hardware failure or loss of power after the firmware itself starts programming SPI/EC. 

