# **多窗口**
# 用ANDROID标准接口启动应用
```
package com.example.multwindow;


import android.app.ActivityOptions;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.graphics.Rect;
import android.os.Bundle;

import java.lang.reflect.Method;

public class StartFreeWindow {
    private static final int WINDOWING_MODE_FREEFORM = 5;

 /*
 StartFreeWindow.startFreeWindow(getContext(), 0, 0, 640, 720,
 "com.example.freew1", "com.example.freew1.SettingsActivity");
 StartFreeWindow.startFreeWindow(getContext(), 641, 0, 1280, 360,
 "com.example.freew2", "com.example.freew2.MapsActivity");
 StartFreeWindow.startFreeWindow(getContext(), 1281, 0, 1920, 720,
 "com.example.freew3", "com.example.freew3.ItemListActivity");

 StartFreeWindow.startFreeWindow(getContext(), 480, 200, 1100, 600,
 "com.example.freew1", "com.example.freew1.ui.login.LoginActivity");

 */
 public static void startFreeWindow(Context context, int left, int top, int right, int bottom,
 String packageName, String activity) {
        Intent intent = new Intent(Intent.ACTION_MAIN);
 intent.setFlags(Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT | Intent.FLAG_ACTIVITY_NEW_TASK
 | Intent.FLAG_ACTIVITY_CLEAR_TOP);
// intent.setFlags(Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT | Intent.FLAG_ACTIVITY_SINGLE_TOP);
// intent.addCategory(Intent.CATEGORY_LAUNCHER);
 ComponentName cn = new ComponentName(packageName, activity);
 intent.setComponent(cn);

 ActivityOptions activityOptions = ActivityOptions.makeBasic();
 try {
            Method method = ActivityOptions.class.getMethod("setLaunchWindowingMode", int.class);
 method.invoke(activityOptions, WINDOWING_MODE_FREEFORM);
 } catch (Exception e) {
            /* Gracefully fail */
 }
        activityOptions.setLaunchBounds(new Rect(left, top, right, bottom));
 Bundle bundle = activityOptions.toBundle();
 context.startActivity(intent, bundle);
 }
}
```

# aosp 修改

## aosp/frameworks/base/core/res/res/values/config.xml
```

    <!-- The device supports freeform window management. Windows have title bars and can be moved
         and resized. If you set this to true, you also need to add
         PackageManager.FEATURE_FREEFORM_WINDOW_MANAGEMENT feature to your device specification.
         The duplication is necessary, because this information is used before the features are
         available to the system.-->
    <bool name="config_freeformWindowManagement">true</bool>
 ```

## aosp/frameworks/base/data/etc/Android.mk
```
########################
include $(CLEAR_VARS)
LOCAL_MODULE := android.software.freeform_window_management.xml
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := $(TARGET_OUT_ETC)/permissions
LOCAL_SRC_FILES := $(LOCAL_MODULE)
include $(BUILD_PREBUILT)

```
##  aosp/frameworks/base/data/etc/android.software.freeform_window_management.xml
```
<?xml version="1.0" encoding="utf-8"?>
<!-- Copyright (C) 2015 The Android Open Source Project

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<permissions>
    <feature name="android.software.freeform_window_management" />
</permissions>
```
## aosp/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java
```
    private void retrieveSettings() {
        final ContentResolver resolver = mContext.getContentResolver();
        /* add start */
        Settings.Global.putInt(resolver, DEVELOPMENT_ENABLE_FREEFORM_WINDOWS_SUPPORT, 1);
        Settings.Global.putInt(resolver, DEVELOPMENT_FORCE_RESIZABLE_ACTIVITIES, 1);
        /* add end */

```

# ADB调试:
## 查询task id
```
adb shell am stack list
```
## 修改窗口尺寸
```
adb shell am task resize <taskId> <left>  <top> <right> <bottom>
adb shell am task resize 9 128 80 327 720
```
