# **dvm_lock_sample和Long monitor分析**
## 如何看events log
文件event-log-tags位于手机的/system/etc/下面，定义了Android系统所有的events事件，可以把这个文件pull出来，根据tag搜索对应的事件定义。下面是部分片段：
```
42 answer (to life the universe etc|3)
314 pi
1003 auditd (avc|3)
1004 logd (dropped|3)
1005 liblog (dropped|1)
2718 e
2719 configuration_changed (config mask|1|5)
2720 sync (id|3),(event|1|5),(source|1|5),(account|1|5)
2721 cpu (total|1|6),(user|1|6),(system|1|6),(iowait|1|6),(irq|1|6),(softirq|1|6)
2722 battery_level (level|1|6),(voltage|1|1),(temperature|1|1)
2723 battery_status (status|1|5),(health|1|5),(present|1|5),(plugged|1|5),(technology|3)
2724 power_sleep_requested (wakeLocksCleared|1|1)
2725 power_screen_broadcast_send (wakelockCount|1|1)
2726 power_screen_broadcast_done (on|1|5),(broadcastDuration|2|3),(wakelockCount|1|1)
2727 power_screen_broadcast_stop (which|1|5),(wakelockCount|1|1)
2728 power_screen_state (offOrOn|1|5),(becauseOfUser|1|5),(totalTouchDownTime|2|3),(touchCycles|1|1)
2729 power_partial_wake_state (releasedorAcquired|1|5),(tag|3)
2730 battery_discharge (duration|2|3),(minLevel|1|6),(maxLevel|1|6)
2731 power_soft_sleep_requested (savedwaketimems|2)
2740 location_controller
2741 force_gc (reason|3)
2742 tickle (authority|3)
2744 free_storage_changed (data|2|2)
2745 low_storage (data|2|2)
2746 free_storage_left (data|2|2),(system|2|2),(cache|2|2)
2747 contacts_aggregation (aggregation time|2|3), (count|1|1)
2748 cache_file_deleted (path|3)
```

其中里面数字的含义如下，可以对照解析出events
```
# The entries in this file map a sparse set of log tag numbers to tag names.
# This is installed on the device, in /system/etc, and parsed by logcat.
#
# Tag numbers are decimal integers, from 0 to 2^31.  (Let's leave the
# negative values alone for now.)
# Tag numbers是十进制的整数，取值从0到2^31
# Tag names are one or more ASCII letters and numbers or underscores, i.e.
# "[A-Z][a-z][0-9]_".  Do not include spaces or punctuation (the former
# impacts log readability, the latter makes regex searches more annoying).
#
# Tag names由1到多个ASCII码的字母和下划线组成，为了方便在log中搜索，name中避免使用空格和标点
# Tag numbers and names are separated by whitespace.  Blank lines and lines
# starting with '#' are ignored.
#
# Optionally, after the tag names can be put a description for the value(s)
# of the tag. Description are in the format
#    (<name>|data type[|data unit])
     (<名字>|数据类型[|数据单位])
# Multiple values are separated by commas.
#
# The data type is a number from the following values:
# 1: int
# 2: long
# 3: string
# 4: list
# 5: float
#
# The data unit is a number taken from the following list:
# 1: Number of objects 对象个数
# 2: Number of bytes 字节个数
# 3: Number of milliseconds 毫秒
# 4: Number of allocations 分配个数
# 5: Id
# 6: Percent 百分比
# Default value for data of type int/long is 2 (bytes).
#
# TODO: generate ".java" and ".h" files with integer constants from this file.

# These are used for testing, do not modify without updating
# tests/framework-tests/src/android/util/EventLogFunctionalTest.java
```

## dvm_lock_sample解析
dvm_lock_sample 是Dalvik VM / ART的事件，是虚拟机对同步锁的一个监控，一旦线程等待锁的时间超过lock_profiling_threshold_(500ms)，就会打印这个事件。通过这个事件，可以发现哪个类的方法存在性能问题，有优化的空间。 dvm_lock_sample的出发条件如下：

    尝试获取锁失败
    锁拥有者的线程id不为0
    log_contention为true，也就是lock_profiling_threshold_不为0
    耗时的百分比不为0(sample_percent不为0)，或是百分比大于一个100内的随机数(static_cast<uint32_t>(rand() % 100) < sample_percent)？？？比较诡异，为什么要做这个对比？？？

