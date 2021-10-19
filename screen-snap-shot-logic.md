# **Android 获取及保存截屏的流程**
# 背景
多窗口编辑功能需要时时获取多窗口的截屏图片，用于编辑页的activity截屏显示。
系统截屏是ams对系统应用的开放接口，需要申请权限"android.permission.READ_FRAME_BUFFER"
获取截屏的时序图如下：
# 一. 获取截屏流程
TaskSnapShotCache getSnapshot()主要逻辑为通过二级缓存来存储数据，一级为mRunningCache 容器，二级为磁盘文件
## 1.TaskSnapShotCache.java  
```
private final ArrayMap<Integer, CacheEntry> mRunningCache = new ArrayMap<>();
 
@Nullable TaskSnapshot getSnapshot(int taskId, int userId, boolean restoreFromDisk,
        boolean reducedResolution) {
 
    synchronized (mService.mWindowMap) {
        // Try the running cache.
        final CacheEntry entry = mRunningCache.get(taskId);
        if (entry != null) {
            return entry.snapshot;
        }
    }
 
    // Try to restore from disk if asked.
    if (!restoreFromDisk) { // ams 一
        return null;
    }
    return tryRestoreFromDisk(taskId, userId, reducedResolution);
}
private TaskSnapshot tryRestoreFromDisk(int taskId, int userId, boolean reducedResolution) {
    final TaskSnapshot snapshot = mLoader.loadTask(taskId, userId, reducedResolution);
    if (snapshot == null) {
        return null;
    }
    return snapshot;
}
 
private static final class CacheEntry {
 
    /** The snapshot. */
    final TaskSnapshot snapshot;
 
    /** The app token that was on top of the task when the snapshot was taken */
    final AppWindowToken topApp;
 
    CacheEntry(TaskSnapshot snapshot, AppWindowToken topApp) {
        this.snapshot = snapshot;
        this.topApp = topApp;
    }
}
```

## 2.WindowManagerService.java   restoreFromDisk = true;
```
public TaskSnapshot getTaskSnapshot(int taskId, int userId, boolean reducedResolution) {
    return mTaskSnapshotController.getSnapshot(taskId, userId, true /* restoreFromDisk */,
            reducedResolution);
}
```

## 3.TaskSnapshotLoader.java
```
TaskSnapshot loadTask(int taskId, int userId, boolean reducedResolution) {
    final File protoFile = mPersister.getProtoFile(taskId, userId);
    final File bitmapFile = reducedResolution
            ? mPersister.getReducedResolutionBitmapFile(taskId, userId)
            : mPersister.getBitmapFile(taskId, userId);
    if (bitmapFile == null || !protoFile.exists() || !bitmapFile.exists()) {
        return null;
    }
    try {
        final byte[] bytes = Files.readAllBytes(protoFile.toPath());
        final TaskSnapshotProto proto = TaskSnapshotProto.parseFrom(bytes);
        final Options options = new Options();
        options.inPreferredConfig = Config.HARDWARE;
        final Bitmap bitmap = BitmapFactory.decodeFile(bitmapFile.getPath(), options);
        if (bitmap == null) {
            Slog.w(TAG, "Failed to load bitmap: " + bitmapFile.getPath());
            return null;
        }
        final GraphicBuffer buffer = bitmap.createGraphicBufferHandle();
        if (buffer == null) {
            Slog.w(TAG, "Failed to retrieve gralloc buffer for bitmap: "
                    + bitmapFile.getPath());
            return null;
        }
        return new TaskSnapshot(buffer, proto.orientation,
                new Rect(proto.insetLeft, proto.insetTop, proto.insetRight, proto.insetBottom),
                reducedResolution, reducedResolution ? REDUCED_SCALE : 1f,
                proto.isRealSnapshot, proto.windowingMode, proto.systemUiVisibility,
                proto.isTranslucent);
    } catch (IOException e) {
        Slog.w(TAG, "Unable to load task snapshot data for taskId=" + taskId);
        return null;
    }
}
```

