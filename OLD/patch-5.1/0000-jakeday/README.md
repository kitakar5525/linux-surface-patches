# IPTS patch changes from 5.0 to 5.1

```bash
git checkout -b ipts-5.0-to-5.1
patch -Np1 -i ipts-5.0
rm $(find . -name "*.rej")
git commit -A && git commit -m "Applied ipts-5.0"
git checkout -b ipts-5.1
git am ipts-5.1
git diff ipts-5.0-to-5.1 ipts-5.1
```

Resolved .rej files and/or build time errors and/or runtime errors
```diff
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index 5cd6c967d..40f24ebf8 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -713,7 +713,7 @@ static int i915_load_modeset_init(struct drm_device *dev)
 	intel_init_ipc(dev_priv);
 
 	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
-        intel_ipts_init(dev);
+		intel_ipts_init(dev);
 
 	return 0;
 
@@ -2091,7 +2091,7 @@ static int i915_drm_resume(struct drm_device *dev)
 	enable_rpm_wakeref_asserts(dev_priv);
 
 	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
-        intel_ipts_resume(dev);
+		intel_ipts_resume(dev);
 
 	return 0;
 }
diff --git a/drivers/gpu/drm/i915/i915_irq.c b/drivers/gpu/drm/i915/i915_irq.c
index f397a4838..ad066f43c 100644
--- a/drivers/gpu/drm/i915/i915_irq.c
+++ b/drivers/gpu/drm/i915/i915_irq.c
@@ -3883,7 +3883,8 @@ static void gen8_gt_irq_postinstall(struct drm_i915_private *dev_priv)
 {
 	/* These are interrupts we'll toggle with the ring mask register */
 	u32 gt_interrupts[] = {
-		GT_RENDER_USER_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
+		GT_RENDER_PIPECTL_NOTIFY_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
+			GT_RENDER_USER_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
 			GT_CONTEXT_SWITCH_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
 			GT_RENDER_USER_INTERRUPT << GEN8_BCS_IRQ_SHIFT |
 			GT_CONTEXT_SWITCH_INTERRUPT << GEN8_BCS_IRQ_SHIFT,
diff --git a/drivers/gpu/drm/i915/intel_lrc.c b/drivers/gpu/drm/i915/intel_lrc.c
index 4cba45461..523bc6354 100644
--- a/drivers/gpu/drm/i915/intel_lrc.c
+++ b/drivers/gpu/drm/i915/intel_lrc.c
@@ -2396,6 +2396,9 @@ int logical_render_ring_init(struct intel_engine_cs *engine)
 	engine->emit_flush = gen8_emit_flush_render;
 	engine->emit_fini_breadcrumb = gen8_emit_fini_breadcrumb_rcs;
 
+	engine->irq_keep_mask |= GT_RENDER_PIPECTL_NOTIFY_INTERRUPT
+				<< GEN8_RCS_IRQ_SHIFT;
+
 	ret = logical_ring_init(engine);
 	if (ret)
 		return ret;
diff --git a/drivers/gpu/drm/i915/intel_lrc.h b/drivers/gpu/drm/i915/intel_lrc.h
index 84df5627f..faa403f13 100644
--- a/drivers/gpu/drm/i915/intel_lrc.h
+++ b/drivers/gpu/drm/i915/intel_lrc.h
@@ -112,14 +112,15 @@ void intel_execlists_show_requests(struct intel_engine_cs *engine,
 							const char *prefix),
 				   unsigned int max);
 
-u32 gen8_make_rpcs(struct drm_i915_private *i915, struct intel_sseu *ctx_sseu);
-
 struct intel_context *
 execlists_context_pin(struct intel_engine_cs *engine,
 		      struct i915_gem_context *ctx);
 void execlists_context_unpin(struct intel_context *ce);
 int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
-					    struct intel_engine_cs *engine,
-						struct intel_context *ce);
+				     struct intel_engine_cs *engine,
+				     struct intel_context *ce);
+
+
+u32 gen8_make_rpcs(struct drm_i915_private *i915, struct intel_sseu *ctx_sseu);
 
 #endif /* _INTEL_LRC_H_ */
```