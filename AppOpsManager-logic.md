# **悬浮窗权限分析**

# 悬浮窗权限检查
应用在使用悬浮窗之前,要先检查是否有悬浮窗权限,调用Settings.canDrawOverlays(),最后的检查代码如下: 总结这部分,可知可以分两部分:

    通过AppOpsManager进行check.
    如果不是返回AppOpsManager.MODE_ALLOWED, 而是AppOpsManager.MODE_DEFAULT,就调用checkCallingOrSelfPermission()进行check,这个方法是Android最常用的权限检查,最终会走到PMS里面进行检查.

通过以上分析可知:对悬浮窗的权限检查以AppOpsManager为准,PMS的检查只是补充,并且从目前的情况来看,没有入口去赋予PMS的悬浮窗权限.设置界面的入口是直接调用的AppOpsManager的接口.

```
 /**
     * Helper method to perform a general and comprehensive check of whether an operation that is
     * protected by appops can be performed by a caller or not. e.g. OP_SYSTEM_ALERT_WINDOW and
     * OP_WRITE_SETTINGS
     * @hide
     */
    public static boolean isCallingPackageAllowedToPerformAppOpsProtectedOperation(Context context,
            int uid, String callingPackage, boolean throwException, int appOpsOpCode, String[]
            permissions, boolean makeNote) {
        if (callingPackage == null) {
            return false;
        }

        AppOpsManager appOpsMgr = (AppOpsManager)context.getSystemService(Context.APP_OPS_SERVICE);
        int mode = AppOpsManager.MODE_DEFAULT;
        if (makeNote) {
            mode = appOpsMgr.noteOpNoThrow(appOpsOpCode, uid, callingPackage);
        } else {
            mode = appOpsMgr.checkOpNoThrow(appOpsOpCode, uid, callingPackage);
        }

        switch (mode) {
            case AppOpsManager.MODE_ALLOWED:
                return true;

            case AppOpsManager.MODE_DEFAULT:
                // this is the default operating mode after an app's installation
                // In this case we will check all associated static permission to see
                // if it is granted during install time.
                for (String permission : permissions) {
                    if (context.checkCallingOrSelfPermission(permission) == PackageManager
                            .PERMISSION_GRANTED) {
                        // if either of the permissions are granted, we will allow it
                        return true;
                    }
                }

            default:
                // this is for all other cases trickled down here...
                if (!throwException) {
                    return false;
                }
        }

        // prepare string to throw SecurityException
        StringBuilder exceptionMessage = new StringBuilder();
        exceptionMessage.append(callingPackage);
        exceptionMessage.append(" was not granted ");
        if (permissions.length > 1) {
            exceptionMessage.append(" either of these permissions: ");
        } else {
            exceptionMessage.append(" this permission: ");
        }
        for (int i = 0; i < permissions.length; i++) {
            exceptionMessage.append(permissions[i]);
            exceptionMessage.append((i == permissions.length - 1) ? "." : ", ");
        }

        throw new SecurityException(exceptionMessage.toString());
    }
```

# 悬浮窗权限校验