# 二. 存入数据
## 1.从ArrayMap<Integer, CacheEntry> mRunningCache入手
### TaskSnapshotCache.java
```
void putSnapshot(Task task, TaskSnapshot snapshot) {
    final CacheEntry entry = mRunningCache.get(task.mTaskId);
    if (entry != null) {
        mAppTaskMap.remove(entry.topApp);
    }
    final AppWindowToken top = task.getTopChild();
    mAppTaskMap.put(top, task.mTaskId);
    mRunningCache.put(task.mTaskId, new CacheEntry(snapshot, task.getTopChild()));
}
```
### TaskSnapshotController.java
```
void snapshotTasks(ArraySet<Task> tasks) {
    for (int i = tasks.size() - 1; i >= 0; i--) {
        final Task task = tasks.valueAt(i);
        final int mode = getSnapshotMode(task);
        final TaskSnapshot snapshot;
        switch (mode) {
            case SNAPSHOT_MODE_NONE:
                continue;
            case SNAPSHOT_MODE_APP_THEME:
                snapshot = drawAppThemeSnapshot(task);
                break;
            case SNAPSHOT_MODE_REAL:
                snapshot = snapshotTask(task);
                break;
            default:
                snapshot = null;
                break;
        }
        if (snapshot != null) {
            final GraphicBuffer buffer = snapshot.getSnapshot();
            if (buffer.getWidth() == 0 || buffer.getHeight() == 0) {
                buffer.destroy();
                Slog.e(TAG, "Invalid task snapshot dimensions " + buffer.getWidth() + "x"
                        + buffer.getHeight());
            } else {
                mCache.putSnapshot(task, snapshot); // mCache一级缓存
                mPersister.persistSnapshot(task.mTaskId, task.mUserId, snapshot);
                if (task.getController() != null) {
                    task.getController().reportSnapshotChanged(snapshot); // disk二级缓存
                }
            }
        }
    }
} 
```
## 2.三个存入数据的入口：
### 入口1：handleClosingApps 关闭app时保存截屏
```
private void handleClosingApps(ArraySet<AppWindowToken> closingApps) {
    if (shouldDisableSnapshots()) {
        return;
    }
 
    // We need to take a snapshot of the task if and only if all activities of the task are
    // either closing or hidden.
    getClosingTasks(closingApps, mTmpTasks);
    snapshotTasks(mTmpTasks);
    mSkipClosingAppSnapshotTasks.clear();
}
```
### 入口2：screenTurningOff 熄灭屏幕时保存截屏
```
void screenTurningOff(ScreenOffListener listener) {
    if (shouldDisableSnapshots()) {
        listener.onScreenOff();
        return;
    }
 
    // We can't take a snapshot when screen is off, so take a snapshot now!
    mHandler.post(() -> {
        try {
            synchronized (mService.mWindowMap) {
                mTmpTasks.clear();
                mService.mRoot.forAllTasks(task -> {
                    if (task.isVisible()) {
                        mTmpTasks.add(task);
                    }
                });
                snapshotTasks(mTmpTasks);
            }
        } finally {
            listener.onScreenOff();
        }
    });
}
```
### 入口3：RecentsAnimationController.java  最近任务
```
private final IRecentsAnimationController mController =
        new IRecentsAnimationController.Stub() {
 
    @Override
    public TaskSnapshot screenshotTask(int taskId) {
       ...
            return snapshotController.getSnapshot(taskId, 0 /* userId */,
                                false /* restoreFromDisk */, false /* reducedResolution */);
       ...
    }
```
## 3.存入数据的来源  
### TaskSnapshotController.java
```
private TaskSnapshot snapshotTask(Task task) {
    final AppWindowToken top = task.getTopChild();
    if (top == null) {
        return null;
    }
    final WindowState mainWindow = top.findMainWindow();
    if (mainWindow == null) {
        return null;
    }
    ...   
 
    // 从SurfaceControl获取GraphicBuffer
    final GraphicBuffer buffer = SurfaceControl.captureLayers(
            task.getSurfaceControl().getHandle(), mTmpRect, scaleFraction);
    final boolean isWindowTranslucent = mainWindow.getAttrs().format != PixelFormat.OPAQUE;
    if (buffer == null || buffer.getWidth() <= 1 || buffer.getHeight() <= 1) {
        if (DEBUG_SCREENSHOT) {
            Slog.w(TAG_WM, "Failed to take screenshot for " + task);
        }
        return null;
    }
    return new TaskSnapshot(buffer, top.getConfiguration().orientation,
            getInsets(mainWindow), isLowRamDevice /* reduced */, scaleFraction /* scale */,
            true /* isRealSnapshot */, task.getWindowingMode(), getSystemUiVisibility(task),
            !top.fillsParent() || isWindowTranslucent);
}
```

