# 命令行

## cat查看log

```text
cat file.log | grep "ActivityManager" //过滤查看包含ActivityManager tag的log

cat file.log | grep "ActivityManager\|NEMO_WD" //过滤查看包含ActivityManager或NEMO_WD tag的log

cat file.log | grep " ANR in" -A 10 -B 10 > ~/anr.txt //过滤Anr前10行+后10行并写入anr.txt文件
```

## linux文件

### 列出目录下文件的权限

```text
ls －al
```

### 修改文件权限

权限的字母表示：u-user g-group o-others a-all 权限的数字表示：4-读 2-写 1-执行

```text
chmod 777 a.txt
chmod u=rwx, g=rwx, o=rwx a.txt
chmod a=rwx a.txt
```

### 修改文件夹权限

```text
chmod -R 777  folder
```

### 查看文件夹的大小

du 命令 disk use 参数含义： s:sum h:human can read

```text
du -sh ./sdk
```

### 查看磁盘使用情况

```text
du -h
```

### 查找文件

以当前目录为根目录查找名字为DownloadManager.Java的文件所在位置

```text
find <path> -name <fileName>
find . -name "DownloadManager.java"
```

忽略大小写

```text
find <path> -iname <regExp fileName>
find . -iname "DownloadManager.java"
```

### 查找字符串

查找某目录下所有包含DownloadManager字符的文件 参数含义： r:recurisive 递归 n:显示line number

```text
grep -rn <regExp string> <path>
grep -rn DownloadManager /Users/peter/Downloads/Android4.4源码
```

### 删除jar包中的文件夹

```text
zip --delete baseproject.jar "com/luajava/*"
```

### 列出历史命令

```text
history

history | more
```

### 链接文件

ln -s \[source\] \[target\]

```text
ln -s /Users/peter/blog/source/post-commit post-commit
```

### 环境变量

#### 列出所有的环境变量

```text
env
```

#### 加入环境变量

```text
export PATH=$PATH:[path]
export PATH=$PATH:/Users/peter/Downloads/protoc-3.3.0-osx-x86_64/bin
```

或

```text
vim ~/.bashrc
加入export PATH=$PATH:/Users/peter/Downloads/protoc-3.3.0-osx-x86_64/bin
```

### echo

```text
echo 'android-8.0.0_r17' >> readme.txt
//>> 是追加内容
//> 是覆盖原有内容
```

### 解压tar.gz

```text
tar -zxvf ×××.tar.gz
```

## python

### 开一个简单的服务器

#### python 2.x

```text
python -m SimpleHTTPServer [port]
```

#### python 3.x

```text
python -m http.server [port]
```

## 内存

### top命令

```text
top -h //help
top -m num //显示的最多进程数
top -s col //Column to sort by (cpu,vss,rss,thr)
top -m 10 -s cpu //ddd
```

## 网络

### ping网速

-c count 在收发指定的count个数目的报文后停止 -f 尽可能快地发送报文 -s packetsize 设置要发送的报文大小\(单位为字节，最大长度是64k，缺省长度是64\) -i wait 设置连续报文发送时报文间的发送时间间隔（单位为秒）

```text
ping -c 5 -s 1024 -i 2 www.baidu.com
```

发送 5次 1024 byte 间隔 2秒 到百度服务器。

## gcc

### 编译单个源文件

gcc xxx.c -o xxx

```text
gcc test.c -o test
```

执行./xxx 可运行

### 编译多个源文件

* gcc -c xxx.c 分别编译源文件为xxx.o
* gcc xxx.o yyy.o -o main

```text
gcc -c hotel.c
```

```text
gcc main.o hotel.o -o main
```

执行./main可运行

## 交互式脚本自动化

使用expect

```text
#!/usr/bin/expect
set timeout 30
spawn ./build.sh config //执行交互式脚本命令

expect "*chips*Choice:"
send "0\r"

expect "*platforms*Choice:"
send "0\r"

interact
```

## 重复执行自动化脚本

```text
#!/bin/sh
repo sync
while [ $? -ne 0 ]    # $? 表示上一个命令的执行结果    -ne表示不等于
do
repo sync
done
```

