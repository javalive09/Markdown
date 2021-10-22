# **surfaceflinger问题排查流程**
# 问题描述

系统表现不好，冷启动的时间较长，只有一个空Activity的demo冷启动时间大概1.2S左右，所以怀疑系统存在问题。

# 排查问题 

## 查看系统的可用memory是否过低


### 1. 执行命令：adb shell dumpsys meminfo

```
Total RAM: 3,903,372K (status normal)

Free RAM: 2,589,911K ( 129,119K cached pss + 671,724K cached kernel + 1,789,068K free)

Used RAM: 1,443,253K (1,031,393K used pss + 411,860K kernel)

Lost RAM: -129,792K

Tuning: 192 (large 512), oom 322,560K, restore limit 107,520K (high-end-gfx)

```
可知整个系统的可用内存还有2.5G以上，内存足够

### 2. 获取系统内存使用情况还有如下方式：
adb shell cat /proc/meminfo
```
fhw@henryfeng:~$ adb shell cat /proc/meminfo

MemTotal: 3903372 kB

MemFree: 1791144 kB

MemAvailable: 2882456 kB

Buffers: 16524 kB

Cached: 1124160 kB

SwapCached: 0 kB

Active: 1204364 kB

Inactive: 594080 kB

Active(anon): 660600 kB
```
adb shell vmstat 1
```
fhw@henryfeng:~$ adb shell vmstat 1

procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----

r b swpd free buff cache si so bi bo in cs us sy id wa

1 0 0 1790524 16604 1124220 0 0 147 11 0 2500 15 7 79 0

2 0 0 1790028 16604 1124232 0 0 0 0 0 8500 15 6 79 0

4 0 0 1789904 16604 1124232 0 0 0 16 0 8075 12 6 82 0

0 0 0 1789904 16604 1124236 0 0 0 0 0 8237 14 6 80 0

0 0 0 1789904 16604 1124236 0 0 0 0 0 8728 15 5 80 0

3 0 0 1789904 16612 1124228 0 0 0 52 0 8558 14 6 80 0

0 0 0 1789408 16620 1124232 0 0 0 56 0 8318 15 8 76 0
```

## 查看系统的cpu是否过高
### 通过adb shell进入终端，执行
```
top -d 1 -m 20
```
可知，整个系统的cpu占用不算高，但是，有个问题，surfaceflinger进程的cpu占用率一直持续在15%左右，就算前台是个空的Activity，并且静止没有做任何操作。接下来就是查找surfaceflinger的问题
```
Mem: 3903372k total, 2115832k used, 1787540k free, 18920k buffers

Swap: 0k total, 0k used, 0k free, 1126228k cached

400%cpu 56%user 0%nice 32%sys 311%idle 1%iow 0%irq 0%sirq 0%host

PID USER PR NI VIRT RES SHR S[%CPU] %MEM TIME+ ARGS

4621 system 20 0 2.0G 339M 125M S 45.0 8.8 29:06.28 com.tencent.wecarspeech

1965 system -2 -8 167M 36M 16M S 14.0 0.9 9:43.35 surfaceflinger

7469 root 20 0 11M 4.1M 3.2M R 7.0 0.1 0:01.25 top -d 1 -m 20

4944 system 20 0 1.6G 124M 103M S 7.0 3.2 4:10.31 com.tencent.wecarspeech:voiceview

2085 system -3 -8 70M 11M 8.5M S 4.0 0.2 2:03.75 android.hardware.graphics.composer@2+

1 root 20 0 19M 3.3M 2.1M S 4.0 0.0 0:33.94 init

2070 audioserver 20 0 22M 10M 8.2M S 3.0 0.2 2:02.86 android.hardware.audio@2.0-service

3202 u0_a55 20 0 1.0G 107M 93M S 1.0 2.8 0:07.72 com.tencent.wecarnavi:subnavi

2276 audioserver 20 0 104M 19M 16M S 1.0 0.4 0:43.42 audioserver

2072 vehicle_net+ 20 0 28M 6.5M 4.9M S 1.0 0.1 0:29.37 android.hardware.automotive.vehicle@+

1958 logd 30 10 23M 6.2M 2.7M S 1.0 0.1 0:29.99 logd

1256 root 20 0 0 0 0 S 1.0 0.0 0:32.92 [galcore deamon ]

7470 root 20 0 0 0 0 I 0.0 0.0 0:00.02 [kworker/u8:2]

7464 root 20 0 9.0M 2.4M 2.0M S 0.0 0.0 0:00.01 sh -

7373 root 20 0 0 0 0 I 0.0 0.0 0:00.07 [kworker/2:1]

7342 root 20 0 0 0 0 I 0.0 0.0 0:00.01 [kworker/3:2]

7247 root 20 0 0 0 0 I 0.0 0.0 0:00.03 [kworker/0:0]

7198 root 20 0 0 0 0 I 0.0 0.0 0:00.00 [kworker/1:1]

7170 root 20 0 0 0 0 I 0.0 0.0 0:00.07 [kworker/2:2]

7136 root 20 0 0 0 0 I 0.0 0.0 0:00.00 [kworker/3:0]
```

