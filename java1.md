# Java1(基本类型、面向对象、异常处理)

## 基本类型

### 数据类型

#### 基本数据类型（4类8种，能自动装箱）

**整形**

**byte（8 bit，1个字节 -128~127）**

存储和传输容量的计量单位，字节。最早的PC机数据总线宽度是8（8根线）用8位表示一个字（c中char是一个字节） 表示方法1：byte b = 1; 注意：默认类型是int型，所以当b在\[-128,127\]之间时编译器不会报错。 例如：byte a = 1；byte b = 2；byte c； c = a + 1; //会报错，相当于把int赋值给byte c = a + b //会报错，相当于把int赋值给byte short （16 bit， 2个字节） short a = 1; int b = a;//编译报错 注意：不可以显示的将范围大的数据类型赋值给范围小的数据类型 总结：可以将范围小的值赋给表示范围大的变量；但不能将表示范围大的值赋给表示范围小的变量，只能通过强制类型转换 范围大的强转成范围小的变量时：前面舍位 范围小的强转成范围大的变量时：前面补位 ,如是正数前面补位0，如是负数前面补位1.

**short（16 bit， 2个字节）**

short a = 1; int b = a;//编译报错 注意：不可以显示的将范围大的数据类型赋值给范围小的数据类型 总结：可以将范围小的值赋给表示范围大的变量；但不能将表示范围大的值赋给表示范围小的变量，只能通过强制类型转换 范围大的强转成范围小的变量时：前面舍位 范围小的强转成范围大的变量时：前面补位 ,如是正数前面补位0，如是负数前面补位1.

**int（32 bit，4个字节）**

整形的默认类型 表示方法: int i = 1;

**long（64 bit，8个字节）**

表示方法1： long m = 1; 表示方法2： long m = 2l; 表示方法3： long m = 2L;

**浮点型**

**float（32bit， 4个字节）**

表示方法1：float f = 11.11f; 表示方法2：float f = 11.11F;

**double（64bit，8个字节 ）**

浮点型的默认类型 表示方法1：double d = 11.11; 表示方法2：double d = 11.11D; 表示方法3：double d = 11.11d;

**字符型**

**char（16 bit， 2个字节）**

表示方法1：char c = ‘c’;表示一个字符 表示方法2：char c = 56;表示一个整数

**布尔型**

**boolean（1 bit）**

为了纪念英国数学家Boole-George，在逻辑运算（与或非，因果关系的运算）中有突出贡献 表示方法： boolean b =true/false;

**强制转换**

**正数**

小范围 --&gt; 大范围 ： 前面空位补0 大范围 --&gt; 小范围 ： 前面空位去0

**负数**

小范围 --&gt; 大范围 ： 前面空位补1 大范围 --&gt; 小范围 ： 前面空位去1

**十六进制字符串转换成int方法**

有时候编写color 相关 的程序时，需要将十六进制字符串转换成int。 String str = "0xC0A80010";  
Integer ss = 0xC0A80010;  
Long longStr = Long.parseLong\(str.substring\(2\), 16\);  
ss = longStr.intValue\(\);  
System.out.println\(ss\);  
System.out.println\(Integer.toHexString\(ss\)\); 打印结果：  
-1062731760 c0a80010

#### 引用数据类型

**类（class）**

**包装类（能自动拆箱）**

Byte，Short， Integer（自动装箱时 ，integer有一个缓存，介于-128~127之间的整数装箱时会用缓存的内容，超过范围会生成新的对象，所以比较integer时要注意 integer == integer） Long，Float，Double，Character，Boolean

**枚举类**

枚举每个元素都是该类的实例 枚举元素必须位于枚举类最开始 枚举只有一个成员时，可以作为一种单例实现形式

**其他类**

**接口（interface）**

#### String字符串

String的本质是char数组 private final char value\[\]; 默认数组长度是0

**StringBuilder**

线程不安全 本质是char数组 char\[\]value 默认数组长度是16 增长时int newCapacity =value.length \* 2 + 2;

**StringBuffer**

线程安全 char\[\]value 默认数组长度是16 增长时int newCapacity = value.length \* 2 + 2;

**String占用字节数**

Char c = “我”;//占用2个byte字节 String str = “我”； byte \[\] bytes = str.getByte\(\); byte长度是3而不是2 因为str.getBytes（encoding）是获取指定编码的byte数组，通常GBK是2个字节，utf-8是3个字节。如果不指定，则去默认的encoding。

**关于分隔**

StringTokenizer（字符串分隔解析器，默认分隔符：空格、回车、Tab） Token被分隔字符串中的一块，两端有分隔符

**对象个数**

String在方法区中有一个字符串池（常量）。每次都会在池中搜寻一次。 Stringa = “abc”; 如果池中有，则直接用池中的，如果没有在池中新建一个 所以String a = new String（“abc”）；会建立两个对象。

