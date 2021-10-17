---
description: 24种设计模式（创建型6、结构型7、行为型11）
---

# 设计模式

## 代码的抽象三原则

### 1.DRY原则

DRY是 Don't repeat yourself 的缩写，意思是"不要重复自己"。 它的涵义是，系统的每一个功能都应该有唯一的实现。也就是说，如果多次遇到同样的问题，就应该抽象出一个共同的解决方法，不要重复开发同样的功能。

### 2.YAGNI原则

YAGNI是 You aren't gonna need it 的缩写，意思是"你不会需要它"。 它背后的指导思想，就是尽可能快、尽可能简单地让软件运行起来（do the simplest thing that could possibly work）。

### 3.Rule Of Three原则

Rule of three 称为"三次原则"，指的是当某个功能第三次出现时，才进行"抽象化"。 它的涵义是，第一次用到某个功能时，你写一个特定的解决方法；第二次又用到的时候，你拷贝上一次的代码；第三次出现的时候，你才着手"抽象化"，写出通用的解决方法。 （1）省事。如果一种功能只有一到两个地方会用到，就不需要在"抽象化"上面耗费时间了。 （2）容易发现模式。"抽象化"需要找到问题的模式，问题出现的场合越多，就越容易看出模式，从而可以更准确地"抽象化"。 （3）防止过度冗余。如果一种功能同时有多个实现，管理起来非常麻烦，修改的时候需要修改多处。在实际工作中，重复实现最多可以容忍出现一次，再多就无法接受了。

## 面向对象设计的原则

面向对象又叫OOD（Object-oriented design），遵循solid原则。

### 1.单一功能原则（SingleResponsibility Principle）

对象应该具有单一的功能。（封装）

### 2.开放闭合原则（Open ClosedPrinciple）

对修改关闭，对扩展开放。（继承）

### 3.Liscov替换原则（Liscov Substitution Principle）

子类功能不能退化（契约式设计）。子类重写父类方法时不应该清空方法体的内容。（继承）

### 4.接口隔离原则（Interface Segregation Principle）

细分的接口要好于宽泛的接口。（多态）

### 5.依赖倒置原则（DependencyInversion Principle）

遵从“依赖于抽象而不是一个实例”（要面向接口编程）。（多态）

## UML（图形化的语言表示）

OO（Object-Orientation）：面向对象，一种系统建模技术 OOP（Object-OrientationProgramming）：面向对象编程，按照OO的方法来开发程序的过程 OOAD（Object-OrientationAnalysis and Design）：面向对象的分析与设计

### UML图分类

#### 静态模型（static model）

**用例图（use case diagrams）**

由参与者（Actor）、用例（Use Case）以及它们之间的关系构成 描述系统功能的动态视图

**类图（class diagrams）**

用于描述系统的结构化设计，模型中存在的类、类的内部结构以及它们与其他类的关系等

**对象图（object diagrams）**

**组件图（component diagrams）**

**部署图（deployment diagrams）**

**动态模型（dynamic model）**

**时序图（sequence diagrams）**

是一种UML行为图，通过描述对象之间发送消息的时间顺序显示多个对象之间的动态协作

**协作图（collaboration diagrams）**

**状态图（state chart diagrams）**

**活动图（activity diagrams）**

### UML关系分类

依赖（use-a）：表现为局部变量、参数、静态方法的调用 （表示图形为虚线箭头）

关联（has-a）：成员变量引用 （表示图形为实线箭头）

* 聚合（Aggregation）：是整体与部分的关联关系，两者可单独存在（表示图形为空心菱形+实线箭头）
* 组合（Composition）：是整体与部分的关联关系，两者不能单独存在（表示图形为实心菱形+实线箭头）

泛化（is-a）：继承类（表示图形为实线三角箭头）

实现（Realization）：实现接口（表示图形为虚线三角箭头）

### UML关系例子

![](.gitbook/assets/uml.jpg)

## 创建型设计模式\(6\)

### singleton \(单例模式\)

保证一个类仅有一个实例。

#### 模型

现实生活中的CEO, CFO 等管理者，或资源控制者。

#### 结构

构造器私有化，类中包含一个自己的实例作为属性并提供对外的接口，要考虑多线程调用的情况

#### 种类

饱汉式 饿汉式 静态内部类式

#### 代码例子

Android SDK 中的InputMethodManager.getInstance\(\) Android SDK 中的SystemConfig.getInstance\(\) Android SDK 中的Watchdog.getInstance\(\) Android SDK 中的WindowManagerGlobal.getInstance\(\) Java JDK 中的Runtime.getRuntime\(\)

#### 类图

![](.gitbook/assets/singleton.jpg)

#### 示例图

![](.gitbook/assets/singleton_pic.jpg)

### Static Factory（静态工厂、简单工厂）

提供同一类型的产品。

#### 模型

现实生活中的单一产品的工厂。如：螺丝长，水泵厂。

#### 结构

