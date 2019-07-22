# README

From [jakeday/linux-surface: Linux Kernel for Surface Devices](https://github.com/jakeday/linux-surface) commit 6bbdaf3679e2c1be7eb1fcf8362c5e53741c468c, which is the last commit with support 4.14 to 4.17.

# IPTS changes from 4.15 to 4.16

```diff
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index abcadfcf4..86c0f6f54 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -696,7 +696,7 @@ static int i915_load_modeset_init(struct drm_device *dev)
 	/* Only enable hotplug handling once the fbdev is fully set up. */
 	intel_hpd_init(dev_priv);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
         intel_ipts_init(dev);
 
 	return 0;
@@ -1424,7 +1424,7 @@ void i915_driver_unload(struct drm_device *dev)
 	struct drm_i915_private *dev_priv = to_i915(dev);
 	struct pci_dev *pdev = dev_priv->drm.pdev;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
 		intel_ipts_cleanup(dev);
 
 	i915_driver_unregister(dev_priv);
diff --git a/drivers/gpu/drm/i915/i915_params.c b/drivers/gpu/drm/i915/i915_params.c
index b5f3eb4fa..a8f2b07d0 100644
--- a/drivers/gpu/drm/i915/i915_params.c
+++ b/drivers/gpu/drm/i915/i915_params.c
@@ -152,7 +152,7 @@ i915_param_named_unsafe(edp_vswing, int, 0400,
 i915_param_named_unsafe(enable_guc, int, 0400,
 	"Enable GuC load for GuC submission and/or HuC load. "
 	"Required functionality can be selected using bitmask values. "
-	"(-1=auto, 0=disable [default], 1=GuC submission, 2=HuC load)");
+	"(-1=auto, 0=disable, 1=GuC submission [default], 2=HuC load)");
 
 i915_param_named(guc_log_level, int, 0400,
 	"GuC firmware logging level (-1:disabled (default), 0-3:enabled)");
diff --git a/drivers/gpu/drm/i915/i915_params.h b/drivers/gpu/drm/i915/i915_params.h
index c96360398..f8f63f023 100644
--- a/drivers/gpu/drm/i915/i915_params.h
+++ b/drivers/gpu/drm/i915/i915_params.h
@@ -47,7 +47,7 @@ struct drm_printer;
 	param(int, disable_power_well, -1) \
 	param(int, enable_ips, 1) \
 	param(int, invert_brightness, 0) \
-	param(int, enable_guc, 0) \
+	param(int, enable_guc, 1) \
 	param(int, guc_log_level, -1) \
 	param(char *, guc_firmware_path, NULL) \
 	param(char *, huc_firmware_path, NULL) \
diff --git a/drivers/gpu/drm/i915/intel_dp.c b/drivers/gpu/drm/i915/intel_dp.c
index 79521da5d..fcee78eb4 100644
--- a/drivers/gpu/drm/i915/intel_dp.c
+++ b/drivers/gpu/drm/i915/intel_dp.c
@@ -2484,8 +2484,8 @@ void intel_dp_sink_dpms(struct intel_dp *intel_dp, int mode)
 		return;
 
 	if (mode != DRM_MODE_DPMS_ON) {
-		if (downstream_hpd_needs_d0(intel_dp))
-			return;
+		//if (downstream_hpd_needs_d0(intel_dp))
+		//	return;
 
 		ret = drm_dp_dpcd_writeb(&intel_dp->aux, DP_SET_POWER,
 					 DP_SET_POWER_D3);
diff --git a/drivers/gpu/drm/i915/intel_guc.h b/drivers/gpu/drm/i915/intel_guc.h
index b973d1b1e..dbc05a22e 100644
--- a/drivers/gpu/drm/i915/intel_guc.h
+++ b/drivers/gpu/drm/i915/intel_guc.h
@@ -64,6 +64,7 @@ struct intel_guc {
 
 	struct intel_guc_client *execbuf_client;
 	struct intel_guc_client *preempt_client;
+	struct intel_guc_client *ipts_client;
 
 	struct guc_preempt_work preempt_work[I915_NUM_ENGINES];
 	struct workqueue_struct *preempt_wq;
@@ -131,9 +132,5 @@ int intel_guc_suspend(struct drm_i915_private *dev_priv);
 int intel_guc_resume(struct drm_i915_private *dev_priv);
 struct i915_vma *intel_guc_allocate_vma(struct intel_guc *guc, u32 size);
 u32 intel_guc_wopcm_size(struct drm_i915_private *dev_priv);
-int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
-				    struct i915_gem_context *ctx);
-void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv);
-void i915_guc_ipts_reacquire_doorbell(struct drm_i915_private *dev_priv);
 
 #endif
diff --git a/drivers/gpu/drm/i915/intel_guc_submission.c b/drivers/gpu/drm/i915/intel_guc_submission.c
index 4d2409466..870ab4d6c 100644
--- a/drivers/gpu/drm/i915/intel_guc_submission.c
+++ b/drivers/gpu/drm/i915/intel_guc_submission.c
@@ -90,6 +90,7 @@ static inline bool is_high_priority(struct intel_guc_client *client)
 
 static int reserve_doorbell(struct intel_guc_client *client)
 {
+	struct drm_i915_private *dev_priv = guc_to_i915(client->guc);
 	unsigned long offset;
 	unsigned long end;
 	u16 id;
@@ -102,11 +103,16 @@ static int reserve_doorbell(struct intel_guc_client *client)
 	 * priority contexts, the second half for high-priority ones.
 	 */
 	offset = 0;
-	end = GUC_NUM_DOORBELLS / 2;
-	if (is_high_priority(client)) {
-		offset = end;
-		end += offset;
+	if (IS_SKYLAKE(dev_priv) || IS_KABYLAKE(dev_priv)) {
+		end = GUC_NUM_DOORBELLS;
 	}
+	else {
+		end = GUC_NUM_DOORBELLS/2;
+		if (is_high_priority(client)) {
+			offset = end;
+			end += offset;
+		}
+ 	}
 
 	id = find_next_zero_bit(client->guc->doorbell_bitmap, end, offset);
 	if (id == end)
@@ -349,8 +355,14 @@ static void guc_stage_desc_init(struct intel_guc *guc,
 	desc = __get_stage_desc(client);
 	memset(desc, 0, sizeof(*desc));
 
-	desc->attribute = GUC_STAGE_DESC_ATTR_ACTIVE |
-			  GUC_STAGE_DESC_ATTR_KERNEL;
+	desc->attribute = GUC_STAGE_DESC_ATTR_ACTIVE;
+	if ((client->priority == GUC_CLIENT_PRIORITY_KMD_NORMAL) ||
+			(client->priority == GUC_CLIENT_PRIORITY_KMD_HIGH)) {
+		desc->attribute  |= GUC_STAGE_DESC_ATTR_KERNEL;
+	} else {
+		desc->attribute  |= GUC_STAGE_DESC_ATTR_PCH;
+	}
+
 	if (is_high_priority(client))
 		desc->attribute |= GUC_STAGE_DESC_ATTR_PREEMPT;
 	desc->stage_id = client->stage_id;
@@ -1206,7 +1218,8 @@ static void guc_interrupts_capture(struct drm_i915_private *dev_priv)
 		I915_WRITE(RING_MODE_GEN7(engine), irqs);
 
 	/* route USER_INTERRUPT to Host, all others are sent to GuC. */
-	irqs = GT_RENDER_USER_INTERRUPT << GEN8_RCS_IRQ_SHIFT |
+	irqs = (GT_RENDER_USER_INTERRUPT | GT_RENDER_PIPECTL_NOTIFY_INTERRUPT)
+							<< GEN8_RCS_IRQ_SHIFT |
 	       GT_RENDER_USER_INTERRUPT << GEN8_BCS_IRQ_SHIFT;
 	/* These three registers have the same bit definitions */
 	I915_WRITE(GUC_BCS_RCS_IER, ~irqs);
@@ -1334,6 +1347,58 @@ void intel_guc_submission_disable(struct intel_guc *guc)
 	intel_engines_reset_default_submission(dev_priv);
 }
 
+int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
+				    struct i915_gem_context *ctx)
+{
+	struct intel_guc *guc = &dev_priv->guc;
+	struct intel_guc_client *client;
+	int err;
+	int ret;
+
+	/* client for execbuf submission */
+	client = guc_client_alloc(dev_priv,
+				  INTEL_INFO(dev_priv)->ring_mask,
+				  GUC_CLIENT_PRIORITY_NORMAL,
+				  ctx);
+	if (IS_ERR(client)) {
+		DRM_ERROR("Failed to create normal GuC client!\n");
+		return -ENOMEM;
+	}
+
+	guc->ipts_client = client;
+
+	err = intel_guc_sample_forcewake(guc);
+	if (err)
+		return err;
+
+	ret = create_doorbell(guc->ipts_client);
+	if (ret)
+		return ret;
+
+	return 0;
+}
+
+void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv)
+{
+	struct intel_guc *guc = &dev_priv->guc;
+
+	if (!guc->ipts_client)
+		return;
+
+	guc_client_free(guc->ipts_client);
+	guc->ipts_client = NULL;
+}
+
+void i915_guc_ipts_reacquire_doorbell(struct drm_i915_private *dev_priv)
+{
+	struct intel_guc *guc = &dev_priv->guc;
+
+	int err = __guc_allocate_doorbell(guc, guc->ipts_client->stage_id);
+
+	if (err)
+		DRM_ERROR("Not able to reacquire IPTS doorbell\n");
+}
+
 #if IS_ENABLED(CONFIG_DRM_I915_SELFTEST)
 #include "selftests/intel_guc.c"
 #endif
diff --git a/drivers/gpu/drm/i915/intel_guc_submission.h b/drivers/gpu/drm/i915/intel_guc_submission.h
index fb081cefe..71fc79865 100644
--- a/drivers/gpu/drm/i915/intel_guc_submission.h
+++ b/drivers/gpu/drm/i915/intel_guc_submission.h
@@ -79,5 +79,9 @@ void intel_guc_submission_disable(struct intel_guc *guc);
 void intel_guc_submission_fini(struct intel_guc *guc);
 int intel_guc_preempt_work_create(struct intel_guc *guc);
 void intel_guc_preempt_work_destroy(struct intel_guc *guc);
+int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
+				    struct i915_gem_context *ctx);
+void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv);
+void i915_guc_ipts_reacquire_doorbell(struct drm_i915_private *dev_priv);
 
 #endif
diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index be9fa6fc3..f8cc5eaf0 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -27,7 +27,7 @@
 #include <linux/intel_ipts_if.h>
 #include <drm/drmP.h>
 
-#include "i915_guc_submission.h"
+#include "intel_guc_submission.h"
 #include "i915_drv.h"
 
 #define SUPPORTED_IPTS_INTERFACE_VERSION	1
@@ -312,7 +312,7 @@ static int set_wq_info(void)
 {
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
 	struct intel_guc *guc = &dev_priv->guc;
-	struct i915_guc_client *client;
+	struct intel_guc_client *client;
 	struct guc_process_desc *desc;
 	void *base = NULL;
 	intel_ipts_wq_info_t *wq_info;
diff --git a/drivers/gpu/drm/i915/intel_panel.c b/drivers/gpu/drm/i915/intel_panel.c
index 9e381c64b..f3e97dc87 100644
--- a/drivers/gpu/drm/i915/intel_panel.c
+++ b/drivers/gpu/drm/i915/intel_panel.c
@@ -676,7 +676,7 @@ static void lpt_disable_backlight(const struct drm_connector_state *old_conn_sta
 	struct drm_i915_private *dev_priv = to_i915(connector->base.dev);
 	u32 tmp;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
 		intel_ipts_notify_backlight_status(false);
 
 	intel_panel_actually_set_backlight(old_conn_state, 0);
@@ -867,7 +867,7 @@ static void lpt_enable_backlight(const struct intel_crtc_state *crtc_state,
 	/* This won't stick until the above enable. */
 	intel_panel_actually_set_backlight(conn_state, panel->backlight.level);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
 		intel_ipts_notify_backlight_status(true);
 }
 
```