## 4. disk二级缓存的保存逻辑
## TaskSnapshotPersister.java
```
void persistSnapshot(int taskId, int userId, TaskSnapshot snapshot) {
    synchronized (mLock) {
        mPersistedTaskIdsSinceLastRemoveObsolete.add(taskId);
        sendToQueueLocked(new StoreWriteQueueItem(taskId, userId, snapshot));
    }
}
private final ArrayDeque<WriteQueueItem> mWriteQueue = new ArrayDeque<>();
private void sendToQueueLocked(WriteQueueItem item) {
    mWriteQueue.offer(item);
    item.onQueuedLocked();
    ensureStoreQueueDepthLocked();
    if (!mPaused) {
        mLock.notifyAll();
    }
}
private static final int MAX_STORE_QUEUE_DEPTH = 2;
private void ensureStoreQueueDepthLocked() {
    while (mStoreQueueItems.size() > MAX_STORE_QUEUE_DEPTH) {
        final StoreWriteQueueItem item = mStoreQueueItems.poll();
        mWriteQueue.remove(item);
        Slog.i(TAG, "Queue is too deep! Purged item with taskid=" + item.mTaskId);
    }
}
 
private static final long DELAY_MS = 100;
 
private Thread mPersister = new Thread("TaskSnapshotPersister") {
    public void run() {
        android.os.Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        while (true) {
            WriteQueueItem next;
            synchronized (mLock) {
                if (mPaused) {
                    next = null;
                } else {
                    next = mWriteQueue.poll();
                    if (next != null) {
                        next.onDequeuedLocked();
                    }
                }
            }
            if (next != null) {
                next.write();
                SystemClock.sleep(DELAY_MS);
            }
            synchronized (mLock) {
                final boolean writeQueueEmpty = mWriteQueue.isEmpty();
                if (!writeQueueEmpty && !mPaused) {
                    continue;
                }
                try {
                    mQueueIdling = writeQueueEmpty;
                    mLock.wait();
                    mQueueIdling = false;
                } catch (InterruptedException e) {
                }
            }
        }
    }
};
 
 
private class StoreWriteQueueItem extends WriteQueueItem {
    private final int mTaskId;
    private final int mUserId;
    private final TaskSnapshot mSnapshot;
 
    StoreWriteQueueItem(int taskId, int userId, TaskSnapshot snapshot) {
        mTaskId = taskId;
        mUserId = userId;
        mSnapshot = snapshot;
    }
 
    @GuardedBy("mLock")
    @Override
    void onQueuedLocked() {
        mStoreQueueItems.offer(this);
    }
 
    @GuardedBy("mLock")
    @Override
    void onDequeuedLocked() {
        mStoreQueueItems.remove(this);
    }
 
    @Override
    void write() {
        if (!createDirectory(mUserId)) {
            Slog.e(TAG, "Unable to create snapshot directory for user dir="
                    + getDirectory(mUserId));
        }
        boolean failed = false;
        if (!writeProto()) {
            failed = true;
        }
        if (!writeBuffer()) {
            failed = true;
        }
        if (failed) {
            deleteSnapshot(mTaskId, mUserId);
        }
    }
 
    boolean writeProto() {
        final TaskSnapshotProto proto = new TaskSnapshotProto();
        proto.orientation = mSnapshot.getOrientation();
        proto.insetLeft = mSnapshot.getContentInsets().left;
        proto.insetTop = mSnapshot.getContentInsets().top;
        proto.insetRight = mSnapshot.getContentInsets().right;
        proto.insetBottom = mSnapshot.getContentInsets().bottom;
        proto.isRealSnapshot = mSnapshot.isRealSnapshot();
        proto.windowingMode = mSnapshot.getWindowingMode();
        proto.systemUiVisibility = mSnapshot.getSystemUiVisibility();
        proto.isTranslucent = mSnapshot.isTranslucent();
        final byte[] bytes = TaskSnapshotProto.toByteArray(proto);
        final File file = getProtoFile(mTaskId, mUserId);
        final AtomicFile atomicFile = new AtomicFile(file);
        FileOutputStream fos = null;
        try {
            fos = atomicFile.startWrite();
            fos.write(bytes);
            atomicFile.finishWrite(fos);
        } catch (IOException e) {
            atomicFile.failWrite(fos);
            Slog.e(TAG, "Unable to open " + file + " for persisting. " + e);
            return false;
        }
        return true;
    }
 
    boolean writeBuffer() {
        final Bitmap bitmap = Bitmap.createHardwareBitmap(mSnapshot.getSnapshot());
        if (bitmap == null) {
            Slog.e(TAG, "Invalid task snapshot hw bitmap");
            return false;
        }
 
        final Bitmap swBitmap = bitmap.copy(Config.ARGB_8888, false /* isMutable */);
        final File reducedFile = getReducedResolutionBitmapFile(mTaskId, mUserId);
        final Bitmap reduced = mSnapshot.isReducedResolution()
                ? swBitmap
                : Bitmap.createScaledBitmap(swBitmap,
                        (int) (bitmap.getWidth() * REDUCED_SCALE),
                        (int) (bitmap.getHeight() * REDUCED_SCALE), true /* filter */);
        try {
            FileOutputStream reducedFos = new FileOutputStream(reducedFile);
            reduced.compress(JPEG, QUALITY, reducedFos);
            reducedFos.close();
        } catch (IOException e) {
            Slog.e(TAG, "Unable to open " + reducedFile +" for persisting.", e);
            return false;
        }
 
        // For snapshots with reduced resolution, do not create or save full sized bitmaps
        if (mSnapshot.isReducedResolution()) {
            swBitmap.recycle();
            return true;
        }
 
        final File file = getBitmapFile(mTaskId, mUserId);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            swBitmap.compress(JPEG, QUALITY, fos);
            fos.close();
        } catch (IOException e) {
            Slog.e(TAG, "Unable to open " + file + " for persisting.", e);
            return false;
        }
        reduced.recycle();
        swBitmap.recycle();
        return true;
    }
```