工厂类角色：提供产品实例的接口。 产品类角色：同一类型产品的抽象以及它的实现。

#### 种类

new的方式实例化产品。 反射的方式实例化产品。

#### 代码例子

Android SDK中的BitmapFactory.decodexxx\(\) JDK中的Class.forName\(\) JDK 中的Calendar.getInstance\(\)

#### 类图

![](.gitbook/assets/staticfactory.jpg)

### Factory Method（工厂方法）

产品、工厂都是变化的，两个都要抽象。

#### 模型

现实生活中的连锁工厂。哪个方便用哪个工厂来生产产品。

#### 结构

工厂角色：抽象的工厂和它的实现类。 产品角色：抽象的产品和它的实现类。

#### 代码例子

JDK中的Threadfactory

#### 类图

![](.gitbook/assets/factorymethod.jpg)

#### 示例图

![](.gitbook/assets/fatory_method_pic.jpg)

### Abstract Factory（抽象工厂）

产品（多类）、工厂都是变化的，两个都要抽象。一般要生成多系列产品，产品族时会用此模式，

#### 模型

生产插座插头的工厂（中国标准插头，美国标准插头）

#### 结构

工厂类角色：提供产品实例的接口。 产品类角色：多个类型产品的抽象以及它的实现。

#### 类图

![](.gitbook/assets/abstractfactory.jpg)

#### 示例图

![](.gitbook/assets/abstract_factory_pic.jpg)

### Builder（建造者）

一个对象由多个部分组成，各个组成部分可以灵活变化的情况。

#### 模型

现实生活中的盖房子。（加阁楼不加阁楼，加门不加门，加窗补加窗）

#### 结构

产品角色：产品 建造者角色：抽象建造者及它的实现。负责提供组合产品各个部分的方法 导演者：负责组合的逻辑。

#### 代码例子

Android SDK中的AlertDialog.Builder JDK中的Java.lang.StringBuilder的 append\(\)

#### 类图

![](.gitbook/assets/builder.jpg)

#### 示例图

![](.gitbook/assets/builder_pic.jpg)

### Protoype（原型模式）

除new以外另一种高效创建对象的方法。

#### 模型

克隆羊。

#### 序列化

本质是流（byte流），对象流不序列化static、transient 。序列化的目的是搬运对象。

#### 反序列化

不调用构造器 new出对象

#### 代码例子

Android SDK中的Intent.clone\(\) jdk中的clone

#### 类图

![](.gitbook/assets/prototype.jpg)

#### 示例图

![](.gitbook/assets/prototype_pic.jpg)

## 结构型设计模式\(7\)

### Adapter（适配器模式）

更好的连接不兼容的接口。

#### 模型

现实生活中的充电器

#### 结构

客户角色：接口使用者 适配器角色：接口转换者 被适配对象角色：接口提供者

#### 代码例子

Android SDK中的adapter Java JDK中的InputStreamReader

#### 类图

![](.gitbook/assets/adapter.png)

#### 示例图

![](.gitbook/assets/adapter_pic.jpg)

### Composite（组合模式）

使对单个对象和组合对象的接口使用一致，

#### 模型

现实生活中的暖器片。

#### 结构

单个对象角色 组合对象角色 单个对象和组合对象抽象的接口角色

#### 代码例子

Android View层级结构

#### 类图

![](.gitbook/assets/composite.png)

#### 示例图

![](.gitbook/assets/composite_pic.jpg)

### Decorator（装饰器模式）

一个对象通过叠加其他对象来增加这个对象的功能。

#### 模型

游戏中的红蓝buff。 坦克大战的加护甲，加攻击的道具。

#### 结构

被装饰对象角色：需要增加功能的对象 装饰器角色：装饰器的抽象及它的实现 抽象角色：被装饰对象和装饰器的抽象接口

#### 代码例子

jdk中的io inputstream

#### 类图

![](.gitbook/assets/decorator.png)

#### 示例图

![](.gitbook/assets/decorator_pic.jpg)

### Proxy（代理模式）

控制对一个对象的访问

#### 模型

现实生活中的各种代理人（物流代理人）

#### 结构

被代理角色 代理角色 抽象角色

#### 代码例子

Android中的phoneproxy

#### 类图

![](.gitbook/assets/proxy.png)

#### 示例图

![](.gitbook/assets/proxy_pic.jpg)

### Flyweight（享元模式）

以共享的方式实现内存最小化。

#### 模型

图书馆

#### 结构

享元工厂角色：控制元对象的数量 元对象角色：元对象的抽象及它的实现

#### 代码例子

Android SDK中的Message.obtain\(\) JDK中的integer的valueOf\(\)

#### 类图

![](.gitbook/assets/flyweight.png)

#### 示例图

![](.gitbook/assets/flyweight_pic.jpg)

### Facade（门面模式）

为复杂系统提供一个简单的接口

#### 模型

医院咨询处

#### 代码例子

jdk中的Class ![](/images/facade.png)

#### 示例图

