# Framework - 调试技巧

## disable dex opt\(用于直接push调试framework.jar\)

1.   build/make/core/dex\_preopt.mk  

```text
   # Non eng linux builds must have preopt enabled so that system server doesn't run as interpreter
   # only. b/74209329
   ifeq (,$(filter eng, $(TARGET_BUILD_VARIANT)))
-    ifneq (true,$(WITH_DEXPREOPT))
+    ifneq (false,$(WITH_DEXPREOPT))
       ifneq (true,$(WITH_DEXPREOPT_BOOT_IMG_AND_SYSTEM_SERVER_ONLY))
         $(call pretty-error, DEXPREOPT must be enabled for user and userdebug builds)
       endif

```

2.  device/qcom/sm6150\_au/AndroidBoard.mk

```text
 # Enable dex pre-opt to speed up initial boot
 ifeq ($(HOST_OS),linux)
     ifeq ($(WITH_DEXPREOPT),)
-      WITH_DEXPREOPT := true
+      WITH_DEXPREOPT := false
       WITH_DEXPREOPT_PIC := true
       ifneq ($(TARGET_BUILD_VARIANT),user)
         # Retain classes.dex in APK's for non-user builds

```