## dvm_lock_sample定义
dvm_lock_sample的定义位于文件system/core/logcat/event.logtags，内容如下：
```
20003 dvm_lock_sample (process|3),(main|1|5),(thread|3),(time|1|3),(file|3),(line|1|5),(ownerfile|3),(ownerline|1|5),(sample_percent|1|6)
```
在生成的event-log-tags文件中的定义如下：
```
20003 dvm_lock_sample (process|3),(main|1|5),(thread|3),(time|1|3),(file|3),(line|1|5),(ownerfile|3),(ownerline|1|5),(sample_percent|1|6)
```

## dvm_lock_sample日志解析
```
08-05 03:13:19.437  1558 29210 I dvm_lock_sample: [system_server,1,Binder:1558_14,11,ActivityManagerService.java,12168,android.app.ContentProviderHolder com.android.server.am.ActivityManagerService.getContentProviderImpl(android.app.IApplicationThread, java.lang.String, android.os.IBinder, boolean, int),-,12634,void com.android.server.am.ActivityManagerService.removeContentProvider(android.os.IBinder, boolean),2]

08-05 03:13:19.437  1558 29210 I ：打印日志的时间，打印日志的进程号，打印日志的线程号，日志的等级
dvm_lock_sample：事件的类型
[system_server,1,Binder:1558_14,11,ActivityManagerService.java,12168,android.app.ContentProviderHolder com.android.server.am.ActivityManagerService.getContentProviderImpl(android.app.IApplicationThread, java.lang.String, android.os.IBinder, boolean, int),-,12634,void com.android.server.am.ActivityManagerService.removeContentProvider(android.os.IBinder, boolean),2]
总共11项，和上面的定义有出入，但是和代码时一致的，分别是，
进程名，该线程是否为StrictMode会监控的对象， 等待锁的线程名，等待锁的时间(ms)，等待锁所在的类，等待锁所在的行号，等待锁所在的方法，持有锁的文件名(同一个文件为“-”，或者为空)，持有锁的行号，持有锁的方法名，锁等待的百分比(超过500ms为百分之百)
```

## 生成dvm_lock_sample事件的代码解析
位置：art/runtime/monitor_android.cc
```
void Monitor::LogContentionEvent(Thread* self,
                                 uint32_t wait_ms,
                                 uint32_t sample_percent,
                                 ArtMethod* owner_method,
                                 uint32_t owner_dex_pc) {
  android_log_event_list ctx(EVENT_LOG_TAG_dvm_lock_sample);

  const char* owner_filename;
  int32_t owner_line_number;
  TranslateLocation(owner_method, owner_dex_pc, &owner_filename, &owner_line_number);

  // Emit the process name, <= 37 bytes.
  {
    int fd = open("/proc/self/cmdline", O_RDONLY);
    char procName[33];
    memset(procName, 0, sizeof(procName));
    read(fd, procName, sizeof(procName) - 1);
    close(fd);
    ctx << procName;
  }

  // Emit the sensitive thread ("main thread") status. We follow tradition that this corresponds
  // to a C++ bool's value, but be explicit.
  constexpr uint32_t kIsSensitive = 1u;
  constexpr uint32_t kIsNotSensitive = 0u;
  ctx << (Thread::IsSensitiveThread() ? kIsSensitive : kIsNotSensitive);

  // Emit self thread name string.
  {
    std::string thread_name;
    self->GetThreadName(thread_name);
    ctx << thread_name;
  }

  // Emit the wait time.
  ctx << wait_ms;

  const char* filename = nullptr;
  {
    uint32_t pc;
    ArtMethod* m = self->GetCurrentMethod(&pc);
    int32_t line_number;
    TranslateLocation(m, pc, &filename, &line_number);

    // Emit the source code file name.
    ctx << filename;

    // Emit the source code line number.
    ctx << line_number;

    // Emit the method name.
    ctx << ArtMethod::PrettyMethod(m);
  }

  // Emit the lock owner source code file name.
  if (owner_filename == nullptr) {
    owner_filename = "";
  } else if (strcmp(filename, owner_filename) == 0) {
    // Common case, so save on log space.
    owner_filename = "-";
  }
  ctx << owner_filename;

  // Emit the source code line number.
  ctx << owner_line_number;

  // Emit the owner method name.
  ctx << ArtMethod::PrettyMethod(owner_method);

  // Emit the sample percentage.
  ctx << sample_percent;

  ctx << LOG_ID_EVENTS;
}

```

