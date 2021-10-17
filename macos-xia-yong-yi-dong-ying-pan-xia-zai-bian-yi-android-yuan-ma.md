# MacOS下用移动硬盘下载编译Android源码

## 下载

[android细分版本号列表](https://source.android.com/setup/start/build-numbers)

### 制作一个大小写敏感的磁盘分区

磁盘工具 &gt; 选中移动硬盘 &gt; 文件 &gt; 新建映像 &gt; 空白映像 &gt; 设置映像 &gt; 存储 &gt; 启动磁盘分区

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

#### 遇到的问题1

```text
ninja: build stopped: subcommand failed.
build/core/ninja.mk:148: recipe for target ‘ninja_wrapper’ failed
make: *** [ninja_wrapper] Error 1
```

解决：

```text
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
out/host/linux-x86/bin/jack-admin kill-server
out/host/linux-x86/bin/jack-admin start-server
```

#### 遇到的问题2

```text
warning: gpg (GnuPG) is not available. warning: Installing it is strongly encouraged.
```

解决：

```text
brew install gpg
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

### 环境

```text
xcode-select --install
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
export PATH=/opt/local/bin:$PATH
POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg
POSIXLY_CORRECT=1 sudo port install bison
```

### 开始编译

```text
source build/envsetup.sh
make -j4
```

## 查看Android源码版本

```text
build\core\version_defaults.mk //搜索该文件中的 PLATFORM_VERSION值
```

