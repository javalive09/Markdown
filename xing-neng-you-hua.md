# 性能优化

## ui优化

### 防止过度绘制

#### view层级控制

1. 逻辑上控制层级
2. merge

   **避免过多主动调用invalidata\(\)**

   **避免过多主动调用requestlayout\(\)**

   **使用canvas cliprect\(\) 局部绘制**

   **懒加载**

   viewstub

   **使用硬件加速**

   **使用surfaceView**

   **检测工具**

   traceView 

   monitors的gpu选项

## 内存优化

### 防止内存泄漏

资源要及时回收 cursor, io, 注册，单例，handler， 静态view引用，异步持有view引用

### 减少内存

bitmap采样，arrayMap代替hashmap，sparseArray代替hashmap，使用intentservice，使用asynctask， handlerThread，localbroadCastReceiver，缓存lrucache，listview holder，threadpool

### 为android 应用申请更多内存

#### 获取Dalvik初始分配的内存大小

**1. 查看/system/build.prop文件**

```text
dalvik.vm.heapstartsize=8m
dalvik.vm.heapgrowthlimit=192m
dalvik.vm.heapsize=512m
```

dalvik.vm.heapstartsize=8m 相当于虚拟机的 -Xms配置，该项用来设置堆内存的初始大小。 dalvik.vm.heapgrowthlimit=192m 相当于虚拟机的 -XX:HeapGrowthLimit配置，该项用来设置一个标准的应用的最大堆内存大小。一个标准的应用就是没有使用android:largeHeap的应用。 dalvik.vm.heapsize=512m 相当于虚拟机的 -Xmx配置，该项设置了使用android:largeHeap的应用的最大堆内存大小。

#### 2. 使用adb查看

```text
adb shell getprop|grep dalvik.vm.heapstartsize 应用启动后分配的初始内存
adb shell getprop|grep heapgrowthlimit 单个标准应用程序最大内存限制（没设置android:largeHeap）
adb shell getprop|grep dalvik.vm.heapsize 单个dalvik虚拟机最大的内存限制（设置了android:largeHeap）
```

#### 3. 使用ActivityManager

```text
ActivityManager activityManager =(ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);
activityManager.getMemoryClass();
activityManager.getLargeMemoryClass();
```

ActivityManager的getMemoryClass\(\)获得内用正常情况下内存的大小 ActivityManager的getLargeMemoryClass\(\)可以获得开启largeHeap最大的内存大小

#### 申请大内存

在application元素上加入android:largeHeap=”true”申请大内存

### 检测工具

monitors的memery选项

## 包大小优化

proguard minify，shrink png

## 省电

### 屏幕

屏幕锁

### 网络

1. 压缩数据
2. 延后发送数据
3. 打包数据发送

   **检测工具**

   monitors的cpu选项

   monitors的network选项

## 计算速度优化

### for循环

#### 优化前：

```text
for(int i = 0; i < list.size;  i++){}
```

#### 优化后：

```text
for(int i = 0,int len = list.size; i < len ; i++){}
```

### 如何提高 x &gt; 0 ? x : 0 的效率？

使用 Math.max\(0, x\); 更高效

### ThreadLocalRandom 速度优于 Random

## 性能优化工具

### monitors的cpu选项

## 随机排序数组的方法

shuffle方式：

```text
Integer[] arr = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
List<Integer> list = Arrays.asList(arr);
Collections.shuffle(list);
Integer[] random = list.toArray(new Integer[arr.size()]);
```

set方式：

```text
Integer[] arr = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
Set<Integer> indexes = new HashSet<>(Arrays.asList(arr));
Integer[] random = indexes.toArray(new Integer[arr.size()]);
```



## String技巧

### StringFormat 补0；

```text
        for (int i = 0; i < 24; i++) {
            if (i < 10) {
                mHours.add("0" + String.valueOf(i));
            } else {
                mHours.add(String.valueOf(i));
            }
        }

        for (int i = 0; i < 60; i++) {
            if (i < 10) {
                mMinutes.add("0" + String.valueOf(i));
            } else {
                mMinutes.add(String.valueOf(i));
            }
        }
```

```text
        for (int i = 0; i < 24; i++) {
            mHours.add(String.format(Local.getDefault(), "%02d"), i));
        }

        for (int i = 0; i < 60; i++) {
            mMinutes.add(String.format(Local.getDefault(), "%02d",i));
        }
```