#### 面试题

**int 和 Integer 有什么区别？**

Int是Java的原始数据类型，Integer是java为int提供的封装类。

**short s1 = 1; s1 = s1 + 1;有什么错?**

s1+1运算结果是int型，需要强制转换类型。

**char型变量中能不能存贮一个中文汉字?**

在C语言中，char类型占1一个字节，而汉子占2个字节，所以不能存储。 在Java中，char类型占2个字节，而且Java默认采用Unicode编码，以个Unicode码是16位，所以一个Unicode码占两个字节，Java中无论汉字还是英文字母都是用Unicode编码来表示的。所以，在Java中，char类型变量可以存储一个中文汉字。

**Java支持的数据类型有哪些？什么是自动拆装箱？**

Java语言支持的8中基本数据类型是：byte short int long float double boolean char 自动装箱是Java编译器在基本数据类型和对应的对象包装类型之间做的一个转化。比如：把int转化成Integer，double转化成double，等等。反之就是自动拆箱。

### 编码知识

#### Bit（binary  digit二进制数，比特）

电脑数据量的最小单位，二进制系统中每个1或0就是一个位（bit） 1 Byte = 8 bit 1 KB = 1024 Byte 1 MB = 1024 KB 1 GB = 1024 MB 1 TB = 1024 GB （10TB大约等于一个人脑的存储量）

#### ASCII