![](.gitbook/assets/facade.png)

![](.gitbook/assets/facade_pic.jpg)

### Bridge \(桥接模式\)

一个对象有多维度变化，这些维度有组合的关系，避免生成大量的组合类。

#### 模型

需要改变形状和颜色的图形。

#### 结构

一个维度抽象类使用另一个维度抽象类。 接口调用接口。

#### 类图

![](.gitbook/assets/bridge.jpeg)

#### 示例图

![](.gitbook/assets/bridge_pic.jpg)

## 行为型设计模式\(11\)

### State（状态模式）

动态的控制一个对象状态的切换。

#### 模型

汽车档位切换

#### 结构

需要改变状态的对象角色 状态角色：状态的抽象及它的实现

#### 类图

![](.gitbook/assets/state.png)

#### 示例图

![](.gitbook/assets/state_pic.jpg)

### Strategy（策略模式）

把一个对象的算法，行为抽象出来，使他们能够互相替换。

#### 模型

多功能榨汁机

#### 结构

上下文角色： 策略角色：策略抽象及它的实现

#### 代码例子

jdk中的collections sort 排序 jdk中的treemap Comparator 参数的构造器 jdk中的TreeSet Comparator 参数的构造器 Android中的animation 的setInterpolator\(\)

#### 类图

![](.gitbook/assets/strategy.png)

#### 示例图

![](.gitbook/assets/strategy_pic.jpg)

### Observer（观察者）

一个对象状态的改变能影响一系列对象状态的改变

#### 模型

喇叭广播

#### 结构

被观察角色： 观察者角色：观察者的抽象及它的实现

#### 代码例子

jdk中的Observable Android中的广播

#### 类图

![](.gitbook/assets/observe.png)

#### 示例图

![](.gitbook/assets/observer_pic.jpg)

### Chain of Responsibility（责任链）

使多个对象都有机会处理消息。

#### 模型

多米诺骨牌

#### 结构

消息发送角色： 消息处理节点角色；消息处理节点的抽象及它的实现 通过链表的形势实现消息的传递

#### 代码例子

Android中的view层消息传递，绘图

#### 类图

![](.gitbook/assets/chainofresponsibility.png)

#### 示例图

![](.gitbook/assets/chain_of_responsibility_pic.jpg)

### Command（命令模式）

把对象的一系列命令抽象并管理起来使对象支持对命令的管理。如：排队，撤销，日志等操作

#### 模型

现实生活中去餐馆的点菜

#### 结构

客户：食客 调用者角色：服务员 命令角色：点的菜 接受者角色：厨师

#### 类图

![](.gitbook/assets/command.jpg)

#### 示例图

![](.gitbook/assets/command_pic.jpg)

### Memento（备忘录模式）

在对象之外保存该对象的状态以便恢复

#### 模型

游戏存档

#### 结构

需要保存对象的角色：游戏 备忘录角色：存档数据 备忘录持有者角色：数据库

#### 代码例子

Android中activity onRestoreInstanceState\(\)

#### 类图

![](.gitbook/assets/memento.png)

#### 示例图

![](.gitbook/assets/memento_pic.jpg)

### Iterator（迭代子）

将遍历集合的方法抽象出来。

#### 结构

集合角色：集合的抽象及它的实现 迭代子角色：迭代子的抽象及他的实现

#### 代码例子

jdk中的collection 的 iterator

#### 类图

![](.gitbook/assets/iterator.png)

#### 示例图

![](.gitbook/assets/iterator_pic.jpg)

### TemplateMethod（模版方法）

对象做一件事的流程是固定的但是流程中的各个节点的内容是变化的。

#### 模型

自动买票软件

#### 结构

模版角色： 实现类角色：

#### 代码例子

Android 中activity的生命周期的回调函数

#### 类图

![](.gitbook/assets/templatemethod.png)

#### 示例图

![](.gitbook/assets/template_method_pic.jpg)

### Interpreter \(解释器模式\)

通过组合多种单元算法来解释问题。

#### 模型

计算器

#### 结构

抽象表达式角色 具体终结表达式 具体非终结表达式

#### 类图

![](.gitbook/assets/interpreter.jpeg)

#### 示例图

![](.gitbook/assets/interpreter_pic.jpg)

### Mediator 中介者模式

一个功能的内部的各个模块有千丝万缕的联系。中介者用来解耦。

#### 模型

内部通讯工具软件。

#### 结构

各个模块角色 中介者角色

#### 类图

![](.gitbook/assets/mediator.jpeg)

#### 示例图

![](.gitbook/assets/mediator_pic.jpg)

### Visitor \(访问者模式\)

封装（用容器）一些施加于某种数据结构元素之上的操作，在不改变数据结构的前提下，修改元素操作。

#### 模型

不同级别领导视察

#### 结构

结构对象角色 元素角色 访问者角色

#### 类图

![](.gitbook/assets/visitor.jpeg)

#### 示例图

![](.gitbook/assets/visitor_pic.jpg)

