# **init.rc语法**

# 此文件是原始版init.rc中语法配置讲解。 readme.txt
```

Android Init Language
---------------------

The Android Init Language consists of four broad classes of statements,
which are Actions, Commands, Services, and Options.

All of these are line-oriented, consisting of tokens separated by
whitespace.  The c-style backslash escapes may be used to insert
whitespace into a token.  Double quotes may also be used to prevent
whitespace from breaking text into multiple tokens.  The backslash,
when it is the last character on a line, may be used for line-folding.

Lines which start with a # (leading whitespace allowed) are comments.

Actions and Services implicitly declare a new section.  All commands
or options belong to the section most recently declared.  Commands
or options before the first section are ignored.

Actions and Services have unique names.  If a second Action or Service
is declared with the same name as an existing one, it is ignored as
an error.  (??? should we override instead)


Actions
-------
Actions are named sequences of commands.  Actions have a trigger which
is used to determine when the action should occur.  When an event
occurs which matches an action's trigger, that action is added to
the tail of a to-be-executed queue (unless it is already on the
queue).

Each action in the queue is dequeued in sequence and each command in
that action is executed in sequence.  Init handles other activities
(device creation/destruction, property setting, process restarting)
"between" the execution of the commands in activities.

Actions take the form of:

on <trigger>
   <command>
   <command>
   <command>


Services
--------
Services are programs which init launches and (optionally) restarts
when they exit.  Services take the form of:

service <name> <pathname> [ <argument> ]*
   <option>
   <option>
   ...


Options
-------
Options are modifiers to services.  They affect how and when init
runs the service.

critical
   This is a device-critical service. If it exits more than four times in
   four minutes, the device will reboot into recovery mode.

disabled
   This service will not automatically start with its class.
   It must be explicitly started by name.

setenv <name> <value>
   Set the environment variable <name> to <value> in the launched process.

socket <name> <type> <perm> [ <user> [ <group> ] ]
   Create a unix domain socket named /dev/socket/<name> and pass
   its fd to the launched process.  <type> must be "dgram", "stream" or "seqpacket".
   User and group default to 0.

user <username>
   Change to username before exec'ing this service.
   Currently defaults to root.  (??? probably should default to nobody)
   Currently, if your process requires linux capabilities then you cannot use
   this command. You must instead request the capabilities in-process while
   still root, and then drop to your desired uid.

group <groupname> [ <groupname> ]*
   Change to groupname before exec'ing this service.  Additional
   groupnames beyond the (required) first one are used to set the
   supplemental groups of the process (via setgroups()).
   Currently defaults to root.  (??? probably should default to nobody)

seclabel <securitycontext>
  Change to securitycontext before exec'ing this service.
  Primarily for use by services run from the rootfs, e.g. ueventd, adbd.
  Services on the system partition can instead use policy-defined transitions
  based on their file security context.
  If not specified and no transition is defined in policy, defaults to the init context.

oneshot
   Do not restart the service when it exits.

class <name>
   Specify a class name for the service.  All services in a
   named class may be started or stopped together.  A service
   is in the class "default" if one is not specified via the
   class option.

onrestart
    Execute a Command (see below) when service restarts.

Triggers
--------
   Triggers are strings which can be used to match certain kinds
   of events and used to cause an action to occur.

boot
   This is the first trigger that will occur when init starts
   (after /init.conf is loaded)

<name>=<value>
   Triggers of this form occur when the property <name> is set
   to the specific value <value>.

device-added-<path>
device-removed-<path>
   Triggers of these forms occur when a device node is added
   or removed.

service-exited-<name>
   Triggers of this form occur when the specified service exits.


Commands
--------

exec <path> [ <argument> ]*
   Fork and execute a program (<path>).  This will block until
   the program completes execution.  It is best to avoid exec
   as unlike the builtin commands, it runs the risk of getting
   init "stuck". (??? maybe there should be a timeout?)

export <name> <value>
   Set the environment variable <name> equal to <value> in the
   global environment (which will be inherited by all processes
   started after this command is executed)

ifup <interface>
   Bring the network interface <interface> online.

import <filename>
   Parse an init config file, extending the current configuration.

hostname <name>
   Set the host name.

chdir <directory>
   Change working directory.

chmod <octal-mode> <path>
   Change file access permissions.

chown <owner> <group> <path>
   Change file owner and group.

chroot <directory>
  Change process root directory.

class_start <serviceclass>
   Start all services of the specified class if they are
   not already running.

class_stop <serviceclass>
   Stop all services of the specified class if they are
   currently running.

domainname <name>
   Set the domain name.

insmod <path>
   Install the module at <path>

mkdir <path> [mode] [owner] [group]
   Create a directory at <path>, optionally with the given mode, owner, and
   group. If not provided, the directory is created with permissions 755 and
   owned by the root user and root group.

mount <type> <device> <dir> [ <mountoption> ]*
   Attempt to mount the named device at the directory <dir>
   <device> may be of the form mtd@name to specify a mtd block
   device by name.
   <mountoption>s include "ro", "rw", "remount", "noatime", ...

restorecon <path>
   Restore the file named by <path> to the security context specified
   in the file_contexts configuration.
   Not required for directories created by the init.rc as these are
   automatically labeled correctly by init.

setcon <securitycontext>
   Set the current process security context to the specified string.
   This is typically only used from early-init to set the init context
   before any other process is started.

setenforce 0|1
   Set the SELinux system-wide enforcing status.
   0 is permissive (i.e. log but do not deny), 1 is enforcing.

setkey
   TBD

setprop <name> <value>
   Set system property <name> to <value>.

setrlimit <resource> <cur> <max>
   Set the rlimit for a resource.

setsebool <name> <value>
   Set SELinux boolean <name> to <value>.
   <value> may be 1|true|on or 0|false|off

start <service>
   Start a service running if it is not already running.

stop <service>
   Stop a service from running if it is currently running.

symlink <target> <path>
   Create a symbolic link at <path> with the value <target>

sysclktz <mins_west_of_gmt>
   Set the system clock base (0 if system clock ticks in GMT)

trigger <event>
   Trigger an event.  Used to queue an action from another
   action.

wait <path> [ <timeout> ]
  Poll for the existence of the given file and return when found,
  or the timeout has been reached. If timeout is not specified it
  currently defaults to five seconds.

write <path> <string> [ <string> ]*
   Open the file at <path> and write one or more strings
   to it with write(2)


Properties
----------
Init updates some system properties to provide some insight into
what it's doing:

init.action 
   Equal to the name of the action currently being executed or "" if none

init.command
   Equal to the command being executed or "" if none.

init.svc.<name>
   State of a named service ("stopped", "running", "restarting")


Example init.conf
-----------------

# not complete -- just providing some examples of usage
#
on boot
   export PATH /sbin:/system/sbin:/system/bin
   export LD_LIBRARY_PATH /system/lib

   mkdir /dev
   mkdir /proc
   mkdir /sys

   mount tmpfs tmpfs /dev
   mkdir /dev/pts
   mkdir /dev/socket
   mount devpts devpts /dev/pts
   mount proc proc /proc
   mount sysfs sysfs /sys

   write /proc/cpu/alignment 4

   ifup lo

   hostname localhost
   domainname localhost

   mount yaffs2 mtd@system /system
   mount yaffs2 mtd@userdata /data

   import /system/etc/init.conf

   class_start default

service adbd /sbin/adbd
   user adb
   group adb

service usbd /system/bin/usbd -r
   user usbd
   group usbd
   socket usbd 666

service zygote /system/bin/app_process -Xzygote /system/bin --zygote
   socket zygote 666

service runtime /system/bin/runtime
   user system
   group system

on device-added-/dev/compass
   start akmd

on device-removed-/dev/compass
   stop akmd

service akmd /sbin/akmd
   disabled
   user akmd
   group akmd

Debugging notes
---------------
By default, programs executed by init will drop stdout and stderr into
/dev/null. To help with debugging, you can execute your program via the
Andoird program logwrapper. This will redirect stdout/stderr into the
Android logging system (accessed via logcat).

For example
service akmd /system/bin/logwrapper /sbin/akmd

```

