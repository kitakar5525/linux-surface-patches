# Changes from 5.2 to 5.3rc

## IPTS changes

IPTS is not working. It turned out that GuC submission will not be allowed on 5.3. Thus, IPTS will not work on 5.3.
- [drm/i915/guc: Don't allow GuC submission · torvalds/linux@a2904ad](https://github.com/torvalds/linux/commit/a2904ade3dc28cf1a1b7deded41f4369f75e664c)
- [drm/i915/guc: Updates for GuC 32.0.3 firmware · torvalds/linux@ffd5ce2](https://github.com/torvalds/linux/commit/ffd5ce22faa4d07a07085b497717d7650f72fd5f)

### Path changes
- `drivers/gpu/drm/i915/i915_gem_context.c` to `drivers/gpu/drm/i915/gem/i915_gem_context.c`
- `drivers/gpu/drm/i915/intel_dp.c` to `drivers/gpu/drm/i915/display/intel_dp.c`
- `drivers/gpu/drm/i915/intel_lrc.c` to `drivers/gpu/drm/i915/gt/intel_lrc.c`
- `drivers/gpu/drm/i915/intel_lrc.h` to `drivers/gpu/drm/i915/gt/intel_lrc.h`
- `drivers/gpu/drm/i915/intel_panel.c` to `drivers/gpu/drm/i915/display/intel_panel.c`

- [drm/i915: Move more GEM objects under gem/ · torvalds/linux@10be98a](https://github.com/torvalds/linux/commit/10be98a77c558f8cfb823cd2777171fbb35040f6)
- [drm/i915: move modesetting output/encoder code under display/ · torvalds/linux@379bc10](https://github.com/torvalds/linux/commit/379bc100232acd45b19421bd0748f9f549da8a8a)
- [drm/i915: Move GraphicsTechnology files under gt/ · torvalds/linux@112ed2d](https://github.com/torvalds/linux/commit/112ed2d31a46f4704085ad925435b77e62b8abee)

### intel_execlists_submission_setup

5.2
```diff
int logical_render_ring_init(struct intel_engine_cs *engine)
{
	int ret;

	ret = logical_ring_setup(engine);
	if (ret)
		return ret;

	/* Override some for render ring. */
	engine->init_context = gen8_init_rcs_context;
	engine->emit_flush = gen8_emit_flush_render;
	engine->emit_fini_breadcrumb = gen8_emit_fini_breadcrumb_rcs;

+	engine->irq_keep_mask |= GT_RENDER_PIPECTL_NOTIFY_INTERRUPT
+				<< GEN8_RCS_IRQ_SHIFT;

	ret = logical_ring_init(engine);
	if (ret)
		return ret;

	ret = intel_init_workaround_bb(engine);
	if (ret) {
		/*
		 * We continue even if we fail to initialize WA batch
		 * because we only expect rare glitches but nothing
		 * critical to prevent us from using GPU
		 */
		DRM_ERROR("WA batch buffer initialization failed: %d\n",
			  ret);
	}

	intel_engine_init_whitelist(engine);

	return 0;
}
```

5.3
```diff
int intel_execlists_submission_setup(struct intel_engine_cs *engine)
{
	/* Intentionally left blank. */
	engine->buffer = NULL;

	tasklet_init(&engine->execlists.tasklet,
		     execlists_submission_tasklet, (unsigned long)engine);

	logical_ring_default_vfuncs(engine);
	logical_ring_default_irqs(engine);

	if (engine->class == RENDER_CLASS) {
		engine->init_context = gen8_init_rcs_context;
		engine->emit_flush = gen8_emit_flush_render;
		engine->emit_fini_breadcrumb = gen8_emit_fini_breadcrumb_rcs;

+		engine->irq_keep_mask |= GT_RENDER_PIPECTL_NOTIFY_INTERRUPT
+					<< GEN8_RCS_IRQ_SHIFT;
	}

	return 0;
}
```

### drivers/gpu/drm/i915/intel_ipts.c:89:12: error: implicit declaration of function ‘i915_gem_object_create’; did you mean ‘i915_gem_object_free’? [-Werror=implicit-function-declaration]
- [drm/i915: Move shmem object setup to its own file · torvalds/linux@8475355](https://github.com/torvalds/linux/commit/8475355f7a2645a022288301c03555c31fb4de17#diff-b33e15b3f5b3b0ee4d341dfba50afe04)
- [drm/i915: Move GEM domain management to its own file · torvalds/linux@f0e4a06](https://github.com/torvalds/linux/commit/f0e4a06397526d3352a3c80b0575ac22ab24da94)

- `i915_gem_object_create()` to `i915_gem_object_create_shmem()`

### drivers/gpu/drm/i915/intel_ipts.c:139:14: error: ‘struct i915_gem_context’ has no member named ‘ppgtt’
- [drm/i915: Pull kref into i915_address_space · torvalds/linux@e568ac3](https://github.com/torvalds/linux/commit/e568ac3874be7dcef3da0cc3bd6b91ca9dd14aa0)

- `ppgtt` to `vm`
- `&ppgtt->vm` to `vm`

### drivers/gpu/drm/i915/intel_ipts.c:195:7: error: implicit declaration of function ‘intel_context_pin’; did you mean ‘intel_ring_pin’? [-Werror=implicit-function-declaration]

- [drm/i915: Export intel_context_instance() · torvalds/linux@fa9f668](https://github.com/torvalds/linux/commit/fa9f668141f4e5590837845ffc1dc4f5aca7a0a5)
- [drm/i915: Pass intel_context to intel_context_pin_lock() · torvalds/linux@6b736de](https://github.com/torvalds/linux/commit/6b736de5746a304692fc5f7f5fc46cd9c2e8bd29#diff-a6faf8f8247554f2549043d630f7d051)

- `intel_context_pin()` to `intel_context_create()` (?)
- Add `#include "gt/intel_context.h"`

### __intel_context_do_pin
- [drm/i915: Export intel_context_instance() · torvalds/linux@fa9f668](https://github.com/torvalds/linux/commit/fa9f668141f4e5590837845ffc1dc4f5aca7a0a5)

### drivers/gpu/drm/i915/intel_ipts.c:229:3: error: implicit declaration of function ‘i915_gem_context_put’; did you mean ‘i915_gem_object_put’? [-Werror=implicit-function-declaration]
Add `#include "gem/i915_gem_context.h"`

### drivers/gpu/drm/i915/intel_ipts.c:247:7: error: implicit declaration of function ‘intel_context_lookup’; did you mean ‘intel_context_put’? [-Werror=implicit-function-declaration]

- [drm/i915: Switch back to an array of logical per-engine HW contexts · torvalds/linux@5e2a041](https://github.com/torvalds/linux/commit/5e2a0419ef7cb25d0f9a5fd6a62372bb47ce948d#diff-a6faf8f8247554f2549043d630f7d051)

- `intel_context_lookup()` to `intel_context_create()` (?)



### Complete changes

ipts-5.3rc: Resolved .rej files
```diff
From 3f864dcf03f11247a8cf9f80d76b2f1859797dc9 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Fri, 2 Aug 2019 06:45:34 +0900
Subject: [PATCH 05/12] Resolved .rej files for ipts-5.2

---
 drivers/gpu/drm/i915/gt/intel_lrc.c | 3 +++
 drivers/gpu/drm/i915/gt/intel_lrc.h | 5 +++++
 drivers/gpu/drm/i915/i915_debugfs.c | 1 +
 drivers/gpu/drm/i915/i915_drv.c     | 1 +
 drivers/gpu/drm/i915/i915_irq.c     | 1 +
 5 files changed, 11 insertions(+)

diff --git a/drivers/gpu/drm/i915/gt/intel_lrc.c b/drivers/gpu/drm/i915/gt/intel_lrc.c
index 04645e5c6..59724a70e 100644
--- a/drivers/gpu/drm/i915/gt/intel_lrc.c
+++ b/drivers/gpu/drm/i915/gt/intel_lrc.c
@@ -2686,6 +2686,9 @@ int intel_execlists_submission_setup(struct intel_engine_cs *engine)
 		engine->init_context = gen8_init_rcs_context;
 		engine->emit_flush = gen8_emit_flush_render;
 		engine->emit_fini_breadcrumb = gen8_emit_fini_breadcrumb_rcs;
+
+		engine->irq_keep_mask |= GT_RENDER_PIPECTL_NOTIFY_INTERRUPT
+					<< GEN8_RCS_IRQ_SHIFT;
 	}
 
 	return 0;
diff --git a/drivers/gpu/drm/i915/gt/intel_lrc.h b/drivers/gpu/drm/i915/gt/intel_lrc.h
index c2bba82bc..7e764797f 100644
--- a/drivers/gpu/drm/i915/gt/intel_lrc.h
+++ b/drivers/gpu/drm/i915/gt/intel_lrc.h
@@ -131,4 +131,9 @@ int intel_virtual_engine_attach_bond(struct intel_engine_cs *engine,
 				     const struct intel_engine_cs *master,
 				     const struct intel_engine_cs *sibling);
 
+int execlists_context_pin(struct intel_context *ce);
+void execlists_context_unpin(struct intel_context *ce);
+int execlists_context_deferred_alloc(struct intel_context *ce,
+				     struct intel_engine_cs *engine);
+
 #endif /* _INTEL_LRC_H_ */
diff --git a/drivers/gpu/drm/i915/i915_debugfs.c b/drivers/gpu/drm/i915/i915_debugfs.c
index 520357eee..19970a0b9 100644
--- a/drivers/gpu/drm/i915/i915_debugfs.c
+++ b/drivers/gpu/drm/i915/i915_debugfs.c
@@ -48,6 +48,7 @@
 #include "intel_guc_submission.h"
 #include "intel_pm.h"
 #include "intel_sideband.h"
+#include "intel_ipts.h"
 
 static inline struct drm_i915_private *node_to_i915(struct drm_info_node *node)
 {
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index 8c5c1535e..b0e3b0f9d 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -76,6 +76,7 @@
 #include "intel_drv.h"
 #include "intel_pm.h"
 #include "intel_uc.h"
+#include "intel_ipts.h"
 
 static struct drm_driver driver;
 
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
From a92b4a2c0191ad419215ad964e1f9c2ad7b101da Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Fri, 2 Aug 2019 06:42:02 +0900
Subject: [PATCH 05/12] Resolved build time errors for ipts-5.2

---
 drivers/gpu/drm/i915/intel_ipts.c | 17 ++++++++++-------
 1 file changed, 10 insertions(+), 7 deletions(-)

diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index 3d3c35398..9fb92c760 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -27,6 +27,9 @@
 #include <linux/intel_ipts_if.h>
 #include <drm/drmP.h>
 
+#include "gt/intel_context.h"
+#include "gem/i915_gem_context.h"
+
 #include "intel_guc_submission.h"
 #include "i915_drv.h"
 
@@ -86,7 +89,7 @@ static intel_ipts_object_t *ipts_object_create(size_t size, u32 flags)
 	}
 
 	/* Allocate the new object */
-	gem_obj = i915_gem_object_create(dev_priv, size);
+	gem_obj = i915_gem_object_create_shmem(dev_priv, size);
 	if (gem_obj == NULL) {
 		ret = -ENOMEM;
 		goto err_out;
@@ -136,8 +139,8 @@ static int ipts_object_pin(intel_ipts_object_t* obj,
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
 	int ret = 0;
 
-	if (ipts_ctx->ppgtt) {
-		vm = &ipts_ctx->ppgtt->vm;
+	if (ipts_ctx->vm) {
+		vm = ipts_ctx->vm;
 	} else {
 		vm = &dev_priv->ggtt.vm;
 	}
@@ -192,7 +195,7 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ce = intel_context_pin(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
 	if (IS_ERR(ce)) {
 		DRM_ERROR("Failed to create intel context (error %ld)\n",
 			  PTR_ERR(ce));
@@ -241,7 +244,7 @@ static void destroy_ipts_context(void)
 
 	ipts_ctx = intel_ipts.ipts_context;
 
-	ce = intel_context_lookup(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
 
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
@@ -452,8 +455,8 @@ static int intel_ipts_map_buffer(u64 gfx_handle, intel_ipts_mapbuffer_t *mapbuf)
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

ipts5.3-rc: update by qzed
```diff
From f0d2fa6a996e7f65ea3ac86d87943f9bc40de0a6 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Tue, 6 Aug 2019 08:49:41 +0900
Subject: [PATCH] ipts5.3-rc: update by qzed

---
 drivers/gpu/drm/i915/intel_ipts.c | 22 ++++++++++++++++++----
 1 file changed, 18 insertions(+), 4 deletions(-)

diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index 5ca04106e..b9f788dae 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -195,12 +195,19 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = i915_gem_context_get_engine(ipts_ctx, RCS0);
 	if (IS_ERR(ce)) {
 		DRM_ERROR("Failed to create intel context (error %ld)\n",
 			  PTR_ERR(ce));
 		ret = PTR_ERR(ce);
-		goto err_unlock;
+		goto err_ctx;
+	}
+
+	ret = intel_context_pin(ce);
+	if (ret) {
+		DRM_ERROR("Failed to pin intel context (error %d)\n", ret);
+		ret = PTR_ERR(ce);
+		goto err_ctx;
 	}
 
 	ret = execlists_context_deferred_alloc(ce, ce->engine);
@@ -226,7 +233,9 @@ static int create_ipts_context(void)
 	return 0;
 
 err_ctx:
-	if (ipts_ctx)
+	if (!IS_ERR_OR_NULL(ce))
+		intel_context_put(ce);
+	if (!IS_ERR_OR_NULL(ipts_ctx))
 		i915_gem_context_put(ipts_ctx);
 
 err_unlock:
@@ -244,7 +253,11 @@ static void destroy_ipts_context(void)
 
 	ipts_ctx = intel_ipts.ipts_context;
 
-	ce = intel_context_create(ipts_ctx, dev_priv->engine[RCS0]);
+	ce = i915_gem_context_lookup_engine(ipts_ctx, RCS0);
+	if (IS_ERR(ce)) {
+		DRM_ERROR("i915_gem_context_lookup_engine failed: %ld\n", PTR_ERR(ce));
+		return;
+	}
 
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
@@ -255,6 +268,7 @@ static void destroy_ipts_context(void)
 
 	execlists_context_unpin(ce);
 	intel_context_unpin(ce);
+	intel_context_put(ce);
 	i915_gem_context_put(ipts_ctx);
 
 	mutex_unlock(&intel_ipts.dev->struct_mutex);
-- 
2.22.0


```



## 0007-sdcard-reader.patch changes

`USB_STATE_DEFAULT` to `USB_STATE_CONFIGURED`

- [usb: core: hub: Enable/disable U1/U2 in configured state · torvalds/linux@fea3af5](https://github.com/torvalds/linux/commit/fea3af5e0358b014c653561109c11ebd3ecdbff2)



## 0008-wifi.patch changes

- [mwifiex: don't disable hardirqs; just softirqs · torvalds/linux@8a7f9fd](https://github.com/torvalds/linux/commit/8a7f9fd8a3e09c829c9fc2a86fe2d370ebcafd95#diff-bd00101a648dddd22d709b04ec311460)

Resolved .rej files for wifi-5.2
```diff
From f128a72f283df3a126efa7c119377babd23cfc25 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Wed, 7 Aug 2019 16:13:14 +0900
Subject: [PATCH 2/2] Resolved .rej files for wifi-5.2

---
 drivers/net/wireless/marvell/mwifiex/main.c | 10 +++++++++-
 1 file changed, 9 insertions(+), 1 deletion(-)

diff --git a/drivers/net/wireless/marvell/mwifiex/main.c b/drivers/net/wireless/marvell/mwifiex/main.c
index d25113557..ba99d84a3 100644
--- a/drivers/net/wireless/marvell/mwifiex/main.c
+++ b/drivers/net/wireless/marvell/mwifiex/main.c
@@ -172,16 +172,18 @@ void mwifiex_queue_main_work(struct mwifiex_adapter *adapter)
 }
 EXPORT_SYMBOL_GPL(mwifiex_queue_main_work);
 
-static void mwifiex_queue_rx_work(struct mwifiex_adapter *adapter)
+void mwifiex_queue_rx_work(struct mwifiex_adapter *adapter)
 {
 	spin_lock_bh(&adapter->rx_proc_lock);
 	if (adapter->rx_processing) {
+		adapter->more_rx_task_flag = true;
 		spin_unlock_bh(&adapter->rx_proc_lock);
 	} else {
 		spin_unlock_bh(&adapter->rx_proc_lock);
 		queue_work(adapter->rx_workqueue, &adapter->rx_work);
 	}
 }
+EXPORT_SYMBOL_GPL(mwifiex_queue_rx_work);
 
 static int mwifiex_process_rx(struct mwifiex_adapter *adapter)
 {
@@ -190,6 +192,7 @@ static int mwifiex_process_rx(struct mwifiex_adapter *adapter)
 
 	spin_lock_bh(&adapter->rx_proc_lock);
 	if (adapter->rx_processing || adapter->rx_locked) {
+		adapter->more_rx_task_flag = true;
 		spin_unlock_bh(&adapter->rx_proc_lock);
 		goto exit_rx_proc;
 	} else {
@@ -219,6 +222,11 @@ static int mwifiex_process_rx(struct mwifiex_adapter *adapter)
 		}
 	}
 	spin_lock_bh(&adapter->rx_proc_lock);
+	if (adapter->more_rx_task_flag) {
+		adapter->more_rx_task_flag = false;
+		spin_unlock_bh(&adapter->rx_proc_lock);
+		goto rx_process_start;
+	}
 	adapter->rx_processing = false;
 	spin_unlock_bh(&adapter->rx_proc_lock);
 
-- 
2.22.0


```