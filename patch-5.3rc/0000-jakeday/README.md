# Changes from 5.2 to 5.3rc

## IPTS changes

### Path changes
- `drivers/gpu/drm/i915/i915_gem_context.c` to `drivers/gpu/drm/i915/gem/i915_gem_context.c`
- `drivers/gpu/drm/i915/intel_dp.c` to `drivers/gpu/drm/i915/display/intel_dp.c`
- `drivers/gpu/drm/i915/intel_lrc.c` to `drivers/gpu/drm/i915/gt/intel_lrc.c`
- `drivers/gpu/drm/i915/intel_lrc.h` to `drivers/gpu/drm/i915/gt/intel_lrc.h`
- `drivers/gpu/drm/i915/intel_panel.c` to `drivers/gpu/drm/i915/display/intel_panel.c`

- [drm/i915: Move more GEM objects under gem/ · torvalds/linux@10be98a](https://github.com/torvalds/linux/commit/10be98a77c558f8cfb823cd2777171fbb35040f6)
- [drm/i915: move modesetting output/encoder code under display/ · torvalds/linux@379bc10](https://github.com/torvalds/linux/commit/379bc100232acd45b19421bd0748f9f549da8a8a)
- [drm/i915: Move GraphicsTechnology files under gt/ · torvalds/linux@112ed2d](https://github.com/torvalds/linux/commit/112ed2d31a46f4704085ad925435b77e62b8abee)

### `i915_gem_object_create()` is not available anymore
- [drm/i915: Move shmem object setup to its own file · torvalds/linux@8475355](https://github.com/torvalds/linux/commit/8475355f7a2645a022288301c03555c31fb4de17#diff-b33e15b3f5b3b0ee4d341dfba50afe04)
- [drm/i915: Move GEM domain management to its own file · torvalds/linux@f0e4a06](https://github.com/torvalds/linux/commit/f0e4a06397526d3352a3c80b0575ac22ab24da94)

- `i915_gem_object_create()` to `i915_gem_object_create_shmem`

### error: ‘struct i915_gem_context’ has no member named ‘ppgtt’
- [drm/i915: Pull kref into i915_address_space · torvalds/linux@e568ac3](https://github.com/torvalds/linux/commit/e568ac3874be7dcef3da0cc3bd6b91ca9dd14aa0)

### error: implicit declaration of function ‘intel_context_pin_lock’ [-Werror=implicit-function-declaration]
- [drm/i915: Pass intel_context to intel_context_pin_lock() · torvalds/linux@6b736de](https://github.com/torvalds/linux/commit/6b736de5746a304692fc5f7f5fc46cd9c2e8bd29)
- [drm/i915: Export intel_context_instance() · torvalds/linux@fa9f668](https://github.com/torvalds/linux/commit/fa9f668141f4e5590837845ffc1dc4f5aca7a0a5#diff-98020f00749192bd43e3eb49d0194818)
- [drm/i915: Switch back to an array of logical per-engine HW contexts · torvalds/linux@5e2a041](https://github.com/torvalds/linux/commit/5e2a0419ef7cb25d0f9a5fd6a62372bb47ce948d#diff-98020f00749192bd43e3eb49d0194818)

### error: implicit declaration of function ‘i915_gem_context_put’



### Complete changes

ipts-5.2rc: Resolved .rej files
```diff
From d93dcf84162fd2e8fb9ee5c5cf23d53702652367 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Mon, 22 Jul 2019 15:22:56 +0900
Subject: [PATCH 06/29] ipts-5.2rc: Resolved .rej files

---
 drivers/gpu/drm/i915/gt/intel_lrc.c | 3 +++
 drivers/gpu/drm/i915/gt/intel_lrc.h | 5 +++++
 drivers/gpu/drm/i915/i915_drv.c     | 5 ++++-
 drivers/gpu/drm/i915/i915_irq.c     | 1 +
 4 files changed, 13 insertions(+), 1 deletion(-)

diff --git a/drivers/gpu/drm/i915/gt/intel_lrc.c b/drivers/gpu/drm/i915/gt/intel_lrc.c
index 2e696dbcf..3a481fbb4 100644
--- a/drivers/gpu/drm/i915/gt/intel_lrc.c
+++ b/drivers/gpu/drm/i915/gt/intel_lrc.c
@@ -2684,6 +2684,9 @@ int intel_execlists_submission_setup(struct intel_engine_cs *engine)
 		engine->init_context = gen8_init_rcs_context;
 		engine->emit_flush = gen8_emit_flush_render;
 		engine->emit_fini_breadcrumb = gen8_emit_fini_breadcrumb_rcs;
+
+		engine->irq_keep_mask |= GT_RENDER_PIPECTL_NOTIFY_INTERRUPT
+					<< GEN8_RCS_IRQ_SHIFT;
 	}
 
 	return 0;
diff --git a/drivers/gpu/drm/i915/gt/intel_lrc.h b/drivers/gpu/drm/i915/gt/intel_lrc.h
index c2bba82bc..3db61569b 100644
--- a/drivers/gpu/drm/i915/gt/intel_lrc.h
+++ b/drivers/gpu/drm/i915/gt/intel_lrc.h
@@ -131,4 +131,9 @@ int intel_virtual_engine_attach_bond(struct intel_engine_cs *engine,
 				     const struct intel_engine_cs *master,
 				     const struct intel_engine_cs *sibling);
 
+int execlists_context_pin(struct intel_context *ce);
+void execlists_context_unpin(struct intel_context *ce);
+int execlists_context_deferred_alloc(struct intel_context *ce,
+					    struct intel_engine_cs *engine);
+
 #endif /* _INTEL_LRC_H_ */
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index e743e2b00..9d6f633ce 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -75,7 +75,7 @@
 #include "intel_csr.h"
 #include "intel_drv.h"
 #include "intel_pm.h"
-#include "intel_uc.h"
+#include "intel_ipts.h"
 
 static struct drm_driver driver;
 
@@ -2094,6 +2094,9 @@ static int i915_drm_suspend(struct drm_device *dev)
 	struct pci_dev *pdev = dev_priv->drm.pdev;
 	pci_power_t opregion_target_state;
 
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
+		intel_ipts_suspend(dev);
+
 	disable_rpm_wakeref_asserts(&dev_priv->runtime_pm);
 
 	/* We do a lot of poking in a lot of registers, make sure they work
diff --git a/drivers/gpu/drm/i915/i915_irq.c b/drivers/gpu/drm/i915/i915_irq.c
index 6450e58e6..909f8ac45 100644
--- a/drivers/gpu/drm/i915/i915_irq.c
+++ b/drivers/gpu/drm/i915/i915_irq.c
@@ -47,6 +47,7 @@
 #include "i915_trace.h"
 #include "intel_drv.h"
 #include "intel_pm.h"
+#include "intel_ipts.h"
 
 /**
  * DOC: interrupt handling
-- 
2.22.0

```

ipts-5.3rc: Resolved build time errors
```diff
From 566dbf113ca6f552b03283bfaad39b1010b07df7 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Mon, 22 Jul 2019 17:04:31 +0900
Subject: [PATCH 07/29] ipts-5.3rc: Resolved build time errors

---
 drivers/gpu/drm/i915/intel_ipts.c | 16 +++++++++-------
 1 file changed, 9 insertions(+), 7 deletions(-)

diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index 2810a0c86..6833a4de2 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -29,6 +29,8 @@
 
 #include "intel_guc_submission.h"
 #include "i915_drv.h"
+#include "gt/intel_context.h"
+#include "gem/i915_gem_context.h"
 
 #define SUPPORTED_IPTS_INTERFACE_VERSION	1
 
@@ -86,7 +88,7 @@ static intel_ipts_object_t *ipts_object_create(size_t size, u32 flags)
 	}
 
 	/* Allocate the new object */
-	gem_obj = i915_gem_object_create(dev_priv, size);
+	gem_obj = i915_gem_object_create_shmem(dev_priv, size);
 	if (gem_obj == NULL) {
 		ret = -ENOMEM;
 		goto err_out;
@@ -136,8 +138,8 @@ static int ipts_object_pin(intel_ipts_object_t* obj,
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
 	int ret = 0;
 
-	if (ipts_ctx->ppgtt) {
-		vm = &ipts_ctx->ppgtt->vm;
+	if (ipts_ctx->vm) {
+		vm = ipts_ctx->vm;
 	} else {
 		vm = &dev_priv->ggtt.vm;
 	}
@@ -193,7 +195,7 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ce = intel_context_pin_lock(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
 	if (IS_ERR(ce)) {
 		DRM_ERROR("Failed to create intel context (error %ld)\n",
 			  PTR_ERR(ce));
@@ -242,7 +244,7 @@ static void destroy_ipts_context(void)
 
 	ipts_ctx = intel_ipts.ipts_context;
 
-	ce = intel_context_pin_lock(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
 
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
@@ -452,8 +454,8 @@ static int intel_ipts_map_buffer(u64 gfx_handle, intel_ipts_mapbuffer_t *mapbuf)
 		obj->cpu_addr = ipts_object_map(obj);
 	}
 
-	if (ipts_ctx->ppgtt) {
-		vm = &ipts_ctx->ppgtt->vm;
+	if (ipts_ctx->vm) {
+		vm = ipts_ctx->vm;
 	} else {
 		vm = &dev_priv->ggtt.vm;
 	}
-- 
2.22.0

```



## 0007-sdcard-reader.patch changes

`USB_STATE_DEFAULT` to `USB_STATE_CONFIGURED`

- [usb: core: hub: Enable/disable U1/U2 in configured state · torvalds/linux@fea3af5](https://github.com/torvalds/linux/commit/fea3af5e0358b014c653561109c11ebd3ecdbff2)