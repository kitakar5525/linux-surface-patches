# Changes from 4.19 to 5.0

See
- [Porting patches to Linux 5.0 (surface-acpi, ipts) · Issue #417 · jakeday/linux-surface](https://github.com/jakeday/linux-surface/issues/417)

## IPTS changes

### Major changes

Use `__guc_client_enable()` instead of `create_doorbell()`

>Apparently (I mean, after blindly digging around i915 for 2 days) this problem was introduced by commit [torvalds/linux@48b426a](https://github.com/torvalds/linux/commit/48b426a9b9ab93481a2c5b913c2c6add5fb1001), which moves the initialization of GuC descriptors out of the initial allocation. This can be fixed by simply calling the new __guc_client_enable function instead of create_doorbell when initializing IPTS. The new function takes care of everything for us automatically.

- https://github.com/jakeday/linux-surface/issues/417#issuecomment-471146629

### Complete changes

Resolved .rej files and build time errors
```diff
From 67c8f227097c2d16ba0e01d5f49608c0231e520e Mon Sep 17 00:00:00 2001
From: "Hayataka@kitakar5525" <34676735+kitakar5525@users.noreply.github.com>
Date: Tue, 5 Mar 2019 15:33:12 +0900
Subject: [PATCH] ipts-419to50

---
 drivers/gpu/drm/i915/i915_drv.c | 1 +
 drivers/hid/hid-multitouch.c    | 3 ++-
 2 files changed, 3 insertions(+), 1 deletion(-)

diff --git a/drivers/gpu/drm/i915/i915_drv.c b/drivers/gpu/drm/i915/i915_drv.c
index 0b55872c8..eb095be71 100644
--- a/drivers/gpu/drm/i915/i915_drv.c
+++ b/drivers/gpu/drm/i915/i915_drv.c
@@ -53,6 +53,7 @@
 #include "i915_vgpu.h"
 #include "intel_drv.h"
 #include "intel_uc.h"
+#include "intel_ipts.h"
 #include "intel_workarounds.h"
 
 static struct drm_driver driver;
diff --git a/drivers/hid/hid-multitouch.c b/drivers/hid/hid-multitouch.c
index 2af3d9dbe..719a0360e 100644
--- a/drivers/hid/hid-multitouch.c
+++ b/drivers/hid/hid-multitouch.c
@@ -1562,12 +1562,13 @@ static int mt_input_configured(struct hid_device *hdev, struct hid_input *hi)
         /* already handled by hid core */
         break;
     case HID_DG_TOUCHSCREEN:
-        /* we do not set suffix = "Touchscreen" */
+        suffix = "Touchscreen";
         hi->input->name = hdev->name;
         break;
     case HID_DG_STYLUS:
         /* force BTN_STYLUS to allow tablet matching in udev */
         __set_bit(BTN_STYLUS, hi->input->keybit);
+        __set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
         break;
     case HID_VD_ASUS_CUSTOM_MEDIA_KEYS:
         suffix = "Custom Media Keys";
-- 
2.21.0


```

Resolved runtime errors
```diff
diff --git a/drivers/gpu/drm/i915/intel_guc_submission.c b/drivers/gpu/drm/i915/intel_guc_submission.c
index 3b07bb9eb..d75c997cd 100644
--- a/drivers/gpu/drm/i915/intel_guc_submission.c
+++ b/drivers/gpu/drm/i915/intel_guc_submission.c
@@ -1379,7 +1379,7 @@ int i915_guc_ipts_submission_enable(struct drm_i915_private *dev_priv,
 	if (err)
 		return err;
 
-	ret = create_doorbell(guc->ipts_client);
+	ret = __guc_client_enable(guc->ipts_client);
 	if (ret)
 		return ret;
 
@@ -1393,6 +1393,7 @@ void i915_guc_ipts_submission_disable(struct drm_i915_private *dev_priv)
 	if (!guc->ipts_client)
 		return;
 
+	__guc_client_disable(guc->ipts_client);
 	guc_client_free(guc->ipts_client);
 	guc->ipts_client = NULL;
 }
```



## surface-acpi changes
```diff
diff --git a/drivers/platform/x86/surface_acpi.c b/drivers/platform/x86/surface_acpi.c
index 6a3aa6a01..b2d0f4f19 100644
--- a/drivers/platform/x86/surface_acpi.c
+++ b/drivers/platform/x86/surface_acpi.c
@@ -650,6 +650,7 @@ inline static int surfacegen5_ssh_writer_flush(struct surfacegen5_ec *ec)
 {
 	struct surfacegen5_ec_writer *writer = &ec->writer;
 	struct serdev_device *serdev = ec->serdev;
+	int status;
 
 	size_t len = writer->ptr - writer->data;
 
@@ -657,7 +658,8 @@ inline static int surfacegen5_ssh_writer_flush(struct surfacegen5_ec *ec)
 	print_hex_dump_debug("send: ", DUMP_PREFIX_OFFSET, 16, 1,
 	                     writer->data, writer->ptr - writer->data, false);
 
-	return serdev_device_write(serdev, writer->data, len, SG5_WRITE_TIMEOUT);
+	status = serdev_device_write(serdev, writer->data, len, SG5_WRITE_TIMEOUT);
+	return status >= 0 ? 0 : status;
 }
 
 inline static void surfacegen5_ssh_write_msg_cmd(struct surfacegen5_ec *ec,
@@ -895,6 +897,7 @@ inline static bool surfacegen5_ssh_is_valid_crc(const u8 *begin, const u8 *end)
 
 static int surfacegen5_ssh_send_ack(struct surfacegen5_ec *ec, u8 seq)
 {
+	int status;
 	u8 buf[SG5_MSG_LEN_CTRL];
 	u16 crc;
 
@@ -916,7 +919,8 @@ static int surfacegen5_ssh_send_ack(struct surfacegen5_ec *ec, u8 seq)
 	print_hex_dump_debug("send: ", DUMP_PREFIX_OFFSET, 16, 1,
 	                     buf, SG5_MSG_LEN_CTRL, false);
 
-	return serdev_device_write(ec->serdev, buf, SG5_MSG_LEN_CTRL, SG5_WRITE_TIMEOUT);
+	status = serdev_device_write(ec->serdev, buf, SG5_MSG_LEN_CTRL, SG5_WRITE_TIMEOUT);
+	return status >= 0 ? 0 : status;
 }
 
 static void surfacegen5_event_work_ack_handler(struct work_struct *_work)
```