# **Android Q 分流广播队列 介绍**

# 官方简介

Android 10 在现有的后台和前台队列中添加了一个新的分流 BroadcastQueue。分流队列具有与后台队列相同的优先级和超时行为。为了防止屏蔽后台队列（该队列中可能会出现更加有趣或用户可见的广播），分流队列会处理 BOOT_COMPLETED 广播（该广播受到多个应用的监听并且需要很长时间才能完成）。分流队列目前仅处理 BOOT_COMPLETED 广播，但可能会处理其他较长的广播。
# 代码体现
## 定义
```
    BroadcastQueue mFgBroadcastQueue;
    BroadcastQueue mBgBroadcastQueue;
    BroadcastQueue mOffloadBroadcastQueue;
```
## 初始化
```
        mFgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "foreground", foreConstants, false);
        mBgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "background", backConstants, true);
        mOffloadBroadcastQueue = new BroadcastQueue(this, mHandler,
                "offload", offloadConstants, true)；

        final BroadcastConstants offloadConstants = new BroadcastConstants(
                Settings.Global.BROADCAST_OFFLOAD_CONSTANTS);
        offloadConstants.TIMEOUT = BROADCAST_BG_TIMEOUT;
        // by default, no "slow" policy in this queue
        offloadConstants.SLOW_TIME = Integer.MAX_VALUE;
        mEnableOffloadQueue = SystemProperties.getBoolean(        
"persist.device_config.activity_manager_native_boot.offload_queue_enabled", false);
```
# 怎么用

intent添加如下flag 就会使用分流队列
```
    /**
     * If set, when sending a broadcast the recipient will be run on the offload queue.
     *
     * @hide
     */
    public static final int FLAG_RECEIVER_OFFLOAD = 0x80000000;
```
比如Android Q中的开机广播
```
final Intent bootIntent = new Intent(Intent.ACTION_BOOT_COMPLETED, null);
        bootIntent.putExtra(Intent.EXTRA_USER_HANDLE, userId);
        bootIntent.addFlags(Intent.FLAG_RECEIVER_NO_ABORT
                | Intent.FLAG_RECEIVER_INCLUDE_BACKGROUND
                | Intent.FLAG_RECEIVER_OFFLOAD);
```                



