# **OSFramework jar 源码集成方式**
# 一.源码拷贝
  由于所有源码都在/aosp/vendor/xx/proprietary下，所以直接找一个有源码的项目copy proprietary到要用源码的项目的/aosp/vendor/xx下，并删除/android/aosp/vendor/xx的所有文件。

# 二.代码修改
按照如下方式更改。
## aosp/frameworks/base/Android.bp
```
static_libs: [
       "framework-protos",
       "xx-plugs",
       "xx-frames",//删除
        
       "android.hardware.automotive.audiocontrol-V1.0-java",
       "android.hardware.automotive.vehicle-V2.0-java",
       "android.hidl.base-V1.0-java",
       "android.hardware.cas-V1.0-java",
       "android.hardware.contexthub-V1.0-java",
       "android.hardware.health-V1.0-java-constants",
       "android.hardware.thermal-V1.0-java-constants",
       "android.hardware.tv.input-V1.0-java-constants",
       "android.hardware.usb-V1.0-java-constants",
       "android.hardware.usb-V1.1-java-constants",
```

## aosp/frameworks/base/services/Android.bp
```
    app_image: true,
    profile: "art-profile",
},
 
srcs: [
    "java/**/*.java",
],
 
// The convention is to name each service module 'services.$(module_name)'
static_libs: [
    "xx-servers",//删除
    "services.core",
    "services.accessibility",
    "services.appwidget",
    "services.autofill",
    "services.backup",
    "services.companion",
    "services.coverage",
    "services.devicepolicy",
    "services.midi",
    "services.net",
```
## aosp/vendor/xx/build/xx.mk
```

#PRODUCT_PACKAGES += \ //打开注释
    xx-plugs \
    xx-frames \
    xx-servers \
 
#PRODUCT_BOOT_JARS +=  xx-plugs xx-frames //打开注释
#PRODUCT_SYSTEM_SERVER_JARS += xx-servers //打开注释
 
 
# add for overlays, only for add  new res, modify origin res by RRO(runtime-resource-overlay)
PRODUCT_PACKAGE_OVERLAYS := \
    vendor/xx/opensource/xx_overlays \
    $(PRODUCT_PACKAGE_OVERLAYS)
 
# add for init.xx.rc */
PRODUCT_COPY_FILES +=  vendor/xx/build/init.xx.rc:root/init.xx.rc
```
## aosp/build/make/core/tasks/check_boot_jars/package_whitelist.txt
```
#加上这些
com\.xx\.utils\..*
com\.xx\.frame
com\.xx\.frame\..*
com\.xx\.plug
com\.xx\.plug\..*
com\.xx\.process
com\.xx\.stability\..*
com\.xx\.condition
```
## frameworks/native/services/inputflinger/Android.bp
```
// Copyright (C) 2013 The Android Open Source Project
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//      http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
 
cc_library_shared {
    name: "libinputflinger",
 
    srcs: [
        "EventHub.cpp",
        "InputApplication.cpp",
        "InputDispatcher.cpp",
        "InputListener.cpp",
        "InputManager.cpp",
        "InputReader.cpp",
        "InputWindow.cpp",
    ],
 
    shared_libs: [
        "libbase",
        "libbinder",
        "libcrypto",
        "libcutils",
        "libinput",
        "liblog",
        "libutils",
        "libui",
        "libhardware_legacy",
    ],
//  加上这段
// xx begin
    static_libs: [
        "libxx_native",
    ],
// xx end
 
    cflags: [
        "-Wall",
        "-Wextra",
        "-Werror",
        "-Wno-unused-parameter",
        // TODO: Move inputflinger to its own process and mark it hidden
        //-fvisibility=hidden
    ],
 
    export_include_dirs: ["."],
}
 
subdirs = [
    "host",
    "tests",
]
``` 

# 三.开始编译

在android目录下 ./Tmake abc -v user -n --no_apk