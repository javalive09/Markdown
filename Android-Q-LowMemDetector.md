# **Android Q LowMemDetector**

# 简介

android Q 我们在AMS中看到了一个新的类，名字叫LowMemDetector，从字面意思看就是低内存探测器，我们来看下它的实现。
```
/**
 * Detects low memory using PSI.
 *
 * If the kernel doesn't support PSI, then this class is not available.
 */
```
# 启动

在AMS构造函数中创建对象：
```
public ActivityManagerService(Context systemContext, ActivityTaskManagerService atm){
     //省略部分代码
    mLowMemDetector = new LowMemDetector(this);
     //省略部分代码
 }
```

构造函数：
```
    LowMemDetector(ActivityManagerService am) {
        mAm = am;
        mLowMemThread = new LowMemThread();
        if (init() != 0) {
            mAvailable = false;
        } else {
            mAvailable = true;
            mLowMemThread.start();
        }
    }
```
首先创建LowMemThread，然后执行init操作，并启动LowMemThread：

```
    private final class LowMemThread extends Thread {
        public void run() {


            while (true) {
                // sleep waiting for a PSI event
                int newPressureState = waitForPressure();
                if (newPressureState == -1) {
                    // epoll broke, tear this down
                    mAvailable = false;
                    break;
                }
                // got a PSI event? let's update lowmem info
                synchronized (mPressureStateLock) {
                    mPressureState = newPressureState;
                }
            }
        }
    }
```

init操作的实现在native层，主要是epoll机制监控内核节点上报的psi信息：

```
// amount of stall in us for each level
static constexpr int PSI_LOW_STALL_US = 15000;
static constexpr int PSI_MEDIUM_STALL_US = 30000;
static constexpr int PSI_HIGH_STALL_US = 50000;

static jint android_server_am_LowMemDetector_init(JNIEnv*, jobject) {
    int epollfd;
    int low_psi_fd;
    int medium_psi_fd;
    int high_psi_fd;

    epollfd = epoll_create(PRESSURE_LEVEL_COUNT);
    if (epollfd == -1) {
        ALOGE("epoll_create failed: %s", strerror(errno));
        return -1;
    }

    low_psi_fd = init_psi_monitor(PSI_SOME, PSI_LOW_STALL_US, PSI_WINDOW_SIZE_US);
    if (low_psi_fd < 0 ||
        register_psi_monitor(epollfd, low_psi_fd, (void*)PRESSURE_LOW) != 0) {
        goto low_fail;
    }

    medium_psi_fd =
        init_psi_monitor(PSI_FULL, PSI_MEDIUM_STALL_US, PSI_WINDOW_SIZE_US);
    if (medium_psi_fd < 0 || register_psi_monitor(epollfd, medium_psi_fd,
                                                  (void*)PRESSURE_MEDIUM) != 0) {
        goto medium_fail;
    }

    high_psi_fd =
        init_psi_monitor(PSI_FULL, PSI_HIGH_STALL_US, PSI_WINDOW_SIZE_US);
    if (high_psi_fd < 0 ||
        register_psi_monitor(epollfd, high_psi_fd, (void*)PRESSURE_HIGH) != 0) {
        goto high_fail;
    }

    psi_epollfd = epollfd;
    return 0;

high_fail:
    unregister_psi_monitor(epollfd, medium_psi_fd);
medium_fail:
    unregister_psi_monitor(epollfd, low_psi_fd);
low_fail:
    ALOGE("Failed to register psi trigger");
    close(epollfd);
    return -1;
}
```
init_psi_monitor和register_psi_monitor的实现都在psi.c中：
```
int init_psi_monitor(enum psi_stall_type stall_type,
             int threshold_us, int window_us) {
    int fd;
    int res;
    char buf[256];

    fd = TEMP_FAILURE_RETRY(open(PSI_MON_FILE_MEMORY, O_WRONLY | O_CLOEXEC));
    if (fd < 0) {
        ALOGE("No kernel psi monitor support (errno=%d)", errno);
        return -1;
    }

    switch (stall_type) {
    case (PSI_SOME):
    case (PSI_FULL):
        res = snprintf(buf, sizeof(buf), "%s %d %d",
            stall_type_name[stall_type], threshold_us, window_us);
        break;
    default:
        ALOGE("Invalid psi stall type: %d", stall_type);
        errno = EINVAL;
        goto err;
    }

    if (res >= (ssize_t)sizeof(buf)) {
        ALOGE("%s line overflow for psi stall type '%s'",
            PSI_MON_FILE_MEMORY, stall_type_name[stall_type]);
        errno = EINVAL;
        goto err;
    }

    res = TEMP_FAILURE_RETRY(write(fd, buf, strlen(buf) + 1));
    if (res < 0) {
        ALOGE("%s write failed for psi stall type '%s'; errno=%d",
            PSI_MON_FILE_MEMORY, stall_type_name[stall_type], errno);
        goto err;
    }

    return fd;

err:
    close(fd);
    return -1;
}

int register_psi_monitor(int epollfd, int fd, void* data) {
    int res;
    struct epoll_event epev;

    epev.events = EPOLLPRI;
    epev.data.ptr = data;
    res = epoll_ctl(epollfd, EPOLL_CTL_ADD, fd, &epev);
    if (res < 0) {
        ALOGE("epoll_ctl for psi monitor failed; errno=%d", errno);
    }
    return res;
}
```

