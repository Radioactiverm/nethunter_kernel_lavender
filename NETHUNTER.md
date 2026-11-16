# Kali NetHunter kernel — Xiaomi Redmi Note 7 (lavender)

A NetHunter-enabled build of the **Mimir** kernel for the Xiaomi Redmi Note 7
(codename **lavender**, SD660 / `msm-4.4`). It adds wireless monitor-mode +
packet injection, an external RTL8812AU driver, and HID/BadUSB support on top of
rehork's kernel, so you can run Kali NetHunter on a modern Android 13 ROM.

> **Forked from [rehork/kernel_lavender](https://github.com/rehork/kernel_lavender)**
> (branch `EAS`). All the base hardware support and credit for the kernel itself
> belong to rehork and the Mimir kernel project. This branch only adds the
> NetHunter patches.

## Verified working

- **Kernel:** `4.4.302-MimirKernel-EAS` — **NOT** 4.19. crDroid 9.12 (LineageOS 20,
  Android 13) for lavender ships a **4.4** kernel; a 4.19 NetHunter kernel
  bootloops. This is the 4.4 build and it boots cleanly.
- **ROM tested:** crDroid 9.12 (Android 13), rooted with Magisk.
- Internal Wi-Fi (bcmdhd / `phy0`) **monitor mode** works.
- External **RTL8812AU** adapter: `88XXau.ko` loads and does monitor + injection.
- **HID / BadUSB** (`/dev/uhid`, USB HID gadget) available.
- Full Kali NetHunter chroot + KeX desktop run on it.

## What this branch changes

| Area | Change |
|------|--------|
| `net/mac80211/*`, `net/wireless/chan.c` | Monitor mode + packet-injection patches |
| `drivers/net/wireless/ath/regd.c` | Regulatory-domain unlock (all channels / TX power) |
| `drivers/net/wireless/realtek/rtl8812au/` | **New** — RTL8812AU/21AU driver (`88XXAU=m`) |
| `drivers/net/wireless/{Kconfig,realtek/Makefile}` | Wire the rtl8812au driver into the build |
| `drivers/net/wireless/zd1211rw`, `rtl818x/rtl8187` | Injection patches for those adapters |
| `arch/arm64/configs/nethunter_defconfig` | **New** — NetHunter defconfig |
| `scripts/sign-file.c`, `scripts/extract-cert.c` | Build fixes for **OpenSSL 3.0+** host toolchains |

Key options enabled in `nethunter_defconfig`: `CONFIG_MAC80211=y`,
`CONFIG_CFG80211_WEXT=y`, `CONFIG_CFG80211_INTERNAL_REGDB=y`, `CONFIG_88XXAU=m`,
`CONFIG_UHID=y`, `CONFIG_HIDRAW=y`, `CONFIG_USB_CONFIGFS_F_HID=y`.

## Flashing (easiest)

1. Download the flashable zip from the **[Releases](../../releases)** page.
2. Boot into **OrangeFox / TWRP**.
3. `adb sideload NetHunter-kernel-lavender-*-mimir44.zip`
   (or copy it to the phone and flash it from the recovery UI).
4. Reboot. Install the [Kali NetHunter generic arm64 installer](https://www.kali.org/get-kali/#kali-mobile)
   as a Magisk module to get the app + Kali chroot.

> ⚠️ **lavender + a 4.4 (LineageOS-20 / crDroid-9) ROM only.** Do not flash on a
> ROM that ships a different kernel version.

## Building from source

```bash
# arm64 cross toolchain + clang in your PATH
make O=out ARCH=arm64 nethunter_defconfig
make O=out ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
     CROSS_COMPILE_ARM32=arm-linux-gnueabi- -j"$(nproc)"
# output: out/arch/arm64/boot/Image.gz-dtb
```

The external adapter module is built from
`drivers/net/wireless/realtek/rtl8812au/` (`88XXau.ko`); its vermagic must match
the running kernel (`4.4.302-MimirKernel-EAS/<hash>`).

If your host has OpenSSL 3.0+ and the build fails in `scripts/sign-file` or
`scripts/extract-cert`, this branch already contains the fixes for it.

## Credits

- **[rehork/kernel_lavender](https://github.com/rehork/kernel_lavender)** / the
  **Mimir** kernel — the base kernel this is built on.
- **[Kali NetHunter](https://www.kali.org/docs/nethunter/)** — the wireless
  injection / HID patch set and the userland.
- **LineageOS / crDroid** — the ROM and kernel lineage.

## License

GPLv2, same as the Linux kernel (see `COPYING`). Kernel sources distributed with
a NetHunter build must remain available — that is the purpose of this repository.