# 三.清理数据
## 1.清理mRunningCache一级缓存
### TaskSnapshotCache.java
```
private void removeRunningEntry(int taskId) {
    final CacheEntry entry = mRunningCache.get(taskId);
    if (entry != null) {
        mAppTaskMap.remove(entry.topApp);
        mRunningCache.remove(taskId);
    }
}
 
//1. recentTask手动删除task
void onTaskRemoved(int taskId) {
    removeRunningEntry(taskId);
}
//2. app died
void onAppDied(AppWindowToken wtoken) {
    final Integer taskId = mAppTaskMap.get(wtoken);
    if (taskId != null) {
        removeRunningEntry(taskId);
    }
}
//3.app token has been removed （触发activity ondestory）
void onAppRemoved(AppWindowToken wtoken) {
    final Integer taskId = mAppTaskMap.get(wtoken);
    if (taskId != null) {
        removeRunningEntry(taskId);
    }
}
```
## 2.清理disk 二级缓存
```
//自动删除class RemoveObsoleteFilesQueueItem extends WriteQueueItem {
...
 @Override
 void write() {
 final ArraySet<Integer> newPersistedTaskIds;
 synchronized (mLock) {
 newPersistedTaskIds = new ArraySet<>(mPersistedTaskIdsSinceLastRemoveObsolete);
 }
 for (int userId : mRunningUserIds) {
 final File dir = getDirectory(userId);
 final String[] files = dir.list();
 if (files == null) {
 continue;
 }
 for (String file : files) {
 final int taskId = getTaskId(file);
 if (!mPersistentTaskIds.contains(taskId)
 && !newPersistedTaskIds.contains(taskId)) {
 new File(dir, file).delete();
 }
 }
 }
 }//  手动在任务列表删除private class DeleteWriteQueueItem extends WriteQueueItem {
...
 @Override
 void write() {
 deleteSnapshot(mTaskId, mUserId);
 }
}
```

