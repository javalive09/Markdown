# Ubunut20.04下载/编译/调试Android源码

## 下载

[android细分版本号列表](https://source.android.com/setup/start/build-numbers)

### 从官网下载android源码 <a id="cong-guan-wang-xia-zai-android-yuan-ma"></a>

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

### 使用清华源下载源码 <a id="shi-yong-qing-hua-yuan-xia-zai-yuan-ma"></a>

```text
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

export HTTP_PROXY=
export HTTPS_PROXY=
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.1.0_r10 // -b 后面为选择的一个[android细分版本号列表](https://source.android.com/setup/start/build-numbers) 中的一个分支号

repo sync // start download
```

### 查看源码版本

```text
build\core\version_defaults.mk //搜索该文件中的 PLATFORM_VERSION值
```

## 编译

### 安装基本的依赖软件 <a id="an-zhuang-ji-ben-de-yi-lai-ruan-jian"></a>

```text
sudo apt update
sudo apt upgrade
```

```text
sudo apt install openssh-server screen python git openjdk-8-jdk android-tools-adb bc bison \
build-essential curl flex g++-multilib gcc-multilib gnupg gperf imagemagick lib32ncurses-dev \
lib32readline-dev lib32z1-dev  liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev \
libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc yasm zip zlib1g-dev \
libtinfo5 libncurses5
```

### 开始编译

```text
source build/envsetup.sh // 用于运行shell脚本命令，功能等价于”.”，因此该命令也等价于. build/envsetup.sh
lunch // 指定此次编译的目标设备以及编译类型
make update-api //更新系统接口
make  -j4 2>&1 | tee build.log
```

如果出现out of memery编译错误，需要执行如下操作

```text
export _JAVA_OPTIONS="-Xmx8g"
make  -j4 2>&1 | tee build.log
```

## 调试

### 生成IDE配置文件

检查out/host/linux-x86/framework/目录下是否存在idegen.jar文件,存在则说明你已经编译过该模块,否者,则需要编译.执行如下命令即可:

```text
source build/envsetup.sh
lunch xx
mmm development/tools/idegen/
sudo ./development/tools/idegen/idegen.sh
```

其中mmm development/tools/idegen/执行完成后会生成idegen.jar,而sodo ./development/tools/idegen/idegen.sh则会在源码目录下生成IEDA工程配置文件:android.ipr,android.iml及android.iws

### 导入源码

打开Android Studio,点击File-&gt;Open,选择刚才生成的android.ipr文件即可,然后就是漫长的等待。

### 配置代码依赖

File -&gt; Project Structure 打开 Module，然后选中 Dependencies， 保留 JDK 跟 Module Source 项，并添加源码的 external 和 frameworks 依赖 然后是 SDK 的设置，确保关联对应版本的 SDK 于系统版本一致。

### 开始调试

[参考文章](https://blog.csdn.net/qq_32452623/article/details/53983563) usb连接设备后 点击debug图标 --&gt; show all processes --&gt; 要调试的进程 在需要的地方加入断点即可调试



