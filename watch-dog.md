# **watchdog 简介**

启动类：SystemServer.java
实现类：Watchdog.java
dump实现类：ActivityManagerService.java

watchdog
    软件看门狗，参考硬件看门狗
# 一    分为两种模式
1 DB模式，超时时间10秒    5秒检测一次
2 标准模式  超时时间60秒    30秒检测一次

# 二    检测模式：
1 monitor对象锁，  主要是检测死锁问题，常见的持锁超时不释放和逻辑错误的循环死锁问题
2  handler 队列       主要是检测消息队列的处理是否良好，常见的消息处理不及时卡顿超时问题

# 三   检测原理：

## 3.1  monitor，直接获取对象锁，若是有死锁问题，将会获取不到，一直等待，60秒后超时，上报SWT问题
handlercheck，设置四种状态
    static final int COMPLETED = 0;        
    static final int WAITING = 1;        0----30 
    static final int WAITED_HALF = 2;    30----60  
    static final int OVERDUE = 3;        60---

## 3.2  根据入队列开始时间，和当前检测的时间，记录不同的状态


## 3.3 根据状态
不同的处理策略
COMPLETED   不处理
WAITING        不处理

WAITED_HALF    打印当前进程的堆栈
OVERDUE    上报SWT

## 3.4  上报SWT问题，会打印堆栈信息
这里使用ANR的堆栈打印函数  打印出问题进程堆栈和对应的native堆栈信息

```
  final File stack = ActivityManagerService.dumpStackTraces(
                    !waitedHalf, pids, null, null, getInterestingNativePids());
```

## 3.5  日志顺序
1 event日志
2 java堆栈信息
3 kernal日志
4 dropbox文件日志信息

# 3.6  最后是SWT的处理策略

1  debug模式，只打印日志，不杀进程，不重启
2  allowRestart   标识。不允许杀进程的，只打印日志，不杀进程，不重启（比如启动期间，超时SWT问题是不重启的）
2  标准模式， 查进程，退出systemserver 进程，造成系统进程软重启
```
                Slog.w(TAG, "*** WATCHDOG KILLING SYSTEM PROCESS: " + subject);
                WatchdogDiagnostics.diagnoseCheckers(blockedCheckers);
                Slog.w(TAG, "*** GOODBYE!");
                Process.killProcess(Process.myPid());
                System.exit(10);
```
# 四   补充内容：
dumpStackTraces（）

这里先说一个记录的arrylist
4.0  clearTraces     是否清理原来的日志
		firstPids       这是个出问题的进程队列
		processCpuTracker  CPU的堆栈信息
		lastPids        这个是根据算法记录的最近使用的进程队列，尽可能的保存现场
		nativePids      native本地进程队列
		extraPids   根据CPU使用情况，从lastPids 中挑选的最近在使用的进程

对于SWT来说，只使用   firstPids         nativePids                    

