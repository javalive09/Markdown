# Ubunut16.04下载/编译/调试Android源码

## 下载

[android细分版本号列表](https://source.android.com/setup/start/build-numbers)

### 从官网下载android源码

[官方文档](https://source.android.com/source/downloading)

```text
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

repo init -u https://android.googlesource.com/platform/manifest
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1

repo sync //start download
```

### 使用清华源下载源码

#### 下载repo

```text
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

#### 下载

```text
export HTTP_PROXY=
export HTTPS_PROXY=
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.1.0_r10 // -b 后面为选择的一个[android细分版本号列表](https://source.android.com/setup/start/build-numbers) 中的一个分支号
repo sync // start download
```

## 编译

### 安装基本的依赖软件

因为用的是mac环境，所以先安装Homebrew（这是mac下超强的包管理工具）。 homebrew官网：[http://brew.sh/index\_zh-cn.html](http://brew.sh/index_zh-cn.html)

```text
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386 
sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib
sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386
sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
sudo apt-get install git-core gnupg flex bison gperf build-essential 
sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib
sudo apt-get install libc6-dev-i386
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev
sudo apt-get install lib32z-dev ccache
sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
sudo apt-get install gawk
```

### 根据自己所下载的Android代码的版本，选择安装所需的jdk版本

#### Android版本是7.1.2，所需的JDK版本是openjdk-8-jdk。Ubuntu默认自带的openjdk-8-jdk的安装源，可以直接安装，使用下面的命令安装即可

```text
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

#### 如果使用的OpenJDK7，由于Ubuntu 16.04没有OpenJDK7的源，因此要先添加源，然后在安装OpenJDK7，按下面的命令操作即可

```text
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update
sudo apt-get install openjdk-7-jdk
```

#### 安装完以后，执行下面的命令添加JAVA\_HOME相关配置

```text
sudo gedit /etc/profile

// 在打开的profile文件的末尾添加下面的内容：
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH

// 修改了/etc/profile文件需要重启才能生效，但使用下面命令可以在不重启的情况下在当前bash环境生效：
source /etc/profile

// 最后，检查一下jdk是否安装成功：
java -version
```

### 切换java版本（方法一）

默认使用jdk 8 当要用到jdk7时在终端执行：（只对当前终端有效）

```text
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH
```

### 切换java版本（方法二）

```text
sudo update-alternatives --config java
```

### 开始编译

```text
source build/envsetup.sh // 用于运行shell脚本命令，功能等价于”.”，因此该命令也等价于. build/envsetup.sh
lunch // 指定此次编译的目标设备以及编译类型
make update-api //更新系统接口
make -j4
```

#### 编译某个模块

```text
m：编译所有的模块
mm：编译当前目录下的模块，当前目录下要有Android.mk文件 
mmm：编译指定路径下的模块，指定路径下要有Android.mk文件
```

#### 清理编译产物

```text
make clobber // 相当于 rm -rf out/
make install clean // 在切换编译目标时需要执行make installclean，用以清除之前编译生成的文件。
```

#### [编译相关知识的帖子](http://gityuan.com/2016/03/19/android-build/)

## 查看Android源码版本

```text
build\core\version_defaults.mk //搜索该文件中的 PLATFORM_VERSION值
```

## 调试

### 准备

检查out/host/linux-x86/framework/目录下是否存在idegen.jar文件,存在则说明你已经编译过该模块,否者,则需要编译.执行如下命令即可:

```text
source build/envsetup.sh
lunch xx
mmm development/tools/idegen/
sudo ./development/tools/idegen/idegen.sh
```

其中mmm development/tools/idegen/执行完成后会生成idegen.jar,而sodo ./development/tools/idegen/idegen.sh则会在源码目录下生成IEDA工程配置文件:android.ipr,android.iml及android.iws.

#### 三个文件的作用

android.ipr:通常是保存工程相关的设置,比如编译器配置,入口,相关的libraries等 android.iml:则是主要是描述了modules,比如modules的路径,依赖关系等. android.iws:则主要是包含了一些个人工作区的设置.

### 导入

#### 修改AS配置文件

修改一下Android studio的配置:32位系统下修改idea.vmoptions,64位下修改idea64.vmotions 调整其中的-Xms和-Xmx参数值,官方要求至少在748m以上,根据实际情况进行配置即可.

#### 导入源码

打开Android Studio,点击File-&gt;Open,选择刚才生成的android.ipr文件即可,然后就是漫长的等待。很多情况下,我们希望不导入某些模块,那么就可以在导入前修改android.iml文件 只保留了framworks和packages模块,将其他模块全部排除,如下:

```text
<excludeFolder url="file://$MODULE_DIR$/.repo" />
<excludeFolder url="file://$MODULE_DIR$/abi" />
<excludeFolder url="file://$MODULE_DIR$/art" />
<excludeFolder url="file://$MODULE_DIR$/bionic" />
<excludeFolder url="file://$MODULE_DIR$/bootable" />
<excludeFolder url="file://$MODULE_DIR$/build" />
<excludeFolder url="file://$MODULE_DIR$/cts" />
<excludeFolder url="file://$MODULE_DIR$/dalvik" />
<excludeFolder url="file://$MODULE_DIR$/developers" />
<excludeFolder url="file://$MODULE_DIR$/development" />
<excludeFolder url="file://$MODULE_DIR$/device" />
<excludeFolder url="file://$MODULE_DIR$/docs" />
<excludeFolder url="file://$MODULE_DIR$/external" />
<excludeFolder url="file://$MODULE_DIR$/hardware" />
<excludeFolder url="file://$MODULE_DIR$/libcore" />
<excludeFolder url="file://$MODULE_DIR$/libnativehelper" />
<excludeFolder url="file://$MODULE_DIR$/ndk" />
<excludeFolder url="file://$MODULE_DIR$/out" />
<excludeFolder url="file://$MODULE_DIR$/pdk" />
<excludeFolder url="file://$MODULE_DIR$/prebuilt" />
<excludeFolder url="file://$MODULE_DIR$/prebuilts" />
<excludeFolder url="file://$MODULE_DIR$/sdk" />
<excludeFolder url="file://$MODULE_DIR$/system" />
<excludeFolder url="file://$MODULE_DIR$/tools" />
```

AS加载完成后，查看目录Poject 列表下 红色的目录代表被 exclude 排除了

#### 配置代码依赖，确保代码跳转正确

[参考文章](https://blog.csdn.net/aaa111/article/details/43227367) File -&gt; Project Structure 打开 Module，然后选中 Dependencies， 保留 JDK 跟 Module Source 项，并添加源码的 external 和 frameworks 依赖 然后是 SDK 的设置，确保关联对应版本的 SDK 于系统版本一致

#### 遇到Android studio 不停 scanning files to index

Open module setting --&gt; Modules --&gt; 找到gen文件夹 --&gt; 右键选择Resources

### 开始调试

[参考文章](https://blog.csdn.net/qq_32452623/article/details/53983563) usb连接设备后 点击debug图标 --&gt; show all processes --&gt; 要调试的进程 在需要的地方加入断点即可调试

## 遇到问题

### ubuntu 无法识别移动硬盘

```text
sudo apt-get install exfat-fuse exfat-utils
```

### Ubuntu 16.04安装sogou 拼音输入法

[下载](http://pinyin.sogou.com/linux/?r=pinyin) [参考](https://blog.csdn.net/ljheee/article/details/52966456)

```text
sudo apt-get install -f
sudo dpkg -i sogoupinyin_2.0.0.0072_amd64.deb
```

### Ubuntu 配置adb环境

```text
vi ~/.bashrc
export PATH=/home/homer/android-sdk-linux/platform-tools/:$PATH //重启后生效
source ~/.bashrc //立即生效
```

### 生成android.jar

1.首先编译Android的系统，其实直接编译framework也行，在编译完成后在out\target\common\obj\JAVA\_LIBRARIES\framework\_intermediates下面有个classes.jar的文件，我们就需要这个jar文件。

2.将classes.jar放在某个文件夹下面，然后将其解压，我们可以得到一个android文件夹和META\_INF文件夹，

3.找到我们常使用的sdk版本目录下面的android.jar,比如:sdk\platforms\android-19\android.jar,将其放在某个文件夹下面解压。解压后会得到很多个文件夹，包括:android,java,com,javax,org,META\_INF等文件夹以及resources.arsc文件。

4.将上面classes.jar解压出来的android文件夹下面的所有文件，复制到android.jar解压出来的android文件夹里面，并覆盖相同文件名的文件及文件夹。其实你注意上面的两个android文件夹内容大致一样。

5.面到了最关键的一步，就是把现有的文件夹中的classes打包成java文件，首先在解压android.jar的根目录下面打开cmd命令窗口，输入命令jar cvfm android.jar META-INF/MANIFEST.MF ./ 。如果找不到jar，请先配置环境变量。接着就看到这个打包的详细信息在控制台输出。关于jar的详细命令参数请看这里 [jar命令详解](https://blog.csdn.net/xyw_blog/article/details/8827937)顺利的话，就可以得到一个android.jar 文件了。

## 开机自动挂载硬盘

### 查看分区信息

```text
sudo fdisk -l
```

### 创建/data目录（/data目录为硬盘将挂载的地方）：

```text
sudo mkdir /data
```

### 挂载硬盘

```text
sudo mount /dev/sdb1 /data
```

### 查看磁盘分区的UUID

```text
sudo blkid
```

### 配置开机自动挂载

因为mount命令会在重启服务器后失效，所以要将分区信息写到/etc/fstab文件中让它永久挂载

```text
sudo vim /etc/fstab
```

加入

```text
UUID=11263962-9715-473f-9421-0b604e895aaa /data ext4 defaults 0 1
```

### 验证

命令验证一下配置是否正确。如果配置不正确可能会导致系统无法正常启动。

```text
sudo mount -a
```

### 重启

