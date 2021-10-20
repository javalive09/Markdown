# **使用VsCode调试Android Framework C/C++源代码**
AndroidStudio支持对Framework中的java源码调试，这极大方便了我们对源码逻辑的跟踪及调试。但是AndroidStudio不支持对C/C++的调试，通常我们使用gdb对C/C++源码进行跟踪调试https://www.gnu.org/software/gdb/，但是使用gdb调试及为不便，效率不高。android官网推荐我们使用VsCode + gdb进行调试https://source.android.google.cn/devices/tech/debug/gdb?hl=zh-cn，具体使用过程如下（以xxBaseOS_android10为例）：

# 1.下载源码并编译
```
repo init -u gerritserver:manifest -m xxBaseOS_android10.xml
repo sync -c -j4
./Tmake xxBaseOSFramework -v userdebug -n
```
# 2.启动emulator模拟器
```
cd android/release/xxBaseOSFramework-T0016
./emulator.s
```
# 3.查找System_Server进程
```
adb shell ps | grep system_server
system        1923  1656 4413480 206276 0                   0 S system_server
```
# 4.执行gdbclient.py
```
source build/envsetup.sh
lunch aosp_car_x86_64-userdebug
gdbclient.py -p 1923 --setup-forwarding vscode
```

输出结果如下：
```
peter@peter:~/Documents/xxBaseOS_android10/android$ gdbclient.py -p 1923 --setup-forwarding vscode
WARNING:root:Couldn't find local unstripped executable in /home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols, symbols may not be available.
Redirecting gdbserver output to /tmp/gdbclient.log
 
{
    "miDebuggerPath": "/home/peter/Documents/xxBaseOS_android10/android/prebuilts/gdb/linux-x86/bin/gdb",
    "program": "/tmp/gdbclient-binary-8658",
    "setupCommands": [
        {
            "text": "-enable-pretty-printing",
            "description": "Enable pretty-printing for gdb",
            "ignoreFailures": true
        },
        {
            "text": "-environment-directory /home/peter/Documents/xxBaseOS_android10/android",
            "description": "gdb command: dir",
            "ignoreFailures": false
        },
        {
            "text": "-gdb-set solib-search-path /home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/hw:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/ssl/engines:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/drm:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/egl:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/soundfx:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/hw:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/egl",
            "description": "gdb command: set solib-search-path",
            "ignoreFailures": false
        },
        {
            "text": "-gdb-set solib-absolute-prefix /home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols",
            "description": "gdb command: set solib-absolute-prefix",
            "ignoreFailures": false
        },
        {
            "text": "-interpreter-exec console \"source /home/peter/Documents/xxBaseOS_android10/android/development/scripts/gdb/dalvik.gdb\"",
            "description": "gdb command: source art commands",
            "ignoreFailures": false
        }
    ],
    "name": "(gdbclient.py) Attach gdbclient-binary-8658 (port: 5039)",
    "miDebuggerServerAddress": "localhost:5039",
    "request": "launch",
    "type": "cppdbg",
    "cwd": "/home/peter/Documents/xxBaseOS_android10/android",
    "MIMode": "gdb"
}
 
 
Paste the above json into .vscode/launch.json and start the debugger as
normal. Press enter in this terminal once debugging is finished to shutdown
the gdbserver and close all the ports.
 
Press enter to shutdown gdbserver
```
# 5. vscode添加配置

在 VS Code 的“调试”标签页中，选择添加配置，然后选择 LLDB：自定义启动。这将打开一个 launch.json 文件，并将新的 JSON 对象添加到列表中，删除新添加的调试程序配置。复制 lldbclient.py 输出的 JSON 对象并将其粘贴到您刚删除的对象中。保存更改。
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "miDebuggerPath": "/home/peter/Documents/xxBaseOS_android10/android/prebuilts/gdb/linux-x86/bin/gdb",
            "program": "/tmp/gdbclient-binary-8658",
            "setupCommands": [
                {
                    "text": "-enable-pretty-printing",
                    "description": "Enable pretty-printing for gdb",
                    "ignoreFailures": true
                },
                {
                    "text": "-environment-directory /home/peter/Documents/xxBaseOS_android10/android",
                    "description": "gdb command: dir",
                    "ignoreFailures": false
                },
                {
                    "text": "-gdb-set solib-search-path /home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/hw:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/ssl/engines:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/drm:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/egl:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/system/lib64/soundfx:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/hw:/home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols/vendor/lib64/egl",
                    "description": "gdb command: set solib-search-path",
                    "ignoreFailures": false
                },
                {
                    "text": "-gdb-set solib-absolute-prefix /home/peter/Documents/xxBaseOS_android10/android/out/target/product/generic_x86_64/symbols",
                    "description": "gdb command: set solib-absolute-prefix",
                    "ignoreFailures": false
                },
                {
                    "text": "-interpreter-exec console \"source /home/peter/Documents/xxBaseOS_android10/android/development/scripts/gdb/dalvik.gdb\"",
                    "description": "gdb command: source art commands",
                    "ignoreFailures": false
                }
            ],
            "name": "(gdbclient.py) Attach gdbclient-binary-8658 (port: 5039)",
            "miDebuggerServerAddress": "localhost:5039",
            "request": "launch",
            "type": "cppdbg",
            "cwd": "/home/peter/Documents/xxBaseOS_android10/android",
            "MIMode": "gdb"
        }
    ]
}
```

# 6.调试

加入断点，选择新的调试程序配置，然后按运行。调试程序应在 10 到 30 秒后连接。
![](.gitbook/assets/vscode-debug-framework.png)