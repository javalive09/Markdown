# ADB调试



[adbshell.com](http://adbshell.com/)

## sync

```text
adb root && adb remount && adb sync system && adb reboot //同步代码调试系统应用
```

## install

```text
adb install test.apk 
adb install -l test.apk //forward lock application 将应用安装到保护目录 /mnt/asec
adb install -r test.apk //replace existing application 覆盖安装 保持原程序数据
adb install -t test.apk // allow test packages 允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用
adb install -s test.apk // install application on sdcard 将应用安装到 sdcard
adb install -d test.apk // allow version code downgrade 允许降级覆盖安装
adb install -p test.apk // partial application install 授予所有运行时权限
```

## uninstall

```text
adb uninstall com.xx.xx 
adb uninstall －k com.xx.xxx // 卸载，保留缓存
```

## log

```text
adb logcat | grep xxx // 过滤显示字符串xxx
adb logcat | grep "^E.123" //过滤显示error级 tag123
adb logcat | grep "^W.123" //过滤显示warning级tag 123
adb logcat | grep "^..123" //过滤显示所有tag 123
adb logcat -c && adb logcat // 清除log后显示
adb logcat > log.txt // 保存到电脑
logcat -v threadtime --pid 17393 -f /storage/emulated/0/.peter/logs/codebag/codebag.log -r 20480 -n 3 -s *:I // 保存到设备中  官方文档：https://developer.android.com/studio/command-line/logcat.html#filteringOutput
```

## am

```text
adb shell am force-stop com.xx.xx // kill app

adb shell am start -a android.intent.action.CALL -d tel:10086 // 打电话拨打10086

adb shell am start -a android.intent.action.VIEW -d http://www.baidu.com // 打开网页

adb shell am start -n com.android.settings/com.android.settings.Settings
am start -a android.settings.SETTINGS // 打开原生设置

adb shell am start -n com.android.launcher3/com.android.launcher3.Launcher // 打开原生launcher

adb shell am start -a "android.intent.action.MAIN"  // 打开入口程序列表

am broadcast -a duershow.settings.ota.findnewversion // 广播

adb shell am broadcast -a com.android.test --es test_string "this is test string" --ei test_int 100 --ez test_boolean true // 带参数广播

am broadcast -a com.baidu.duershow.action.SEND_LAUNCHER_VER_INFO --es launcherVersion "1.0" --es DCSVersion "2.0" --es VoiceVersion "3.0" // 带参数广播

adb shell am set-debug-app -w com.peter.util.searcher // 启动时等待调试 -w: wait

adb shell am set-debug-app -w --persistent com.peter.util.searcher // 持续调试 --persistent: 每次启动都调试

adb shell am clear-debug-app // 清除持续调试
```

### 获取app启动时间

```text
adb shell am start -W -n packagename/packageName.MainActivity

ThisTime: 355   // 调用有界面 Activity 的 onCreate、onResume。表示一连串启动 Activity 的最后一个 Activity 的启动耗时；表示新应用启动的耗时，包括新进程的启动和 Activity 的启动，但不包括前一个应用 Activity pause 的耗时。
TotalTime: 355  // 启动进程、调用无界面 Activity 的 onCreate() 等、 pause/finish 无界面的 Activity。 
WaitTime: 365  //AMS 创建 ActivityRecord 记录块和选择合理的 Task、将当前Resume 的 Activity 进行 pause。 就是总的耗时，包括前一个应用 Activity pause 的时间和新应用启动的时间；开发者一般只要关心 TotalTime 即可，这个时间才是自己应用真正启动的耗时。
```

## pm

```text
adb shell pm list package // 列出所有安装包

adb shell pm clear <packagename> // 清除应用数据与缓存

adb shell pm list packages -f // See their associated file.

adb shell pm list packages -d // Filter to only show disabled packages.

adb shell pm list packages -e // Filter to only show enabled packages.

adb shell pm list packages -s // Filter to only show system packages.

adb shell pm list packages -3 // Filter to only show third party packages.

adb shell pm list packages -i // See the installer for the packages.

adb shell pm list packages -u // Also include uninstalled packages.

adb shell pm list packages --user <USER_ID> // The user space to query.

adb shell pm path com.android.phone // 获取包名apk所在路径
```

## 修改host

```text
//android
adb shell  
su  
vi /etc/hosts

//mac
vi /private/etc/hosts
```

## wm

```text
adb shell wm size // 获取分辨率
adb shell wm size 540x960 // 设置分辨率
adb shell wm size reset // 还原分辨率
adb shell wm density // 获取屏幕密度
adb shell wm density 320  //设置屏幕密度
adb shell wm density reset // 还原屏幕密度
```

## prop

对应的文件为

```text
init.rc
default.prop
/system/build.prop
```

```text
adb shell getprop ro.build.version.release // 查询系统版本6.0.1 

adb shell getprop ro.build.version.sdk // 查询系统版本23

adb shell setprop ro.duershow.ota.productid 1094 // 设置系统属性

adb shell setprop ro.duershow.ota.productkey ugMgwCwntaXbiKb5ySE1xQ== // 设置系统属性
```

## 删除wif配置

```text
adb shell rm /data/misc/wifi/*.conf
adb reboot
```

## svc

```text
svc wifi enable // 打开wifi开关
svc wifi disable // 关闭wifi开关
```

## 查看IP

```text
adb shell ifconfig wlan0
```

## “Read-only file system”解决办法

```text
adb remount
```

## 修改系统时间

```text
adb shell toybox date

adb shell toolbox date

//use YYYYMMDD.HHmmss format:
adb shell "su 0 toolbox date -s 20161231.235959"

//MMDDhhmm[[CC]YY][.ss] format:
adb shell "su 0 toybox date 123123592016.59"
```

## input 输入

```text
adb shell input keyevent 4 //back


0 -->  "KEYCODE_UNKNOWN" 
1 -->  "KEYCODE_MENU" 
2 -->  "KEYCODE_SOFT_RIGHT" 
3 -->  "KEYCODE_HOME" 
4 -->  "KEYCODE_BACK" 
5 -->  "KEYCODE_CALL" 
6 -->  "KEYCODE_ENDCALL" 
7 -->  "KEYCODE_0" 
8 -->  "KEYCODE_1" 
9 -->  "KEYCODE_2" 
10 -->  "KEYCODE_3" 
11 -->  "KEYCODE_4" 
12 -->  "KEYCODE_5" 
13 -->  "KEYCODE_6" 
14 -->  "KEYCODE_7" 
15 -->  "KEYCODE_8" 
16 -->  "KEYCODE_9" 
17 -->  "KEYCODE_STAR" 
18 -->  "KEYCODE_POUND" 
19 -->  "KEYCODE_DPAD_UP" 
20 -->  "KEYCODE_DPAD_DOWN" 
21 -->  "KEYCODE_DPAD_LEFT" 
22 -->  "KEYCODE_DPAD_RIGHT" 
23 -->  "KEYCODE_DPAD_CENTER" 
24 -->  "KEYCODE_VOLUME_UP" 
25 -->  "KEYCODE_VOLUME_DOWN" 
26 -->  "KEYCODE_POWER" 
27 -->  "KEYCODE_CAMERA" 
28 -->  "KEYCODE_CLEAR" 
29 -->  "KEYCODE_A" 
30 -->  "KEYCODE_B" 
31 -->  "KEYCODE_C" 
32 -->  "KEYCODE_D" 
33 -->  "KEYCODE_E" 
34 -->  "KEYCODE_F" 
35 -->  "KEYCODE_G" 
36 -->  "KEYCODE_H" 
37 -->  "KEYCODE_I" 
38 -->  "KEYCODE_J" 
39 -->  "KEYCODE_K" 
40 -->  "KEYCODE_L" 
41 -->  "KEYCODE_M" 
42 -->  "KEYCODE_N" 
43 -->  "KEYCODE_O" 
44 -->  "KEYCODE_P" 
45 -->  "KEYCODE_Q" 
46 -->  "KEYCODE_R" 
47 -->  "KEYCODE_S" 
48 -->  "KEYCODE_T" 
49 -->  "KEYCODE_U" 
50 -->  "KEYCODE_V" 
51 -->  "KEYCODE_W" 
52 -->  "KEYCODE_X" 
53 -->  "KEYCODE_Y" 
54 -->  "KEYCODE_Z" 
55 -->  "KEYCODE_COMMA" 
56 -->  "KEYCODE_PERIOD" 
57 -->  "KEYCODE_ALT_LEFT" 
58 -->  "KEYCODE_ALT_RIGHT" 
59 -->  "KEYCODE_SHIFT_LEFT" 
60 -->  "KEYCODE_SHIFT_RIGHT" 
61 -->  "KEYCODE_TAB" 
62 -->  "KEYCODE_SPACE" 
63 -->  "KEYCODE_SYM" 
64 -->  "KEYCODE_EXPLORER" 
65 -->  "KEYCODE_ENVELOPE" 
66 -->  "KEYCODE_ENTER" 
67 -->  "KEYCODE_DEL" 
68 -->  "KEYCODE_GRAVE" 
69 -->  "KEYCODE_MINUS" 
70 -->  "KEYCODE_EQUALS" 
71 -->  "KEYCODE_LEFT_BRACKET" 
72 -->  "KEYCODE_RIGHT_BRACKET" 
73 -->  "KEYCODE_BACKSLASH" 
74 -->  "KEYCODE_SEMICOLON" 
75 -->  "KEYCODE_APOSTROPHE" 
76 -->  "KEYCODE_SLASH" 
77 -->  "KEYCODE_AT" 
78 -->  "KEYCODE_NUM" 
79 -->  "KEYCODE_HEADSETHOOK" 
80 -->  "KEYCODE_FOCUS" 
81 -->  "KEYCODE_PLUS" 
82 -->  "KEYCODE_MENU" 
83 -->  "KEYCODE_NOTIFICATION" 
84 -->  "KEYCODE_SEARCH" 
85 -->  "TAG_LAST_KEYCODE"
```

## settings

```text
adb shell settings put global captive_portal_mode 0 // 关闭网络探测  disable-0  prompt-1  disconnect-2 

adb shell settings get secure bluetooth_address // 获取蓝牙地址

/data/system/users/0/settings_global.xml
adb shell settings put global <key> <value>
adb shell settings get global <key> <default value>

/data/system/users/0/settings_secure.xml
adb shell settings put secure <key> <value>
adb shell settings get secure <key> <default value>

/data/system/users/0/settings_system.xml
adb shell settings put system <key> <value>
adb shell settings get system <key> <default value>

adb shell settings get secure default_input_method // 获取默认输入法

adb shell settings put secure default_input_method com.sohu.inputmethod.sogouoem/.SogouIME // 设置默认输入法

adb shell settings get system screen_brightness_mode // 获取亮度是否为自动获取

adb shell settings get system screen_brightness // 获取当前亮度值

adb shell settings put system screen_brightness 150 // 设置当前亮度值

adb shell settings get system screen_off_timeout // 获取屏幕休眠时间

adb shell settings put system screen_off_timeout 600000 // 更改休眠时间，10分钟

adb shell settings get global auto_time // 获取日期时间选项中通过网络获取时间的状态，1为允许、0为不允许

adb shell settings get global ntp_server // 获取当前NTP服务器url

adb shell settings put global ntp_server ntp.aliyun.com // 修改NTP服务器url
```

### 查询端口所在进程

```text
sudo netstat -tulpn | grep :8000
```

### kill pid

```text
kill -9 <进程 pid>
```

### 查看最大线程数

```text
cat /proc/sys/kernel/threads-max
```

### 查看系统各个限制的数值

```text
ulimit -a
```

## 录制gif

android 4.4 以上

### 录制

```text
adb shell screenrecord  --bit-rate 2000000 --size 480x800 /sdcard/test.mp4
```

### 导出

```text
adb pull /sdcard/test.mp4 ~/Desktop/test.mp4
```

## 修改文件所在组

```text
sudo chown -R $USER /usr/local
```

```text
sudo chown -R $peter /usr/local
```

## 导出anr文件

anr 数据保存位置：/data/anr/traces.txt

```text
adb pull /data/anr/traces.txt ~/Desktop/
```

## adb参数

### Intent参数

| 参数 | 意义 | 解释 |
| :--- | :--- | :--- |
| -a | &lt;ACTION&gt; | 指定Intent action  实现原理Intent.setAction\(\) |
| -n | &lt;COMPONENT&gt; | 指定组件名  格式为{包名}/.{主Activity名}，实现原理Intent.setComponent\(） |
| -d | &lt;DATA\_URI&gt; | 指定Intent data URI |
| -t | &lt;MIME\_TYPE&gt; | 指定Intent MIME Type |
| -c | &lt;CATEGORY&gt; | 指定Intent category  实现原理Intent.addCategory\(\) |
| -p | &lt;PACKAGE&gt; | 指定包名    实现原理Intent.setPackage\(\) |
| -f | &lt;FLAGS&gt; | 添加flags 实现原理Intent.setFlags\(int \) |

```text
am start -a android.intent.action.VIEW
am start -n com.gityuan.app/.MainActivity
am start -d content://contacts/people/1
am start -t image/png
am start -c android.intent.category.APP_CONTACTS
```

### Extra参数

#### 基本类型

| 参数 | --e/--es | --esn | --ez | --ei | --el | --ef | --eu | --ecn |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 类型 | String | \(String\)null | boolean | int | long | float | uri | component |

```text
am start -n com.javalive09.app/.MainActivity --es website 200code.com
// 等价于Intent.putExtra(“website”, 200code.com”);
```

#### 数组类型

| 参数 | -esa | -eia | -ela | -efa |
| :---: | :---: | :---: | :---: | :---: |
| 数组类型 | String\[\] | int\[\] | long\[\] | float\[\] |

```text
am start -n com.javalive09.app/.MainActivity --ela weekday 1,2,3,4,5
// 等价于Intent.putExtra(“weekday”, new int[]{1,2,3,4,5});
```

#### ArrayList类型

| 参数 | -esal | -eial | -elal | -efal |
| :---: | :---: | :---: | :---: | :---: |
| List类型 | String | int | long | float |

```text
am start -n com.javalive09.app/.MainActivity --efal nums 1.2,2.2
// 等价于先构造ArrayList变量，再通过putExtra放入第二个参数
```

#### Flag参数

-f ，是通过Intent.setFlags\(int \)方法，来设置Intent的flags

```text
[--grant-read-uri-permission] [--grant-write-uri-permission]
[--grant-persistable-uri-permission] [--grant-prefix-uri-permission]
[--debug-log-resolution]
[--exclude-stopped-packages] [--include-stopped-packages]
[--activity-brought-to-front] [--activity-clear-top]
[--activity-clear-when-task-reset] [--activity-exclude-from-recents]
[--activity-launched-from-history] [--activity-multiple-task]
[--activity-no-animation] [--activity-no-history]
[--activity-no-user-action] [--activity-previous-is-top]
[--activity-reorder-to-front] [--activity-reset-task-if-needed]
[--activity-single-top] [--activity-clear-task]
[--activity-task-on-home]
[--receiver-registered-only] [--receiver-replace-pending]

//发送action=”broadcast.demo”的广播，并且对于forceStopPackage()的应用不允许接收该广播
am broadcast -a broadcast.demo --exclude-stopped-packages
```

## dumpsys

### [官放文档](https://developer.android.com/studio/command-line/dumpsys) [参考文章](http://gityuan.com/2016/05/14/dumpsys-command/)

```text
adb shell dumpsys -l //列出所支持的dump

adb shell dumpsys power // 查看和power manager service相关的信息，比如屏幕锁信息

adb shell dumpsys activity // 查询AMS服务相关信息, 比如查看activity栈信息

adb shell dumpsys cpuinfo // 查询CPU情况

adb shell dumpsys window // 查询WMS信息

adb shell dumpsys meminfo // 查询内存情况 [官方文档](https://developer.android.com/studio/profile/investigate-ram?hl=zh-cn)

dumpsys meminfo packageName // 查看某个应用的内存占用

adb shell dumpsys package // 查询PMS服务相关信息

adb shell dumpsys gfxinfo com.baidu.launcher // 查询gpu每帧处理时间

adb shell dumpsys meminfo | grep "duershowsettings"  // 查询pid
cat proc/<pid>/oom_adj // 查看adj
```

### Android 显示错位问题如何快速定位

首先排除应用window错位，查看window的 visible rect 查看是否有错位及遮挡问题

```text
adb shell dumpsys window windows | grep -E 'Window\ \#|    content\='
```

```text
 Window #0 Window{3ea2a2e u0 HVAC Passenger Temp}:
        content=[1776,162][1888,720] visible=[1776,162][1888,720]
  Window #1 Window{746973 u0 HVAC Driver Temp}:
        content=[32,162][144,720] visible=[32,162][144,720]
  Window #2 Window{3d00bc4 u0 HVAC Passenger Temp collapsed}:
        content=[1776,720][1888,720] visible=[1776,720][1888,720]
  Window #3 Window{a743871 u0 HVAC Driver Temp collapsed}:
        content=[32,720][144,720] visible=[32,720][144,720]
  Window #4 Window{7c45b8a u0 HVAC Container}:
        content=[0,0][1920,720] visible=[0,0][1920,720]
  Window #5 Window{ba24a01 u0 com.wt.launcher3}:
        content=[-1917,0][23,720] visible=[-1917,0][23,720]
  Window #6 Window{d7a4722 u0 StatusBar}:
        content=[0,0][1920,80] visible=[0,0][1920,80]
  Window #7 Window{b74f937 u0 AssistPreviewPanel}:
        content=[0,720][1920,720] visible=[0,720][1920,720]
  Window #8 Window{cbad522 u0 com.wt.multimedia.platform3/com.wt.multimedia.platform3.video.view.VideoActivity}:
        content=[40,100][632,620] visible=[40,100][632,620]
  Window #9 Window{9ccf7dd u0 com.wt.hub/com.wt.hubimpl.SETMiniAct}:
        content=[1288,284][1880,436] visible=[1288,284][1880,436]
  Window #10 Window{6157bd8 u0 com.wt.hub/com.wt.hubimpl.FREMiniAct}:
        content=[1288,100][1880,252] visible=[1288,100][1880,252]
  Window #11 Window{77b814 u0 com.tencent.wecarnavi/com.tencent.wecarnavi.BriefActivity}:
        content=[664,100][1256,620] visible=[664,100][1256,620]
  Window #12 Window{930ae7 u0 com.wt.launcher3/com.wt.launcher3.MainActivity}:
        content=[0,0][1920,720] visible=[0,0][1920,720]
  Window #13 Window{27282b u0 com.tencent.wecarnavi/com.tencent.wecarnavi.MainActivity}:
        content=[0,0][1920,720] visible=[0,0][1920,720]
  Window #14 Window{c402d03 u0 com.android.systemui.ImageWallpaper}:
        content=[0,0][1920,720] visible=[0,0][1920,720]

```

## content

```text
usage: adb shell content [subcommand] [options]

usage: adb shell content insert --uri <URI> [--user <USER_ID>] --bind <BINDING> [--bind <BINDING>...]
  <URI> a content provider URI.
  <BINDING> binds a typed value to a column and is formatted:
  <COLUMN_NAME>:<TYPE>:<COLUMN_VALUE> where:
  <TYPE> specifies data type such as:
  b - boolean, s - string, i - integer, l - long, f - float, d - double
  Note: Omit the value for passing an empty string, e.g column:s:
  Example:
  # Add "new_setting" secure setting with value "new_value".
  adb shell content insert --uri content://settings/secure --bind name:s:new_setting --bind value:s:new_value

usage: adb shell content update --uri <URI> [--user <USER_ID>] [--where <WHERE>]
  <WHERE> is a SQL style where clause in quotes (You have to escape single quotes - see example below).
  Example:
  # Change "new_setting" secure setting to "newer_value".
  adb shell content update --uri content://settings/secure --bind value:s:newer_value --where "name='new_setting'"

usage: adb shell content delete --uri <URI> [--user <USER_ID>] --bind <BINDING> [--bind <BINDING>...] [--where <WHERE>]
  Example:
  # Remove "new_setting" secure setting.
  adb shell content delete --uri content://settings/secure --where "name='new_setting'"

usage: adb shell content query --uri <URI> [--user <USER_ID>] [--projection <PROJECTION>] [--where <WHERE>] [--sort <SORT_ORDER>]
  <PROJECTION> is a list of colon separated column names and is formatted:
  <COLUMN_NAME>[:<COLUMN_NAME>...]
  <SORT_ORDER> is the order in which rows in the result should be sorted.
  Example:
  # Select "name" and "value" columns from secure settings where "name" is equal to "new_setting" and sort the result by name in ascending order.
  adb shell content query --uri content://settings/secure --projection name:value --where "name='new_setting'" --sort "name ASC"

usage: adb shell content call --uri <URI> --method <METHOD> [--arg <ARG>]
       [--extra <BINDING> ...]
  <METHOD> is the name of a provider-defined method
  <ARG> is an optional string argument
  <BINDING> is like --bind above, typed data of the form <KEY>:{b,s,i,l,f,d}:<VAL>

usage: adb shell content read --uri <URI> [--user <USER_ID>]
  Example:
  # cat default ringtone to a file, then pull to host
  adb shell 'content read --uri content://settings/system/ringtone > /mnt/sdcard/tmp.ogg' && adb pull /mnt/sdcard/tmp.ogg

usage: adb shell content gettype --uri <URI> [--user <USER_ID>]
  Example:
  # Show the mime-type of the URI
  adb shell content gettype --uri content://media/internal/audio/media/
```

### contentprovider call调试

```text
adb shell content call --uri content://com.baidu.duer.duershowsettings.provider --method abc --arg abc --extra <KEY>:{b,s,i,l,f,d}:<VAL>
```