### 另外查看系统cpu的方法
#### adb shell dumpsys cpuinfo
```
fhw@henryfeng:~$ adb shell dumpsys cpuinfo

Load: 4.45 / 4.22 / 4.25

CPU usage from 496321ms to 196261ms ago (2019-08-09 00:18:44.781 to 2019-08-09 00:23:44.841):

43% 4621/com.tencent.wecarspeech: 37% user + 5.8% kernel / faults: 8034 minor

14% 1965/surfaceflinger: 8.8% user + 5.3% kernel / faults: 3 minor

6.4% 4944/com.tencent.wecarspeech:voiceview: 4.4% user + 2% kernel / faults: 306 minor

3.1% 2070/android.hardware.audio@2.0-service: 1.6% user + 1.4% kernel

3% 2085/android.hardware.graphics.composer@2.1-service: 1.4% user + 1.5% kernel

1.1% 6474/kworker/1:2: 0% user + 1.1% kernel

1.1% 2145/commandservice: 0% user + 1% kernel

1% 2276/audioserver: 0.7% user + 0.3% kernel

0.8% 1256/galcore deamon : 0% user + 0.8% kernel

0.6% 2072/android.hardware.automotive.vehicle@2.0-service: 0.1% user + 0.5% kernel

0.6% 1//init: 0% user + 0.5% kernel / faults: 2007 minor

0.4% 6700/kworker/1:0: 0% user + 0.4% kernel

0.4% 3171/com.tencent.wecarnavi:wecarbase: 0.3% user + 0% kernel / faults: 2509 minor

0.3% 1958/logd: 0.1% user + 0.1% kernel / faults: 14 minor

0.2% 8/rcu_preempt: 0% user + 0.2% kernel

0.2% 3090/wtlogger: 0.1% user + 0.1% kernel / faults: 1 minor

0.1% 2263/system_server: 0.1% user + 0% kernel / faults: 315 minor

0.1% 2761/sugov:0: 0% user + 0.1% kernel

0.1% 957/HandleReportPoi: 0% user + 0.1% kernel

0.1% 953/HandleReportPoi: 0% user + 0.1% kernel

0.1% 6577/kworker/u8:0: 0% user + 0.1% kernel

0.1% 2146/brlinkd: 0% user + 0% kernel

0.1% 2083/android.hardware.gnss@1.0-service-ubx: 0% user + 0% kernel

0.1% 3573/com.wt.music: 0% user + 0% kernel / faults: 85 minor

0.1% 6319/kworker/u8:1: 0% user + 0.1% kernel

0.1% 3202/com.tencent.wecarnavi:subnavi: 0% user + 0% kernel / faults: 69 minor

0% 108/kworker/u8:4: 0% user + 0% kernel

0% 1401/kworker/0:1H: 0% user + 0% kernel

0% 2088/android.hardware.memtrack@1.0-service: 0% user + 0% kernel

0% 1959/servicemanager: 0% user + 0% kernel

0% 3535/com.autopai.dockview.dock: 0% user + 0% kernel / faults: 116 minor

0% 2425/com.android.car: 0% user + 0% kernel / faults: 206 minor

0% 2560/com.android.phone: 0% user + 0% kernel / faults: 12 minor

0% 2164/kworker/1:2H: 0% user + 0% kernel

0% 1266/kworker/3:1H: 0% user + 0% kernel

0% 1976/jbd2/mmcblk0p14: 0% user + 0% kernel

0% 1490/kworker/2:1H: 0% user + 0% kernel

0% 2069/healthd: 0% user + 0% kernel

0% 2086/android.hardware.health@2.0-service.imx: 0% user + 0% kernel

0% 6965/kworker/2:0: 0% user + 0% kernel

0% 3054/com.android.bluetooth: 0% user + 0% kernel / faults: 2 minor

0% 7/ksoftirqd/0: 0% user + 0% kernel

0% 9/rcu_sched: 0% user + 0% kernel

0% 2092/com.android.car.procfsinspector: 0% user + 0% kernel

0% 7170/kworker/2:2: 0% user + 0% kernel

0% 7247/kworker/0:0: 0% user + 0% kernel

0% 20/ksoftirqd/2: 0% user + 0% kernel

0% 25/ksoftirqd/3: 0% user + 0% kernel

0% 2057/netd: 0% user + 0% kernel / faults: 15 minor

0% 2160/statsd: 0% user + 0% kernel

0% 2538/com.wt.stability.monitor: 0% user + 0% kernel / faults: 46 minor

0% 2937/VosMCThread: 0% user + 0% kernel

0% 3868/com.tencent.wecarcontrol: 0% user + 0% kernel

+0% 7342/kworker/3:2: 0% user + 0% kernel

+0% 7373/kworker/2:1: 0% user + 0% kernel

20% TOTAL: 14% user + 5.8% kernel + 0% iowait + 0% softirq

```

