# IPTS module changes from 5.1 to 5.2

See also
- [linux-surface-patches/patch-5.2/0000-jakeday at master · kitakar5525/linux-surface-patches](https://github.com/kitakar5525/linux-surface-patches/tree/master/patch-5.2/0000-jakeday)



## Major patch changes

### Adapted to i915 functions signature changes
- `execlists_context_pin()`
- `execlists_context_deferred_alloc()`

- [drm/i915: Make context pinning part of intel_context_ops · torvalds/linux@95f697e](https://github.com/torvalds/linux/commit/95f697eb024d7def7f9050cd5eba9502034dd94d)

- `i915_gem_create_context()`

- [drm/i915: Separate GEM context construction and registration to users… · torvalds/linux@3aa9945](https://github.com/torvalds/linux/commit/3aa9945a528e7616b5c8fe5d7aa7d4aaf52b0af2)
- [drm/i915: Allow contexts to share a single timeline across all engines · torvalds/linux@ea593db](https://github.com/torvalds/linux/commit/ea593dbba4c8ed841630fa5445202627e1046ba6)

### Adapted to i915 name changes
- `ring_mask` to `engine_mask`
- `RCS` to `RCS0`

- [drm/i915: Store the BIT(engine->id) as the engine's mask · torvalds/linux@8a68d46](https://github.com/torvalds/linux/commit/8a68d464366efb5b294fa11ccf23b51306cc2695)

### `to_intel_context()` is not available anymore
- Candidate is `intel_context_pin()` or `intel_context_pin_lock()`

- [drm/i915: Move over to intel_context_lookup() · torvalds/linux@c4d52fe](https://github.com/torvalds/linux/commit/c4d52feb2c46ddcdde4058cf03f8b9eb996bb09b)



### Continue using `execlists_context_pin()`

Using `intel_context_pin()` like this resulted in black screen right after boot:
```diff
-    pin_ret = execlists_context_pin(dev_priv->engine[RCS], ipts_ctx);
+    pin_ret = intel_context_pin(ipts_ctx, dev_priv->engine[RCS0]);
    if (IS_ERR(pin_ret)) {
        DRM_DEBUG("lr context pinning failed :  %ld\n", PTR_ERR(pin_ret));
        goto err_ctx;
    }
```



## Complete changes

ipts-5.2: resolved .rej files
```diff
From fe16909732f8bce2f9f8126060d9b603adb6ef54 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Thu, 18 Jul 2019 22:03:36 +0900
Subject: [PATCH 1/2] ipts-5.2: resolved .rej files

Stop exposing __execlists_context_pin()
---
 drivers/gpu/drm/i915/i915_irq.c    | 4 +++-
 drivers/gpu/drm/i915/intel_lrc.c   | 6 ++----
 drivers/gpu/drm/i915/intel_lrc.h   | 9 +++------
 drivers/gpu/drm/i915/intel_panel.c | 1 +
 4 files changed, 9 insertions(+), 11 deletions(-)

diff --git a/drivers/gpu/drm/i915/i915_irq.c b/drivers/gpu/drm/i915/i915_irq.c
index c542e27e9..78fcd4b78 100644
--- a/drivers/gpu/drm/i915/i915_irq.c
+++ b/drivers/gpu/drm/i915/i915_irq.c
@@ -41,6 +41,7 @@
 #include "i915_trace.h"
 #include "intel_drv.h"
 #include "intel_psr.h"
+#include "intel_ipts.h"
 
 /**
  * DOC: interrupt handling
@@ -4058,7 +4059,8 @@ static void gen8_gt_irq_postinstall(struct drm_i915_private *dev_priv)
 
 	/* These are interrupts we'll toggle with the ring mask register */
 	u32 gt_interrupts[] = {
-		(GT_RENDER_USER_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
+		(GT_RENDER_PIPECTL_NOTIFY_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
+		 GT_RENDER_USER_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
 		 GT_CONTEXT_SWITCH_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
 		 GT_RENDER_USER_INTERRUPT << GEN8_BCS_IRQ_SHIFT |
 		 GT_CONTEXT_SWITCH_INTERRUPT << GEN8_BCS_IRQ_SHIFT),
diff --git a/drivers/gpu/drm/i915/intel_lrc.c b/drivers/gpu/drm/i915/intel_lrc.c
index 247576bf4..10b5dd1a4 100644
--- a/drivers/gpu/drm/i915/intel_lrc.c
+++ b/drivers/gpu/drm/i915/intel_lrc.c
@@ -166,8 +166,6 @@
 
 #define ACTIVE_PRIORITY (I915_PRIORITY_NOSEMAPHORE)
 
-static int execlists_context_deferred_alloc(struct intel_context *ce,
-					    struct intel_engine_cs *engine);
 static void execlists_init_reg_state(u32 *reg_state,
 				     struct intel_context *ce,
 				     struct intel_engine_cs *engine,
@@ -1285,7 +1283,7 @@ __execlists_context_pin(struct intel_context *ce,
 	return ret;
 }
 
-static int execlists_context_pin(struct intel_context *ce)
+int execlists_context_pin(struct intel_context *ce)
 {
 	return __execlists_context_pin(ce, ce->engine);
 }
@@ -2884,7 +2882,7 @@ static struct i915_timeline *get_timeline(struct i915_gem_context *ctx)
 		return i915_timeline_create(ctx->i915, NULL);
 }
 
-static int execlists_context_deferred_alloc(struct intel_context *ce,
+int execlists_context_deferred_alloc(struct intel_context *ce,
 					    struct intel_engine_cs *engine)
 {
 	struct drm_i915_gem_object *ctx_obj;
diff --git a/drivers/gpu/drm/i915/intel_lrc.h b/drivers/gpu/drm/i915/intel_lrc.h
index 4297cfff2..bfef20e4f 100644
--- a/drivers/gpu/drm/i915/intel_lrc.h
+++ b/drivers/gpu/drm/i915/intel_lrc.h
@@ -115,13 +115,10 @@ void intel_execlists_show_requests(struct intel_engine_cs *engine,
 							const char *prefix),
 				   unsigned int max);
 
-struct intel_context *
-execlists_context_pin(struct intel_engine_cs *engine,
-		      struct i915_gem_context *ctx);
+int execlists_context_pin(struct intel_context *ce);
 void execlists_context_unpin(struct intel_context *ce);
-int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
-				     struct intel_engine_cs *engine,
-				     struct intel_context *ce);
+int execlists_context_deferred_alloc(struct intel_context *ce,
+					    struct intel_engine_cs *engine);
 
 
 u32 gen8_make_rpcs(struct drm_i915_private *i915, struct intel_sseu *ctx_sseu);
diff --git a/drivers/gpu/drm/i915/intel_panel.c b/drivers/gpu/drm/i915/intel_panel.c
index 0b3f2cfc2..2d3c523ba 100644
--- a/drivers/gpu/drm/i915/intel_panel.c
+++ b/drivers/gpu/drm/i915/intel_panel.c
@@ -37,6 +37,7 @@
 #include "intel_connector.h"
 #include "intel_drv.h"
 #include "intel_panel.h"
+#include "intel_ipts.h"
 
 #define CRC_PMIC_PWM_PERIOD_NS	21333
 
-- 
2.22.0


```

ipts-5.2: resolved build time errors
```diff
From e639952e323f0bfb813ac60ce038236afa9d7be9 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Thu, 18 Jul 2019 22:16:13 +0900
Subject: [PATCH 2/2] ipts-5.2: resolved build time errors

Update according to @qzed finding

More update according to @qzed finding
---
 drivers/gpu/drm/i915/i915_gem_context.c     |  2 +-
 drivers/gpu/drm/i915/intel_guc_submission.c |  2 +-
 drivers/gpu/drm/i915/intel_ipts.c           | 14 +++++++-------
 3 files changed, 9 insertions(+), 9 deletions(-)

diff --git a/drivers/gpu/drm/i915/i915_gem_context.c b/drivers/gpu/drm/i915/i915_gem_context.c
index 03977569c..ae3209b79 100644
--- a/drivers/gpu/drm/i915/i915_gem_context.c
+++ b/drivers/gpu/drm/i915/i915_gem_context.c
@@ -572,7 +572,7 @@ struct i915_gem_context *i915_gem_context_create_ipts(struct drm_device *dev)
 
 	BUG_ON(!mutex_is_locked(&dev->struct_mutex));
 
-	ctx = i915_gem_create_context(dev_priv, NULL);
+	ctx = i915_gem_create_context(dev_priv, 0);
 
 	return ctx;
 }
diff --git a/drivers/gpu/drm/i915/intel_guc_submission.c b/drivers/gpu/drm/i915/intel_guc_submission.c
index fad19db49..475bebc91 100644
--- a/drivers/gpu/drm/i915/intel_guc_submission.c
+++ b/drivers/gpu/drm/i915/intel_guc_submission.c
@@ -1476,7 +1476,7 @@ int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
 
 	/* client for execbuf submission */
 	client = guc_client_alloc(dev_priv,
-				  INTEL_INFO(dev_priv)->ring_mask,
+				  INTEL_INFO(dev_priv)->engine_mask,
 				  IS_SKYLAKE(dev_priv) || IS_KABYLAKE(dev_priv) ? GUC_CLIENT_PRIORITY_HIGH : GUC_CLIENT_PRIORITY_NORMAL,
 				  ctx);
 	if (IS_ERR(client)) {
diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index b276a2f78..6647c92d1 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -175,7 +175,6 @@ static int create_ipts_context(void)
 	struct i915_gem_context *ipts_ctx = NULL;
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
 	struct intel_context *ce = NULL;
-	struct intel_context *pin_ret;
 	int ret = 0;
 
 	/* Initialize the context right away.*/
@@ -193,7 +192,7 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+	ce = intel_context_pin(ipts_ctx, dev_priv->engine[RCS0]);
 	if (IS_ERR(ce)) {
 		DRM_ERROR("Failed to create intel context (error %ld)\n",
 			  PTR_ERR(ce));
@@ -201,15 +200,15 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ret = execlists_context_deferred_alloc(ipts_ctx, dev_priv->engine[RCS], ce);
+	ret = execlists_context_deferred_alloc(ce, ce->engine);
 	if (ret) {
 		DRM_DEBUG("lr context allocation failed : %d\n", ret);
 		goto err_ctx;
 	}
 
-	pin_ret = execlists_context_pin(dev_priv->engine[RCS], ipts_ctx);
-	if (IS_ERR(pin_ret)) {
-		DRM_DEBUG("lr context pinning failed :  %ld\n", PTR_ERR(pin_ret));
+	ret = execlists_context_pin(ce);
+	if (ret) {
+		DRM_DEBUG("lr context pinning failed :  %d\n", ret);
 		goto err_ctx;
 	}
 
@@ -242,7 +241,7 @@ static void destroy_ipts_context(void)
 
 	ipts_ctx = intel_ipts.ipts_context;
 
-	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+	ce = intel_context_lookup(ipts_ctx, dev_priv->engine[RCS0]);
 
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
@@ -252,6 +251,7 @@ static void destroy_ipts_context(void)
 	}
 
 	execlists_context_unpin(ce);
+	intel_context_unpin(ce);
 	i915_gem_context_put(ipts_ctx);
 
 	mutex_unlock(&intel_ipts.dev->struct_mutex);
-- 
2.22.0


```



## Feature additions / Bug fixes

Fix RCS0 GPU hang on module removing
```diff
From ee64e1cd19364d7e1150757d1c25e9f0d4540d4e Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Wed, 24 Jul 2019 21:57:47 +0900
Subject: [PATCH] Fix RCS0 GPU hang on module removing
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

Before this commit, when you removed intel_ipts module, you will see
this GPU hang:
\```
kernel: IPTS ipts_mei_cl_exit() is called
kernel: ipts mei::3e8d0870-271a-4208-8eb5-9acb9402ae04:0F: error in reading m2h msg
kernel: IPTS removed
kernel: Asynchronous wait on fence i915:systemd-logind[547]/0:75a timed out (hint:intel_atomic_commit_ready+0x0/0x50 [i915])
kernel: i915 0000:00:02.0: GPU HANG: ecode 9:0:0x00000000, no progress on rcs0
kernel: [drm] GPU hangs can indicate a bug anywhere in the entire gfx stack, including userspace.
kernel: [drm] Please file a _new_ bug report on bugs.freedesktop.org against DRI -> DRM/Intel
kernel: [drm] drm/i915 developers can then reassign to the right component if it's not a kernel issue.
kernel: [drm] The gpu crash dump is required to analyze gpu hangs, so please always attach it.
kernel: [drm] GPU crash dump saved to /sys/class/drm/card0/error
kernel: i915 0000:00:02.0: Resetting rcs0 for no progress on rcs0
kernel: i915 0000:00:02.0: Resetting rcs0 for hang on rcs0
kernel: [drm:intel_guc_send_mmio [i915]] *ERROR* MMIO: GuC action 0x3 failed with error -110 0x3
kernel: i915 0000:00:02.0: Resetting chip for hang on rcs0
kernel: i915 0000:00:02.0: GPU reset not supported
\```

This commit fixes GPU hang on removing intel_ipts module.

See:
・ Porting patches to Linux 5.2 (ipts) · Issue #544 · jakeday/linux-surface
https://github.com/jakeday/linux-surface/issues/544#issuecomment-514367983
---
 drivers/misc/ipts/ipts-msg-handler.c | 6 ++++++
 1 file changed, 6 insertions(+)

diff --git a/drivers/misc/ipts/ipts-msg-handler.c b/drivers/misc/ipts/ipts-msg-handler.c
index 8b214f975..db5356a1c 100644
--- a/drivers/misc/ipts/ipts-msg-handler.c
+++ b/drivers/misc/ipts/ipts-msg-handler.c
@@ -151,6 +151,9 @@ void ipts_stop(ipts_info_t *ipts)
 	old_state = ipts_get_state(ipts);
 	ipts_set_state(ipts, IPTS_STA_STOPPING);
 
+	ipts_send_sensor_quiesce_io_cmd(ipts);
+	ipts_send_sensor_clear_mem_window_cmd(ipts);
+
 	if (old_state < IPTS_STA_RESOURCE_READY)
 		return;
 
@@ -267,6 +270,9 @@ int ipts_handle_resp(ipts_info_t *ipts, touch_sensor_msg_m2h_t *m2h_msg,
 				break;
 			}
 
+			if (ipts_get_state(ipts) == IPTS_STA_STOPPING)
+				break;
+
 			/* allocate default resource : common & hid only */
 			if (!ipts_is_default_resource_ready(ipts)) {
 				ret = ipts_allocate_default_resource(ipts);
-- 
2.22.0


```