## curl

curl\(CommandLine Uniform Resource Locator\) [官网](https://curl.haxx.se/) [参考](https://piaosanlang.gitbooks.io/spiders/10day/section10.2.html) [参考](http://blog.fpliu.com/it/software/curl) 安装

```text
brew install curl
```

### 构造http请求

通用语法

```text
curl [option] [URL...]
```

#### 获取网页内容

```text
curl http://www.baidu.com
```

#### 获取网页内容并转码 iconv

```text
curl http://ip138.com|iconv -f gb2312
```

#### get方式提交数据

```text
curl -G -d "name=value&name2=value2" http://www.baidu.com
```

#### post方式提交数据

```text
curl -d "name=value&name2=value2" http://www.baidu.com #post数据
curl -d a=b&c=d&txt@/tmp/txt http://www.baidu.com  #post文件
```

#### 以表单的方式上传文件

```text
curl -F file=@/tmp/me.txt http://www.aiezu.com
```

#### 保存访问的网页

```text
curl http://www.baidu.com > page.html
```

#### 设置Header

```text
curl -H 'Host: 157.166.226.25'-H 'Accept-Language: es'-H 'Cookie: ID=1234' http://cnn.com
```

#### -I：显示文档信息

```text
curl -I http://www.sina.com.cn/ -H Accept-Encoding:gzip,defalte
```

#### -x ：可以指定http访问所使用的proxy服务器及其端口

```text
curl -x 123.45.67.89:1080 -o page.html http://www.linuxidc.com
curl -x http://username:pwd@ip:port http://www.baidu.com
```

#### -c： 保存http的response里面的cookie信息。

```text
curl -c cookiec.txt  http://www.baidu.com
```

#### -D： 保存http的response里面的header信息

```text
curl -D cookied.txt http://www.baidu.com
```

执行后cookie信息就被存到了cookied.txt里面了 注意：-c\(小写\)产生的cookie和-D里面的cookie是不一样的。

#### -b：使用cookie

很多网站都是通过监视你的cookie信息来判断你是否按规矩访问他们的网站的，因此我们需要使用保存的cookie信息。

```text
curl -b cookiec.txt http://www.baidu.com
```

#### 模仿浏览器信息

-A :指定浏览器去访问网站

```text
curl -A "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.0)" http://www.baidu.com
```

#### 伪造referer（盗链）

很多服务器会检查http访问的referer从而来控制访问。比如：你是先访问首页，然后再访问首页中的邮箱页面，这里访问邮箱的referer地址就是访问首页成功后的页面地址，如果服务器发现对邮箱页面访问的referer地址不是首页的地址，就断定那是个盗连了 -e：设定referer

```text
curl -e "www.baidu.com" http://news.baidu.com/
```

#### 下载文件

-o： 把输出写到该文件中

```text
curl -o dodo1.jpg http://www.shaimn.com/uploads/allimg/160613/1-160613121111.jpg
```

-O：把输出写到该文件中，保留远程文件的文件名

```text
curl -O http://www.shaimn.com/uploads/allimg/160613/1-160613121111.jpg
```

循环下载,有时候下载图片可以能是前面的部分名称是一样的，就最后的尾椎名不一样

```text
curl -O http://www.shaimn.com/uploads/allimg/160613/1-16061312111[1-5].jpg
```

下载重命名

```text
curl http://www.shaimn.com/uploads/allimg/160613/1-16061312111[1-5].jpg -o dodo#1.jpg
```

分块下载

```text
curl -r 0-100 -o dodo1_part1.JPG http://www.shaimn.com/uploads/allimg/160613/1-160613121111.jpg
curl -r 100-200 -o dodo1_part2.JPG http://www.shaimn.com/uploads/allimg/160613/1-160613121111.jpg
curl -r 200- -o dodo1_part3.JPG http://www.shaimn.com/uploads/allimg/160613/1-160613121111.jpg
cat dodo1_part* > dodo1.JPG
```

通过ftp下载文件

```text
curl -O -u 用户名:密码 ftp://www.linux.com/dodo1.JPG
curl -O ftp://用户名:密码@www.linux.com/dodo1.JPG
```

显示下载进度条

```text
curl -# -O http://www.linux.com/dodo1.JPG
```

不会显示下载进度信息

```text
curl -s -O http://www.linux.com/dodo1.JPG
```

断点续传 curl可以通过内置option:-C同样可以达到相同的效果 如果在下载dodo1.JPG的过程中突然掉线了，可以使用以下的方式续传

```text
curl -C -O http://www.linux.com/dodo1.JPG
```

#### 上传文件

curl不仅仅可以下载文件，还可以上传文件。通过内置option:-T来实现 这样就向ftp服务器上传了文件dodo1.JPG

```text
curl -T dodo1.JPG -u 用户名:密码 ftp://www.linux.com/img/
```

#### 对CURL使用网络限速

通过--limit-rate选项对CURL的最大网络使用进行限制

```text
# 下载速度最大不会超过1000B/second

curl --limit-rate 1000B -O http://www.gnu.org/software/gettext/manual/gettext.html
```

### linux curl命令集

```text
-a/--append 上传文件时，附加到目标文件
-A/--user-agent <string> 设置用户代理发送给服务器
- anyauth 可以使用“任何”身份验证方法
-b/--cookie <name=string/file> cookie字符串或文件读取位置
- basic 使用HTTP基本验证
-B/--use-ascii 使用ASCII /文本传输
-c/--cookie-jar <file> 操作结束后把cookie写入到这个文件中
-C/--continue-at <offset> 断点续转
-d/--data <data> HTTP POST方式传送数据
--data-ascii <data> 以ascii的方式post数据
--data-binary <data> 以二进制的方式post数据
--negotiate 使用HTTP身份验证
--digest 使用数字身份验证
--disable-eprt 禁止使用EPRT或LPRT
--disable-epsv 禁止使用EPSV
-D/--dump-header <file> 把header信息写入到该文件中
--egd-file <file> 为随机数据(SSL)设置EGD socket路径
--tcp-nodelay 使用TCP_NODELAY选项
-e/--referer 来源网址
-E/--cert <cert[:passwd]> 客户端证书文件和密码 (SSL)
--cert-type <type> 证书文件类型 (DER/PEM/ENG) (SSL)
--key <key> 私钥文件名 (SSL)
--key-type <type> 私钥文件类型 (DER/PEM/ENG) (SSL)
--pass <pass> 私钥密码 (SSL)
--engine <eng> 加密引擎使用 (SSL). "--engine list" for list
--cacert <file> CA证书 (SSL)
--capath <directory> CA目录 (made using c_rehash) to verify peer against (SSL)
--ciphers <list> SSL密码
--compressed 要求返回是压缩的形势 (using deflate or gzip)
--connect-timeout <seconds> 设置最大请求时间
--create-dirs 建立本地目录的目录层次结构
--crlf 上传是把LF转变成CRLF
-f/--fail 连接失败时不显示http错误
--ftp-create-dirs 如果远程目录不存在，创建远程目录
--ftp-method [multicwd/nocwd/singlecwd] 控制CWD的使用
--ftp-pasv 使用 PASV/EPSV 代替端口
--ftp-skip-pasv-ip 使用PASV的时候,忽略该IP地址
--ftp-ssl 尝试用 SSL/TLS 来进行ftp数据传输
--ftp-ssl-reqd 要求用 SSL/TLS 来进行ftp数据传输
-F/--form <name=content> 模拟http表单提交数据
-form-string <name=string> 模拟http表单提交数据
-g/--globoff 禁用网址序列和范围使用{}和[]
-G/--get 以get的方式来发送数据
-h/--help 帮助
-H/--header <line>自定义头信息传递给服务器
--ignore-content-length 忽略的HTTP头信息的长度
-i/--include 输出时包括protocol头信息
-I/--head 只显示文档信息
从文件中读取-j/--junk-session-cookies忽略会话Cookie
- 界面<interface>指定网络接口/地址使用
- krb4 <级别>启用与指定的安全级别krb4
-j/--junk-session-cookies 读取文件进忽略session cookie
--interface <interface> 使用指定网络接口/地址
--krb4 <level> 使用指定安全级别的krb4
-k/--insecure 允许不使用证书到SSL站点
-K/--config 指定的配置文件读取
-l/--list-only 列出ftp目录下的文件名称
--limit-rate <rate> 设置传输速度
--local-port<NUM> 强制使用本地端口号
-m/--max-time <seconds> 设置最大传输时间
--max-redirs <num> 设置最大读取的目录数
--max-filesize <bytes> 设置最大下载的文件总量
-M/--manual 显示全手动
-n/--netrc 从netrc文件中读取用户名和密码
--netrc-optional 使用 .netrc 或者 URL来覆盖-n
--ntlm 使用 HTTP NTLM 身份验证
-N/--no-buffer 禁用缓冲输出
-o/--output 把输出写到该文件中
-O/--remote-name 把输出写到该文件中，保留远程文件的文件名
-p/--proxytunnel 使用HTTP代理
--proxy-anyauth 选择任一代理身份验证方法
--proxy-basic 在代理上使用基本身份验证
--proxy-digest 在代理上使用数字身份验证
--proxy-ntlm 在代理上使用ntlm身份验证
-P/--ftp-port <address> 使用端口地址，而不是使用PASV
-Q/--quote <cmd>文件传输前，发送命令到服务器
-r/--range <range>检索来自HTTP/1.1或FTP服务器字节范围
--range-file 读取（SSL）的随机文件
-R/--remote-time 在本地生成文件时，保留远程文件时间
--retry <num> 传输出现问题时，重试的次数
--retry-delay <seconds> 传输出现问题时，设置重试间隔时间
--retry-max-time <seconds> 传输出现问题时，设置最大重试时间
-s/--silent静音模式。不输出任何东西
-S/--show-error 显示错误
--socks4 <host[:port]> 用socks4代理给定主机和端口
--socks5 <host[:port]> 用socks5代理给定主机和端口
--stderr <file>
-t/--telnet-option <OPT=val> Telnet选项设置
--trace <file> 对指定文件进行debug
--trace-ascii <file> Like --跟踪但没有hex输出
--trace-time 跟踪/详细输出时，添加时间戳
-T/--upload-file <file> 上传文件
--url <URL> Spet URL to work with
-u/--user <user[:password]>设置服务器的用户和密码
-U/--proxy-user <user[:password]>设置代理用户名和密码
-v/--verbose
-V/--version 显示版本信息
-w/--write-out [format]什么输出完成后
-x/--proxy <host[:port]>在给定的端口上使用HTTP代理
-X/--request <command>指定什么命令
-y/--speed-time 放弃限速所要的时间。默认为30
-Y/--speed-limit 停止传输速度的限制，速度时间'秒
-z/--time-cond 传送时间设置
-0/--http1.0 使用HTTP 1.0
-1/--tlsv1 使用TLSv1（SSL）
-2/--sslv2 使用SSLv2的（SSL）
-3/--sslv3 使用的SSLv3（SSL）
--3p-quote like -Q for the source URL for 3rd party transfer
--3p-url 使用url，进行第三方传送
--3p-user 使用用户名和密码，进行第三方传送
-4/--ipv4 使用IP4
-6/--ipv6 使用IP6
-#/--progress-bar 用进度条显示当前的传送状态
```

## ubuntu使用ssh

### 安装ssh-server

```text
apt-get install openssh-server
```

### 安装ssh-client

```text
apt-get install openssh-client
```

### 确认安装SSH服务成功与否

```text
ps -e|grep sshd
```

### SSH服务启动、重启、关闭

```text
service ssh start
service ssh restart
service ssh stop
```

### 从服务器上下载文件

```text
scp username@servername:/path/filename /var/www/local_dir（本地目录）
```

### 上传本地文件到服务器

```text
scp /path/filename username@servername:/path
```

### 从服务器下载整个目录

```text
scp -r username@servername:/var/www/remote_dir/（远程目录） /var/www/local_dir（本地目录）
```

### 上传目录到服务器

```text
scp  -r local_dir username@servername:remote_dir
```