# Android init.rc文件详解

本文主要来自$ANDROID_SOURCE/system/init/readme.txt的翻译.

## 1 简述

Android init.rc文件由系统第一个启动的init程序解析，此文件由语句组成，主要包含了四种类型的语句:Action,Commands,Services,Options.在init.rc文件中一条语句通常是占据一行.单词之间是通过空格符来相隔的.如果需要在单词内使用空格，那么得使用转义字符"\",如果在一行的末尾有一个反斜杠，那么是换行折叠符号，应该和下一行合并成一起来处理，这样做主要是为了避免一行的字符太长，与C语言中的含义是一致的。注释是以#号开头。 Action和services显式声明了一个语句块，而commands和options属于最近声明的语句块。在第一个语句块之前 的commands和options会被忽略.
在具体讲解这之前，有些关键词得先了解.

## 2 关键字

token:  计算机语言中的一个单词，就跟英文中的单词差不多一人概念.
Section: 语句块，相当于C语言中大括号内的一个块。一个Section以Service或On开头的语句块.以Service开头的Section叫做服务,而以On开头的叫做动作(Action).
services: 服务.
Action: 动作
commands:命令.
options:选项.
trigger:触发器，或者叫做触发条件.
class: 类属，即可以为多个service指定一个相同的类属，方便操作同时启动或停止.

