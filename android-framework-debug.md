# **Andorid framework 源码调试**
# 1.首先新建module,选择 java or Kotlin library
# 2.在gradle里导入Java sdk，如果要导入梧桐的项目，应按照以下的写法
```
apply plugin: 'java-library'
dependencies {
  implementation fileTree(dir: 'libs', include: ['*.jar'])
}
sourceCompatibility = "8"
targetCompatibility = "8"
def defaultEncoding = 'UTF-8'
compileJava {
  options.encoding = defaultEncoding
//  def clspath = rootProject.ext.oslibs.join(File.pathSeparator)
//  println "bootClasspath before => " + options.bootClasspath
//  options.bootClasspath = clspath
//  println "bootClasspath after => " + options.bootClasspath
}
dependencies {
  implementation fileTree(dir: 'libs', include: ['*.jar'])
//  implementation rootProject.ext.oslibs
}
sourceSets {
  def and_root = 'D:/w/a/a_android/k_sdk/sources/android-28/'
  main.java.srcDirs += and_root
//要导入梧桐的项目应按照如下的写法
def xx_root = 'D:\\xx\\xx28\\'(绝对路径)
 
main.java.srcDirs += xx_root + 'xx_servers\\java'
main.java.srcDirs += xx_root + 'xx_servers\\aosp\\frameworks\\base\\services\\core\\java'
main.java.srcDirs += xx_root + 'xx_plugs\\java'
 
main.java.srcDirs += xx_root + 'xx_frames\\java'
}
```