在增加窗口的时候,会由WindowManagerService调用到PhoneWindowManager的checkAddPermission()方法进行权限校验. 从代码可知道,如果权限是TYPE_APPLICATION_OVERLAY(也就是SYSTEM_ALERT_WINDOW),会调用AppOpsManager的noteOpNoThrow()方法.如果返回的AppOpsManager.MODE_ALLOWED和AppOpsManager.MODE_IGNORED,就放过.如果返回的AppOpsManager.MODE_DEFAULT,就继续校验的PMS的悬浮窗权限.
```
/** {@inheritDoc} */
    @Override
    public int checkAddPermission(WindowManager.LayoutParams attrs, int[] outAppOp) {
        final int type = attrs.type;
        final boolean isRoundedCornerOverlay =
                (attrs.privateFlags & PRIVATE_FLAG_IS_ROUNDED_CORNERS_OVERLAY) != 0;

        if (isRoundedCornerOverlay && mContext.checkCallingOrSelfPermission(INTERNAL_SYSTEM_WINDOW)
                != PERMISSION_GRANTED) {
            return ADD_PERMISSION_DENIED;
        }

        outAppOp[0] = AppOpsManager.OP_NONE;

        if (!((type >= FIRST_APPLICATION_WINDOW && type <= LAST_APPLICATION_WINDOW)
                || (type >= FIRST_SUB_WINDOW && type <= LAST_SUB_WINDOW)
                || (type >= FIRST_SYSTEM_WINDOW && type <= LAST_SYSTEM_WINDOW))) {
            return WindowManagerGlobal.ADD_INVALID_TYPE;
        }

        if (type < FIRST_SYSTEM_WINDOW || type > LAST_SYSTEM_WINDOW) {
            // Window manager will make sure these are okay.
            return ADD_OKAY;
        }

        if (!isSystemAlertWindowType(type)) {
            switch (type) {
                case TYPE_TOAST:
                    // Only apps that target older than O SDK can add window without a token, after
                    // that we require a token so apps cannot add toasts directly as the token is
                    // added by the notification system.
                    // Window manager does the checking for this.
                    outAppOp[0] = OP_TOAST_WINDOW;
                    return ADD_OKAY;
                case TYPE_DREAM:
                case TYPE_INPUT_METHOD:
                case TYPE_WALLPAPER:
                case TYPE_PRESENTATION:
                case TYPE_PRIVATE_PRESENTATION:
                case TYPE_VOICE_INTERACTION:
                case TYPE_ACCESSIBILITY_OVERLAY:
                case TYPE_QS_DIALOG:
                    // The window manager will check these.
                    return ADD_OKAY;
            }
            return mContext.checkCallingOrSelfPermission(INTERNAL_SYSTEM_WINDOW)
                    == PERMISSION_GRANTED ? ADD_OKAY : ADD_PERMISSION_DENIED;
        }

        // Things get a little more interesting for alert windows...
        outAppOp[0] = OP_SYSTEM_ALERT_WINDOW;

        final int callingUid = Binder.getCallingUid();
        // system processes will be automatically granted privilege to draw
        if (UserHandle.getAppId(callingUid) == Process.SYSTEM_UID) {
            return ADD_OKAY;
        }

        ApplicationInfo appInfo;
        try {
            appInfo = mContext.getPackageManager().getApplicationInfoAsUser(
                            attrs.packageName,
                            0 /* flags */,
                            UserHandle.getUserId(callingUid));
        } catch (PackageManager.NameNotFoundException e) {
            appInfo = null;
        }

        if (appInfo == null || (type != TYPE_APPLICATION_OVERLAY && appInfo.targetSdkVersion >= O)) {
            /**
             * Apps targeting >= {@link Build.VERSION_CODES#O} are required to hold
             * {@link android.Manifest.permission#INTERNAL_SYSTEM_WINDOW} (system signature apps)
             * permission to add alert windows that aren't
             * {@link android.view.WindowManager.LayoutParams#TYPE_APPLICATION_OVERLAY}.
             */
            return (mContext.checkCallingOrSelfPermission(INTERNAL_SYSTEM_WINDOW)
                    == PERMISSION_GRANTED) ? ADD_OKAY : ADD_PERMISSION_DENIED;
        }

        // check if user has enabled this operation. SecurityException will be thrown if this app
        // has not been allowed by the user
        final int mode = mAppOpsManager.noteOpNoThrow(outAppOp[0], callingUid, attrs.packageName);
        switch (mode) {
            case AppOpsManager.MODE_ALLOWED:
            case AppOpsManager.MODE_IGNORED:
                // although we return ADD_OKAY for MODE_IGNORED, the added window will
                // actually be hidden in WindowManagerService
                return ADD_OKAY;
            case AppOpsManager.MODE_ERRORED:
                // Don't crash legacy apps
                if (appInfo.targetSdkVersion < M) {
                    return ADD_OKAY;
                }
                return ADD_PERMISSION_DENIED;
            default:
                // in the default mode, we will make a decision here based on
                // checkCallingPermission()
                return (mContext.checkCallingOrSelfPermission(SYSTEM_ALERT_WINDOW)
                        == PERMISSION_GRANTED) ? ADD_OKAY : ADD_PERMISSION_DENIED;
        }
    }

```

# AppOps说明

appops是在现有权限机制上新增的一套权限管理机制，主要针对一些高危的非必须系统应用的权限，比如在其他应用上显示悬浮窗。
# AppOpsService初始化

AppOpsService的整个初始化和结束流程依赖于AMS. 在ActivityManagerService构造函数中new AppOpsService,代码如下:
```
mAppOpsService = mInjector.getAppOpsService(new File(systemDir, "appops.xml"), mHandler);
```
此处,同时把/data/system/appops.xml文件和 AMS的mHandler 传入. appops.xml文件主要记录各应用权限情况.也就是这个文件记录了各个应用权限授予的情况. 当然,AppOpsService初始化时,会把appops.xml文件的内容读取到内存中.

