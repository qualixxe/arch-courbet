# arch-courbet

Arch Linux ARM bring-up for the Xiaomi Mi 11 Lite 4G (`courbet`, SM7150 / Snapdragon 732G).

This is an early device bring-up, not a ready-to-install phone image. The goal is to get a real Arch ARM userspace booting on the device, then bring up a local graphical session with a terminal.

## Current state

- A separate mainline Linux 6.17 bring-up for `courbet` reaches a serial shell and renders the console on the phone display. Its source and tested boot images are in [qualixxe/mainline-courbet](https://github.com/qualixxe/mainline-courbet).
- The current console image is a temporary bring-up initramfs. Seeing an Arch `os-release` file in an initramfs is not enough to make it an Arch installation; the phone previously reported Alpine and had no `pacman`.
- The generic Arch Linux ARM AArch64 rootfs archive is available, but it has not yet been booted on `courbet`.
- No graphical desktop has been tested on the phone. Weston/Wayland, GPU acceleration, touchscreen input, and a usable terminal session remain work in progress.

## Target

1. Provide the real Arch ARM root filesystem on a separate filesystem, without overwriting Android data.
2. Extend the initramfs to discover and mount that root filesystem, then `switch_root` into Arch.
3. Verify basic userspace on-device (`/etc/os-release`, `pacman`, `uname -m`, serial login and a terminal on the display).
4. Bring up a compositor (start with Weston if the available DRM/KMS path permits it), then connect a terminal such as `foot`.
5. Test touchscreen, rendering stability, and a repeatable temporary boot before considering any persistent install.

## Safety

The device reports `userdata` as F2FS. Do not format or write to it as part of bring-up without a verified backup and an explicit installation plan. Early tests should use `fastboot boot` rather than flashing partitions. The known fixed boot image and the Alpine-based pmOS boot are not Arch desktop images.

## Device notes

- Device: Xiaomi Mi 11 Lite 4G, codename `courbet`
- SoC: Qualcomm SM7150 / Snapdragon 732G
- Panel: K9A DSI panel
- Mainline kernel baseline: Linux 6.17 `courbet` bring-up
- Known working area: display console and serial shell
- Not yet established: full device support, persistent Arch install, GPU-accelerated Wayland, touch in a desktop session

## Related work

- [Mainline kernel and display bring-up](https://github.com/qualixxe/mainline-courbet)
- [Arch Linux ARM](https://archlinuxarm.org/)
