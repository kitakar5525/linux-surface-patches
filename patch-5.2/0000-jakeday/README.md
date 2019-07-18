# IPTS patch changes from 5.1 to 5.2

resolved-.rej-files
```diff
From 27dd2fb66f3d8f93479e26a711ea8d0bb781aba4 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Thu, 18 Jul 2019 22:03:36 +0900
Subject: [PATCH 2/3] ipts-5.2: resolved .rej files

---
 drivers/gpu/drm/i915/i915_irq.c    |  4 +++-
 drivers/gpu/drm/i915/intel_lrc.c   |  9 +++------
 drivers/gpu/drm/i915/intel_lrc.h   | 11 +++++------
 drivers/gpu/drm/i915/intel_panel.c |  1 +
 4 files changed, 12 insertions(+), 13 deletions(-)

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
index 247576bf4..131761781 100644
--- a/drivers/gpu/drm/i915/intel_lrc.c
+++ b/drivers/gpu/drm/i915/intel_lrc.c
@@ -166,8 +166,6 @@
 
 #define ACTIVE_PRIORITY (I915_PRIORITY_NOSEMAPHORE)
 
-static int execlists_context_deferred_alloc(struct intel_context *ce,
-					    struct intel_engine_cs *engine);
 static void execlists_init_reg_state(u32 *reg_state,
 				     struct intel_context *ce,
 				     struct intel_engine_cs *engine,
@@ -1235,8 +1233,7 @@ __execlists_update_reg_state(struct intel_context *ce,
 			gen8_make_rpcs(engine->i915, &ce->sseu);
 }
 
-static int
-__execlists_context_pin(struct intel_context *ce,
+int __execlists_context_pin(struct intel_context *ce,
 			struct intel_engine_cs *engine)
 {
 	void *vaddr;
@@ -1285,7 +1282,7 @@ __execlists_context_pin(struct intel_context *ce,
 	return ret;
 }
 
-static int execlists_context_pin(struct intel_context *ce)
+int execlists_context_pin(struct intel_context *ce)
 {
 	return __execlists_context_pin(ce, ce->engine);
 }
@@ -2884,7 +2881,7 @@ static struct i915_timeline *get_timeline(struct i915_gem_context *ctx)
 		return i915_timeline_create(ctx->i915, NULL);
 }
 
-static int execlists_context_deferred_alloc(struct intel_context *ce,
+int execlists_context_deferred_alloc(struct intel_context *ce,
 					    struct intel_engine_cs *engine)
 {
 	struct drm_i915_gem_object *ctx_obj;
diff --git a/drivers/gpu/drm/i915/intel_lrc.h b/drivers/gpu/drm/i915/intel_lrc.h
index 4297cfff2..450d780d0 100644
--- a/drivers/gpu/drm/i915/intel_lrc.h
+++ b/drivers/gpu/drm/i915/intel_lrc.h
@@ -115,13 +115,12 @@ void intel_execlists_show_requests(struct intel_engine_cs *engine,
 							const char *prefix),
 				   unsigned int max);
 
-struct intel_context *
-execlists_context_pin(struct intel_engine_cs *engine,
-		      struct i915_gem_context *ctx);
+int __execlists_context_pin(struct intel_context *ce,
+			struct intel_engine_cs *engine);
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

resolved-build-time-errors
```diff
From 80b98ca5e1c5f02ce393f933dca30621c6c4ef13 Mon Sep 17 00:00:00 2001
From: kitakar5525 <34676735+kitakar5525@users.noreply.github.com>
Date: Thu, 18 Jul 2019 22:16:13 +0900
Subject: [PATCH 3/3] ipts-5.2: resolved build time errors

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
index b276a2f78..97219a8cf 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -175,7 +175,7 @@ static int create_ipts_context(void)
 	struct i915_gem_context *ipts_ctx = NULL;
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
 	struct intel_context *ce = NULL;
-	struct intel_context *pin_ret;
+	int pin_ret;
 	int ret = 0;
 
 	/* Initialize the context right away.*/
@@ -193,7 +193,7 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+	ce = intel_context_pin_lock(ipts_ctx, dev_priv->engine[RCS0]);
 	if (IS_ERR(ce)) {
 		DRM_ERROR("Failed to create intel context (error %ld)\n",
 			  PTR_ERR(ce));
@@ -201,15 +201,15 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ret = execlists_context_deferred_alloc(ipts_ctx, dev_priv->engine[RCS], ce);
+	ret = execlists_context_deferred_alloc(ce, dev_priv->engine[RCS0]);
 	if (ret) {
 		DRM_DEBUG("lr context allocation failed : %d\n", ret);
 		goto err_ctx;
 	}
 
-	pin_ret = execlists_context_pin(dev_priv->engine[RCS], ipts_ctx);
-	if (IS_ERR(pin_ret)) {
-		DRM_DEBUG("lr context pinning failed :  %ld\n", PTR_ERR(pin_ret));
+	pin_ret = __execlists_context_pin(ce, dev_priv->engine[RCS0]);
+	if (pin_ret) {
+		DRM_DEBUG("lr context pinning failed :  %d\n", pin_ret);
 		goto err_ctx;
 	}
 
@@ -242,7 +242,7 @@ static void destroy_ipts_context(void)
 
 	ipts_ctx = intel_ipts.ipts_context;
 
-	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+	ce = intel_context_pin_lock(ipts_ctx, dev_priv->engine[RCS0]);
 
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
-- 
2.22.0


```