#### adb shell vmstat 1
```
fhw@henryfeng:~$ adb shell vmstat 1

procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----

r b swpd free buff cache si so bi bo in cs us sy id wa

8 0 0 1787004 19240 1126524 0 0 100 10 0 2554 14 6 79 0

0 0 0 1787004 19240 1126524 0 0 0 0 0 8618 12 6 81 0

4 0 0 1786756 19240 1126528 0 0 0 0 0 8514 14 6 79 0

1 0 0 1786508 19240 1126528 0 0 0 0 0 8387 15 5 80 0

0 0 0 1786632 19240 1126532 0 0 0 20 0 8590 15 7 78 0

0 0 0 1786632 19248 1126532 0 0 0 52 0 8677 14 5 80 0

2 0 0 1786508 19248 1126532 0 0 0 0 0 8164 13 4 82 0

2 0 0 1786260 19248 1126532 0 0 0 0 0 8916 13 6 81 0

2 0 0 1786260 19248 1126536 0 0 0 0 0 8380 14 6 79 0

0 0 0 1786260 19248 1126536 0 0 0 12 0 8442 14 5 81 0

0 0 0 1786260 19256 1126540 0 0 0 52 0 8850 14 7 78 0
```

# 查找surfaceflinger的cpu持续占用15%左右的原因
## 查看surfaceflinger的进程号那个线程持续占用cpu，通过数据可知surfaceflinger的主线程占用cpu最高
获取surfaceflinger的进程号
```
fhw@henryfeng:~$ adb shell ps -ef | grep surfaceflinger

system 1965 1 14 23:12:59 ? 00:10:25 surfaceflinger
```

