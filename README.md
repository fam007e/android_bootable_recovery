# How to Compile TWRP Touch Recovery
This guide will help you compile TWRP (Team Win Recovery Project) touch recovery from source. It's intended for users familiar with basic 
Linux commands and the Android Open Source Project (AOSP) build system. If you are new to these concepts, it might be 
challenging to follow this guide.

## Prerequisites
- Basic knowledge of Linux commands.
- A Linux environment set up for Android development.
- Source code for a compatible ROM (OmniROM or LineageOS).

## Recommended Sources
You can use the source code from the following:
- Omni 6.0, 7.1, 8.1, 9.0
- CyanogenMod (CM) 13.0, 14.1, 15.1
- LineageOS 16.0

For the best compatibility, Omni 9.0 is recommended unless your device uses a super partition.

## Getting TWRP Source Code
### OmniROM
OmniROM already includes TWRP source code by default. However, for older versions, pull the latest branch from the TWRP repository:
```sh
git clone https://github.com/TeamWin/android_bootable_recovery -b <latest-branch>
```
### LineageOS
For CM/LineageOS, place TWRP in the `LineageOS/bootable/recovery-twrp` folder and set the following in your `BoardConfig.mk`:
```makefile
RECOVERY_VARIANT := twrp
```

## Minimal Manifest
For a smaller tree, you can use a minimal manifest:
```sh
git clone https://github.com/minimal-manifest-twrp
```
Note: Additional repositories might be needed for specific devices.

## Preparation
### Build Flags
Set or change build flags in your `BoardConfig.mk` file located in `device/manufacturer/codename`. Include architecture and platform settings. 
For all devices, specify the TWRP theme:
```makefile
TW_THEME := portrait_hdpi
```

Additional flags:
- RECOVERY_SDCARD_ON_DATA := true
- BOARD_HAS_NO_REAL_SDCARD := true
- TW_NO_BATT_PERCENT := true
- TW_CUSTOM_POWER_BUTTON := 107
- TW_NO_REBOOT_BOOTLOADER := true
- TW_NO_REBOOT_RECOVERY := true
- RECOVERY_TOUCHSCREEN_SWAP_XY := true
- RECOVERY_TOUCHSCREEN_FLIP_Y := true
- RECOVERY_TOUCHSCREEN_FLIP_X := true
- TWRP_EVENT_LOGGING := true
- BOARD_HAS_FLIPPED_SCREEN := true

### Recovery.fstab
TWRP supports new recovery.fstab features for backup/restore capabilities. Example for Galaxy S4:
```fstab
/boot emmc /dev/block/platform/msm_sdcc.1/by-name/boot
/system ext4 /dev/block/platform/msm_sdcc.1/by-name/system
/data ext4 /dev/block/platform/msm_sdcc.1/by-name/userdata length=-16384
/cache ext4 /dev/block/platform/msm_sdcc.1/by-name/cache
/recovery emmc /dev/block/platform/msm_sdcc.1/by-name/recovery
/efs ext4 /dev/block/platform/msm_sdcc.1/by-name/efs flags=display="EFS";backup=1
/external_sd vfat /dev/block/mmcblk1p1 flags=display="Micro SDcard";storage;wipeingui;removable
/usb-otg vfat /dev/block/sda1 flags=display="USB-OTG";storage;wipeingui;removable
/preload ext4 /dev/block/platform/msm_sdcc.1/by-name/hidden flags=display="Preload";wipeingui;backup=1
/modem ext4 /dev/block/platform/msm_sdcc.1/by-name/apnhlos
/mdm emmc /dev/block/platform/msm_sdcc.1/by-name/mdm
```
Add your specific partitions and flags as necessary.

## Building TWRP
1. Source the build environment:
```sh
source ./build/envsetup.sh
```
2. Select the device:
```sh
lunch omni_<device_codename>-eng
```
3. Compile the recovery image:
```sh
make clean && make -j$(nproc) recoveryimage
```
Replace `$(nproc)` with the number of CPU cores +1 (e.g., for a quad-core CPU, use `-j5`).

### Samsung Devices
For most Samsung devices:
```sh
make -j$(nproc) bootimage
```

## A/B Devices
For A/B devices (devices with duplicate partitions):
1. Set the following in `BoardConfig.mk`:
```makefile
AB_OTA_UPDATER := true
BOARD_USES_RECOVERY_AS_BOOT := true
BOARD_BUILD_SYSTEM_ROOT_IMAGE := true
```
2. Update `recovery.fstab` for slot-select support:
```fstab
/boot emmc /dev/block/bootdevice/by-name/boot flags=slotselect
/system ext4 /dev/block/bootdevice/by-name/system flags=slotselect
/vendor ext4 /dev/block/bootdevice/by-name/vendor flags=slotselect;display="Vendor";backup=1
```
3. Compile the boot image:
```sh
make bootimage
```

## Flashing TWRP on A/B Devices
Using Fastboot

1. Check the active slot:
```sh
adb shell getprop ro.boot.slot_suffix
```
2. Switch to the inactive slot:
```sh
fastboot --set-active=_a
```
3. Flash TWRP:
```sh
fastboot flash boot twrp.img && fastboot reboot
```

## Additional Resources
- [TWRP Source Code](https://github.com/TeamWin/android_bootable_recovery)
- [Minimal Manifest TWRP](https://github.com/minimal-manifest-twrp)
- [Old Guide](http://xdaforums.com/showpost.php?p=65482905&postcount=1471)

## Community and Support
For questions, join #twrp on Freenode or visit the XDA forums. If you successfully port TWRP to a new device, 
please share your success stories with the community.

Happy building!

**Team Win Recovery Project (TWRP)**
