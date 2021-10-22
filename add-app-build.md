# **预置apk问题 总结**

# 内部含有so 的apk 在预置后会出现无法运行，崩溃的现象。此处写一下预置app 遇到的问题

## 1 小场景预置  到system/app后，会出现卡死等问题。具体log 表现为
```
E linker  : library "/data/user/0/com.tencent.wecarmas/app_tbs/core_share/libmttwebview.so" ("/system/app/WTWeCarMas/lib/arm/libtbs.libmttwebview.so.so") needed or dlopened by "/system/lib/libnativeloader.so" is not accessible for the namespace: [name="classloader-namespace", ld_library_paths="", default_library_paths="/data/data/com.tencent.wecarmas/app_tbs/core_share", permitted_paths="/data:/mnt/expand"]
```
这个是因为so 在system/app中，而system/app 不在permitted_paths 中。 需要把system/app加到kWhitelistedDirectories 中。修改如下

aosp/system/core/libnativeloader/native_loader.cpp
```
// (http://b/27588281) This is a workaround for apps using custom classloaders and calling
// System.load() with an absolute path which is outside of the classloader library search path.
// This list includes all directories app is allowed to access this way.
static constexpr const char* kWhitelistedDirectories = "/data:/mnt/expand:/system/app";  // add

static bool is_debuggable() {
  char debuggable[PROP_VALUE_MAX];
```


另外需要将so 名字加到LOCAL_PREBUILT_JNI_LIBS 中，在Android.mk 中添加如下：
```
WT_JNI_LIBS :=
$(foreach FILE,$(shell find $(LOCAL_PATH)/lib/ -name *.so), $(eval WT_JNI_LIBS += $(FILE)))
LOCAL_PREBUILT_JNI_LIBS := $(subst $(LOCAL_PATH),,$(WT_JNI_LIBS))
```
小场景单发版需要把下面几个so 集成到system/lib64 下
```
$(shell mkdir -p $(PRODUCT_OUT)/system/lib64/)
$(shell cp $(LOCAL_PATH)/lib/armeabi-v7a/libwebviewchromium.so $(PRODUCT_OUT)/system/lib64/)
$(shell cp $(LOCAL_PATH)/lib/armeabi-v7a/libzh_cn.so $(PRODUCT_OUT)/system/lib64/)
$(shell cp $(LOCAL_PATH)/lib/armeabi-v7a/libresources.so $(PRODUCT_OUT)/system/lib64/)
```

# 2微信预置，因微信不允许重新签名，也不能对此编译。建议放到vendor分区，拷贝到vendor/app/WeChat 下。

将Makefile以下语句中error 改成warning
```
define check-product-copy-files $(if $(filter-out $(TARGET_COPY_OUT_SYSTEM_OTHER)/%,$(2)), \ $(if $(filter %.apk, $(2)),$(error \ Prebuilt apk found in PRODUCT_COPY_FILES: $(1), use BUILD_PREBUILT instead!))) endef
```
将so 解压出来，并把apk和so拷贝到vendor 下
```
$(shell unzip -qo packages/apps/WeChat/WeChat.apk -d packages/apps/WeChat)
PRODUCT_COPY_FILES += packages/apps/WeChat/WeChat.apk:vendor/app/WeChat/WeChat.apk
PRODUCT_COPY_FILES += $(call find-copy-subdir-files,*,packages/apps/WeChat/lib/armeabi/,vendor/app/WeChat/lib/arm/)
```
# 3：王者荣耀预置， 因王者荣耀apk 较大，临时演示版本的话只需要放到data分区
修改链接如下

需要将预置的目录放到directories_to_exclude 中，避免因为加密开不了机

