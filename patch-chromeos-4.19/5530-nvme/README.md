# Update patch set for issue "NVMe breaks suspend (s2idle)"

From:
Bug 199689 - s2idle does not work in Dell XPS 9370
https://bugzilla.kernel.org/show_bug.cgi?id=199689#c91

- Reverting the patch 0000-jakeday/0002-suspend.patch of nvme part.

- This patch set is a successor of this patch set I proposed before on jakeday repo:
Surface Book with Performance Base: NVMe SSD breaks suspend (s2idle)
jakeday/linux-surface#123 (comment)
- Originally, meant to reduce NVMe disk power consumption
while suspend (s2idle) on some NVMe disks.
- Also, fixes resuming form s2idle on some devices
- Will be hopefully merged into Linux 5.3
until then, I apply this patch set personally.

Especially, on some Surface devices, for example, my
"Surface Book with Performance Base", which uses TOSHIBA NVMe disk:
```
THNSN5512GPU7 TOSHIBA
Toshiba America Info Systems NVMe Controller [1179:010f]
```
it will wake up from s2idle only once, second times or later, it will
not wake up from s2idle anymore without this patch set.