## 3.自动清理的逻辑
### TaskPersister.java
```
private class LazyTaskWriterThread extends Thread {
    ...
    @Override
    public void run() {
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        ArraySet<Integer> persistentTaskIds = new ArraySet<>();
        while (true) {
            ...
                    mRecentTasks.getPersistableTaskIds(persistentTaskIds); //获取persistent task info
                    mService.mWindowManager.removeObsoleteTaskFiles(persistentTaskIds,
                            mRecentTasks.usersWithRecentsLoadedLocked()); //自动清理
            ...
        }
    }
```
### RecentTasks.java
```
void getPersistableTaskIds(ArraySet<Integer> persistentTaskIds) {
    final int size = mTasks.size();
    for (int i = 0; i < size; i++) {
        ...
        if ((task.isPersistable || task.inRecents) // 在recent task 列表 或 isPersistable
                && (stack == null || !stack.isHomeOrRecentsStack())) {
            if (TaskPersister.DEBUG) Slog.d(TAG, "adding to persistentTaskIds task=" + task);
            persistentTaskIds.add(task.taskId);
        }
        ...
    }
}
```
## isPersistable() 逻辑，ActivityRecord.java
```
boolean isPersistable() {
    return (info.persistableMode == PERSIST_ROOT_ONLY ||
            info.persistableMode == PERSIST_ACROSS_REBOOTS) &&
            (intent == null || (intent.getFlags() & FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS) == 0);
}
 
android:persistableMode="persistRootOnly" // 仅仅会作用在根activity
android:persistableMode="persistAcrossReboots" // 设备重启
```
# 四.调试
## 1. mRunningCache 一级缓存查看
```
adb shell dumpsys window windows | grep  SnapshotCache -A 10
SnapshotCache
  Entry taskId=34
    topApp=AppWindowToken{dbeda88 token=Token{7a6b62b ActivityRecord{989fe7a u0 com.tencent.wecarnavi/.BriefActivity t34}}}
    snapshot=TaskSnapshot{mSnapshot=android.graphics.GraphicBuffer@e03d1cb (592x520) mOrientation=2 mContentInsets=[0,0][0,0]
```
## 2.  disk 二级缓存查看
```
adb shell ls /data/system_ce/0/snapshots

peter@peter:~$ adb shell ls /data/system_ce/0/snapshots
34.jpg
34.proto
34_reduced.jpg
35.jpg
35.proto
35_reduced.jpg
```
# 五.抛砖引玉

1. surfaceFilinger中GraphicBuffer获取逻辑

2.  在一级缓存，二级缓存都是null的情况下，如何保证时时获取截屏
