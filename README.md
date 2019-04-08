# linux-surface patches

- Intended for **Surface Book 1 (especially, with Performance Base)** and **Surface 3**, but all patches (or equivalent) from [jakeday repository](https://github.com/jakeday/linux-surface) are included

See also:
- [kitakar5525/note-linux-on-surface-book-1: Notes to use Linux on Surface Book 1 with Performance Base](https://github.com/kitakar5525/note-linux-on-surface-book-1)
- [kitakar5525/note-linux-on-surface-3: Notes to use Linux on Surface 3](https://github.com/kitakar5525/note-linux-on-surface-3)



## patch filename prefix

- 4416
	- Indicate that the patch is not from me (Hayataka@kitakar5525)
- 552?
	- Indicate that the patch is made/modified by me
- 4416-*552?
	- Indicate that the patch is from another person and modified by me



## state of 4.19 patches

jakeday patches
- [adding support for 5.0 kernel · jakeday/linux-surface@5c2a640](https://github.com/jakeday/linux-surface/commit/5c2a640308d7628f0af341531e078c996b1ba917)

Improve s0ix
- 4416-01-5525-ipu_patches-419.patch, [201579 – HP Elite x2 1013 G3 unable to enter S0ix](https://bugzilla.kernel.org/show_bug.cgi?id=201579)
- [[v3,0/5] ICL support and other enhancements for PMC Core - Patchwork](https://patchwork.kernel.org/cover/10812541/)

Improve s0ix on Cherry Trail on 4.19 (these two patches are merged into 4.20)
- [platform/x86: Add Intel AtomISP2 dummy / power-management driver · torvalds/linux@49ad712](https://github.com/torvalds/linux/commit/49ad712afa88c502831d37f7089d98eac441fb80)
- [pwm: lpss: Add ACPI HID for second PWM controller on Cherry Trail dev… · torvalds/linux@1688c87](https://github.com/torvalds/linux/commit/1688c8717118f37191d824862a006c8373d261de)

Add quirk for Surface SAM I2C address space
- [i2c: acpi: Introduce i2c_acpi_get_i2c_resource() helper · torvalds/linux@0d5102f](https://github.com/torvalds/linux/commit/0d5102fe85302aa06a3e5fd8e63b09294aed4c48)
- [qzed/linux-surface-sam-hid: Experimental Linux drivers for Surface SAM over HID.](https://github.com/qzed/linux-surface-sam-hid)

Surface 3 sound fix for OEMB devices
- 5525-sound-add-dmi-match-OEMB-for-affected-surface-3.patch



## state of 5.0 patches

jakeday patches
- [adding support for 5.0 kernel · jakeday/linux-surface@5c2a640](https://github.com/jakeday/linux-surface/commit/5c2a640308d7628f0af341531e078c996b1ba917)

Improve s0ix
- [[1/1] ipu3-cio2: Allow probe to succeed if there are no sensors connected - Patchwork](https://patchwork.kernel.org/patch/10714257/)
- [[v3,0/5] ICL support and other enhancements for PMC Core - Patchwork](https://patchwork.kernel.org/cover/10812541/)

Add quirk for Surface SAM I2C address space
- [qzed/linux-surface-sam-hid: Experimental Linux drivers for Surface SAM over HID.](https://github.com/qzed/linux-surface-sam-hid)

Surface 3 sound fix for OEMB devices
- 5525-sound-add-dmi-match-OEMB-for-affected-surface-3.patch



## state of 5.1rc patches

base jakeday patches
- [adding support for 5.0 kernel · jakeday/linux-surface@5c2a640](https://github.com/jakeday/linux-surface/commit/5c2a640308d7628f0af341531e078c996b1ba917)

Surface 3 sound fix for OEMB devices
- 5525-sound-add-dmi-match-OEMB-for-affected-surface-3.patch

Add quirk for Surface SAM I2C address space
- [qzed/linux-surface-sam-hid: Experimental Linux drivers for Surface SAM over HID.](https://github.com/qzed/linux-surface-sam-hid)

### merged into 5.1rc

Improve s0ix
- [[1/1] ipu3-cio2: Allow probe to succeed if there are no sensors connected - Patchwork](https://patchwork.kernel.org/patch/10714257/)
- [[v3,0/5] ICL support and other enhancements for PMC Core - Patchwork](https://patchwork.kernel.org/cover/10812541/)