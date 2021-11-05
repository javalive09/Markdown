# **protected-broadcast 使用规范**
# 从一句log说开去

我们在看log的时候经常会看到下面这样的打印：
```
Sending non-protected broadcast xxxxxx from system xxxxxx/1000 pkg xxxxxxxx
    java.lang.Throwable
```
好多应用开发的同学会认为这是个异常，导致他们的广播异常，这里可以说一下，打印这个并不影响广播的使用，但是确实是个“异常”

# 从源码角度分析

我们从源码的角度看下这个“异常”打印的地方，

```
    private void checkBroadcastFromSystem(Intent intent, ProcessRecord callerApp,
            String callerPackage, int callingUid, boolean isProtectedBroadcast, List receivers) {
        if ((intent.getFlags() & Intent.FLAG_RECEIVER_FROM_SHELL) != 0) {
            // Don't yell about broadcasts sent via shell
            return;
        }


        final String action = intent.getAction();
        if (isProtectedBroadcast
                || Intent.ACTION_CLOSE_SYSTEM_DIALOGS.equals(action)
                || Intent.ACTION_DISMISS_KEYBOARD_SHORTCUTS.equals(action)
                || Intent.ACTION_MEDIA_BUTTON.equals(action)
                || Intent.ACTION_MEDIA_SCANNER_SCAN_FILE.equals(action)
                || Intent.ACTION_SHOW_KEYBOARD_SHORTCUTS.equals(action)
                || Intent.ACTION_MASTER_CLEAR.equals(action)
                || Intent.ACTION_FACTORY_RESET.equals(action)
                || AppWidgetManager.ACTION_APPWIDGET_CONFIGURE.equals(action)
                || AppWidgetManager.ACTION_APPWIDGET_UPDATE.equals(action)
                || LocationManager.HIGH_POWER_REQUEST_CHANGE_ACTION.equals(action)
                || TelephonyIntents.ACTION_REQUEST_OMADM_CONFIGURATION_UPDATE.equals(action)
                || SuggestionSpan.ACTION_SUGGESTION_PICKED.equals(action)
                || AudioEffect.ACTION_OPEN_AUDIO_EFFECT_CONTROL_SESSION.equals(action)
                || AudioEffect.ACTION_CLOSE_AUDIO_EFFECT_CONTROL_SESSION.equals(action)) {
            // Broadcast is either protected, or it's a public action that
            // we've relaxed, so it's fine for system internals to send.
            return;
        }

        // This broadcast may be a problem...  but there are often system components that
        // want to send an internal broadcast to themselves, which is annoying to have to
        // explicitly list each action as a protected broadcast, so we will check for that
        // one safe case and allow it: an explicit broadcast, only being received by something
        // that has protected itself.
        if (intent.getPackage() != null || intent.getComponent() != null) {
            if (receivers == null || receivers.size() == 0) {
                // Intent is explicit and there's no receivers.
                // This happens, e.g. , when a system component sends a broadcast to
                // its own runtime receiver, and there's no manifest receivers for it,
                // because this method is called twice for each broadcast,
                // for runtime receivers and manifest receivers and the later check would find
                // no receivers.
                return;
            }
            boolean allProtected = true;
            for (int i = receivers.size()-1; i >= 0; i--) {
                Object target = receivers.get(i);
                if (target instanceof ResolveInfo) {
                    ResolveInfo ri = (ResolveInfo)target;
                    if (ri.activityInfo.exported && ri.activityInfo.permission == null) {
                        allProtected = false;
                        break;
                    }
                } else {
                    BroadcastFilter bf = (BroadcastFilter)target;
                    if (bf.requiredPermission == null) {
                        allProtected = false;
                        break;
                    }
                }
            }
            if (allProtected) {
                // All safe!
                return;
            }
        }

        // The vast majority of broadcasts sent from system internals
        // should be protected to avoid security holes, so yell loudly
        // to ensure we examine these cases.
        if (callerApp != null) {
            Log.wtf(TAG, "Sending non-protected broadcast " + action
                            + " from system " + callerApp.toShortString() + " pkg " + callerPackage,
                    new Throwable());
        } else {
            Log.wtf(TAG, "Sending non-protected broadcast " + action
                            + " from system uid " + UserHandle.formatUid(callingUid)
                            + " pkg " + callerPackage,
                    new Throwable());
        }
    }
```

我们可以看到这个打印是在checkBroadcastFromSystem 函数中，这个函数调用的地方是在broadcastIntentLocked函数中，也就是说在AMS对广播的处理流程中对每个广播都要进程check，但这个check也是有条件的，只有发起方是“system”才会检查，

```
            if (isCallerSystem) {
                checkBroadcastFromSystem(intent, callerApp, callerPackage, callingUid,
                        isProtectedBroadcast, registeredReceivers);
            }

```

那么什么条件下才属于isCallerSystem呢,下面代码写的很清楚这里就不过多解释了，需要注意的是常驻应用也满足条件。
```
final boolean isCallerSystem;
        switch (UserHandle.getAppId(callingUid)) {
            case ROOT_UID:
            case SYSTEM_UID:
            case PHONE_UID:
            case BLUETOOTH_UID:
            case NFC_UID:
            case SE_UID:
                isCallerSystem = true;
                break;
            default:
                isCallerSystem = (callerApp != null) && callerApp.persistent;
                break;
        }

```
# 什么样的广播才算是Protected广播

我们看下代码中的体现
```
        // Verify that protected broadcasts are only being sent by system code,
        // and that system code is only sending protected broadcasts.
        final boolean isProtectedBroadcast;
        try {
            isProtectedBroadcast = AppGlobals.getPackageManager().isProtectedBroadcast(action);
        } catch (RemoteException e) {
            Slog.w(TAG, "Remote exception", e);
            return ActivityManager.BROADCAST_SUCCESS;
        }
```
保护广播为true，一般在下面三种情况中：

1.framework/base/core/res/AndroidManifest.xml中有声明为保护广播

```
     <protected-broadcast android:name="android.intent.action.DATE_CHANGED" />
     <protected-broadcast android:name="android.intent.action.PRE_BOOT_COMPLETED" />
     <protected-broadcast android:name="android.intent.action.BOOT_COMPLETED" />
```
2.如果是phone进程的，一般在Teleservice下的AndroidManifest.xml中声明保护广播

2.如果是系统应用（UID SYSTEM），则可以在系统应用的AndroidManifest.xml里有声明为保护广播。
# 未设置保护的危害

虽然广播正常发送了，不影响广播的作用，但是这样的使用是不安全的，有如下危害：

1.系统组件自定义的广播可能会被恶意软件接收或者发送，导致系统不稳定
2.这个log打印同时会在系统的dropbox下新生成一个wtf的log文件，发多少条这样的广播，就生成多少个这样的文件。而dropbox中默认最多只有1000个文件，再多了就会冲掉旧的文件。dropbox是我们用于分析ANR tombstone等问题的重要log来源，因此会严重影响稳定性同事对该类问题的统计和解决。

为了提高系统的安全性且避免这样的log，系统应用组件当在使用自发自收的广播时，要尽可能使用明确的广播，及指定接收的包名或者组件名，且对广播发送和接收加权限保护。同时这也使我们使用广播更加规范。