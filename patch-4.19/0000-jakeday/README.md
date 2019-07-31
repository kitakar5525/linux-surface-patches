# IPTS patch changes from 4.18 to 4.19

## changes by jakeday
- in `mt_input_mapped`
```diff
+	if (field->application == HID_DG_TOUCHSCREEN ||
+			field->application == HID_DG_TOUCHPAD) {
+		if (usage->type == EV_KEY || usage->type == EV_ABS)
+			set_bit(usage->type, hi->input->evbit);
+
+		return -1;
+	}
```

- changes was made to use `IS_SKYLAKE(dev_priv) || IS_KABYLAKE(dev_priv) ? GUC_CLIENT_PRIORITY_HIGH : GUC_CLIENT_PRIORITY_NORMAL,` instead of `GUC_CLIENT_PRIORITY_NORMAL,`

diff from commit `67bd6e23acc189099c10fe55a448980223863449`

- in `intel_ipts_cleanup`
```diff
+			struct i915_vma *vma, *vn;
+
+			list_for_each_entry_safe(vma, vn,
+						 &obj->list, obj_link) {
+				vma->flags &= ~I915_VMA_PIN_MASK;
+				i915_vma_destroy(vma);
+			}
+
```

- introduced `intel_ipts_resume` and `intel_ipts_suspend`