## Long monitor解析
打印条件和dvm_lock_sample一样
Long monitor日志解析
打印堆栈的日志
```
08-05 03:58:18.068  2257  2359 W com.tencent.mm: Long monitor contention with owner parallels-0 (2357) at java.lang.String java.lang.Runtime.nativeLoad(java.lang.String, java.lang.ClassLoader)(Runtime.java:-2) waiters=0 in boolean com.tencent.mm.compatible.util.k.md(java.lang.String) for 607ms

Long monitor contention with owner + 持有锁的线程名(持有锁的线程号) + at + 持有锁的方法名(持有锁所在的文件:行号) +  waiters=等待锁的线程数 + in + 等待锁的方法 + for + 等待时间
```
## 不打印堆栈的日志

暂时没抓到
## Long monitor代码解析
```

    //如果wait 的时间超过stack_dump_lock_profiling_threshold_，就dump stack
  const bool should_dump_stacks = stack_dump_lock_profiling_threshold_ > 0 &&
    wait_ms > stack_dump_lock_profiling_threshold_;
    ...
// Give the detailed traces for really long contention.
  if (should_dump_stacks) {
    // This must be here (and not above) because we cannot hold the thread-list lock
    // while running the checkpoint.
    std::ostringstream self_trace_oss;
    self->DumpJavaStack(self_trace_oss);

    uint32_t pc;
    ArtMethod* m = self->GetCurrentMethod(&pc);

    LOG(WARNING) << "Long "
        << PrettyContentionInfo(original_owner_name,
                                original_owner_tid,
                                owners_method,
                                owners_dex_pc,
                                num_waiters)
        << " in " << ArtMethod::PrettyMethod(m) << " for "
        << PrettyDuration(MsToNs(wait_ms)) << "\n"
        << "Current owner stack:\n" << owner_stack_dump
        << "Contender stack:\n" << self_trace_oss.str();
  } else if (wait_ms > kLongWaitMs && owners_method != nullptr) {
    uint32_t pc;
    ArtMethod* m = self->GetCurrentMethod(&pc);
    // TODO: We should maybe check that original_owner is still a live thread.
    LOG(WARNING) << "Long "
        << PrettyContentionInfo(original_owner_name,
                                original_owner_tid,
                                owners_method,
                                owners_dex_pc,
                                num_waiters)
        << " in " << ArtMethod::PrettyMethod(m) << " for "
        << PrettyDuration(MsToNs(wait_ms));
  }
```
## PrettyContentionInfo方法详解
```
std::string Monitor::PrettyContentionInfo(const std::string& owner_name,
                                          pid_t owner_tid,
                                          ArtMethod* owners_method,
                                          uint32_t owners_dex_pc,
                                          size_t num_waiters) {
  Locks::mutator_lock_->AssertSharedHeld(Thread::Current());
  const char* owners_filename;
  int32_t owners_line_number = 0;
  if (owners_method != nullptr) {
    TranslateLocation(owners_method, owners_dex_pc, &owners_filename, &owners_line_number);
  }
  std::ostringstream oss;
  oss << "monitor contention with owner " << owner_name << " (" << owner_tid << ")";
  if (owners_method != nullptr) {
    oss << " at " << owners_method->PrettyMethod();
    oss << "(" << owners_filename << ":" << owners_line_number << ")";
  }
  oss << " waiters=" << num_waiters;
  return oss.str();
}
```

# 相关代码，可以不关注

位置：art/runtime/monitor.cc

