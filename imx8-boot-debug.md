# **imx8 安卓 P 开机调试**
# 背景：好帮手提供 boot，设备树等底层刷机包。梧桐提供 system.img。下载 imx8 9.0 code。
lunch mek_8q_car-userdebug。
# 1 原始 system.img 刷机之后会出现不开机问题：
## 获取校验算法失败
```
[11:52:16][ 3.670835] init: Failed to open FsManagerAvbHandle: File exists
[11:52:16][ 3.676903] init: Failed to setup verity for '/vendor': File exists
[11:52:16][ 3.683275] init: Failed to mount required partitions early ...
```
## 解决方法，取消校验：
```
jinlongxu@jinlongxu:~/src/imx8/android_build/system/core$ git diff
diff --git a/fs_mgr/fs_mgr_avb.cpp b/fs_mgr/fs_mgr_avb.cpp
index 7824cfa70..957855382 100644
--- a/fs_mgr/fs_mgr_avb.cpp
+++ b/fs_mgr/fs_mgr_avb.cpp
@@ -541,8 +541,9 @@ FsManagerAvbUniquePtr FsManagerAvbHandle::DoOpen(FsManagerAvbOps*
avb_ops) {
avb_vbmeta_image_header_to_host_byte_order(
(AvbVBMetaImageHeader*)avb_handle->avb_slot_data_->vbmeta_images[0].vbmeta_data,
&vbmeta_header);
+
/* vbmeta_header.flags is Zero*/
bool verification_disabled =
-((AvbVBMetaImageFlags)vbmeta_header.flags & AVB_VBMETA_IMAGE_FLAGS_VERIFICATION_DISABLED);
+((AvbVBMetaImageFlags)vbmeta_header.flags | AVB_VBMETA_IMAGE_FLAGS_VERIFICATION_DISABLED);
```
# 2 ADB 无法工作问题：
## 分析过程：
1 原始 system 可以工作，自己编译的无法工作。
```
[ 5293.711507] init: Service 'adbd' (pid 14465) received signal 9
[ 5293.712226] init: processing action (init.svc.adbd=stopped) from (/init.usb.configfs.rc:14)
[ 5293.735179] init: Received control message 'start' for 'adbd' from pid: 2855 (/vendor/bin/hw/android.hard
ware.usb@1.1-service.imx)
[ 5293.735259] init: starting service 'adbd'...
[ 5293.750704] read descriptors
[ 5293.750720] read strings
//正常情况下会进行 USB 枚举，异常时则不会枚举
[ 5294.065812] android_work: sent uevent USB_STATE=CONNECTED
[ 5294.072578] android_work: sent uevent USB_STATE=DISCONNECTED
[ 5294.215111] android_work: sent uevent USB_STATE=CONNECTED
[ 5294.220542] configfs-gadget gadget: high-speed config #1: b[ 5294.228078] android_work: sent uevent USB_STATE=CONFIGURED
```
## 2 因此使用 simg2img 工具 将原始的 system.img 解包：
```
sudo ./../android_build/out/host/linux-x86/bin/simg2img system.img system.img.raw
sudo mount -t ext4 -o loop images/system.img.raw mnt-system/
```
## 3 对比原始文件与自己编译的文件发现不同
init.recovery.freescale.rc 出现差异
解决方法，修改 usb 配置参数：
```
jinlongxu@jinlongxu:~/src/imx8/android_build/device/fsl$ git diff
diff --git a/imx8q/mek_8q/init.usb.rc b/imx8q/mek_8q/init.usb.rc
index 79767e0..0f3ab0b 100644
--- a/imx8q/mek_8q/init.usb.rc
+++ b/imx8q/mek_8q/init.usb.rc
@@ -44,7 +44,7 @@ on early-boot
mount
functionfs
mtp /dev/usb-ffs/mtp
ptp /dev/usb-ffs/ptp
rmode=0770,fmode=0660,uid=1024,gid=1024,no_disconnect=1
mount
functionfs
rmode=0770,fmode=0660,uid=1024,gid=1024,no_disconnect=1
setprop sys.usb.mtp.device_type 3
- setprop vendor.usb.config "gadget-cdns3"
+ setprop vendor.usb.config "ci_hdrc.0"
write /sys/module/libcomposite/parameters/disable_l1_for_hs "y"
symlink /config/usb_gadget/g1/configs/b.1 /config/usb_gadget/g1/os_desc/b.1
```
# 3 WIFI 无法工作问题
## 分析过程：
Wifi 无法进行扫描，并且 Kernel 驱动没有建立 wlan0 节点，导致 wifi HAL 无法 open 驱动节点，最终 timeout 失败
```
mek_8q:/sys/class/net # ls
ip6_vti0 ip6tnl0 ip_vti0 lo sit0
//出现 open 驱动超时
02-21 12:28:58.108 2858 2858 E WifiHAL : Timed out wating on Driver ready ...
02-21 12:28:58.108 2858 2858 E android.hardware.wifi@1.0-service: Timed out awaiting driver ready
02-21 12:28:58.108
2858
2858 E android.hardware.wifi@1.0-service: Failed to start legacy HAL:
TIMED_OUT
02-21 12:28:58.109
2978
3241 E HalDevMgr: executeChipReconfiguration: configureChip error: 9 (, timed
out)
```
## 解决方法：
【1】同上解包 system.img,对比发现不同文件：init.freescale.rc 出现差异,需要使能加载 ko 的脚本
```
diff --git a/imx8q/mek_8q/init_car.rc b/imx8q/mek_8q/init_car.rc
index bc154ec..41f4477 100644
--- a/imx8q/mek_8q/init_car.rc
+++ b/imx8q/mek_8q/init_car.rc
@@ -247,7 +247,6 @@ service boot_completed_main_sh /vendor/bin/init.insmod.sh /vendor/etc/setup.main
class main
user rootgroup root system
-
disabled
oneshot
on fs
```
【2】wlan.ko 是 Kernel 驱动编译生成的，需要好帮手提供。我们自己没有这部分代码。
```
adb push wlan.ko /vendor/lib/modules/
```