init成功之后mLowMemThread就不断的等待上报内存压力值：
```
static jint android_server_am_LowMemDetector_waitForPressure(JNIEnv*, jobject) {
    static uint32_t pressure_level = PRESSURE_NONE;
    struct epoll_event events[PRESSURE_LEVEL_COUNT];
    int nevents = 0;

    if (psi_epollfd < 0) {
        ALOGE("Memory pressure detector is not initialized");
        return -1;
    }

    do {
        if (pressure_level == PRESSURE_NONE) {
            /* Wait for events with no timeout */
            nevents = epoll_wait(psi_epollfd, events, PRESSURE_LEVEL_COUNT, -1);
        } else {
            // This is simpler than lmkd. Assume that the memory pressure
            // state will stay high for at least 1s. Within that 1s window,
            // the memory pressure state can go up due to a different FD
            // becoming available or it can go down when that window expires.
            // Accordingly, there's no polling: just epoll_wait with a 1s timeout.
            nevents = epoll_wait(psi_epollfd, events, PRESSURE_LEVEL_COUNT, 1000);
            if (nevents == 0) {
                pressure_level = PRESSURE_NONE;
                return pressure_level;
            }
        }
        // keep waiting if interrupted
    } while (nevents == -1 && errno == EINTR);

    if (nevents == -1) {
        ALOGE("epoll_wait failed while waiting for psi events: %s", strerror(errno));
        return -1;
    }

    // reset pressure_level and raise it based on received events
    pressure_level = PRESSURE_NONE;
    for (int i = 0; i < nevents; i++) {
        if (events[i].events & (EPOLLERR | EPOLLHUP)) {
            // should never happen unless psi got disabled in kernel
            ALOGE("Memory pressure events are not available anymore");
            return -1;
        }
        // record the highest reported level
        if (events[i].data.u32 > pressure_level) {
            pressure_level = events[i].data.u32;
        }
    }

    return pressure_level;
}
```


# android系统对PSI的应用

既然可以通过PSI上报给android系统压力值，那么android系统是如何应用的呢？针对不同的内存压力值有什么不同的策略呢？ 我们看下这个接口调用的地方：
```
/**
     * Returns the current mem factor.
     * Note that getMemFactor returns LowMemDetector.MEM_PRESSURE_XXX
     * which match ProcessStats.ADJ_MEM_FACTOR_XXX values. If they deviate
     * there should be conversion performed here to translate pressure state
     * into memFactor.
     */
    public int getMemFactor() {
        synchronized (mPressureStateLock) {
            return mPressureState;
        }
    }
```
这个函数的调用地方只有一个就是在updateLowMemStateLocked中，调用流程是从updateOomAdjLocked调用过来的：
```
 updateOomAdjLocked
    updateLowMemStateLocked
        getMemFactor
```

PRESSURE和ADJ_MEM_FACTOR对应关系：
```
    /* getPressureState return values */
    public static final int MEM_PRESSURE_NONE = 0;
    public static final int MEM_PRESSURE_LOW = 1;
    public static final int MEM_PRESSURE_MEDIUM = 2;
    public static final int MEM_PRESSURE_HIGH = 3;

    public static final int ADJ_MEM_FACTOR_NORMAL = 0;
    public static final int ADJ_MEM_FACTOR_MODERATE = 1;
    public static final int ADJ_MEM_FACTOR_LOW = 2;
    public static final int ADJ_MEM_FACTOR_CRITICAL = 3;
    public static final int ADJ_MEM_FACTOR_COUNT = ADJ_MEM_FACTOR_CRITICAL+1;
```

从代码逻辑上看这个值影响的是app的trimMemoryLevel回调，应用可以根据这个回调对自己的内存进行管理。
```
          int fgTrimLevel;
            switch (memFactor) {
                case ProcessStats.ADJ_MEM_FACTOR_CRITICAL:
                    fgTrimLevel = ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL;
                    break;
                case ProcessStats.ADJ_MEM_FACTOR_LOW:
                    fgTrimLevel = ComponentCallbacks2.TRIM_MEMORY_RUNNING_LOW;
                    break;
                default:
                    fgTrimLevel = ComponentCallbacks2.TRIM_MEMORY_RUNNING_MODERATE;
                    break;
            }
```