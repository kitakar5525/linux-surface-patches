# linux-surface patches

- Intended for **Surface Book 1 (especially, with Performance Base)** and **Surface 3**, but all patches (or equivalent) from [jakeday repository](https://github.com/jakeday/linux-surface) are included

Please refer to every patch header comment or patch name for why the patch is added.

## See also

### My memos
- [kitakar5525/note-linux-on-surface-book-1: Notes to use Linux on Surface Book 1 with Performance Base](https://github.com/kitakar5525/note-linux-on-surface-book-1)
- [kitakar5525/note-linux-on-surface-3: Notes to use Linux on Surface 3](https://github.com/kitakar5525/note-linux-on-surface-3)

How to build chromeos kernel with patches (I sometimes release a prebuilt kernel binary for chromeos here)
- [kitakar5525/chromeos-kernel-linux-surface: linux-surface kernel for Chromium OS/Chrome OS based OSes.](https://github.com/kitakar5525/chromeos-kernel-linux-surface)

How to multiboot Chromium OS
- [kitakar5525/chromium-os-on-surface-devices: Multiboot Chromium OS with another OSes on Surface devices](https://github.com/kitakar5525/chromium-os-on-surface-devices)

### PKGBUILD for Arch Linux
I sometimes release a prebuilt kernel binary for Arch Linux here.
- [kitakar5525/arch-kernel-linux-lts419-surface: linux-surface for Arch Linux linux-lts kernel](https://github.com/kitakar5525/arch-kernel-linux-lts419-surface)
- [kitakar5525/arch-kernel-linux-surface: linux-surface for Arch Linux latest stable kernel](https://github.com/kitakar5525/arch-kernel-linux-surface)
- [kitakar5525/arch-kernel-linux-rc-surface: linux-surface for Arch Linux latest rc kernel](https://github.com/kitakar5525/arch-kernel-linux-rc-surface)

## patch-5.3

~~IPTS is not working yet.~~

### 2019-08-07
It turned out that GuC submission will not be allowed on 5.3. Thus, IPTS will not work on 5.3.
- [drm/i915/guc: Don't allow GuC submission · torvalds/linux@a2904ad](https://github.com/torvalds/linux/commit/a2904ade3dc28cf1a1b7deded41f4369f75e664c)
- [drm/i915/guc: Updates for GuC 32.0.3 firmware · torvalds/linux@ffd5ce2](https://github.com/torvalds/linux/commit/ffd5ce22faa4d07a07085b497717d7650f72fd5f)

## 2019-09-16
Currently, using i915 modules from 5.2.y (i915_legacy) to use GuC submission (StollD/linux-surface-patches@ce48afa).