```
template <LockReason reason>
void Monitor::Lock(Thread* self) {
  ScopedAssertNotHeld sanh(self, monitor_lock_);
  bool called_monitors_callback = false;
  monitor_lock_.Lock(self);
  while (true) {
      //不断的尝试获取锁，如果获取成功，就退出
    if (TryLockLocked(self)) {
      break;
    }
    // Contended. 如果lock_profiling_threshold_不等于0,就打印日志，lock_profiling_threshold_一般等于500ms
    const bool log_contention = (lock_profiling_threshold_ != 0);
    uint64_t wait_start_ms = log_contention ? MilliTime() : 0;
    ArtMethod* owners_method = locking_method_;
    uint32_t owners_dex_pc = locking_dex_pc_;
    // Do this before releasing the lock so that we don't get deflated.
    //num_waiters_表示等待锁的线程数
    size_t num_waiters = num_waiters_;
    ++num_waiters_;

    // If systrace logging is enabled, first look at the lock owner. Acquiring the monitor's
    // lock and then re-acquiring the mutator lock can deadlock.
    bool started_trace = false;
    if (ATRACE_ENABLED()) {
      if (owner_ != nullptr) {  // Did the owner_ give the lock up?
        std::ostringstream oss;
        std::string name;
        owner_->GetThreadName(name);
        oss << PrettyContentionInfo(name,
                                    owner_->GetTid(),
                                    owners_method,
                                    owners_dex_pc,
                                    num_waiters);
        // Add info for contending thread.
        uint32_t pc;
        ArtMethod* m = self->GetCurrentMethod(&pc);
        const char* filename;
        int32_t line_number;
        TranslateLocation(m, pc, &filename, &line_number);
        oss << " blocking from "
            << ArtMethod::PrettyMethod(m) << "(" << (filename != nullptr ? filename : "null")
            << ":" << line_number << ")";
        ATRACE_BEGIN(oss.str().c_str());
        started_trace = true;
      }
    }

    monitor_lock_.Unlock(self);  // Let go of locks in order.
    // Call the contended locking cb once and only once. Also only call it if we are locking for
    // the first time, not during a Wait wakeup.
    if (reason == LockReason::kForLock && !called_monitors_callback) {
      called_monitors_callback = true;
      Runtime::Current()->GetRuntimeCallbacks()->MonitorContendedLocking(this);
    }
    self->SetMonitorEnterObject(GetObject());
    {
      ScopedThreadSuspension tsc(self, kBlocked);  // Change to blocked and give up mutator_lock_.
      uint32_t original_owner_thread_id = 0u;
      {
        // Reacquire monitor_lock_ without mutator_lock_ for Wait.
        MutexLock mu2(self, monitor_lock_);
        if (owner_ != nullptr) {  // Did the owner_ give the lock up?
          original_owner_thread_id = owner_->GetThreadId();
          monitor_contenders_.Wait(self);  // Still contended so wait.
        }
      }
      if (original_owner_thread_id != 0u) {
        // Woken from contention.
        if (log_contention) {
            //获取等待的时间
          uint64_t wait_ms = MilliTime() - wait_start_ms;
          uint32_t sample_percent;
          //如果等待的时间大于等于500ms,等待的百分比就为100
          if (wait_ms >= lock_profiling_threshold_) {
            sample_percent = 100;
          } else {
            sample_percent = 100 * wait_ms / lock_profiling_threshold_;
          }
          //这个随机数用的很诡异
          if (sample_percent != 0 && (static_cast<uint32_t>(rand() % 100) < sample_percent)) {
            // Reacquire mutator_lock_ for logging.
            ScopedObjectAccess soa(self);

            bool owner_alive = false;
            pid_t original_owner_tid = 0;
            std::string original_owner_name;
              LOG(WARNING) << "Long sample_percent = " << sample_percent << " , wait_ms = "<< wait_ms <<
              " , lock_profiling_threshold_ = " << lock_profiling_threshold_ << " , stack_dump_lock_profiling_threshold_ =" <<stack_dump_lock_profiling_threshold_;
                //如果wait 的时间超过stack_dump_lock_profiling_threshold_，就dump stack
              const bool should_dump_stacks = stack_dump_lock_profiling_threshold_ > 0 &&
                wait_ms > stack_dump_lock_profiling_threshold_;
            std::string owner_stack_dump;

            // Acquire thread-list lock to find thread and keep it from dying until we've got all
            // the info we need.
            {
              Locks::thread_list_lock_->ExclusiveLock(Thread::Current());

              // Re-find the owner in case the thread got killed.
              Thread* original_owner = Runtime::Current()->GetThreadList()->FindThreadByThreadId(
                  original_owner_thread_id);

              if (original_owner != nullptr) {
                owner_alive = true;
                original_owner_tid = original_owner->GetTid();
                original_owner->GetThreadName(original_owner_name);

                if (should_dump_stacks) {
                  // Very long contention. Dump stacks.
                  struct CollectStackTrace : public Closure {
                    void Run(art::Thread* thread) OVERRIDE
                        REQUIRES_SHARED(art::Locks::mutator_lock_) {
                      thread->DumpJavaStack(oss);
                    }

                    std::ostringstream oss;
                  };
                  CollectStackTrace owner_trace;
                  // RequestSynchronousCheckpoint releases the thread_list_lock_ as a part of its
                  // execution.
                  original_owner->RequestSynchronousCheckpoint(&owner_trace);
                  owner_stack_dump = owner_trace.oss.str();
                } else {
                  Locks::thread_list_lock_->ExclusiveUnlock(Thread::Current());
                }
              } else {
                Locks::thread_list_lock_->ExclusiveUnlock(Thread::Current());
              }
              // This is all the data we need. Now drop the thread-list lock, it's OK for the
              // owner to go away now.
            }

            // If we found the owner (and thus have owner data), go and log now.
            if (owner_alive) {
              // Give the detailed traces for really long contention.
              if (should_dump_stacks) {
                // This must be here (and not above) because we cannot hold the thread-list lock
                // while running the checkpoint.
                std::ostringstream self_trace_oss;
                self->DumpJavaStack(self_trace_oss);

                uint32_t pc;
                ArtMethod* m = self->GetCurrentMethod(&pc);

                LOG(WARNING) << "Long "
                    << PrettyContentionInfo(original_owner_name,
                                            original_owner_tid,
                                            owners_method,
                                            owners_dex_pc,
                                            num_waiters)
                    << " in " << ArtMethod::PrettyMethod(m) << " for "
                    << PrettyDuration(MsToNs(wait_ms)) << "\n"
                    << "Current owner stack:\n" << owner_stack_dump
                    << "Contender stack:\n" << self_trace_oss.str();
              } else if (wait_ms > kLongWaitMs && owners_method != nullptr) {
                uint32_t pc;
                ArtMethod* m = self->GetCurrentMethod(&pc);
                // TODO: We should maybe check that original_owner is still a live thread.
                LOG(WARNING) << "Long "
                    << PrettyContentionInfo(original_owner_name,
                                            original_owner_tid,
                                            owners_method,
                                            owners_dex_pc,
                                            num_waiters)
                    << " in " << ArtMethod::PrettyMethod(m) << " for "
                    << PrettyDuration(MsToNs(wait_ms));
              }
              //08-01 10:54:53.751 W/system_server( 1524): Long monitor contention with owner main (1524)
              // at void com.android.server.am.ActivityManagerService.finishReceiver(android.os.IBinder, int, java.lang.String, android.os.Bundle, boolean, int)
              // (ActivityManagerService.java:22063) waiters=1 in android.app.ContentProviderHolder
              // com.android.server.am.ActivityManagerService.getContentProviderImpl(android.app.IApplicationThread, java.lang.String, android.os.IBinder, boolean, int) for 102ms
              //打印event日志
              LogContentionEvent(self,
                                wait_ms,
                                sample_percent,
                                owners_method,
                                owners_dex_pc);
              //08-01 10:54:53.752  1524  3010 I dvm_lock_sample: [system_server,1,Binder:1524_E,102,ActivityManagerService.java,12168,
              // android.app.ContentProviderHolder com.android.server.am.ActivityManagerService.getContentProviderImpl(android.app.IApplicationThread, java.lang.String, android.os.I
              // -,22063,void com.android.server.am.ActivityManagerService.finishReceiver(android.os.IBinder, int, java.lang.String, android.os.Bundle, boolean, int),20]
            }
          }
        }
      }
    }
    if (started_trace) {
      ATRACE_END();
    }
    self->SetMonitorEnterObject(nullptr);
    monitor_lock_.Lock(self);  // Reacquire locks in order.
    --num_waiters_;
  }
  monitor_lock_.Unlock(self);
  // We need to pair this with a single contended locking call. NB we match the RI behavior and call
  // this even if MonitorEnter failed.
  if (called_monitors_callback) {
    CHECK(reason == LockReason::kForLock);
    Runtime::Current()->GetRuntimeCallbacks()->MonitorContendedLocked(this);
  }
}



```






