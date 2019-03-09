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



## Status for 4.19

jakeday patches
- [updating patches · jakeday/linux-surface@e09d9e1](https://github.com/jakeday/linux-surface/commit/e09d9e11f58281aec3780849cdaf9336579a4169)

Improve s0ix
- 4416-01-5525-ipu_patches-419.patch, [201579 – HP Elite x2 1013 G3 unable to enter S0ix](https://bugzilla.kernel.org/show_bug.cgi?id=201579)
- [[v3,0/5] ICL support and other enhancements for PMC Core - Patchwork](https://patchwork.kernel.org/cover/10812541/)

Improve s0ix on Cherry Trail on 4.19
- [platform/x86: Add Intel AtomISP2 dummy / power-management driver · torvalds/linux@49ad712](https://github.com/torvalds/linux/commit/49ad712afa88c502831d37f7089d98eac441fb80)
- [pwm: lpss: Add ACPI HID for second PWM controller on Cherry Trail dev… · torvalds/linux@1688c87](https://github.com/torvalds/linux/commit/1688c8717118f37191d824862a006c8373d261de)

Surface 3 sound fix for OEMB devices
- 5525-sound-add-dmi-match-OEMB-for-affected-surface-3.patch



## Status for 5.00

jakeday patches
- [updating patches · jakeday/linux-surface@e09d9e1](https://github.com/jakeday/linux-surface/commit/e09d9e11f58281aec3780849cdaf9336579a4169)
- [Porting patches to Linux 5.0 (surface-acpi, ipts) · Issue #417 · jakeday/linux-surface](https://github.com/jakeday/linux-surface/issues/417)

Improve s0ix
- [[1/1] ipu3-cio2: Allow probe to succeed if there are no sensors connected - Patchwork](https://patchwork.kernel.org/patch/10714257/)
- [[v3,0/5] ICL support and other enhancements for PMC Core - Patchwork](https://patchwork.kernel.org/cover/10812541/)

Surface 3 sound fix for OEMB devices
- 5525-sound-add-dmi-match-OEMB-for-affected-surface-3.patch