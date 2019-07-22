# README

From [jakeday/linux-surface: Linux Kernel for Surface Devices](https://github.com/jakeday/linux-surface) commit 6bbdaf3679e2c1be7eb1fcf8362c5e53741c468c, which is the last commit with support 4.14 to 4.17.

## IPTS changes from 4.14 to 4.15

```diff
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index c8a254753..94bce131e 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -692,7 +692,7 @@ static int i915_load_modeset_init(struct drm_device *dev)
 
 	drm_kms_helper_poll_init(dev);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
         intel_ipts_init(dev);
 
 	return 0;
@@ -1398,7 +1398,7 @@ void i915_driver_unload(struct drm_device *dev)
 	struct drm_i915_private *dev_priv = to_i915(dev);
 	struct pci_dev *pdev = dev_priv->drm.pdev;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
 		intel_ipts_cleanup(dev);
 
 	i915_driver_unregister(dev_priv);
diff --git a/drivers/gpu/drm/i915/i915_params.c b/drivers/gpu/drm/i915/i915_params.c
index a1b57e65d..a906694db 100644
--- a/drivers/gpu/drm/i915/i915_params.c
+++ b/drivers/gpu/drm/i915/i915_params.c
@@ -164,11 +164,11 @@ i915_param_named_unsafe(edp_vswing, int, 0400,
 
 i915_param_named_unsafe(enable_guc_loading, int, 0400,
 	"Enable GuC firmware loading "
-	"(-1=auto, 0=never [default], 1=if available, 2=required)");
+	"(-1=auto, 0=never, 1=if available [default], 2=required)");
 
 i915_param_named_unsafe(enable_guc_submission, int, 0400,
 	"Enable GuC submission "
-	"(-1=auto, 0=never [default], 1=if available, 2=required)");
+	"(-1=auto, 0=never, 1=if available [default], 2=required)");
 
 i915_param_named(guc_log_level, int, 0400,
 	"GuC firmware logging level (-1:disabled (default), 0-3:enabled)");
diff --git a/drivers/gpu/drm/i915/i915_params.h b/drivers/gpu/drm/i915/i915_params.h
index c7292268e..23366950d 100644
--- a/drivers/gpu/drm/i915/i915_params.h
+++ b/drivers/gpu/drm/i915/i915_params.h
@@ -39,13 +39,13 @@
 	param(int, enable_dc, -1) \
 	param(int, enable_fbc, -1) \
 	param(int, enable_ppgtt, -1) \
-	param(int, enable_execlists, -1) \
+	param(int, enable_execlists, 0) \
 	param(int, enable_psr, -1) \
 	param(int, disable_power_well, -1) \
 	param(int, enable_ips, 1) \
 	param(int, invert_brightness, 0) \
-	param(int, enable_guc_loading, 0) \
-	param(int, enable_guc_submission, 0) \
+	param(int, enable_guc_loading, 1) \
+	param(int, enable_guc_submission, 1) \
 	param(int, guc_log_level, -1) \
 	param(char *, guc_firmware_path, NULL) \
 	param(char *, huc_firmware_path, NULL) \
diff --git a/drivers/gpu/drm/i915/intel_guc.h b/drivers/gpu/drm/i915/intel_guc.h
index 418450b1a..c1bd36c10 100644
--- a/drivers/gpu/drm/i915/intel_guc.h
+++ b/drivers/gpu/drm/i915/intel_guc.h
@@ -56,6 +56,7 @@ struct intel_guc {
 	struct ida stage_ids;
 
 	struct i915_guc_client *execbuf_client;
+	struct i915_guc_client *ipts_client;
 
 	DECLARE_BITMAP(doorbell_bitmap, GUC_NUM_DOORBELLS);
 	/* Cyclic counter mod pagesize	*/
@@ -116,5 +117,9 @@ int intel_guc_suspend(struct drm_i915_private *dev_priv);
 int intel_guc_resume(struct drm_i915_private *dev_priv);
 struct i915_vma *intel_guc_allocate_vma(struct intel_guc *guc, u32 size);
 u32 intel_guc_wopcm_size(struct drm_i915_private *dev_priv);
+int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
+				    struct i915_gem_context *ctx);
+void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv);
+void i915_guc_ipts_reacquire_doorbell(struct drm_i915_private *dev_priv);
 
 #endif
diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index f5fa1118b..be9fa6fc3 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -27,6 +27,7 @@
 #include <linux/intel_ipts_if.h>
 #include <drm/drmP.h>
 
+#include "i915_guc_submission.h"
 #include "i915_drv.h"
 
 #define SUPPORTED_IPTS_INTERFACE_VERSION	1
@@ -328,7 +329,7 @@ static int set_wq_info(void)
 	base = client->vaddr;
 	desc = (struct guc_process_desc *)((u64)base + client->proc_desc_offset);
 
-	desc->wq_base_addr = (u64)base + client->wq_offset;
+	desc->wq_base_addr = (u64)base + GUC_DB_SIZE;
 	desc->db_base_addr = (u64)base + client->doorbell_offset;
 
 	/* IPTS expects physical addresses to pass it to ME */
@@ -338,7 +339,7 @@ static int set_wq_info(void)
         wq_info->db_phy_addr = phy_base + client->doorbell_offset;
         wq_info->db_cookie_offset = offsetof(struct guc_doorbell_info, cookie);
         wq_info->wq_addr = desc->wq_base_addr;
-        wq_info->wq_phy_addr = phy_base + client->wq_offset;
+        wq_info->wq_phy_addr = phy_base + GUC_DB_SIZE;
         wq_info->wq_head_addr = (u64)&desc->head;
         wq_info->wq_head_phy_addr = phy_base + client->proc_desc_offset +
 					offsetof(struct guc_process_desc, head);
diff --git a/drivers/gpu/drm/i915/intel_panel.c b/drivers/gpu/drm/i915/intel_panel.c
index c23c8c167..a3294c46a 100644
--- a/drivers/gpu/drm/i915/intel_panel.c
+++ b/drivers/gpu/drm/i915/intel_panel.c
@@ -720,7 +720,7 @@ static void lpt_disable_backlight(const struct drm_connector_state *old_conn_sta
 	struct drm_i915_private *dev_priv = to_i915(connector->base.dev);
 	u32 tmp;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
 		intel_ipts_notify_backlight_status(false);
 
 	intel_panel_actually_set_backlight(old_conn_state, 0);
@@ -911,7 +911,7 @@ static void lpt_enable_backlight(const struct intel_crtc_state *crtc_state,
 	/* This won't stick until the above enable. */
 	intel_panel_actually_set_backlight(conn_state, panel->backlight.level);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915.enable_guc_submission)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc_submission)
 		intel_ipts_notify_backlight_status(true);
 }
 
diff --git a/drivers/gpu/drm/i915/intel_uc.h b/drivers/gpu/drm/i915/intel_uc.h
index 088a35735..e18d3bb02 100644
--- a/drivers/gpu/drm/i915/intel_uc.h
+++ b/drivers/gpu/drm/i915/intel_uc.h
@@ -35,9 +35,4 @@ void intel_uc_fini_fw(struct drm_i915_private *dev_priv);
 int intel_uc_init_hw(struct drm_i915_private *dev_priv);
 void intel_uc_fini_hw(struct drm_i915_private *dev_priv);
 
-int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
-				    struct i915_gem_context *ctx);
-void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv);
-void i915_guc_ipts_reacquire_doorbell(struct drm_i915_private *dev_priv);
-
 #endif

```