## 3 语句解析

### 3.1 动作(Action)
动作表示了一组命令(commands)组成.动作包含一个触发器，决定了何时执行这个动作。当触发器的条件满足时，这个动作会被加入到已被执行的队列尾。如果此动作在队列中已经存在，那么它将不会执行.
 一个动作所包含的命令将被依次执行。动作的语法如下所示:
	
```
on <trigger>
  <command>
  <command>
  <command>
```

### 3.2 服务(services)

服务是指那些需要在系统初始化时就启动或退出时自动重启的程序.

它的语法结构如下所示:
```	

service <name> <pathname> [ <argument> ]*
  <option>
  <option>
  ...
```
### 3.3 选项（options)

选项是用来修改服务的。它们影响如何及何时运行这个服务.


|选项|描述|
|--|--|
|critical	|据设备相关的关键服务，如果在4分钟内，此服务重复启动了4次，那么设备将会重启进入还原模式。|
|disabled	|服务不会自动运行，必须显式地通过服务器来启动。|
|setenv <name> <value>	|设置环境变量|
|socket <name> <type> <perm> [ <user> [ <group> ] ]|	在/dev/socket/下创建一个unix domain的socket，并传递创建的文件描述符fd给服务进程.其中type必须为dgram或stream,seqpacket.用户名和组名默认为0|
|user <username>	|在执行此服务之前先切换用户名。当前默认为root.|
|group <groupname> [ <groupname> ]*	|类似于user,切换组名，在启动这个服务前改变该服务的组名。除了（必需的）第一个组名，附加的组名通常被用于设置进程的补充组|
|oneshot	|当此服务退出时不会自动重启.|
|class <name>	|给服务指定一个类属,这样方便操作多个服务同时启动或停止.默认情况下为default.|
|onrestart	|当服务重启时执行一条指令，|


### 3.4 触发器(trigger)

触发器用来描述一个触发条件，当这个触发条件满足时可以执行动作.


|触发器	|描述|
|-|-|
|boot	|当init程序执行，并载入/init.conf文件时触发.|
|<name>=<value>	|当属性名对应的值设置为指定值时触发.|
|device-added-<path>	|当添加设备时触发.|
|device-removed-<path>	|当设备移除时触发.|
|service-exited-<name>	|当指定的服务退出时触发.|


