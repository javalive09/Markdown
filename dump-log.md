# **用解耦的方式为Service添加dump log功能**
我们知道Binder类提供了dump方法，我可以重写该方法，在其中加入我们想要打印的信息，甚至可以加入一些动态开启关闭调试开关的功能。但是如果service中的信息较为复杂，dump方法会变得十分臃肿，不好阅读及管理，我们可以通过动态注册专门用于dump信息的service来解决这个问题。
# 一.动态注册log service
在service的构造方法中加入动态注册log service的逻辑
```
private static final String CALLBACK = "xxcallback";   
public xxServer(Context context) {
    mContext = context;
    registerstart(context);
     
    // 动态注册log service
    ServiceManager.addService(CALLBACK, new xxCallbackBinder(this), /* allowIsolated= */ false,
            DUMP_FLAG_PRIORITY_HIGH);
    Logs.w(TAG, "xxServer init");
}
```
# 二.在log  service中加入我们想要打印的信息
```
static class xxCallbackBinder extends Binder {
    xxServer mxxServer;
    private final PriorityDump.PriorityDumper mPriorityDumper =
            new PriorityDump.PriorityDumper() {
                @Override
                public void dumpHigh(FileDescriptor fd, PrintWriter pw, String[] args,
                                     boolean asProto) {
                    dump(fd, pw, new String[] {" "}, asProto);
                }
 
                @Override
                public void dump(FileDescriptor fd, PrintWriter pw, String[] args, boolean asProto) {
                    mxxServer.dumpxxCallbackInfo(
                            fd, pw, "  ", args, false, null, asProto);
                }
            };
 
    xxCallbackBinder(xxServer xxServer) {
        mxxServer = xxServer;
    }
 
    @Override
    protected void dump(FileDescriptor fd, PrintWriter pw, String[] args) {
        if (!DumpUtils.checkDumpAndUsageStatsPermission(mxxServer.mContext,
                CALLBACK, pw)) return;
        PriorityDump.dump(mPriorityDumper, fd, pw, args);
    }
}
 
private void dumpxxCallbackInfo(FileDescriptor fd, PrintWriter pw, String prefix,
                              String[] args, boolean brief, PrintWriter categoryPw, boolean asProto) {
    pw.println("mEnableCallbackMethods=");
    pw.println("  " + mEnableCallbacks);
 
    int count = mCallbackList.getRegisteredCallbackCount();
    pw.println("mCallbackList=");
    for(int i = 0; i< count; i++) {
        ICallback iCallback = mCallbackList.getRegisteredCallbackItem(i);
        pw.println("  iCallback=" + iCallback);
        pw.println("  iCallback asBinder =" + iCallback.asBinder());
    }
}
```
# 三.调试
## 1.列出所有的可以dumpsys 信息的service列表
```
adb shell dumpsys -l
```
结果如下：
```
...
wallpaper
webviewupdate
wifi
wificond
wifip2p
wifiscanner
window
xx_audio_service
xx_service
xxcallback // 我们动态添加的service
xxtheme
```
## 2.打印xxcallback 信息
```
adb shell dumpsys xxcallback
```


结果如下：
```
mEnableCallbackMethods=
  {android.os.BinderProxy@2ca844a=[FingerSlideFilter{mBundle=Bundle[{to=80, from=48, rect=Rect(0, 0 - 1920, 720), token=50eaf899-f74e-4904-bcce-47d40e5eef17, distance=100, fingerNum=[I@307dd0c, function=5}]}]}
mCallbackList=
  iCallback=com.xx.utils.ICallback$Stub$Proxy@2faf655
  iCallback asBinder =android.os.BinderProxy@2ca844a
```