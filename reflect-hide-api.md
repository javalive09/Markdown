# **不修改源码情况反射调用系统hide API**

## 一, 从 Android 9（API 级别 28）开始，Android平台对应用能使用的非 SDK 接口实施了限制。

限制级别分别是黑名单（不可用），受限灰色名单（有条件限制），灰名单（不支持），白名单（可用）。

具体限制名单，请参看官方文档：https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces
## 二, 白名单

android开发指南公开的API，白名单属于完全公开不受使用限制。

官网：https://developer.android.com/reference
## 三, 灰名单

android源码中标注hide，不建议使用，但开发者可以通过反射方式调用的API。

各个android版本的灰名单有所不同，如下：

Android9：https://android.googlesource.com/platform/frameworks/base/+/pie-release/config/hiddenapi-light-greylist.txt

Android10：https://dl.google.com/developers/android/qt/non-sdk/hiddenapi-flags.csv

Android11：https://dl.google.com/developers/android/rvc/non-sdk/hiddenapi-flags.csv

如果当前版本为Android9，我们需要使用 StorageVolume的getPath方法扫描usb盘进行歌曲扫描。可以首先查询 https://android.googlesource.com/platform/frameworks/base/+/pie-release/config/hiddenapi-light-greylist.txt 这个文档，发现在4270行 有这个方法Landroid/os/storage/StorageVolume;->getPath()Ljava/lang/String;  这个方法在灰名单中，我们可以通过反射方式调用。
## 四, 如何反射调用开发效率更高

推荐使用joor库，这个库使反射变得非常简单，以on(xxxclass).call(xxxmethod)方式反射调用方法。

官网代码地址：https://github.com/jOOQ/jOOR

使用方法：
```
//1、通过类名转换成一个Reflect对象：
public static Reflect on(String name);
public static Reflect on(String name, ClassLoader classLoader);
public static Reflect on(Class<?> clazz);
  
//2、将一个类对象转换成一个Reflect对象：
public static Reflect on(Object object);
  
//3、修改一个AccessibleObject类型的对象的访问权限：
public static <T extends AccessibleObject> T accessible(T accessible);
  
//4、返回Reflect对象具体包装的类型，类型为Class或者对象，由操作符on()的重载参数决定：
public <T> T get()；
  
//5、将name指定的field转换成一个Reflect对象，后续对field的操作变为对Reflect对象的操作：
public Reflect field(String name);
  
//6、返回当前Reflect对象的所有field属性，并转换成Reflect对象的map：
public Map<String, Reflect> fields();
  
//7、修改(获取)field属性值：
public Reflect set(String name, Object value);
public <T> T get(String name);
  
//8、反射调用name指定的函数，此函数封装了对函数的签名的精准匹配和近似匹配：
public Reflect call(String name);
public Reflect call(String name, Object... args);
  
//9、反射调用指定类的构造函数，也封装了精准匹配和近似匹配：
public Reflect create();
public Reflect create(Object... args);
  
//10、返回当前Reflect对象封装的对象类型：
public Class<?> type()；

获取StorageVolume代码示例：
StorageManager storageManager = codeActivity.getSystemService(StorageManager.class);
List<StorageVolume> list = Reflect.on(storageManager).call("getStorageVolumes").get();
List<String> ps = new ArrayList<>();
for (StorageVolume storageVolume : list) {
    String s = Reflect.on(storageVolume).call("getPath").get();
    ps.add(s);
}
codeActivity.showText(ps.toString());
```

# 五, 黑名单

需要修改系统源码，可能会影响系统稳定性，一般不建议修改。