### 3.5 命令(commands)


|命令	|描述|
|-|-|
|exec <path> [ <argument> ]*	|执行指定路径下的程序，并传递参数.|
|export <name> <value>	|设置全局环境参数，此参数被设置后对所有进程都有效.|
|ifup <interface>	|使指定的网络接口"上线",相当激活指定的网络接口|
|import <filename>	|导入一个额外的init配置文件.|
|hostname <name>	|设置主机名|
|chdir <directory>	|改变工作目录.|
|chmod <octal-mode> <path>	|改变指定文件的读取权限.|
|chown <owner> <group> <path>	|改变指定文件的拥有都和组名的属性.|
|chroot <directory>	|改变进行的根目录.|
|class_start <serviceclass>	|启动指定类属的所有服务，如果服务已经启动，则不再重复启动.|
|class_stop <serviceclass>	|停止指定类属的所胡服务.|
|domainname <name>	|设置域名|
|insmod <path>|	安装模块到指定路径.|
|mkdir <path> [mode] [owner] [group]	|用指定参数创建一个目录，在默认情况下，创建的目录读取权限为755.用户名为root,组名为root.|
|mount <type> <device> <dir> [ <mountoption> ]*	|类似于linux的mount指令|
|setkey|	TBD(To Be Determined),待定.|
|setprop <name> <value>	|设置属性及对应的值.|
|setrlimit <resource> <cur> <max>	|设置资源的rlimit(资源限制），不懂就百度一下rlimit|
|start <service>	|如果指定的服务未启动，则启动它.|
|stop <service>	|如果指定的服务当前正在运行，则停止它.|
|symlink <target> <path>	|创建一个符号链接.|
|sysclktz <mins_west_of_gmt>	|设置系统基准时间.|
|trigger <event>	T|rigger an event.  Used to queue an action from another action.这名话没有理解，望高手指点.|
|write <path> <string> [ <string> ]*	|往指定的文件写字符串.|


### 3.6 属性(Properties)


|属性名	|描述|
|-|-|
|init.action	|当前正在执行的动作，如果没有则为空字符串""|
|init.command	|当前正在执行的命令.没有则为空字符串.|
|init.svc.<name>	|当前某个服务的状态，可为"stopped", "running", "restarting"|


## 4 一个 init.conf例子

```
# not complete -- just providing some examples of usage
#
on boot
  export PATH /sbin:/system/sbin:/system/bin
  export LD_LIBRARY_PATH /system/lib
  
  mkdir /dev
  mkdir /proc
  mkdir /sys
  
  mount tmpfs tmpfs /dev
  mkdir /dev/pts
  mkdir /dev/socket
  mount devpts devpts /dev/pts
  mount proc proc /proc
  mount sysfs sysfs /sys
  
  write /proc/cpu/alignment 4
  
  ifup lo
  
  hostname localhost
  domainname localhost
  
  mount yaffs2 mtd@system /system
  mount yaffs2 mtd@userdata /data
  
  import /system/etc/init.conf
  
  class_start default
  
service adbd /sbin/adbd
  user adb
  group adb
  
service usbd /system/bin/usbd -r
  user usbd
  group usbd
  socket usbd 666
  
service zygote /system/bin/app_process -Xzygote /system/bin --zygote
  socket zygote 666
  
service runtime /system/bin/runtime
  user system
  group system
  
on device-added-/dev/compass
  start akmd
  
on device-removed-/dev/compass
  stop akmd
  
service akmd /sbin/akmd
  disabled
  user akmd
  group akmd 
```
	

## 5 调试注意事项

在默认情况下，通过init程序启动的程序的标准输出stdout和标准错误输出stderr会重定向到/dev/null.如:
```
service akmd /system/bin/logwrapper /sbin/akmd
```
为了更方便调试你的程序，你可以使用Android的log系统，标准输出和标准错误输出会重定义到Android的log系统中来.