# 框架

## volley

库大小100k左右。

```text
compile 'com.android.volley:volley:1.0.0'
```

### 1.请求

cache dispatcher ----&gt; network dispatcher 1）如果 http请求的头信息中 的cache-control max-age字段 未过期则使用 cache ttl\(time to live\)存活时间 BasicNetwork.java

```text
    @Override
    public NetworkResponse performRequest(Request<?> request) throws VolleyError {
        long requestStart = SystemClock.elapsedRealtime();
        while (true) {
            HttpResponse httpResponse = null;
            byte[] responseContents = null;
            Map<String, String> responseHeaders = Collections.emptyMap();
            try {
                // Gather headers.
                Map<String, String> headers = new HashMap<String, String>();
                addCacheHeaders(headers, request.getCacheEntry());
                ...
```

addCacheHeaders\(\) 请求head会加入 "If-None-Match"（服务器上次返回的Last-Modified响应头中的日期值）， "If-Modified-Since"（服务器上次返回的ETag响应头的值） 进行条件请求，以便返回304

```text
    private void addCacheHeaders(Map<String, String> headers, Cache.Entry entry) {
        // If there's no cache entry, we're done.
        if (entry == null) {
            return;
        }

        if (entry.etag != null) {
            headers.put("If-None-Match", entry.etag);
        }

        if (entry.lastModified > 0) {
            Date refTime = new Date(entry.lastModified);
            headers.put("If-Modified-Since", DateUtils.formatDate(refTime));
        }
    }
```

2）如果 max-age 过期 则先显示缓存再发network dispatcher 发netwokr请求 返回304／200 把缓存写入硬盘 返回结果

### 2. 缓存

二级缓存 1）DiskCache  
2）LruImageCache

### 3.重试

DefaultRetryPolicy DEFAULT\_MAX\_RETRIES

### 4.超时

DefaultRetryPolicy DEFAULT\_TIMEOUT\_MS 1）请求超时 2）接收数据超时

### 5.日志

VolleyLog

## retrofit

retrofit-2.1.0.jar大小为86.2K okhttp-3.5.0.jar包大小342K

```text
compile 'com.squareup.retrofit2:retrofit:2.1.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
```

### 创建

```text
new Retrofit.Builder()  
                .addConverterFactory(GsonConverterFactory.create())  
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())  
                .build();
```

### 流程

![](.gitbook/assets/retrofit.jpg)

![](.gitbook/assets/retrofit2.jpg)

### API

| 序号 | 名称 | 类别 | 备注 |
| :--- | :--- | :--- | :--- |
| 1 | @GET | 请求方法 | http get |
| 2 | @POST | 请求方法 | http post |
| 3 | @PUT | 请求方法 | http put |
| 4 | @DELETE | 请求方法 | http delete |
| 5 | @PATCH | 请求方法 | http patch |
| 6 | @HEAD | 请求方法 | http head |
| 7 | @OPTIONS | 请求方法 | http options |
| 8 | @HTTP | 请求方法 | 可代替以上7个及扩展方法；有3个属性：method、path、hasBody |
| 9 | @FormUrlEncoded | 方法标记 | 请求体是 From 表单 |
| 10 | @Multipart | 方法标记 | 请求体是支持文件上传的 From 表单 |
| 11 | @Streaming | 方法标记 | 响应体的数据用流的形式返回 |
| 12 | @Headers | 方法标记 | 添加固定的请求头 |
| 13 | @Header | 参数标记 | 添加动态的请求头 |
| 14 | @Body | 参数标记 | 请求体 |
| 15 | @Field | 参数标记 | 单个表单字段 |
| 16 | @FieldMap | 参数标记 | 多个表单字段 |
| 17 | @Part | 参数标记 | 单文件上传参数 |
| 18 | @PartMap | 参数标记 | 多文件上传参数 |
| 19 | @Path | 参数标记 | URL拼接 |
| 20 | @Query | 参数标记 | 单个get参数 |
| 21 | @QueryMap | 参数标记 | 多个get参数 |
| 22 | @Url | 参数标记 | url |

