# README

From [jakeday/linux-surface: Linux Kernel for Surface Devices](https://github.com/jakeday/linux-surface) commit 67bd6e23acc189099c10fe55a448980223863449, which is the last commit with support 4.18.

## IPTS changes from 4.17 to 4.18

```diff
diff --git a/drivers/gpu/drm/i915/i915_irq.c b/drivers/gpu/drm/i915/i915_irq.c
index 90d8f8d8..8fb9160d 100644
--- a/drivers/gpu/drm/i915/i915_irq.c
+++ b/drivers/gpu/drm/i915/i915_irq.c
@@ -1477,7 +1477,7 @@ gen8_cs_irq_handler(struct intel_engine_cs *engine, u32 iir)
 		tasklet |= USES_GUC_SUBMISSION(engine->i915);
 	}
 
-	if (iir & (GT_RENDER_PIPECTL_NOTIFY_INTERRUPT << test_shift))
+	if (iir & GT_RENDER_PIPECTL_NOTIFY_INTERRUPT)
 		intel_ipts_notify_complete();
 
 	if (tasklet)
diff --git a/drivers/gpu/drm/i915/i915_params.c b/drivers/gpu/drm/i915/i915_params.c
index 4f54d74f..b256e8a9 100644
--- a/drivers/gpu/drm/i915/i915_params.c
+++ b/drivers/gpu/drm/i915/i915_params.c
@@ -154,8 +154,8 @@ i915_param_named_unsafe(enable_guc, int, 0400,
 	"Required functionality can be selected using bitmask values. "
 	"(-1=auto, 0=disable, 1=GuC submission [default], 2=HuC load)");
 
-i915_param_named_unsafe(enable_ipts, bool, 0400,
-	"Enable IPTS Touchscreen and Pen support (default: true)");
+i915_param_named_unsafe(enable_ipts, int, 0400,
+	"Enable IPTS Touchscreen and Pen support (default: 1)");
 
 i915_param_named(guc_log_level, int, 0400,
 	"GuC firmware logging level. Requires GuC to be loaded. "
diff --git a/drivers/gpu/drm/i915/i915_params.h b/drivers/gpu/drm/i915/i915_params.h
index d6b8178c..ca1128dc 100644
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
@@ -71,7 +71,7 @@ struct drm_printer;
 	param(bool, enable_dp_mst, true) \
 	param(bool, enable_dpcd_backlight, false) \
 	param(bool, enable_gvt, false) \
-	param(bool, enable_ipts, true)
+	param(int, enable_ipts, 1)
 
 #define MEMBER(T, member, ...) T member;
 struct i915_params {
diff --git a/drivers/gpu/drm/i915/intel_guc_submission.c b/drivers/gpu/drm/i915/intel_guc_submission.c
index 1b7b8dc2..4f8f3845 100644
--- a/drivers/gpu/drm/i915/intel_guc_submission.c
+++ b/drivers/gpu/drm/i915/intel_guc_submission.c
@@ -116,7 +116,7 @@ static int reserve_doorbell(struct intel_guc_client *client)
 			offset = end;
 			end += offset;
 		}
- 	}
+	}
 
 	id = find_next_zero_bit(client->guc->doorbell_bitmap, end, offset);
 	if (id == end)
diff --git a/drivers/hid/hid-multitouch.c b/drivers/hid/hid-multitouch.c
index daa48c95..468710ac 100644
--- a/drivers/hid/hid-multitouch.c
+++ b/drivers/hid/hid-multitouch.c
@@ -1342,6 +1342,7 @@ static int mt_input_configured(struct hid_device *hdev, struct hid_input *hi)
 			suffix = "Pen";
 			/* force BTN_STYLUS to allow tablet matching in udev */
 			__set_bit(BTN_STYLUS, hi->input->keybit);
+            __set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
 		}
 	}
 
@@ -1357,12 +1358,13 @@ static int mt_input_configured(struct hid_device *hdev, struct hid_input *hi)
 			/* already handled by hid core */
 			break;
 		case HID_DG_TOUCHSCREEN:
-			/* we do not set suffix = "Touchscreen" */
+			suffix = "Touchscreen";
 			hi->input->name = hdev->name;
 			break;
 		case HID_DG_STYLUS:
 			/* force BTN_STYLUS to allow tablet matching in udev */
 			__set_bit(BTN_STYLUS, hi->input->keybit);
+            __set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
 			break;
 		case HID_VD_ASUS_CUSTOM_MEDIA_KEYS:
 			suffix = "Custom Media Keys";
diff --git a/drivers/misc/ipts/ipts-hid.c b/drivers/misc/ipts/ipts-hid.c
index 3b3be617..e85844dc 100644
--- a/drivers/misc/ipts/ipts-hid.c
+++ b/drivers/misc/ipts/ipts-hid.c
@@ -406,7 +406,7 @@ static int handle_outputs(ipts_info_t *ipts, int parallel_idx)
 						err_payload->code[2],
 						err_payload->code[3],
 						err_payload->string);
-				
+
 				break;
 			}
 			default:
diff --git a/drivers/misc/ipts/ipts-kernel.c b/drivers/misc/ipts/ipts-kernel.c
index ca5e24ce..86fd359d 100644
--- a/drivers/misc/ipts/ipts-kernel.c
+++ b/drivers/misc/ipts/ipts-kernel.c
@@ -272,7 +272,7 @@ static int bin_read_cmd_buffer(ipts_info_t *ipts,
 		return -EINVAL;
 
 	ipts_dbg(ipts, "cmd buf size = %d\n", cmd->size);
-		
+
 	num_of_parallels = ipts_get_num_of_parallel_buffers(ipts);
 	/* command buffers are located after the other allocations */
 	cmdbuf_idx = num_of_parallels * alloc_info->num_of_allocations;
@@ -344,7 +344,7 @@ static int bin_read_res_list(ipts_info_t *ipts,
 	int buf_idx, num_of_alloc;
 	u32 buf_size, flags, io_buf_type;
 	bool initialize;
-	
+
 	parsed = parse_info->parsed;
 	size = parse_info->size;
 	bin_data = parse_info->data;
@@ -357,7 +357,7 @@ static int bin_read_res_list(ipts_info_t *ipts,
 
 	ipts_dbg(ipts, "number of resources %u\n", res_list->num);
 	for (i = 0; i < res_list->num; i++) {
-		initialize = false; 
+		initialize = false;
 		io_buf_type = 0;
 		flags = 0;
 
@@ -538,7 +538,7 @@ static int bin_read_patch_list(ipts_info_t *ipts,
 			if(alloc_info->buffs[buf_idx].buf != NULL) {
 				gtt_offset = (u32)(u64)
 					alloc_info->buffs[buf_idx].buf->gfx_addr;
-			} 
+			}
 			gtt_offset += patch[i].alloc_offset;
 
 			batch += patch[i].patch_offset;
@@ -562,7 +562,7 @@ static int bin_read_guc_wq_item(ipts_info_t *ipts,
 	u8 *wi_data;
 	int size, parsed, hdr_size, wi_size;
 	int i, batch_offset;
-	
+
 	parsed = parse_info->parsed;
 	size = parse_info->size;
 	bin_guc_wq = (ipts_bin_guc_wq_info_t *)&parse_info->data[parsed];
@@ -607,7 +607,7 @@ static int bin_setup_guc_workqueue(ipts_info_t *ipts,
 	bin_buffer_t *bin_buf;
 	int wq_size, wi_size, parallel_idx, cmd_idx, k_idx, iter_size;
 	int i, num_of_parallels, batch_offset, k_num, total_workload;
-	
+
 	wq_addr = (u8*)ipts->resource.wq_info.wq_addr;
 	wq_size = ipts->resource.wq_info.wq_size;
 	num_of_parallels = ipts_get_num_of_parallel_buffers(ipts);
@@ -631,7 +631,7 @@ static int bin_setup_guc_workqueue(ipts_info_t *ipts,
 			batch_offset = kernel->guc_wq_item->batch_offset;
 			wi_size = kernel->guc_wq_item->size;
 			wi_data = &kernel->guc_wq_item->data[0];
-			
+
 			cmd_idx = wl[parallel_idx].cmdbuf_index;
 			bin_buf = &alloc_info->buffs[cmd_idx];
 
@@ -774,38 +774,38 @@ static int load_kernel(ipts_info_t *ipts, bin_parse_info_t *parse_info,
 
 	ret = bin_read_allocation_list(ipts, parse_info, alloc_info);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_allocation_list\n");
+		ipts_dbg(ipts, "error read_allocation_list\n");
 		goto setup_error;
 	}
 
 	ret = bin_read_cmd_buffer(ipts, parse_info, alloc_info, wl);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_cmd_buffer\n");
+		ipts_dbg(ipts, "error read_cmd_buffer\n");
 		goto setup_error;
 	}
 
 	ret = bin_read_res_list(ipts, parse_info, alloc_info, wl);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_res_list\n");
+		ipts_dbg(ipts, "error read_res_list\n");
 		goto setup_error;
 	}
 
 	ret = bin_read_patch_list(ipts, parse_info, alloc_info, wl);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_patch_list\n");
+		ipts_dbg(ipts, "error read_patch_list\n");
 		goto setup_error;
 	}
 
 	ret = bin_read_guc_wq_item(ipts, parse_info, &guc_wq_item);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_guc_workqueue\n");
+		ipts_dbg(ipts, "error read_guc_workqueue\n");
 		goto setup_error;
 	}
 
 	memset(&bufid_patch, 0, sizeof(bufid_patch));
 	ret = bin_read_bufid_patch(ipts, parse_info, &bufid_patch);
 	if (ret) {
-        	ipts_dbg(ipts, "error read_bufid_patch\n");
+		ipts_dbg(ipts, "error read_bufid_patch\n");
 		goto setup_error;
 	}
 
@@ -815,7 +815,7 @@ static int load_kernel(ipts_info_t *ipts, bin_parse_info_t *parse_info,
 	kernel->guc_wq_item = guc_wq_item;
 	memcpy(&kernel->bufid_patch, &bufid_patch, sizeof(bufid_patch));
 
-        return 0;
+	return 0;
 
 setup_error:
 	vfree(guc_wq_item);
@@ -944,16 +944,16 @@ static int setup_kernel(ipts_info_t *ipts, bin_fw_list_t *fw_list)
 	}
 
 	ipts_set_wq_item_size(ipts, total_workload);
-	
+
 	ret = bin_setup_guc_workqueue(ipts, kernel_list);
 	if (ret) {
-        	ipts_dbg(ipts, "error setup_guc_workqueue\n");
+		ipts_dbg(ipts, "error setup_guc_workqueue\n");
 		goto error_exit;
 	}
 
 	ret = bin_setup_bufid_buffer(ipts, kernel_list);
 	if (ret) {
-        	ipts_dbg(ipts, "error setup_lastbubmit_buffer\n");
+		ipts_dbg(ipts, "error setup_lastbubmit_buffer\n");
 		goto error_exit;
 	}
 
@@ -996,7 +996,7 @@ static void release_kernel(ipts_info_t *ipts)
 	for (k_idx = 0; k_idx < k_num; k_idx++) {
 		unload_kernel(ipts, kernel);
 		kernel++;
-	}	
+	}
 
 	ipts_unmap_buffer(ipts, kernel_list->bufid_buf);
 
diff --git a/drivers/misc/ipts/ipts-mei.c b/drivers/misc/ipts/ipts-mei.c
index 39667e75..199e49cb 100644
--- a/drivers/misc/ipts/ipts-mei.c
+++ b/drivers/misc/ipts/ipts-mei.c
@@ -222,7 +222,7 @@ static int ipts_mei_cl_probe(struct mei_cl_device *cldev,
 
 disable_mei :
 	mei_cldev_disable(cldev);
-	
+
 	return ret;
 }
 
diff --git a/drivers/misc/ipts/ipts-msg-handler.c b/drivers/misc/ipts/ipts-msg-handler.c
index 1396ecc7..8b214f97 100644
--- a/drivers/misc/ipts/ipts-msg-handler.c
+++ b/drivers/misc/ipts/ipts-msg-handler.c
@@ -156,7 +156,7 @@ void ipts_stop(ipts_info_t *ipts)
 
 	if (old_state == IPTS_STA_RAW_DATA_STARTED ||
 					old_state == IPTS_STA_HID_STARTED) {
-        	ipts_free_default_resource(ipts);
+		ipts_free_default_resource(ipts);
 		ipts_free_raw_data_resource(ipts);
 
 		return;
@@ -172,7 +172,7 @@ int ipts_restart(ipts_info_t *ipts)
 	ipts_stop(ipts);
 
 	ipts->retry++;
-	if (ipts->retry == IPTS_MAX_RETRY && 
+	if (ipts->retry == IPTS_MAX_RETRY &&
 			ipts->sensor_mode == TOUCH_SENSOR_MODE_RAW_DATA) {
 		/* try with HID mode */
 		ipts->sensor_mode = TOUCH_SENSOR_MODE_HID;
diff --git a/drivers/misc/ipts/ipts-msg-handler.h b/drivers/misc/ipts/ipts-msg-handler.h
index b8e27d30..15038814 100644
--- a/drivers/misc/ipts/ipts-msg-handler.h
+++ b/drivers/misc/ipts/ipts-msg-handler.h
@@ -22,7 +22,7 @@ int ipts_start(ipts_info_t *ipts);
 void ipts_stop(ipts_info_t *ipts);
 int ipts_switch_sensor_mode(ipts_info_t *ipts, int new_sensor_mode);
 int ipts_handle_resp(ipts_info_t *ipts, touch_sensor_msg_m2h_t *m2h_msg,
-                        					u32 msg_len);
+                     u32 msg_len);
 int ipts_handle_processed_data(ipts_info_t *ipts);
 int ipts_send_feedback(ipts_info_t *ipts, int buffer_idx, u32 transaction_id);
 int ipts_send_sensor_quiesce_io_cmd(ipts_info_t *ipts);

```