通过进程号获取各个线程cpu的占用情况
```
top -H -d 1 -p 1965

Tasks: 12 total, 0 running, 12 sleeping, 0 stopped, 0 zombie

Mem: 3903372k total, 2121344k used, 1782028k free, 19560k buffers

Swap: 0k total, 0k used, 0k free, 1126840k cached

400%cpu 61%user 0%nice 23%sys 315%idle 0%iow 0%irq 1%sirq 0%host

PID USER PR NI VIRT RES SHR S[%CPU] %MEM TIME+ THREAD PROCESS

1965 system -2 -8 167M 36M 16M S 13.0 0.9 10:35.91 surfaceflinger surfaceflinger

1965 system 12 -8 167M 36M 16M S 2.0 0.9 1:06.42 surfaceflinger surfaceflinger

1965 system -3 -8 167M 36M 16M S 2.0 0.9 0:23.28 sfEventThread surfaceflinger

1965 system 20 0 167M 36M 16M S 2.0 0.9 0:30.06 HwBinder:1965_1 surfaceflinger

1965 system -3 -9 167M 36M 16M S 1.0 0.9 0:24.56 DispSync surfaceflinger

1965 system -3 -8 167M 36M 16M S 1.0 0.9 0:19.68 appEventThread surfaceflinger

1965 system 20 0 167M 36M 16M S 1.0 0.9 0:20.70 Binder:1965_1 surfaceflinger

1965 system 20 0 167M 36M 16M S 0.0 0.9 0:00.73 Binder:1965_5 surfaceflinger

1965 system 20 0 167M 36M 16M S 0.0 0.9 0:00.68 Binder:1965_4 surfaceflinger

1965 system 20 0 167M 36M 16M S 0.0 0.9 0:05.93 Binder:1965_3 surfaceflinger

1965 system 12 -8 167M 36M 16M S 0.0 0.9 0:00.02 surfaceflinger surfaceflinger

1965 system 20 0 167M 36M 16M S 0.0 0.9 0:17.62 Binder:1965_2 surfaceflinger
```

## 查看那个进程一直在调用surfaceflinger
surfaceflinger是个被动调用的进程，这种现象的原因及有可能是有应用一直在绘制，接下来的思路就是查看是那个进程一直在调用surfaceflinger，因为surfaceflinger的调用都是通过binder进行的，此时可以查看surfaceflinger的binder call情况。(由于写文档期间Android系统有重启，以下surfaceflinger进程号会改变)
```
surfaceflinger的进程号为：1964

cat /sys/kernel/debug/binder/transaction_log | grep 1964 | grep call

567877: call from 3434:3602 to 1964:0 context binder node 42778 handle 26 size 104:0 ret 0/0 l=0

567882: call from 1964:1964 to 2085:0 context hwbinder node 445 handle 3 size 460:80 ret 0/0 l=0

567884: call from 1964:1964 to 2085:0 context hwbinder node 445 handle 3 size 220:32 ret 0/0 l=0

567891: call from 3434:3602 to 1964:0 context binder node 42778 handle 26 size 192:8 ret 0/0 l=0

567893: call from 3434:3602 to 1964:0 context binder node 42778 handle 26 size 104:0 ret 0/0 l=0

567896: call from 1964:1964 to 2085:0 context hwbinder node 445 handle 3 size 460:80 ret 0/0 l=0

567901: call from 1964:1964 to 2085:0 context hwbinder node 445 handle 3 size 220:32 ret 0/0 l=0
```

从输出可以看到：进程号为3434，线程号为3602的线程一直在通过binder 调用surfaceflinger。

查看3434为哪个进程，可知3434为语音的应用的一个子进程。把语音应用的disable后，surfaceflinger的cpu占用率降为百分之零，验证了这一猜想。
```
mek_8q:/ $ ps -ef | grep 3434

system 3434 3049 7 00:50:58 ? 00:01:46 com.tencent.wecarspeech:voiceview

shell 5545 5497 1 01:16:51 pts/1 00:00:00 grep 3434

```
