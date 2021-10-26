# **init.rc实例-create daemon process**
本次主要是在init.rc文件中创建一个抓取log的进程，通过配置service标记来声明service进程，property属性出发开启关闭service进程。

# 第一种service方式：启动对应的可执行进程，并传递参数

service logcat_service /system/bin/logcat -b system -b events -b main -n 1000 -v threadtime -r10240 -f /sdcard/WTLog/mobilelog
user root                        执行此进程时候切换用户为配置的用户
group log system          执行此进程时候用户组配置为配置用户组（在启动这个服务前改变该服务的组名。除了（必需的）第一个组名，附加的组名通常被用于设置进程的补充组）
class main                     指定一个服务类。所有同一类的服务可以同时启动和停止。如果不通过class选项指定一个类，则默认为"default"类服务。比如：init.rc中的 class_start main就是启动配置了此配置的服务
disabled                        说明这个服务不会同与他同trigger（触发器）下的服务自动启动。他必须被明确的按名启动。

# 第二种service方式：把对应的sh脚本当作一个可执行进程，在sh脚本中可以一句条件选取参数然后启动对应的logcat。

service logd /system/bin/sh /system/bin/logd.sh
class main
user root
group root shell
oneshot                 代表init中只启动一次，此后进程结束掉后不会重新多次尝试启动。

两种方式修改配置连接(4.4平台版本)：http://10.1.120.4/gerrit/#/c/WT01_LogPick/+/1170/。  d701994.diff.zip

ps:新的平台版本需要配置selinux权限，请大家注意。