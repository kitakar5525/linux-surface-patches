# Update patch set for issue "NVMe breaks suspend (s2idle)" and "SSD heating while suspend"

For 5.2
- (Revert NVMe part of 0002-suspend.patch first to apply this patch set)
- [nvme: export get and set features · torvalds/linux@1a87ee6](https://github.com/torvalds/linux/commit/1a87ee657c530bb2f3e39e4ac184d48f5f959cda)
- [nvme-pci: use host managed power state for suspend · torvalds/linux@d916b1b](https://github.com/torvalds/linux/commit/d916b1be94b6dc8d293abed2451f3062f6af7551#diff-bc4c090f021c046a7d256a3fcf86b7da)

For 4.19, this patch is also needed
- [nvme-pci: Sync queues on reset · torvalds/linux@d6135c3](https://github.com/torvalds/linux/commit/d6135c3a1ec0cddda7b8b8e1b5b4abeeafd98289#diff-bc4c090f021c046a7d256a3fcf86b7da)

## What is the problem?
It seems that some Surface devices, for example, my
"Surface Book with Performance Base", which uses TOSHIBA NVMe disk:
```
THNSN5512GPU7 TOSHIBA
Toshiba America Info Systems NVMe Controller [1179:010f]
```
only work well with host managed power state on their NVMe SSD instead
of generic PCI power settings.
The device makes a lot of heat during suspend and never wakes up after
second attempt of suspend.

## Patch set description
- Originally, meant to reduce NVMe SSD heating while s2idle on
all NVMe SSD disks which only works well with host managed power state
- As a result, also fixes unable to wakeup from s2idle on all NVMe SSD
disks which only works well with host managed power state
- Will be hopefully merged into Linux 5.3
When it arrived, remove this patch set.

## Changes from current 0002-suspend.patch
- It enables host managed power state on all NVMe SSD disks which supports
the state instead of generic PCI power settings
- whereas current 0002-suspend.patch in this repo enables that state on only
SK Hynix 0x1527 disks and Toshiba 0x010f disks only

https://github.com/jakeday/linux-surface/blob/3d0abed6c461fd269694b66b9bb6372be230fa20/patches/5.1/0002-suspend.patch#L75-L78




References
- [196907 – [Regression] s2idle does not work with PC300 NVMe SK hynix 512GB - Dell XPS 13 9360](https://bugzilla.kernel.org/show_bug.cgi?id=196907)
- [199689 – s2idle does not work in Dell XPS 9370](https://bugzilla.kernel.org/show_bug.cgi?id=199689)
