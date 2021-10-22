# **Android RRO机制**
RRO,全称Runtime Resource Overlay。运行时资源替换机制。
具体实现方式：
## 1 以Dialer 举例,比如想替换中间的请输入电话号码，首先用Android studio 工具layout Inspector 工具找到此TextView id。然后用反编译工具apktool 反编译Dialer.apk  。根据id 找到字符串
```
   <string name="input_number">请输入电话号码</string>
```   

## 2 在framework/base/packages/overlays 下创建DialerOverlays 目录。结构如下。放在packages/app 下亦可。
```
framework/base/packages/overlays/DailerOverlays
-AndroidManifest.xml
-Android.mk
-res
   |-values
        |-strings.xml 
```
* AndroidManifest.xml
需要注意的是android:targetPackage 这个是要替换的apk 包名

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="om.autopai.car.dialer.overlay"
android:versionCode="1"
android:versionName="1.0">
<overlay android:targetPackage="com.autopai.car.dialer"
android:priority="1" android:isStatic="true"/>
</manifest>

```

* Android.mk 
```
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_RRO_THEME := DialerOverlay
LOCAL_CERTIFICATE := platform

LOCAL_SRC_FILES := $(call all-subdir-java-files)

LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res

LOCAL_PACKAGE_NAME := DialerOverlay
LOCAL_SDK_VERSION := current

include $(BUILD_RRO_PACKAGE)
```

* 将要替换的字符串写入res/values/strings.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
<string name="input_number">11111111电话号码</string>
</resources>
```

# 3. 单编DialerOverlays 模块
生成路径为
out/target/product/mek_8q/vendor/overlay/DialerOverlay

然后将DialerOverlay push 到vendor/overlay 目录重启即可
