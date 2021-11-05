# **Android恢复出厂设置流程**
# 一，Setting发送广播

文件：android/packages/apps/Settings/src/com/android/settings/MasterClearConfirm.java 在settings里面发送广播
```
    private void doMasterClear() {
        Intent intent = new Intent(Intent.ACTION_FACTORY_RESET);
        intent.setPackage("android");
        intent.addFlags(Intent.FLAG_RE![enter description here](./images/mcr.png)CEIVER_FOREGROUND);
        intent.putExtra(Intent.EXTRA_REASON, "MasterClearConfirm");
        intent.putExtra(Intent.EXTRA_WIPE_EXTERNAL_STORAGE, mEraseSdCard);
        intent.putExtra(Intent.EXTRA_WIPE_ESIMS, mEraseEsims);
        getActivity().sendBroadcast(intent);
        // Intent handling is asynchronous -- assume it will happen soon.
    }
```
文件：base/core/java/android/content/Intent.java
```
    @SystemApi
    @SdkConstant(SdkConstantType.BROADCAST_INTENT_ACTION)
    public static final String ACTION_FACTORY_RESET = "android.intent.action.FACTORY_RESET";
```

# 二，接收广播

文件：android/frameworks/base/services/core/java/com/android/server/MasterClearReceiver.java 接收的广播在 android/frameworks/base/core/res/AndroidManifest.xml 中声明
```
        <receiver android:name="com.android.server.MasterClearReceiver"
            android:permission="android.permission.MASTER_CLEAR">
            <intent-filter
                    android:priority="100" >
                <!-- For Checkin, Settings, etc.: action=FACTORY_RESET -->
                <action android:name="android.intent.action.FACTORY_RESET" />
                <!-- As above until all the references to the deprecated MASTER_CLEAR get updated to
                     FACTORY_RESET. -->
                <action android:name="android.intent.action.MASTER_CLEAR" />

                <!-- MCS always uses REMOTE_INTENT: category=MASTER_CLEAR -->
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="android.intent.category.MASTER_CLEAR" />
            </intent-filter>
        </receiver>

接收广播后，会调用RecoverySystem.rebootWipeUserData()进行处理。
```

# 三，接收到广播后的处理

文件： android/frameworks/base/core/java/android/os/RecoverySystem.java android/frameworks/base/services/core/java/com/android/server/RecoverySystemService.java

处理流程如下： rebootWipeUserData() bootCommand() rebootRecoveryWithCommand() setupOrClearBcb() 在setupOrClearBcb()中有两个非常重要的属性设置：
```
if (isSetup) {
	SystemProperties.set("ctl.start", "setup-bcb");
} else {
	SystemProperties.set("ctl.start", "clear-bcb");
}
```			

这两个属性设置后，会启动对应的服务。 android/bootable/recovery/uncrypt/uncrypt.rc
```
service uncrypt /system/bin/uncrypt
    class main
    socket uncrypt stream 600 system system
    disabled
    oneshot

service setup-bcb /system/bin/uncrypt --setup-bcb
    class main
    socket uncrypt stream 600 system system
    disabled
    oneshot

service clear-bcb /system/bin/uncrypt --clear-bcb
    class main
    socket uncrypt stream 600 system system
    disabled
    oneshot
```

看uncrypt.rc文件可知，服务启动时会同时创建socket服务端，socket服务端创建后，就等待连接。在setupOrClearBcb()方法里面设置属性后，会去创建socket的客户端，连接刚刚创建的服务端，如下：
```
            // Connect to the uncrypt service socket.
            LocalSocket socket = connectService();
            if (socket == null) {
                Slog.e(TAG, "Failed to connect to uncrypt socket");
                return false;
            }
```

# 四，uncrypt真正对恢复出厂设置处理

文件：android/bootable/recovery/uncrypt/uncrypt.cpp

```
//bootable/recovery/uncrypt/uncrypt.cpp
int main(int argc, char** argv) {
    // 定义三种action类型
    enum { UNCRYPT, SETUP_BCB, CLEAR_BCB, UNCRYPT_DEBUG } action;
    const char* input_path = nullptr;
    const char* map_file = CACHE_BLOCK_MAP.c_str();
    // rc文件中参数判断
    if (argc == 2 && strcmp(argv[1], "--clear-bcb") == 0) {
        action = CLEAR_BCB;
    } else if (argc == 2 && strcmp(argv[1], "--setup-bcb") == 0) {
        action = SETUP_BCB;
    } else if (argc == 1) {
        action = UNCRYPT;
    } else if (argc == 3) {
        input_path = argv[1];
        map_file = argv[2];
        action = UNCRYPT_DEBUG;
    } else {
        usage(argv[0]);
        return 2;
    }
    // 读取分区文件
    if (!ReadDefaultFstab(&fstab)) {
        LOG(ERROR) << "failed to read default fstab";
        log_uncrypt_error_code(kUncryptFstabReadError);
        return 1;
    }
    if (action == UNCRYPT_DEBUG) {
        LOG(INFO) << "uncrypt called in debug mode, skip socket communication";
        bool success = uncrypt_wrapper(input_path, map_file, -1);
        if (success) {
            LOG(INFO) << "uncrypt succeeded";
        } else{
            LOG(INFO) << "uncrypt failed";
        }
        return success ? 0 : 1;
    }
    // 当启动服务时由init创建该socket。uncrypt将通过socket与其调用者进行通信。
    android::base::unique_fd service_socket(android_get_control_socket(UNCRYPT_SOCKET.c_str()));
    if (service_socket == -1) {
        PLOG(ERROR) << "failed to open socket \"" << UNCRYPT_SOCKET << "\"";
        log_uncrypt_error_code(kUncryptSocketOpenError);
        return 1;
    }
    fcntl(service_socket, F_SETFD, FD_CLOEXEC);
    if (listen(service_socket, 1) == -1) {
        PLOG(ERROR) << "failed to listen on socket " << service_socket.get();
        log_uncrypt_error_code(kUncryptSocketListenError);
        return 1;
    }
    android::base::unique_fd socket_fd(accept4(service_socket, nullptr, nullptr, SOCK_CLOEXEC));
    if (socket_fd == -1) {
        PLOG(ERROR) << "failed to accept on socket " << service_socket.get();
        log_uncrypt_error_code(kUncryptSocketAcceptError);
        return 1;
    }
    bool success = false;
    switch (action) {
        case UNCRYPT:
            success = uncrypt_wrapper(input_path, map_file, socket_fd);
            break;
        case SETUP_BCB:
            success = setup_bcb(socket_fd);
            break;
        case CLEAR_BCB:
            success = clear_bcb(socket_fd);
            break;
        default:
            LOG(ERROR) << "Invalid uncrypt action code: " << action;
            return 1;
    }
    // 在解密退出之前，从客户端读取4字节代码。目的是为了确保客户端在socket被销毁之前接收最后的状态码。
    int code;
    if (android::base::ReadFully(socket_fd, &code, 4)) {
        LOG(INFO) << "  received " << code << ", exiting now";
    } else {
        PLOG(ERROR) << "failed to read the code";
    }
    return success ? 0 : 1;
}
```

关于后面调用pm重启的流程不做过多介绍。