原始 7位二进制表示字符 现代 8位二进制表示字符 中文 2个byte字节表示 标准ASCII表：[http://baike.baidu.com/view/15482.htm](http://baike.baidu.com/view/15482.htm)

#### ISO8859-1 （8位二进制，1个byte字节表示字符）

加入αβγ（西方文字）

#### Unicode（16位二进制，2个byte字节表示字符，又叫万国码）java编码格式

中文 3个byte字节表示 αβ 2个byte字节表示 英文，数字 1个字节表示

#### GBK

中文 2个byte字节表示 其他 1个byte字节表示

#### base64 编码

由于某些系统中只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法。 Base64中是一种基于64个可打印字符来表示二进制数据的表示方法。Base64中的可打印字符包括字母A-Z、a-z、数字0-9，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。 注：在android中遇到解码时调用 ：Base64.decode\(input, NO\_WRAP\) 总崩溃，出现IllegalArgumentException: bad base-64 异常。这是传参错误引起的。

#### 关于字母

大写字母和小写字符之间数值上差32 a 是 97；A是65 获取content-length 字符串的长度： length = result.getBytes\("utf-8"\).length\(\);

### 关于变量

#### 命名规范

以下划线、字母、$开头都可以。唯独不能用数字开头。 首字母小写，单词间驼峰标示 常量全部大写，单词间用“\_”连接

#### 分类

**静态变量（内存低地址区域，静态区）**

Static关键字的变量

**成员变量（存放在Heap堆中）**

new中存放的变量

**局部变量（local variables内存高地址区域，存放在Stack栈中）**

方法中的变量 包括基本变量、对象地址 方法参数属于局部变量 可变参数（…表示，代替数组使用起来更方便）：只能放在参数列表最后；调用含可变参数的方法时编译器为该可变参数隐式的创建一个数组对象。所以在方法体中以数组形式访问。

#### 运算

当有若干个变量参与运算时，结果类型取决于这些变量中表示范围最大的那个变量类型。比如，参与运算的变量中，有int，有double，有short，那么最后的结果就是double

### 操作符

#### 分类

一元操作符（有一个变量参与的操作符）如： ! , &gt; , &lt; , = , instanceOf 二元操作符（有两个变量参与的操作符）+ , - , \* , / 元，表示变量

#### 取模运算

余数的作用，可以实现循环取连续的值。如 for \(a = 0 ; a &lt; 100 ; a++\){ System.out.println\( a % 5\);

} 7 % 3的意义是7中有2个3，余下1. 7 % 20 的意义是，7中有0个20，余下7。 运算的规律：结果的符号永远与被除数（7）符号相同

#### 位移运算

位移运算可以提高运算速度

```text
1. 左移 '<<'
a << n   相当于 a*2n
2. 右移 '>>'
a >> n   相当于 a/2n
```

#### 按位运算

**按位与 '&'**

两个都是1，结果才是1，串联关系。例如： 1&0 = 0 ;1&1= 1; 0xff0000ff & 0x12345678 = 0x12000078 16进制f = 2进制 1111 与运算可以用于取段位。

**按位或 \|**

只要有一个是1，结果就是1，并联关系。例如：1\|1 = 1;1\|0 = 1; 0xff0000ff \| 0x12345678 = 0xff3456ff

**按位异或 ^**

不同为1，相同为0，比较关系。例如：1^1 =0;1^0 = 1;

#### 取反 ~

00000000 00000000 00000000 00000101 这是5 11111111 11111111 11111111 11111010 这是5取反的结果-6

#### 面试题

3&5 = ？ 3: 0x00000011 -------- 1x2^1 + 1x2^0 = 2 + 1 = 3 5: 0x00000101 -------- 1x2^2 + 1x2^0 = 4 + 1 = 5 &: 0x00000001 -------- 1

### 补码

在计算机系统中，数值一律用补码来表示（存储）。 补码的好处： 使用补码可以将符号位和其他位统一处理（可以符号位运算） 减法可以按照加法来处理 补码的原理： 利用溢出来实现减法变加法 如： 10进制： 计算9 9– 44 = ？ -44的反码是55（10机制的最大值是9减去4得到） -44的补码是55+1 = 56 上式的计算可以变成：9 9+ 56（-44的反码）=155 溢出舍弃 = 55 2进制同理： 计算 - 1 + 2的结果 -1 1111 1111（补码） 2 0000 0010（补码） 1 0000 0001（补码结果

### 流程控制

#### 循环控制

**For**

在知道循环次数的条件下使用

**While**

在不知道循环次数的条件下使用

**中断**

跳出循环用 break； 跳出本次循环用 continue；

#### 条件控制

**if else**

**switch（变量）**

变量类型为：byte，short，int，char，enum，string

### 关键字

#### static

表示静态的，内存中有唯一的一份，其他成员公用的资源。放在静态区 生命周期和JVM一致，不会被回收 static不可以修饰局部变量，局部变量放在栈中会频繁的生死 static方法必须使用static变量也和生命周期有关。区域A能使用区域B的原则是：区域B的生命周期 &gt;=区域A的生命周期

#### this

this只存在于方法中 编译器暗中传递的操作此方法的对象的引用。谁调用了这个方法，this就指谁 用于调用对象的其他成员，一般可以省略 每当启动一个线程的时候，JVM就为他分配一个栈。栈是以帧为单位保存当前运行的状态。每当调用一个方法时。JVM会在对应的栈中压入一个帧，执行方法时，他使用这个帧存储参数、局部变量、this。 this（）用来在构造器中调用构造器，构造器中只能调用一次构造器，并且必须在第一行调用

#### super

super只存在于方法中 编译器暗中传递的操作此方法的的对象的父类的引用 用于调用父类方法 super（）用来在构造器重调用父类构造器，只能调用一次，并且必须在第一行调用

#### package

解决类命名冲突问题 域名的反转 包名小写 lang是language缩写lang包表示Java自带的语言包 io是input/output缩写io包表示java的输入输出流 文件打包 jar –cvf xx.jar_._例如：jar –cvf class.jar Foo.class Bar.class

#### final

final修饰数据，表示数据不变，是个常量 final修饰引用，表示引用指向不可变 final修饰方法，表示方法不可被重写，是不变的 final 修饰类，表示类不可以被继承，断子绝孙 局部内部类访问同一个方法中的局部变量。那个局部变量必须是final修饰的。原因是： 局部变量的生命周期小于内部类的生命周期。解决的方法是拷贝一个变量副本到内部类，语法上是同一个变量。为了这两个变量一致性，用final来修饰。用final保持不可变的方法来解决一致性的问题。

#### volatile

为了防止编译器对代码优化 每次都从主存中提取变量，防止多线程时，变量被改变，而用的仍然是线程中旧的拷贝版本的数据

#### transient

修饰不想被序列化的变量

### 数组array

对应的工具类Arrays包含了sort、copy、binarySearch、asList（不可改变大小的List）等方法 数组时效率最高的容器。在固定尺寸的情况下，一定要用数组 数组的拷贝System.lang.arrayCopy\(\);效率最高。用的事拷贝整块内存的方式 数组的内存模型（二维数组）：int\[\]\[\] scores = new int\[3\]\[2\]; 多维数组的length以最外层的大小为准：socres.length = 3; 

![](.gitbook/assets/array.jpg)

### 访问权限控制

private：私有权限 default（friendly，什么也不写）：包权限 protected：继承权限 public：公共权限

## 面向对象

### 类与对象

#### 类

具体事物的抽象 体现了很好的封装 由数据（成员变量）和方法构成 命名约定：类名首字母大写，类中成员名首字母小写（成员变量、方法）单词间驼峰原则 抽象类：有abstract方法的类

#### 对象

人认识世界中事物的基本单位 用引用操纵对象，相当于遥控器操纵电视

**生成对象的方法**

**反射法**

Class.forName（”Foo”）.newInstance（）; 此方法动态性强，主要用于配置文件，后期给定对象

**New法**

Foo foo = new Foo（）； new 对象执行的流程: classloadeer载入类 在堆中开辟空间 如果有静态块，则由父类到子类先把静态块运行完。再往下面走。具体如下：父类·静态属性 &gt; 父类·静态块 &gt; 子类·静态属性 &gt; 子类·静态块 父类的构造方法super（）；//父类属性，父类构造器方法体 总之：由内向外，（开辟空间、属性、构造器依次运行）层层构造。 总体顺序： 静态 &gt; main入口（如果类中有的话）&gt; 非静态。 静态：按由内向外、由上到下的顺序执行。 非静态：按属性-&gt; 构造器顺序执行

**对象初始化与清理**

**成员变量**

可以不初始化，在堆中创建时会赋上默认值

**局部变量**

必须初始化

**引用**

赋空值的时候，JVM会自动回收new出的空间

### 复用类

#### 继承

通过继承其他类来复用

#### 组合

通过包含其他引用来复用

#### 克隆

**浅clone**

只clone当前类的属性。 适用于clone只含有基本类型成员的对象

**深clone**

clone 所有属性，包括引用对象的属性。 适用于clone 包含有引用类型成员的对象

### 多态

多态和继承的关系：有了继承然后才有了多态，继承是为了复用功能，多态是为了变化功能 概念：同一操作，不同结果（体现在方法上） 本质：实现了动态绑定（运行时绑定、后期绑定） 体现：主要体现在方法重写上 特点：父类引用指向子类对象

### 接口interface

#### 标识接口

serializable、cloneable

#### 普通接口

#### 重载和重写

**重载overLoard**

重新载入。重载的是参数 名字相同 参数列表不同 返回值无要求（一般相同） 权限无要求

**重写overwrite**

重写父类方法。重写的是方法体中的内容 所有东西都相同（名字、参数列表、返回值） 方法体中的内容不同 权限不能萎缩

### 抽象

抽取共同的东西。面向对象语言最大的本事是 应对变化。 通过抽象的方式来应对变化，哪里变化就抽象哪里。 所有涉及到 extend 和 implement 的东西，都是抽象。 因为父类有共同的东西。 java中到处都是抽象。

#### 分类

**属性的抽象**

把共有的属性抽象到父类。 如：Android中的 interface BaseColumns

**方法的抽象**

把共有的行为抽象到父类。 如：android中的 interface Observer

**属性，方法混合抽象**

把共有的属性，方法抽象到父类。 这个的例子非常多。 如： android中的 context ， View，等等。

## 异常处理

在java中引入exception目的是 增加对程序出现异常的管控。

### 分类

#### Error

虚拟机层面的错误，这种错误非常严重，错误所在线程会直接崩溃。

#### CheckException

一种概念，并非具体异常，除了RuntimeException都属于CheckException。API层面的错误，提供的API不能保证百分百的稳定性，有可能会出现某些问题，这种API会强制要求程序员必须 try catch 捕获，所以程序不会崩溃。 常见的如： IOException JSONException SQLException TimeoutException XmlPullParserException

#### RunTimeException

程序编写层面出现的错误，这种错误比较严重，一般是逻辑或不符合规范造成的，程序所在线程会崩溃。 出现了这种错误，说明程序有严重的问题，程序员需要对程序编写逻辑重新处理。 常见的如： NullPointerException NumberFormatException ClassCastException StringIndexOutOfBoundsException ArrayIndexOutOfBoundsException IllegalArgumentException ArithmeticException

### try-with-resource

对于使用完一定会关闭的逻辑 可以使用try with resource 代替try catch finally 实时上java8 后中很多io相关操作都已经支持 AutoCloseable 接口。

```text
        try (Resource resource = new Resource()){
            resource.doSomething();
        } catch (Exception e) {
            e.printStackTrace();
        }

        class Resource implements AutoCloseable{

        @Override
        public void close() throws Exception {
            Log.i("peter", "auto close");
        }

        public void doSomething() {
            Log.i("peter", "doSomething");
        }
    }
```

### 结构

```text
Throwable
        |_Error
        |_Exception 
                |__ RuntimeException
                |__ other（checkException）
```

### UncaughtExceptionHandler

> 如何获取及保存exception log 使用UncaughtExceptionHandler来捕获unchecked异常保存到SDcard，到一定大小后发送，然后重置。

在程序入口处， application的 onCreate\(\) 或者Activity的onCreate\(\)里面加入如下代码：

```text
Thread.setDefaultUncaughtExceptionHandler(new CrashHandler());
```

实现UncaughtExceptionHandler中的uncaughtException方法：

```text
public class CrashHandler implements UncaughtExceptionHandler {

    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        //处理异常
        Log.e("崩溃",thread.getName()+ex.toString());
        //发送到服务器
        //dialog提醒
    }

}
```