```
public void setSystemProcess() {

        ...

                // Start watching app ops after we and the package manager are up and running.
                mAppOpsService.startWatchingMode(AppOpsManager.OP_RUN_IN_BACKGROUND, null,
                        new IAppOpsCallback.Stub() {
                            @Override public void opChanged(int op, int uid, String packageName) {
                                if (op == AppOpsManager.OP_RUN_IN_BACKGROUND && packageName != null) {
                                    if (mAppOpsService.checkOperation(op, uid, packageName)
                                            != AppOpsManager.MODE_ALLOWED) {
                                        runInBackgroundDisabled(uid);
                                    }
                                }
                            }
                        });

    }


    //在AMS的start中发布服务
    private void start() {

    ...
    mAppOpsService.publish(mContext);
    ...

    }

    //在关机时,调用shutdown,持久化appops.xml文件
    @Override
    public boolean shutdown(int timeout) {

        ...
        mAppOpsService.shutdown();
        ...
    }
    //在AMS的systemReady中调用AppOpsService的systemReady(),做一些初始化工作.
    public void systemReady(final Runnable goingCallback, TimingsTraceLog traceLog) {
        ...
        mAppOpsService.systemReady();
        ...
    }
```

# appops.xml文件解析和字段说明

NA
# AppOpsManager说明

AppOpsManager定义了Ops权限的值以及默认状态, 还有,就是通过bind把相关的调用传给AppOpsService. 以SYSTEM_ALERT_WINDOW权限为例,说明下:

```
    //定义值
    @TestApi
    public static final int OP_SYSTEM_ALERT_WINDOW = 24;

        /** Required to draw on top of other apps. */
    public static final String OPSTR_SYSTEM_ALERT_WINDOW
            = "android:system_alert_window";
    //权限定义
    /** Required to draw on top of other apps. */
    public static final String OPSTR_SYSTEM_ALERT_WINDOW
            = "android:system_alert_window";

     /**
     * This maps each operation to the operation that serves as the
     * switch to determine whether it is allowed.  Generally this is
     * a 1:1 mapping, but for some things (like location) that have
     * multiple low-level operations being tracked that should be
     * presented to the user as one switch then this can be used to
     * make them all controlled by the same single operation.
     */
    private static int[] sOpToSwitch = new int[] {
        ...
        //此处外部传递过来的值, 转换为ops定义的值
        OP_SYSTEM_ALERT_WINDOW,             // SYSTEM_ALERT_WINDOW
        ...
    }

    //此处定义了权限的默认状态,比如OP_SYSTEM_ALERT_WINDOW的默认状态定义的为AppOpsManager.MODE_DEFAULT,
    //如果改为AppOpsManager.MODE_ALLOWED, 那么悬浮窗的权限默认是允许的. 就不用再去手动赋予
    /**
     * This specifies the default mode for each operation.
     */
    private static int[] sOpDefaultMode = new int[] {
        ...

        AppOpsManager.MODE_DEFAULT, // OP_SYSTEM_ALERT_WINDOW
        ...
    }
```

# AppOpsService说明
## api接口说明

AppOpsManager提供标准的API供APP调用，但google有明确说明，大部分只针对系统应用。但是想使用的话，可以尝试把Android源码里AppOpsManager.java打包一下，把jar包导入自己的工程，就可以使用了。

int checkOp(Stringop, int uid,StringpackageName)

Op对应一个权限操作，该接口来检测应用是否具有该项操作权限。

int noteOp(Stringop, int uid,StringpackageName)

和checkOp基本相同，但是在检验后会做记录。

int checkOpNoThrow(Stringop, int uid,StringpackageName)

和checkOp类似，但是权限错误，不会抛出SecurityException，而是返回AppOpsManager.MODE_ERRORED。

int noteOpNoThrow(Stringop, int uid,StringpackageName)

类似noteOp，但不会抛出SecurityException。

void setMode( int code, int uid, String packageName, int mode)

code代表具体的操作权限，mode代表要更改成的类型（允许/禁止/提示）

比较关键的就是这个setMode方法，如果要通过代码设置权限, 就需要调用这个方法.

## 通过命令设置权限
```
adb shell appops set xxx包名 SYSTEM_ALERT_WINDOW allow

adb shell appops set xxx包名 READ_EXTERNAL_STORAGE ignore
```
