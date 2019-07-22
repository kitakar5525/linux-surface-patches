# README

From [jakeday/linux-surface: Linux Kernel for Surface Devices](https://github.com/jakeday/linux-surface) commit 6bbdaf3679e2c1be7eb1fcf8362c5e53741c468c, which is the last commit with support 4.14 to 4.17.

## IPTS changes from 4.16 to 4.17

```diff
diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index 803359cd..31968ebd 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -714,7 +714,7 @@ static int i915_load_modeset_init(struct drm_device *dev)
 	/* Only enable hotplug handling once the fbdev is fully set up. */
 	intel_hpd_init(dev_priv);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
         intel_ipts_init(dev);
 
 	return 0;
@@ -1442,7 +1442,7 @@ void i915_driver_unload(struct drm_device *dev)
 	struct drm_i915_private *dev_priv = to_i915(dev);
 	struct pci_dev *pdev = dev_priv->drm.pdev;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
 		intel_ipts_cleanup(dev);
 
 	i915_driver_unregister(dev_priv);
diff --git a/drivers/gpu/drm/i915/i915_params.c b/drivers/gpu/drm/i915/i915_params.c
index e7861fa5..db321ac5 100644
--- a/drivers/gpu/drm/i915/i915_params.c
+++ b/drivers/gpu/drm/i915/i915_params.c
@@ -154,6 +154,9 @@ i915_param_named_unsafe(enable_guc, int, 0400,
 	"Required functionality can be selected using bitmask values. "
 	"(-1=auto, 0=disable, 1=GuC submission [default], 2=HuC load)");
 
+i915_param_named_unsafe(enable_ipts, bool, 0400,
+	"Enable IPTS Touchscreen and Pen support (default: true)");
+
 i915_param_named(guc_log_level, int, 0400,
 	"GuC firmware logging level. Requires GuC to be loaded. "
 	"(-1=auto [default], 0=disable, 1..4=enable with verbosity min..max)");
diff --git a/drivers/gpu/drm/i915/i915_params.h b/drivers/gpu/drm/i915/i915_params.h
index 430f5f9d..2b80220d 100644
--- a/drivers/gpu/drm/i915/i915_params.h
+++ b/drivers/gpu/drm/i915/i915_params.h
@@ -47,7 +47,7 @@ struct drm_printer;
 	param(int, disable_power_well, -1) \
 	param(int, enable_ips, 1) \
 	param(int, invert_brightness, 0) \
-	param(int, enable_guc, 0) \
+	param(int, enable_guc, 1) \
 	param(int, guc_log_level, 0) \
 	param(char *, guc_firmware_path, NULL) \
 	param(char *, huc_firmware_path, NULL) \
@@ -69,7 +69,8 @@ struct drm_printer;
 	param(bool, nuclear_pageflip, false) \
 	param(bool, enable_dp_mst, true) \
 	param(bool, enable_dpcd_backlight, false) \
-	param(bool, enable_gvt, false)
+	param(bool, enable_gvt, false) \
+	param(bool, enable_ipts, true)
 
 #define MEMBER(T, member, ...) T member;
 struct i915_params {
diff --git a/drivers/gpu/drm/i915/intel_panel.c b/drivers/gpu/drm/i915/intel_panel.c
index fb644f21..c0de071d 100644
--- a/drivers/gpu/drm/i915/intel_panel.c
+++ b/drivers/gpu/drm/i915/intel_panel.c
@@ -680,7 +680,7 @@ static void lpt_disable_backlight(const struct drm_connector_state *old_conn_sta
 	struct drm_i915_private *dev_priv = to_i915(connector->base.dev);
 	u32 tmp;
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
 		intel_ipts_notify_backlight_status(false);
 
 	intel_panel_actually_set_backlight(old_conn_state, 0);
@@ -871,7 +871,7 @@ static void lpt_enable_backlight(const struct intel_crtc_state *crtc_state,
 	/* This won't stick until the above enable. */
 	intel_panel_actually_set_backlight(conn_state, panel->backlight.level);
 
-	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc)
+	if (INTEL_GEN(dev_priv) >= 9 && i915_modparams.enable_guc && i915_modparams.enable_ipts)
 		intel_ipts_notify_backlight_status(true);
 }
 
```