### timeout

OkHttpClient.java default:

```text
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
```

### retry

rxjava: Observable.java

```text
@GET("/data.json")
Observable<DataResponse> fetchSomeData();
```

```text
restApi.fetchSomeData()
.retry(5)  // Retry the call 5 times if it errors
.subscribeOn(Schedulers.io())  // execute the call asynchronously
.observeOn(AndroidSchedulers.mainThread())  // handle the results in the ui thread
.subscribe(onComplete, onError); 
// onComplete and onError are of type Action1<DataResponse>, Action1<Throwable>
// Here you can define what to do with the results
```

### cancel task

rxjava:

```text
if (!subscription.isUnsubscribed()) {
    subscription.unsubscribe();
}
```

[原理](https://gank.io/post/56e80c2c677659311bed9841)

### 线程控制

Rxjava： computation\(\): newFixedThreadPool 固定线程池，线程池大小等于cpu数 io\(\): CachedThreadScheduler 缓冲线程池，大小1~ Integer.MAX\_VALUE newThread\(\): 新线程 trampoline\(\): 为当前线程建立一个队列，将当前任务加入到队列中依次执行 immediate\(\): 这个调度器允许你立即在当前线程执行你指定的工作

### https

[okHttp完美支持Https传输](http://blog.csdn.net/sk719887916/article/details/51597816) [ Retrofit中如何正确的使用https？](http://blog.csdn.net/dd864140130/article/details/52625666)

### Cookie

[Okhttp完美同步持久Cookie实现免登录](http://www.jianshu.com/p/1a5f14b63f47)

### 文件上传

[轻松实现多文件/图片上传/Json字符串/表单](http://www.jianshu.com/p/acfefb0a204f)

### 断点续传

[完成大文件断点下载](http://www.jianshu.com/p/582e0a4a4ee9)

## glide

glide-3.6.1.jar 464.1 KB okhttp-integration-1.3.1.jar 5.32 KB okhttp-3.5.0.jar包大小342K

```text
compile 'com.github.bumptech.glide:glide:3.6.1'
```

### cache \(3级缓存\)

#### memeryCache

memeryCache 默认2屏

GlideBuilder.setMemoryCache\(new LruResourceCache\(memorySizeCalculator.getMemoryCacheSize\(\)\);

```text
    static final int MEMORY_CACHE_TARGET_SCREENS = 2;
    private float memoryCacheScreens = MEMORY_CACHE_TARGET_SCREENS;
```

#### bitmap cache

bitmap pool 默认缓存4屏 GlideBuilder.setBitmapPool\(new LruBitmapPool\(size\)\);

```text
    static final int BITMAP_POOL_TARGET_SCREENS = 4;
    private float bitmapPoolScreens = BITMAP_POOL_TARGET_SCREENS;
```

#### disk cache

DiskCache.java GlideBuilder.setDiskCache\(new InternalCacheDiskCacheFactory\(context\)\);

```text
      /** 250 MB of cache. */
      int DEFAULT_DISK_CACHE_SIZE = 250 * 1024 * 1024;
```

### 线程

#### sourceExecutor

GlideExecutor.java calculateBestThreadCount\(\);

```text
Math.min(MAXIMUM_AUTOMATIC_THREAD_COUNT, Math.max(availableProcessors, cpuCount))
```

#### diskCacheExecutor

默认1个线程

```text
  public static final int DEFAULT_DISK_CACHE_EXECUTOR_THREADS = 1;
```

### timeout

#### 默认HttpURLConnection timeout 2500；

HttpUrlFetcher.java

```text
        urlConnection.setConnectTimeout(2500);
        urlConnection.setReadTimeout(2500);
        urlConnection.setUseCaches(false);
```

#### 自定义timeout

**加入okhttp集成包**

```text
dependencies {  
    // your other dependencies
    // ...

    // Glide
    compile 'com.github.bumptech.glide:glide:3.6.1'

    // Glide's OkHttp Integration 
    compile 'com.github.bumptech.glide:okhttp-integration:1.3.1@aar'
    compile 'com.squareup.okhttp:okhttp:2.5.0'
}
```

**自定义OkHttpGlideModule**

**第一步：gradle 引入okhttp3-integration 资源**

```text
   compile 'com.github.bumptech.glide:okhttp3-integration:1.4.0@aar'
```

**第二步：自定义OkHttpGlideModule**

```text
public class CustomGlideModule extends OkHttpGlideModule {
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // stub
    }

    @Override
    public void registerComponents(Context context, Glide glide) {
        final OkHttpClient.Builder builder = new OkHttpClient.Builder();

        // set your timeout here
        builder.readTimeout(30, TimeUnit.SECONDS);
        builder.writeTimeout(30, TimeUnit.SECONDS);
        builder.connectTimeout(30, TimeUnit.SECONDS);

        glide.register(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(builder.build()));
    }
}
```

**第三步： AndroidManifest添加metedata标记**

```text
        <meta-data
            android:name="peter.util.searcher.net.CustomGlideModule"
            android:value="GlideModule" />
```

**第四步： 添加keep到proguard文件中**

```text
-keep class com.bumptech.glide.integration.okhttp3.OkHttpGlideModule
```

## cron4j

[官网](http://www.sauronsoftware.it/projects/cron4j/manual.php) [github](https://github.com/Takuto88/cron4j)

### demo

```text
import it.sauronsoftware.cron4j.Scheduler;

public class Quickstart {

    private static Scheduler s;
    private static String scheduleTaskId;

    public static void main(String[] args) {
        // Creates a Scheduler instance.
        s = new Scheduler();
        // Schedule a once-a-minute task.
        scheduleTaskId = s.schedule("* * * * *", new Runnable() {
            public void run() {
                System.out.println("Another minute ticked away...");
            }
        });
        // Starts the scheduler.
        s.start();
        // Will run for ten minutes.
        try {
            Thread.sleep(1000L * 60L * 10L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // Stops the scheduler. this method will block, Before returning, it waits the
        // end of all the running tasks previously launched
        s.stop();
    }

    public void cancelTask() {
        s.deschedule(scheduleTaskId);
    }

}
```

### TimerThread如何保证在每分钟的毫秒起始点执行Launch

run 代码如下：

```text
    public void run() {
        // What time is it?
        long millis = System.currentTimeMillis();
        // Calculating next minute.
        long nextMinute = ((millis / 60000) + 1) * 60000;
        // Work until the scheduler is started.
        for (;;) {
            // Coffee break 'till next minute comes!
            long sleepTime = (nextMinute - System.currentTimeMillis());
            if (sleepTime > 0) {
                try {
                    safeSleep(sleepTime);
                } catch (InterruptedException e) {
                    // Must exit!
                    break;
                }
            }
            // What time is it?
            millis = System.currentTimeMillis();
            // Launching the launching thread!
            scheduler.spawnLauncher(millis);
            // Calculating next minute.
            nextMinute = ((millis / 60000) + 1) * 60000;
        }
        // Discard scheduler reference.
        scheduler = null;
    }
```

核心代码：

```text
long nextMinute = ((millis / 60000) + 1) * 60000; //下一个整点分钟对应的毫秒

long sleepTime = (nextMinute - System.currentTimeMillis()); //当前时间和整点毫秒的差值

safeSleep(sleepTime);//睡眠差值毫秒时间  此后即在整点毫秒执行launch
```

## Gson

[官网](https://sites.google.com/site/gson/Home) Gson是目前功能最全的Json解析神器。

### 基本使用

Gson提供了fromJson\(\) 和toJson\(\) 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化。

```text
User user = gson.fromJson(jsonString, User.class);
String jsonObject = gson.toJson(user);
```

### 容错 @SerializedName

```text
//单容错
@SerializedName("email_address")
public String emailAddress;

//多容错
@SerializedName(value = "emailAddress", alternate = {"email", "email_address"})
public String emailAddress;
```

### 泛型 TypeToken

使用泛型不再重复定义Pojo类

```text
// data 为 object 的情况
{"code":"0","message":"success","data":{}}
// data 为 array 的情况
{"code":"0","message":"success","data":[]}
...
```

```text
public class Result<T> {
    public int code;
    public String message;
    public T data;
}
```

```text
Type userType = new TypeToken<Result<User>>(){}.getType();
Result<User> userResult = gson.fromJson(json,userType);
User user = userResult.data;

Type userListType = new TypeToken<Result<List<User>>>(){}.getType();
Result<List<User>> userListResult = gson.fromJson(json,userListType);
List<User> users = userListResult.data;
```

### 自定义json输出字符 GsonBuilder

#### 导出null值、格式化输出、日期时间等

Gson在默认情况下是不动导出值null的键的。

```text
public class User {
    //省略其它
    public String name;
    public int age;
    public String email;
}
```

email字段是没有在json中出现

```text
Gson gson = new Gson();
User user = new User("peter",24);
System.out.println(gson.toJson(user)); //{"name":"peter","age":24}
```

配置之后就可以输出了

```text
Gson gson = new GsonBuilder()
        .serializeNulls()
        .create();
User user = new User("peter", 24);
System.out.println(gson.toJson(user)); //{"name":"peter","age":24,"email":null}
```

#### 字段过滤 @Expose

```text
@Expose //
@Expose(deserialize = true,serialize = true) //序列化和反序列化都都生效
@Expose(deserialize = true,serialize = false) //反序列化时生效
@Expose(deserialize = false,serialize = true) //序列化时生效
@Expose(deserialize = false,serialize = false) // 和不写一样
```

```text
public class Category {
    @Expose public int id;
    @Expose public String name;
    @Expose public List<Category> children;
    //不需要序列化,所以不加 @Expose 注解，
    //等价于 @Expose(deserialize = false,serialize = false)
    public Category parent; 
}
```

```text
Gson gson = new GsonBuilder()
        .excludeFieldsWithoutExposeAnnotation()
        .create();
gson.toJson(category);
```

#### 基于版本导出

@Since 和 @Until都接收一个Double值。 @Since：大于等于Since的值时该字段导出。 @Until：小于Until的值时该该字段导出。

```text
class SinceUntilSample {
    @Since(4)
    public String since;
    @Until(5)
    public String until;
}

public void sineUtilTest(double version){
        SinceUntilSample sinceUntilSample = new SinceUntilSample();
        sinceUntilSample.since = "since";
        sinceUntilSample.until = "until";
        Gson gson = new GsonBuilder().setVersion(version).create();
        System.out.println(gson.toJson(sinceUntilSample));
}
//当version <4时，结果：{"until":"until"}
//当version >=4 && version <5时，结果：{"since":"since","until":"until"}
//当version >=5时，结果：{"since":"since"}
```

#### android studio 自动生成代码插件 RoboPOJOGenerator

[官网](https://github.com/robohorse/RoboPOJOGenerator)

## Rxjava

包大小 2.1M [RxJava 源码](https://github.com/ReactiveX/RxJava) [reactivex 官网](http://reactivex.io/) [reactivex 中文文档](https://mcxiaoke.gitbooks.io/rxdocs/content/Intro.html)

使用

```text
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
    implementation 'io.reactivex.rxjava2:rxjava:2.1.7'
```

![](.gitbook/assets/rxjava.png)

### Rxjava切换线程池

```text
        RxJavaPlugins.setComputationSchedulerHandler(scheduler -> Schedulers
                .from(DuerShowThreadPoolUtils.getDefaultComputationPool()));
        RxJavaPlugins.setIoSchedulerHandler(scheduler -> Schedulers
                .from(DuerShowThreadPoolUtils.getDefaultIOPool()));
        RxJavaPlugins.setNewThreadSchedulerHandler(scheduler -> Schedulers
                .from(AsyncTask.THREAD_POOL_EXECUTOR));
```

### 生命周期回调

```text
Observable.doOnSubscribe(); //subscribe 前执行

Observable.doOnNext(); // onNext 前执行
Observable.doAfterNext(); // onNext 后执行

Observable.doOnDispose(); // dispose 前执行

Observable.doOnError(); // error 前执行

Observable.doOnTerminate(); // 最后执行
```

### 简化版的Observable

#### Single

只发射一条单一的数据，或者一条异常通知，不能发射完成通知，其中数据与通知只能发射一个。

#### Completable

只发射一条完成通知，或者一条异常通知，不能发射数据，其中完成通知与异常通知只能发射一个

#### Maybe

可发射一条单一的数据，以及发射一条完成通知，或者一条异常通知，其中完成通知和异常通知只能发射一个，发射数据只能在发射完成通知或者异常通知之前，否则发射数据无效。

### 操作符列表

All：判断Observable发射的所有数据项是否都满足某个条件 

Amb：给定多个Observable，只让第一个发射数据的Observable发射全部数据 

And/Then/When：通过模式（And条件）和计划（Then次序）组合两个或多个Observable发射的数据集 

Average：计算Observable发射的数据序列的平均值，然后发射这个结果 

Buffer：缓存，可以简单理解为缓存，它定期从Observable收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个 

Catch：捕获，继续序列操作，将错误替换成正常的数据，从onError通知中恢复 

CombineLatest：当两个Observables中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果 

Concat：不交错的连接多个Observable的数据 Connect：指示一个可连接的Observable开始发射数据给订阅者 

Contains：判断Observable是否会发射一个指定的数据项 

Count：计算Observable发射的数据个数，然后发射这个结果 

Create：通过调用观察者的方法从头创建一个Observable 

Debounce：只有在空闲了一段时间后才发射数据，简单来说，就是如果一段时间没有操作，就执行一次操作 

DefaultIfEmpty：发射来自原始Observable的数据，如果原始Observable没有发射数据，就发射一个默认数据 

Defer：在观察者订阅之前不创建这个Observable，为每个观察者创建一个新的Observable 

Delay：延迟一段时间发射结果数据 

Distinct：去重，过滤掉重复数据项 Do：注册一个动作占用一些Observable的生命周期事件，相当于Mock某个操作 

Materialize/Dematerialize：将发射的数据和通知都当作数据发射，或者反过来 

ElementAt：取值，取特定位置的数据项 

Empty/Never/Throw：创建行为受限的特殊Observable 

Filter：过滤，过滤掉没有通过谓词测试的数据项，只发射通过测试的 

First：首项，只发射满足条件的第一条数据 

flatMap：扁平映射，将Observable发射的数据转换为Observable集合，然后将这些Observable发射的数据平坦地放进一个单独的Observable，可以认为是将一个嵌套的数据结构展开的过程 

From：将其他对象或数据结构转换为Observable 

GroupBy：分组，将原来的Observable拆分为Observable集合，将原始Observable发射的数据按Key分组，每个Observable发射一组不同的数据 

IgnoreElements：忽略所以的数据，只保留终止通知（onError或onCompleted） 

Interval：创建一个定时发射整数序列的Observable Join：无论何时，如果一个Observable发射了一个数据项，只要在另一个Observable发射的数据项定义的时间窗口内，就将两个Observable发射的数据合并发射 

Just：将对象或者对象集合转换为一个会发射这些对象的Observable 

Last：末项，只发射最后一个数据 

Map：映射，对序列的每一项都应用一个函数变换Observable发射的数据，实质是对序列中的没一项执行一个函数，函数的参数就是这个数据项 

Max：计算并发射数据序列的最大值 

Merge：将2个Observable发射的数据合并成一个 

Min：计算并发射数据序列的最小值 

ObservableOn：指定观察者观察Observable的调度程序（工作线程） 

Publish：将一个普通的Observable转换为可连接的 

Range：创建发射指定范围的整数序列的Observable 

Reduce：按顺序对数据序列的每一项数据应用某个函数，然后返回这个值 RefCount：使一个可连接的Observable表现得像一个普通的Observable 

Repeat：创建重复发射特定的数据或序列的Observable 

Replay：确保所有的观察者收到同样的数据序列，即使他们在Observable开始发射数据之后才订阅 

Retry：重试，如果Observable发射了一个错误通知，重新订阅它，期待它正常终止辅助操作 

Sample：取样，定期发射最新的数据，等同于数据抽样，有的实现中叫做ThrottleFirst 

Scan：扫描，对Observable发射的每一项数据应用一个函数，然后按顺序依次发射这些值 

SequenceEqual：判断两个Observable是否按相同的数据序列

Serialize：强制Observable按次序发射数据并且功能是有效的 

Skip：跳过前面的若干项数据 

SkipLast：跳过后面的若干数据

SkipUntil：丢弃原始Observable发射的数据，直到第二个Observable发射了一个数据，然后发射原始Observable的剩余数据 

SkipWhile：丢弃原始Observable发射的数据，直到一个特定的条件为假，然后发射原始Observable的剩余数据 

Start：创建发射一个函数返回值的Observable 

StartWith：在发射原来的Observable的数据序列之前，先发射一个指定的数据序列或数据项 

Subscribe：收到Observable发射的数据和通知后执行的操作 

SubscribeOn：指定Observable应该在哪个调度程序上执行 

Sum：计算并发射数据序列和 

Switch：将一个发射Observable序列的Observable转换为这样一个Observable，即它逐个发射那些Observable最近发射的数据 

Take：只保留前面的若干项数据 

TakeLast：只保留后面的若干项数据 

TakeUntil：发射来自原始Observable的数据，直到第二个Observable发射了一个数据或一个通知 

TakeWhile：发射原始Observable的数据，直到一个特定的条件为真，然后跳过剩余的数据 

TimeInterval：将一个Observable转换为发射两个数据之间所耗费时间的Observable 

Timeout：添加超时机制，如果过了指定的一段时间没有发射数据，就发射一个错误通知 

Timer：创建在一个指定的延迟之后发射单个数据的Observable 

Timestamp：给Observable发射的每个数据项添加一个时间戳 

To：将Observable转换为其他对象或数据结构 Using：创建一个只在Observable生命周期内存在的一次性资源 

Window：窗口，定期将来自Observable的数据拆分成一些Observable窗口，然后发射这些窗口，而不是每次发射一项。类似于buffer，但buffer发射的是数据，Window发射的是Observable，每个Observable发射原始Observable数据的一个子集 

Zip：打包，使用一个指定的函数将多个Observable发射的数据组合在一起，然后将这个函数的结果作为单项数据发射

| 类型 | 描述 |
| :--- | :--- |
| Observable | 能够发射0或n个数据，并以成功或错误事件终止。 |
| Flowable | 能够发射0或n个数据，并以成功或错误事件终止。 支持Backpressure，可以控制数据源发射的速度。 |
| Single | 只发射单个数据或错误事件。 |
| Completable | 它从来不发射数据，只处理 onComplete 和 onError 事件。可以看成是Rx的Runnable。 |
| Maybe | 能够发射0或者1个数据，要么成功，要么失败。有点类似于Optional |

subscribeOn 改变发射线程 // 只有第一subscribeOn\(\) 起作用（所以多个 subscribeOn\(\) 毛意义） observeOn 改变接收线程 // observeOn\(\) 可以使用多次，每个 observeOn\(\) 将导致一次线程切换\(\)

### 单纯耗时操作

```text
            Observable.create(new ObservableOnSubscribe<Object>() {
                @Override
                public void subscribe(ObservableEmitter<Object> observableEmitter) throws Exception {
                    // 耗时io操作
                }
            }).subscribeOn(Schedulers.io()).subscribe();
```

### 代替handler下载图片

```text
                    Observable.create(new ObservableOnSubscribe<Bitmap>() {
                        @Override
                        public void subscribe(ObservableEmitter<Bitmap> observableEmitter) throws Exception {
                            // 耗时io操作
                            observableEmitter.onNext(bitmap);
                            observableEmitter.onComplete();
                        }
                    }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread())
                            .subscribe(new Consumer<Bitmap>() {
                                @Override
                                public void accept(Bitmap bitmap) throws Exception {
                                    // main thread 操作
                                }
                            });
```

### view计算点击次数触发操作

```text
    public static Observable<Integer> listen(final View view, int count) {
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(final ObservableEmitter<String> observableEmitter) throws Exception {
                view.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        observableEmitter.onNext("click");
                    }
                });
            }
        }).buffer(count).map(new Function<List<String>, Integer>() {
            @Override
            public Integer apply(List<String> strings) throws Exception {
                return strings.size();
            }
        });
    }
```

### 防止view多次点击触发操作

```text
    public static Disposable filterClick(final View view, final View.OnClickListener listener) {
        final WeakReference<View> viewWeakReference = new WeakReference<>(view);
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(final ObservableEmitter<String> observableEmitter) throws Exception {
                view.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        observableEmitter.onNext("click");
                    }
                });
            }
        }).throttleFirst(1000, TimeUnit.MILLISECONDS).map(new Function<String, String>() {

            @Override
            public String apply(String s) throws Exception {
                return s;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                View view = viewWeakReference.get();
                if (view != null) {
                    listener.onClick(view);
                }
            }
        });
    }
```

### 防止方法多次运行

```text
    private void multiRun() {
        Log.e("peter", "rx run");
    }

    private Runnable realRun = new Runnable() {
        @Override
        public void run() {
            multiRun();
        }
    };

    private java.util.Observable observable = new java.util.Observable() {
        @Override
        public synchronized boolean hasChanged() {
            return true;
        }
    };

    private Disposable runFilter(java.util.Observable observable, Runnable realRunnable) {
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                observable.addObserver(new Observer() {
                    @Override
                    public void update(java.util.Observable o, Object arg) {
                        Log.e("peter", "run");
                        emitter.onNext("abc");
                    }
                });
            }
        }).throttleLast(500, java.util.concurrent.TimeUnit.MILLISECONDS).observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Exception {
                        realRunnable.run();
                    }
                });
    }


    runFilter(observable, realRun); // 注册
    observable.notifyObservers(); // 在方法多次运行处触发
```

### RXJava比较好的资料

[详解 RxJava 的消息订阅和线程切换原理](https://juejin.im/post/5b1fbd796fb9a01e8c5fd847) [RxJava2 只看这一篇文章就够了](https://juejin.im/post/5b17560e6fb9a01e2862246f#heading-15)

## LeakCanray

[官网](https://github.com/square/leakcanary)

### 使用:

#### 第一步：In your build.gradle:

```text
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.6.3'
  releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.6.3'
  // Optional, if you use support library fragments:
  debugImplementation 'com.squareup.leakcanary:leakcanary-support-fragment:1.6.3'
}
```

#### 第二步 In your Application class:

```text
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```

### 源码

#### leakcanary-watcher:

这是一个通用的内存检测器，对外提供一个 RefWatcher\#watch\(Object watchedReference\),它不仅能够检测Activity，还能监测任意常规的 Java Object 的泄漏情况。

#### leakcanary-android:

这个 module 是与 Android 的接入点，用来专门监测 Activity 的泄漏情况，内部使用了 application\#registerActivityLifecycleCallbacks 方法来监听 onDestory 事件，然后利用 leakcanary-watcher 来进行弱引用＋手动 GC 机制进行监控。

#### leakcanary-analyzer:

这个 module 提供了 HeapAnalyzer，用来对 dump 出来的内存进行分析并返回内存分析结果AnalysisResult，内部包含了泄漏发生的路径等信息供开发者寻找定位。

#### leakcanary-android-no-op:

这个 module 是专门给 release 的版本用的，内部只提供了两个完全空白的类 LeakCanary 和 RefWatcher，这两个类不会做任何内存泄漏相关的分析。因为 LeakCanary 本身会由于不断 gc 影响到 app 本身的运行，而且主要用于开发阶段的内存泄漏检测。因此对于 release 则可以 disable 所有泄漏分析。

### 原理

ActivityLifecycleCallbacks -&gt; activity ondestory\(\) -&gt; refWatcher.watch\(activity\) -&gt; mainThrad postdelay 5sed -&gt; ensureGone\(\) -&gt; Runtime.getRuntime\(\).gc\(\); enqueueReferences\(\); System.runFinalization\(\); -&gt; ReferenceQueue.poll != null -&gt; Debug.dumpHprofData\(path\);

主要用到3个重要知识点：weakreference + referenceQueue + dumpHprofData

## okhttp

[官网](https://github.com/square/okhttp)

### 内置拦截器

#### RetryAndFollowupInterceptor 重连及重定向拦截器

创建了StreamAllocation，用于Socket管理 处理重定向 失败重连

#### BridgeInterceptor 请求和响应转化拦截器

在请求阶段补全HTTP Header; 响应阶段保存Cookie 响应阶段处理Gzip解压缩;

#### CacheInterceptor 缓存拦截器

Okhttp的网络缓存是基于http协议 [http缓存协议](https://my.oschina.net/leejun2005/blog/369148)

#### ConnectInterceptor  网络连接拦截器

ConnectInterceptor用来与服务器建立连接

#### CallServerInterceptor

利用okio进行数据传输与解析

### Log

```text
compile 'com.squareup.okhttp3:logging-interceptor:3.2.0'
```

```text
HttpLoggingInterceptor logging = new HttpLoggingInterceptor();  
// set your desired log level
logging.setLevel(HttpLoggingInterceptor.Level.BODY);
OkHttpClient.Builder httpClient = new OkHttpClient.Builder();   
// add your other interceptors …
// add logging as last interceptor
httpClient.addInterceptor(logging);  // <-- this is the important line!
Retrofit retrofit = new Retrofit.Builder()  
.baseUrl(API_BASE_URL)
.addConverterFactory(GsonConverterFactory.create())
.client(httpClient.build())
.build();
```

### cache（文件缓存）

默认无cache，需要自己添加

```text
OkHttpClient okHttpClient = new OkHttpClient();
File cacheFile = new File(context.getCacheDir(), "[缓存目录]");
Cache cache = new Cache(cacheFile, 1024 * 1024 * 100); //100Mb
okHttpClient.setCache(cache);
```

如果超过的最大缓存。linkedHashmap 删除第一条数据（最旧的数据） [参考](http://www.jianshu.com/p/9c3b4ea108a7)

### Interceptor

[官方文档](https://github.com/square/okhttp/wiki/Interceptors) 文档解释：观察，修改以及可能短路的请求输出和响应请求的回来。通常情况下拦截器用来添加，移除或者转换请求或者回应的头部信息。 拦截器接口中有intercept\(Chain chain\)方法，同时返回Response。这里有一个简单的拦截弹，它记录了即将到来的请求和输入的响应。

```text
class LoggingInterceptor implements Interceptor {
  @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    long t1 = System.nanoTime();
    logger.info(String.format("Sending request %s on %s%n%s",
        request.url(), chain.connection(), request.headers()));
    Response response = chain.proceed(request);
    long t2 = System.nanoTime();
    logger.info(String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));
    return response;
  }
}
```

![](.gitbook/assets/okhttp.png)

