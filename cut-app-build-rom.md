# **裁剪应用平台化**
# 实现：

修改aosp/vendor/xx/proprietary/xx_demo/LauncherDemo下的Android.mk

```
$(warning "PRODUCT_REMOVE_PACKAGE " $(PRODUCT_REMOVE_PACKAGE))
LOCAL_OVERRIDES_PACKAGES += $(PRODUCT_REMOVE_PACKAGE)
```

# 举例：
需要裁剪3.0R 的Home 和Launcher2两个模块

aosp/device/xx/xx30R/xx30R.mk
```
PRODUCT_REMOVE_PACKAGE := Home Launcher2
```
只需要在对应项目mk 的PRODUCT_REMOVE_PACKAGE  添加需要裁剪的应用名。这个应用名需要是LOCAL_PACKAGE_NAME 或者是 LOCAL_MODULE  指定的名字  并以空格隔开。


注意：裁剪的变量依托与launcherdemo apk,在编译前应先确认launcherdemo apk是否参与编译

# 需要确认以下两点：

## 1.确认 wutong/aosp/vendor/xx/ 目录下xx_os.mk是否包含LauncherDemo

```
#<StabilityTM> 2020-11-11 modify lepeng start
PRODUCT_PACKAGES += LauncherDemo \
                    xx-jni \
                    libxx_jni \
```                    
## 2.确认 wutong/aosp/vendor/xx/ 目录下xx.mk是否引入xx_os.mk

（1）在jar包编译的情况下，xx_os.mk在vendor/xx/build/prebuilts/build/ 目录下，写法如下
```
-include vendor/xx/build/prebuilts/build/xx_os.mk
```
（2）在源码编译的情况下，xx_os.mk在vendor/xx/proprietary/build/ 目录下，写法如下
```
-include ../wutong/aosp/vendor/xx/proprietary/build/xx_os.mk
```

# code
aosp/vendor/xx/build/xx.mk
```
PRODUCT_PACKAGES += \
    libxx_jni \
#    xx-plugs \
#    xx-frames \
#    xx-servers \
#    xx-jni \

#PRODUCT_BOOT_JARS +=  xx-plugs xx-frames
#PRODUCT_SYSTEM_SERVER_JARS += xx-servers

-include vendor/xx/build/prebuilts/build/xx_os.mk // add

# add for overlays, only for add  new res, modify origin res by RRO(runtime-resource-overlay)
PRODUCT_PACKAGE_OVERLAYS := \
    vendor/xx/opensource/xx_overlays \
    $(PRODUCT_PACKAGE_OVERLAYS)
```

aosp/device/xx/xx30R/xx30R.mk
```
# add activities_on_secondary_displays.xml file
PRODUCT_COPY_FILES += device/xx/xx30R/android.software.activities_on_secondary_displays.xml:system/etc/permissions/android.software.activities_on_secondary_displays.xml

#start usercenter
PRODUCT_PROPERTY_OVERRIDES += persist.sys.xx.start.usercenter=true
PRODUCT_REMOVE_PACKAGE := Home Launcher2  //add
$(warning  "PRODUCT_REMOVE_PACKAGE = "$(PRODUCT_REMOVE_PACKAGE)) //add

```