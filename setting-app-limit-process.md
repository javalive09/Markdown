# **Setting App 中对后台进程限制的实现**

# 功能描述

在Android的开发者选项中，APP栏中有个选项是：Background process limit，这个选项可以限制后台进程，现在顺着调用的接口看一下具体限制的哪些进程，做了哪些操作。
# 源码分析

在Setting中调用的是AMS中的setProcessLimit()方法来设置的后台进程值，代码如下：
```
8675 @Override

8676 public void setProcessLimit(int max) {

8677 //权限检查

8678 enforceCallingPermission(android.Manifest.permission.SET_PROCESS_LIMIT,

8679 "setProcessLimit()");

8680 final int callingUid = Binder.getCallingUid();

8681 Slog.d("fhw", "setProcessLimit, callingUid = "+callingUid+", max = "+max);

8682 synchronized (this) {

8683 //设置一些值，这些在dump()的时候会打印

8684 mConstants.setOverrideMaxCachedProcesses(max);

8685 }

8686 trimApplications();

8687 }

25806 final void trimApplications() {

25807 synchronized (this) {

25808 trimApplicationsLocked();

25809 }

25810 }
```

设置完值后，会通过这个值把缓存的进程数，空进程数等重新计算一下，其他地方会用到这个计算的值，比如updateOomAdjLocked方法，然后调用trimApplicationsLocked方法把空进程给Kill掉，以达到精简进程。
```
25812 final void trimApplicationsLocked() {

25813 // First remove any unused application processes whose package

25814 // has been removed.

25815 // 遍历mRemovedProcesses进程，kill掉空进程

25816 for (int i=mRemovedProcesses.size()-1; i>=0; i--) {

25817 final ProcessRecord app = mRemovedProcesses.get(i);

25818 //只有满足这四个条件，才会被kill掉

25819 if (app.activities.size() == 0 && app.recentTasks.size() == 0

25820 && app.curReceivers.isEmpty() && app.services.size() == 0) {

25821 Slog.i(

25822 TAG+"_fhw", "Exiting empty application process "

25823 + app.toShortString() + " ("

25824 + (app.thread != null ? app.thread.asBinder() : null)

25825 + ")\n");

25826 if (app.pid > 0 && app.pid != MY_PID) {

25827 app.kill("empty", false);

25828 } else if (app.thread != null) {

25829 try {

25830 app.thread.scheduleExit();

25831 } catch (Exception e) {

25832 // Ignore exceptions.

25833 }

25834 }

25835 cleanUpApplicationRecordLocked(app, false, true, -1, false /*replacingPid*/);

25836 mRemovedProcesses.remove(i);

25837 //如果是persistent进程，重新启动

25838 if (app.persistent) {

25839 addAppLocked(app.info, null, false, null /* ABI override */);

25840 }

25841 }

25842 }

25843

25844 // Now update the oom adj for all processes. Don't skip this, since other callers

25845 // might be depending on it.

25846 // 更新进程的adj

25847 updateOomAdjLocked();

25848 }
```

# 总结

    设置后台进程的限制值后，并没有立即去根据这个值比对后台进程，把空进程/缓存进程kill掉；而是把值保存起来，下次更新进程状态的时候才用到相关的值。

    设置值后，会调用trimApplicationsLocked精简进程，精简进程的过程中，persistent进程会重启。

    精简进程后，会更新进程adj值。