aosp/frameworks/base/packages/Shell/src/com/android/shell/BootCompletedReceiver.java
```
public class BootCompletedReceiver extends BroadcastReceiver {
    private static final String TAG = "Shell";
    private static final String WECARMAS = "wecarmas";
    private static final String WECHAT = "wechat";
    private static final String KING = "king"; // add
    private static final String ACTION_INSTALL_MM = "com.tencent.mm.action.install";
    private static final String ACTION_INSTALL_MM_FAILED = "com.tencent.mm.action.INSTALL_FAILED";
    private static boolean haveReceiveBootCompleted = false;
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG,"ACTION = "+intent.getAction());
        if (intent.getAction().equals(Intent.ACTION_BOOT_COMPLETED) && !haveReceiveBootCompleted) {
            context.startService(new Intent(context,InstallService.class));
            haveReceiveBootCompleted = true;
        }
        if (intent.getAction().equals(ACTION_INSTALL_MM)) {
            if(installApp(intent.getStringExtra("path"),context,WECHAT)){
                Intent failIntent = new Intent(ACTION_INSTALL_MM_FAILED);
                context.sendBroadcast(failIntent);
            }else{
                Log.d(TAG,"Install tencent mm success ,open wechat");
                Intent successIntent = context.getPackageManager().getLaunchIntentForPackage("com.tencent.mm");
                if(successIntent != null){
                    context.startActivity(successIntent);
                }
            }
        }
    }


    static  class MyTaskClear implements Runnable {
        private  Context  context =null;

        public MyTaskClear(Context mcontext) {
            context = mcontext;
        }

        @Override
        public void run() {
            Log.d(TAG,"MyTaskClear run ");
           //<StabilityTM> 2019-09-29 add third APP WT_WeCarMas  kang.zhang start
           if(SystemProperties.get("persist.sys.wecarmas","0").equals("0")){
               File file = new File("system/third_app/WeCarMas/WeCarMas.apk");
               if(file.exists()){
                  Log.d(TAG,"installApp WeCarMas");
                  installApp("/system/third_app/WeCarMas/WeCarMas.apk",context,WECARMAS);
               }else{
                  Log.d(TAG,"installApp WT_WeCarMas ");
                  installApp("/system/third_app/WT_WeCarMas/WT_WeCarMas.apk",context,WECARMAS);
               }
           }
           //微信静默安装. add 
           if(SystemProperties.get("persist.sys.wechat","0").equals("0")){
               installApp("/system/third_app/WeChat/WeChat.apk",context,WECHAT);
           }
           if(SystemProperties.get("persist.sys.king","0").equals("0")){
               installApp("/data/third_app/HonorOfKings/HonorOfKings.apk",context,KING);
           }
           //<StabilityTM> 2019-09-29 add third APP WT_WeCarMas kang.zhang end add
        }
    }
    ...

      public static boolean installApp(String apkPath,Context context,String apk) {
          ...
            }
        // add    
        if(apk.equals(KING)){
            if(isSuccess){
                SystemProperties.set("persist.sys.king","1");
            }else{
                SystemProperties.set("persist.sys.king","0");
            }
        }
        // add
        Log.e(TAG, errorMsg.toString());      
```

 aosp/packages/apps/HonorOfKings/HonorOfKings.mk
 ```
 LOCAL_PATH := $(call my-dir)
ifeq ($(strip $(APK_3RD_HONOROfKINGS_SUPPORT)),yes)
PRODUCT_COPY_FILES += packages/apps/HonorOfKings/HonorOfKings.apk:data/third_app/HonorOfKings/HonorOfKings.apk
endif

 ```

 aosp/system/extras/ext4_utils/ext4_crypt_init_extensions.cpp
 ```
     std::vector<std::string> directories_to_exclude = {
        "lost+found",
        "system_ce", "system_de",
        "misc_ce", "misc_de",
        "vendor_ce", "vendor_de",
        "media",
        "data", "user", "user_de",
        "third_app", //add
    };
 ```

 wutong/config/wt_common.mk
 ```
 #<StabilityTM> 2019-07-04 modify zhouhui start
PRODUCT_PACKAGES += StabilityMonitoring \
                    PerformanceMonitor \
#<StabilityTM> 2019-07-04 modify for zhouhui end
$(call inherit-product-if-exists, packages/apps/HonorOfKings/HonorOfKings.mk) // add
ifeq ($(strip $(WT_APK_WT_BTPHONE_SUPPORT)),yes)
	 PRODUCT_PACKAGES += WT_BTPhone
 ```


4：混淆问题，有一些三方apk 在释放完so 之后仍然不能正常运行，但是install 却可以正常运行。这个时候要检查下报错的类或so 是否存在混淆。

预置对混淆的容忍度较低。这个时候要联系应用去掉混淆。

5：LOCAL_MULTILIB 设置,预置在系统中的so只需要存在一份，32或64位。因此 LOCAL_MULTILIB只需要写32 或者64 。这样也能节省空间。