```diff
diff --git a/drivers/gpu/drm/i915/i915_gem_context.c b/drivers/gpu/drm/i915/i915_gem_context.c
index 79ed9b6e..0222f0a5 100644
--- a/drivers/gpu/drm/i915/i915_gem_context.c
+++ b/drivers/gpu/drm/i915/i915_gem_context.c
@@ -469,7 +469,7 @@ static bool needs_preempt_context(struct drm_i915_private *i915)
 
 struct i915_gem_context *i915_gem_context_create_ipts(struct drm_device *dev)
 {
-	struct drm_i915_private *dev_priv = dev->dev_private;
+	struct drm_i915_private *dev_priv = to_i915(dev);
 	struct i915_gem_context *ctx;
 
 	BUG_ON(!mutex_is_locked(&dev->struct_mutex));
diff --git a/drivers/gpu/drm/i915/intel_ipts.c b/drivers/gpu/drm/i915/intel_ipts.c
index f8cc5eaf..1888e450 100644
--- a/drivers/gpu/drm/i915/intel_ipts.c
+++ b/drivers/gpu/drm/i915/intel_ipts.c
@@ -137,9 +137,9 @@ static int ipts_object_pin(intel_ipts_object_t* obj,
 	int ret = 0;
 
 	if (ipts_ctx->ppgtt) {
-		vm = &ipts_ctx->ppgtt->base;
+		vm = &ipts_ctx->ppgtt->vm;
 	} else {
-		vm = &dev_priv->ggtt.base;
+		vm = &dev_priv->ggtt.vm;
 	}
 
 	vma = i915_vma_instance(obj->gem_obj, vm, NULL);
@@ -174,7 +174,8 @@ static int create_ipts_context(void)
 {
 	struct i915_gem_context *ipts_ctx = NULL;
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
-	struct intel_ring *pin_ret;
+	struct intel_context *ce = NULL;
+	struct intel_context *pin_ret;
 	int ret = 0;
 
 	/* Initialize the context right away.*/
@@ -192,7 +193,15 @@ static int create_ipts_context(void)
 		goto err_unlock;
 	}
 
-	ret = execlists_context_deferred_alloc(ipts_ctx, dev_priv->engine[RCS]);
+	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+	if (IS_ERR(ce)) {
+		DRM_ERROR("Failed to create intel context (error %ld)\n",
+			  PTR_ERR(ce));
+		ret = PTR_ERR(ce);
+		goto err_unlock;
+	}
+
+	ret = execlists_context_deferred_alloc(ipts_ctx, dev_priv->engine[RCS], ce);
 	if (ret) {
 		DRM_DEBUG("lr context allocation failed : %d\n", ret);
 		goto err_ctx;
@@ -228,10 +237,13 @@ static void destroy_ipts_context(void)
 {
 	struct i915_gem_context *ipts_ctx = NULL;
 	struct drm_i915_private *dev_priv = to_i915(intel_ipts.dev);
+	struct intel_context *ce = NULL;
 	int ret = 0;
 
 	ipts_ctx = intel_ipts.ipts_context;
 
+	ce = to_intel_context(ipts_ctx, dev_priv->engine[RCS]);
+
 	/* Initialize the context right away.*/
 	ret = i915_mutex_lock_interruptible(intel_ipts.dev);
 	if (ret) {
@@ -239,7 +251,7 @@ static void destroy_ipts_context(void)
 		return;
 	}
 
-	execlists_context_unpin(dev_priv->engine[RCS], ipts_ctx);
+	execlists_context_unpin(ce);
 	i915_gem_context_put(ipts_ctx);
 
 	mutex_unlock(&intel_ipts.dev->struct_mutex);
@@ -441,9 +453,9 @@ static int intel_ipts_map_buffer(u64 gfx_handle, intel_ipts_mapbuffer_t *mapbuf)
 	}
 
 	if (ipts_ctx->ppgtt) {
-		vm = &ipts_ctx->ppgtt->base;
+		vm = &ipts_ctx->ppgtt->vm;
 	} else {
-		vm = &dev_priv->ggtt.base;
+		vm = &dev_priv->ggtt.vm;
 	}
 
 	vma = i915_vma_instance(obj->gem_obj, vm, NULL);
@@ -591,16 +603,19 @@ int intel_ipts_init(struct drm_device *dev)
 	intel_ipts.dev = dev;
 	INIT_DELAYED_WORK(&intel_ipts.reacquire_db_work, reacquire_db_work_func);
 
+	pr_info("ipts: creating ipts context\n");
 	ret = create_ipts_context();
 	if (ret)
 		return -ENOMEM;
 
+	pr_info("ipts: starting ipts work queue\n");
 	ret = intel_ipts_init_wq();
 	if (ret)
 		return ret;
 
 	intel_ipts.initialized = true;
-	DRM_DEBUG_DRIVER("Intel iTouch framework initialized\n");
+	//DRM_DEBUG_DRIVER("Intel iTouch framework initialized\n");
+	pr_info("ipts: Intel iTouch framework initialized\n");
 
 	return ret;
 }
diff --git a/drivers/gpu/drm/i915/intel_lrc.c b/drivers/gpu/drm/i915/intel_lrc.c
index f58de13b..f669087d 100644
--- a/drivers/gpu/drm/i915/intel_lrc.c
+++ b/drivers/gpu/drm/i915/intel_lrc.c
@@ -164,9 +164,6 @@
 #define WA_TAIL_DWORDS 2
 #define WA_TAIL_BYTES (sizeof(u32) * WA_TAIL_DWORDS)
 
-static int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
-					    struct intel_engine_cs *engine,
-					    struct intel_context *ce);
 static void execlists_init_reg_state(u32 *reg_state,
 				     struct i915_gem_context *ctx,
 				     struct intel_engine_cs *engine,
@@ -1292,7 +1289,7 @@ static void execlists_context_destroy(struct intel_context *ce)
 	i915_gem_object_put(ce->state->obj);
 }
 
-static void execlists_context_unpin(struct intel_context *ce)
+void execlists_context_unpin(struct intel_context *ce)
 {
 	intel_ring_unpin(ce->ring);
 
@@ -1379,7 +1376,7 @@ static const struct intel_context_ops execlists_context_ops = {
 	.destroy = execlists_context_destroy,
 };
 
-static struct intel_context *
+struct intel_context *
 execlists_context_pin(struct intel_engine_cs *engine,
 		      struct i915_gem_context *ctx)
 {
@@ -2737,7 +2734,7 @@ populate_lr_context(struct i915_gem_context *ctx,
 	return ret;
 }
 
-static int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
+int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
 					    struct intel_engine_cs *engine,
 					    struct intel_context *ce)
 {
diff --git a/drivers/gpu/drm/i915/intel_lrc.h b/drivers/gpu/drm/i915/intel_lrc.h
index e52a7782..32159231 100644
--- a/drivers/gpu/drm/i915/intel_lrc.h
+++ b/drivers/gpu/drm/i915/intel_lrc.h
@@ -106,12 +106,12 @@ void intel_lr_context_resume(struct drm_i915_private *dev_priv);
 
 void intel_execlists_set_default_submission(struct intel_engine_cs *engine);
 
-struct intel_ring *
+struct intel_context *
 execlists_context_pin(struct intel_engine_cs *engine,
 		      struct i915_gem_context *ctx);
-void execlists_context_unpin(struct intel_engine_cs *engine,
-				    struct i915_gem_context *ctx);
+void execlists_context_unpin(struct intel_context *ce);
 int execlists_context_deferred_alloc(struct i915_gem_context *ctx,
-					    struct intel_engine_cs *engine);
+					    struct intel_engine_cs *engine,
+						struct intel_context *ce);
 
 #endif /* _INTEL_LRC_H_ */
diff --git a/drivers/hid/hid-multitouch.c b/drivers/hid/hid-multitouch.c
index 90894b53..28b729c9 100644
--- a/drivers/hid/hid-multitouch.c
+++ b/drivers/hid/hid-multitouch.c
@@ -173,6 +173,7 @@ struct mt_device {
 static void mt_post_parse_default_settings(struct mt_device *td,
 					   struct mt_application *app);
 static void mt_post_parse(struct mt_device *td, struct mt_application *app);
+static int cc_seen = 0;
 
 /* classes of device behavior */
 #define MT_CLS_DEFAULT				0x0001
@@ -799,8 +800,11 @@ static int mt_touch_input_mapping(struct hid_device *hdev, struct hid_input *hi,
 			app->scantime_logical_max = field->logical_maximum;
 			return 1;
 		case HID_DG_CONTACTCOUNT:
-			app->have_contact_count = true;
-			app->raw_cc = &field->value[usage->usage_index];
+			if(cc_seen != 1) {
+				app->have_contact_count = true;
+				app->raw_cc = &field->value[usage->usage_index];
+				cc_seen++;
+			}
 			return 1;
 		case HID_DG_AZIMUTH:
 			/*
@@ -1339,6 +1343,14 @@ static int mt_input_mapped(struct hid_device *hdev, struct hid_input *hi,
 	struct mt_device *td = hid_get_drvdata(hdev);
 	struct mt_report_data *rdata;
 
+	if (field->application == HID_DG_TOUCHSCREEN ||
+			field->application == HID_DG_TOUCHPAD) {
+		if (usage->type == EV_KEY || usage->type == EV_ABS)
+			set_bit(usage->type, hi->input->evbit);
+
+		return -1;
+	}
+
 	rdata = mt_find_report_data(td, field->report);
 	if (rdata && rdata->is_mt_collection) {
 		/* We own these mappings, tell hid-input to ignore them */
@@ -1548,7 +1560,6 @@ static int mt_input_configured(struct hid_device *hdev, struct hid_input *hi)
 			suffix = "Pen";
 			/* force BTN_STYLUS to allow tablet matching in udev */
 			__set_bit(BTN_STYLUS, hi->input->keybit);
-            __set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
 		}
 	}
 
@@ -1571,7 +1582,7 @@ static int mt_input_configured(struct hid_device *hdev, struct hid_input *hi)
 		case HID_DG_STYLUS:
 			/* force BTN_STYLUS to allow tablet matching in udev */
 			__set_bit(BTN_STYLUS, hi->input->keybit);
-            __set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
+			__set_bit(INPUT_PROP_DIRECT, hi->input->propbit);
 			break;
 		case HID_VD_ASUS_CUSTOM_MEDIA_KEYS:
 			suffix = "Custom Media Keys";
@@ -1688,6 +1699,7 @@ static int mt_probe(struct hid_device *hdev, const struct hid_device_id *id)
 	td->hdev = hdev;
 	td->mtclass = *mtclass;
 	td->inputmode_value = MT_INPUTMODE_TOUCHSCREEN;
+	cc_seen = 0;
 	hid_set_drvdata(hdev, td);
 
 	INIT_LIST_HEAD(&td->applications);

```