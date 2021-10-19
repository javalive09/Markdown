# **命令行方式对APK进行重签名**
# 一.签名前的准备
1.jks签名文件

2.jks别名及密码

3.需要签名的apk文件
# 二.删除旧签名

1.把apk的后缀改成zip格式

2.用解压工具打开压缩文件android.zip，然后删除文件夹中的 META-INF目录。

META-INF存放签名后的CERT和MANIFEST文件，用于识别软件的签名及版权。

3.直接把android.apk文件后缀改为android.apk
# 三.重签名

## 1.命令
```
jarsigner -verbose -keystore <*.jks> -storepass <storepass > -signedjar <sign.apk> -digestalg SHA1 -sigalg MD5withRSA android.apk <alias_name>
```
## 2.参数解析
```
-jarsigner : Java的签名工具

-verbose：显示出签名详细信息

-keystore:  使用当前目录中的*.jks(或者格式为.keystore):签名证书文件

-storepass : Keystore密码

-signedjar:  签名后生成的APK名称

android.apk: 未签名的APK

-digestalg SHA1 -sigalg MD5withRSA：这就是必须加上的参数

alias_name:签名文件的别名
```
# 四.脚本工具

为节省时间，方便使用，可用如下脚本进行重签名

使用方法：

运行./re_sign_apk.sh有如下提示：
```

apk重签名：

./re_sign_apk.sh <apk_name> <jks> <storepass> <key>

============参数详解============

<apk_name>:   要重签名的apk

<jks>:        新的签名文件

<storepass>:  签名文件的密码

<key>:        签名文件的别名

============例子============

./re_sign_apk.sh a.apk b.jks 123 wt

注：

将apk及签名文件放到同一个目录下

本次运行时间>>>>>>>>>>>> 0s
```