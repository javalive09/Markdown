# AndroidC

## activity

### activity launchmode

#### 1.standard

每次都是新的activity对象。

#### 2.singletop

栈顶单例。== Intent.FLAG\_ACTIVITY\_SINGLE\_TOP 特征：如果栈顶有此activity实例，当用startActivity调用此activity时，会复用这个activity，并调用onNewIntent\(\) 刷新intent数据。 应用场景：防止一个activity 显示多个实例。

#### 3.singleTask：

栈单例，复用它时会清除它上面的所有activity。== Intent.FLAG\_ACTIVITY\_NEW\_TASK  
情况1：没有activity taskaffinity -&gt; 情况2：有activity taskaffinity -&gt; 栈中的root activity 特征：如果栈中有此activity实例，当用startActivity调用此activity时，会复用这个activity，通过clear掉此activity之上的所有activity的方式将此activity置于栈顶，并调用onNewIntent\(\) 刷新intent数据。 应用场景：会被多个应用使用的app的main activity。 例子: 浏览器 操作：打开浏览器，输入某一网址，然后打开设置界面，按home键切出应用。点击微信，点击某一个网页消息进入，点击菜单，选使用浏览器打开。选择刚才的浏览器点击打开，此时会显示新的网页内容，点击back键会退出浏览器。相同的操作，用qq操作，结果一样。

#### 4.singleInstance：

栈单例，栈中只有一个activity。 no flag 情况1：没有activity taskaffinity -&gt; rectent task 不会显示 情况2：有activity taskaffinity -&gt; rectent task 会显示 特征：如果栈中有此activity实例，当用startActivity调用此activity时，会复用这个activity，并调用onNewIntent\(\) 刷新intent数据。 应用场景：和任何activity都没有逻辑关系（包括它所在的app）的activity 例子：闹钟 操作：打开闹钟设置界面，按home键进入微信，此时闹钟响了，进入闹钟提示界面，按返回键，回到微信界面。闹钟提示界面和任何app都没有逻辑关联，包括它自己所在的app。

#### onNewintent\(\)

2，3，4 复用时会调用onNewintent（）

### startActivityForResult问题？

在新的栈中startActivityForResult，会导致onActivityResult回调逻辑问题。最好在同一个栈中调用。

### application, task, activity 的关系？

task是任务的概念 本质是个栈 一个task任务对应一个acivity集合；通过配置activity的Android:taskAffinity来控制。 一个task中可以有多个应用的activity。 application是应用的概念，一个application可以有多个task任务（多个栈），默认的application里的activity都在一个栈中。

### task控制

[官方文档](https://developer.android.com/guide/topics/manifest/activity-element.html#aff)

#### taskAffinity

android:taskAffinity. 与 Activity 有着亲和关系的任务。从概念上讲，具有相同亲和关系的 Activity 归属同一任务（从用户的角度来看，则是归属同一“应用”）。 任务的亲和关系由其根 Activity 的亲和关系确定。 value默认值是activity-taskAffinity继承application-taskAffinity，application-taskAffinity的value是包名。 和启动它的task的root activity的taskAnifinity value 相同。

#### android:allowTaskReparenting

基于taskAffinity，是否允许此activity，回到原来的task栈中，认祖归宗（更贴切是含义，应该是该子任务是否允许被主任务认领）。 默认false，保留在启动task中。如一个应用启动了浏览器web页面activity，如果这个浏览器web页activity的allowTaskReparenting=true，这个应用退到后台（按home键），浏览器启动时会把这个web页面activity移到浏览器的栈中。 Activity 的亲和关系由 taskAffinity 属性定义。 任务的亲和关系通过读取其根 Activity 的亲和关系来确定。因此，按照定义，根 Activity 始终位于具有相同亲和关系的任务之中。 由于具有“singleTask”或“singleInstance”启动模式的 Activity 只能位于任务的根，因此更改父项仅限于“standard”和“singleTop”模式。

> 注： 如果之前启动过浏览器activity，那在recenttask列表中切换回浏览器activity，不会reparent。 如果之前没有启动过浏览器activity，退出当前应用，或杀掉应用，再点击launcher icon 启动browser则不会reparent。 reparent的点：是调用应用在后台并且没有被回收，然后点击被调用activity应用\(allowTaskReparenting=true\)的launcher icon 才会reparent。

#### android:alwaysRetainTaskState

是否保持task状态，默认false，是否保留栈列表。true的话不会清除task列表。只适用于根activity，其他的Activity都会被忽略。如浏览器mainActivity设置。

#### android:clearTaskOnLaunch

启动时是否清除task，默认false。只适用于根activity。不想保存task状态。

> 注： 在recententtask列表中切换时不会clear。 点击home回到桌面，触发launcher的icon 图标启动时才会clear。

#### android:finishOnTaskLaunch

每当用户再次启动其任务（在主屏幕上选择任务）时，是否应销毁现有 Activity 实例

> 注： 如果是true，无论是别的应用（别的task）调用，还是当前应用（相同task）应用调用。点击home，在桌面启动launcher icon时 都会销毁这个activity。

#### android:autoRemoveFromRecents

由具有该属性的 Activity 启动的任务是否一直保留在概览屏幕中，直至任务中的最后一个 Activity 完成为止。

> 注： 用于root activity，当此应用退出后，recent task中不保留。

#### android:configChanges

在Activity中添加了 android:configChanges属性，目的是当所指定属性\(Configuration Changes\)发生改变时，通知程序调用 onConfigurationChanged\(\)函数。

#### TaskAffinity

activity所在task栈的标记。是否属于某一逻辑关系task里。

### 横竖屏名字

#### 横屏

landscape风景画。风景画都是横向的，如家里面曾经挂过的黄果树瀑布的那种。

#### 竖屏

portrait肖像。肖像都是竖向的，如毛主席像。

### activity和dialog的关系

dialog的本质是个Window。 虽然dialog的构造函数是Context，但是显示是基于activity。 dialog的显示不会触发activity的任何生命周期回调函数。

### dialog的dismiss\(\)和cancel\(\)的区别

#### dismiss\(\) 源码

```text
/**  
 * Dismiss this dialog, removing it from the screen. This method can be  
 * invoked safely from any thread.  Note that you should not override this  
 * method to do cleanup when the dialog is dismissed, instead implement  
 * that in {@link #onStop}.  
 */  
@Override  
public void dismiss() {  
    if (Looper.myLooper() == mHandler.getLooper()) {  
        dismissDialog();  
    } else {  
        mHandler.post(mDismissAction);  
    }  
}
```

#### cancel\(\) 源码

```text
/**  
 * Cancel the dialog.  This is essentially the same as calling {@link #dismiss()}, but it will  
 * also call your {@link DialogInterface.OnCancelListener} (if registered).  
 */  
public void cancel() {  
    if (!mCanceled && mCancelMessage != null) {  
        mCanceled = true;  
        // Obtain a new message so this dialog can be re-used  
        Message.obtain(mCancelMessage).sendToTarget();  
    }  
    dismiss();  
}
```

#### 结论

从源码可以看出2个方法的效果一样都是让dialog消失， 只是cancel\(\) 多了一个DialogInterface.OnCancelListener的回调。

### 切换activity时他们的回调函数调用逻辑

分别监听两个activity的 onCreate onStart onRestart onResume onPause onStop onDestroy 回调函数。 [demo](http://download.csdn.net/detail/javalive09/6855725)

#### 打开act1

```text
01-16 15:54:32.374: I/peter(13244): ACT1-onCreate  
01-16 15:54:32.374: I/peter(13244): ACT1-onStart  
01-16 15:54:32.374: I/peter(13244): ACT1-onResume
```

#### 打开act2

```text
01-16 15:55:29.509: I/peter(13244): ACT1-onPause  
01-16 15:55:29.524: I/peter(13244): ACT2-onCreate  
01-16 15:55:29.524: I/peter(13244): ACT2-onStart  
01-16 15:55:29.524: I/peter(13244): ACT2-onResume  
01-16 15:55:29.604: I/peter(13244): ACT1-onStop
```

#### 关闭act2

```text
01-16 15:56:11.614: I/peter(13244): ACT2-onPause  
01-16 15:56:11.639: I/peter(13244): ACT1-onRestart  
01-16 15:56:11.639: I/peter(13244): ACT1-onStart  
01-16 15:56:11.639: I/peter(13244): ACT1-onResume  
01-16 15:56:16.714: I/peter(13244): ACT2-onStop  
01-16 15:56:16.714: I/peter(13244): ACT2-onDestroy
```

为什么会出现这种2个activity交叉调用的逻辑呢？ 为什么不全部回调完当前的activity的函数。再回调要显示的activity函数。这样的逻辑不是更简单明了么？

原因就是：动画。  act1 -&gt; act2 时 act1 先onPause （注：此时act1仍然是可见的）然后依次执行act2 onCreate onStart onResume 并做切换动画，动画执行完成后act1彻底不可见，此时调用act1 onStop。

#### 验证

增加动画显示的时间如：用5000毫秒来显示动画 执行上面的3 步骤时发现如下现象。

```text
01-16 16:07:01.544: I/peter(13244): ACT2-onPause  
01-16 16:07:01.569: I/peter(13244): ACT1-onRestart  
01-16 16:07:01.569: I/peter(13244): ACT1-onStart  
01-16 16:07:01.569: I/peter(13244): ACT1-onResume
```

过了很久直到动画显示完成才出现下面的log。

```text
01-16 16:07:06.649: I/peter(13244): ACT2-onStop  
01-16 16:07:06.649: I/peter(13244): ACT2-onDestroy
```

#### 结论

activity支持动画切换造成了这种逻辑。 如果activity切换动画的时间很长。如果直接全部回调完当前的activity的函数。再回调要显示的activity函数。 会造成这种可能：当前的activity 它的ondestory\(\) 被调用了。view 被虚拟机回收了。但是它还再做动画。这时null 空指针出现了。于是软件崩溃。

### 同一个程序不同activity放入不同的栈中

intent中加入flag Intent.FLAG\_ACTIVITY\_NEW\_TASK 标记。即可

### 不同程序的activity放入同一个栈中

activity 标签中加入： android:allowTaskReparenting="true" android:taskAffinity="same"

### activity状态的保存

栈底的activity 在系统内存不足的时候会被回收，规则是越在栈底越容易被回收。那么再次启动activity的时候就会出现问题。 Android系统提供了activity状态保存的机制。

#### 保存

重写onSaveInstanceState\(Bundle outState\) outState 保存状态值。

#### 获取

重写onRestoreInstanceState\(Bundle savedInstanceState\)通过 savedInstanceState 或onCreate\(\)中通过intent获取之前保存的值。

### splashview 闪屏界面实现的逻辑

Android 软件有时初始化时有耗时操作，很影响体验。一般的做法是加一层splash view。加载完成再进入操作界面。这样用户可以在体验上感觉更好一些。主要逻辑。 是在初始化耗时操作时，显示splash view。 耗时操作结束，显示操作界面。 [demo](http://download.csdn.net/detail/javalive09/6852655)

```text
@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  

    setContentView(R.layout.splash);//闪屏界面  

    IdleHandler handler = new IdleHandler() {  

        @Override  
        public boolean queueIdle() {  
            setContentView(R.layout.activity_main);//操作界面  

            //long time consuming Operating  
            longTime();//耗时操作  
            return false;  
        }  

    };  
    Looper.myQueue().addIdleHandler(handler);  

}
```

### idleHandler使用场景

1. 在onDraw执行完后执行某一操作  onRender\(\)。
2. 结合HandlerThread，实现单线程消息通知器。比如多个消息接收完后，执行绘制操作

### activity中addContentView和setContentView的区别

#### setContentView

逻辑在PhoneWindow 类中

```text
@Override  
public void setContentView(int layoutResID) {  
    if (mContentParent == null) {  
        installDecor();  
    } else {  
        mContentParent.removeAllViews();  
    }  
    mLayoutInflater.inflate(layoutResID, mContentParent);  
    final Callback cb = getCallback();  
    if (cb != null && !isDestroyed()) {  
        cb.onContentChanged();  
    }  
}
```

如果多次调用setContentView ，则之后每次都会清空 mContentParent 容器。然后组装资源layoutResID。

#### addContentView

逻辑也在PhoneWindow 类中

```text
@Override  
public void addContentView(View view, ViewGroup.LayoutParams params) {  
    if (mContentParent == null) {  
        installDecor();  
    }  
    mContentParent.addView(view, params);  
    final Callback cb = getCallback();  
    if (cb != null && !isDestroyed()) {  
        cb.onContentChanged();  
    }  
}
```

如果多次调用addContentView ，则之后每次都会将view添加到mContentParent 容器中。最后产生view 的叠加效果。

#### 结论：

调用 setContentView 。设置时会清空容器再添加 view。无view叠加效果 调用addContentView 。直接添加到view容器中，有view叠加效果。

### 隐式Intent

#### 隐式intent的作用

1.在不暴露类的情况下，可以让其他组件或程序调用此组组建（在filter中过滤相关intent，只有满足其条件才触发）。 2.注册给系统列表，使系统打开对应文件或链接时，该程序出现在打开列表中。如：浏览器，图片查看器。

#### 隐式intent的3个子节点

**action**

最关键的一个表示行为的属性，如果要启用隐式意图，AndroidManifest.xml中filter的action集 中必须包含此action。

**category**

对action的辅助说明。 如果启用隐式意图的intent对象中不写category，系统会加入默认的catagory：Android.intent.category.DEFAULT。AndroidManifest.xml中filter的category集合中必须包含

```text
<category android:name="android.intent.category.DEFAULT"/>
```

程序的入口activity除外。

**data**

对action的辅助说明。data属性由两部分构成 ：数据URI 和 数据MIME type; data内容往往和action有关。

### IntentFilter 中 的android:priority 的含义

关于priority 这个属性的描述 ，google 官方文档如下： [官方文档](http://developer.android.com/guide/topics/manifest/intent-filter-element.html)

```text
android:priority  
The priority that should be given to the parent component with regard to handling intents of the type described by the filter. This attribute has meaning for both activities and broadcast receivers:  
It provides information about how able an activity is to respond to an intent that matches the filter, relative to other activities that could also respond to the intent. When an intent could be handled by multiple activities with different priorities, Android will consider only those with higher priority values as potential targets for the intent.  
It controls the order in which broadcast receivers are executed to receive broadcast messages. Those with higher priority values are called before those with lower values. (The order applies only to synchronous messages; it's ignored for asynchronous messages.)  
Use this attribute only if you really need to impose a specific order in which the broadcasts are received, or want to force Android to prefer one activity over others.  

The value must be an integer, such as "100". Higher numbers have a higher priority. The default value is 0. The value must be greater than -1000 and less than 1000.  

Also see setPriority().
```

priority优先级仅用于描述控件的intent的filter的类型。这个属性只对activity和receivers是有意义的。priority 必须是整数，默认是0 范围是\[-1000, 1000\]

#### 隐式调用activity的情况

如果多个activity 满足响应 的条件，系统只会触发 priority 高的那个activity。

#### 有序广播发出的情况

如果多个receiver满足响应的条件，系统会优先触发prioriyt搞的那个receiver。

### activity ondestory\(\)的调用时机

#### 调用finish\(\)

#### 触发back键

#### 触发home键，内存紧张。

### 关闭activity的方法

1. 点击back键
2. 调用finish\(\);
3. clear top的方式 startactivity 

   ```text
   Intent intent = new Intent(getActivity(), TrashClearMainAcitivity.class);  
   intent.putExtra(TrashClearMainAcitivity.CLEARTYPE, 1);  
   intent.setFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);  
   intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);  
   Utils.startActivity(getActivity(), intent);
   ```

## Service

### 应用场景

1.长期存在的后台操作（无界面交互）。 2.公用的（context 上下文对象都可以操作）。

### 使用

1.service默认和应用主线程在同一进程。 2.Android：process=“:xxx” service是应用程序私有的，并且自己单独处在一个进程。 3.android：process=“xxx” service是global的，并且自己单独处在一个进程。

### service重启

通过onStartCommand 方法的return 值来实现。 1.service.START\_STICKY 被杀后重启。默认。 2.service.START\_NO\_STICKY 被杀后不重启。 3.service.START\_REDELIVER\_INTENT 被杀后重启。并传递 data intent。

#### 前台运行

startForeGround

```text
        Notification note = new Notification(R.drawable.ic_launcher, "Video Service",
                System.currentTimeMillis());
        note.setLatestEventInfo(this, "Video Service", null, null);
        note.flags = Notification.FLAG_NO_CLEAR;
        startForeground(1, note);
```

```text
        Notification.Builder builder = new Notification.Builder(this.getApplicationContext());
        builder.setLargeIcon(BitmapFactory.decodeResource(this.getResources(), R.drawable.ic_launcher))
                .setContentTitle(getString(R.string.tcp_content_title))
                .setSmallIcon(R.drawable.ic_launcher)
                .setContentText(getString(R.string.tcp_content_txt))
                .setWhen(System.currentTimeMillis());
        Intent resultIntent = new Intent(this, MainActivity.class);
        TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
        stackBuilder.addNextIntent(resultIntent);
        PendingIntent resultPendingIntent = stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
        builder.setContentIntent(resultPendingIntent);
        Notification notification = builder.build();
        startForeground(110, notification);
```

### IntentService的应用场景

开启一个新的IntentService 就会在它内部新建一个Thread 并在其上加了一个Looper。执行完毕后自动关闭service。 IntentService 被用于执行需要排队执行的一串耗时操作。 IntentService应用的关键点是Looper。

```text
public class SendKeyCodeService extends IntentService {

    private static final String NAME = "keycode";

    public static final String KEYCODE = "send_keycode";

    private static final String PATH_KEY_FILE = "/sys/class/mac_accessory/virtual_key/send_virtual_key";

    public SendKeyCodeService() {
        super(NAME);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        int keyCode = intent.getIntExtra(KEYCODE, 0);
        if(keyCode != 0) {
            sendKey(keyCode);
        }
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void sendKey(int keyCode) {
        try {
            PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(PATH_KEY_FILE, true)));
            out.println(keyCode);
            out.close();
        } catch (IOException e) {
            //exception handling left as an exercise for the reader
        }
    }

}
```

### Service和activity如何通讯

#### 本地Service

**1. 通过startService intent。**

**2. 通过bindeService 中间类binder 获取service引用来调用。**

**3. 通过broadcast广播 intent。**

#### 远程Service

**1. 通过startService intent。**

**2. 通过bindeService 然后使用AIDL进行相互通讯 demo**

**写aidl文件，定义要通讯的接口。**

```text
interface IMyAidlInterface {
   String getName();
}
```

**service 中提供aidl定义接口xxx.stub 的实现类。**

```text
public class MyService extends Service {

    public MyService() {

    }

    @Override
    public IBinder onBind(Intent intent) {
        return new MyBinder();
    }

    class MyBinder extends IMyAidlInterface.Stub {

        @Override
        public String getName() throws RemoteException {
            return "test";
        }
    }
}
```

c. acivity 获取xxx接口引用

```text
public class MainActivity extends AppCompatActivity {

    private IMyAidlInterface iMyAidlInterface;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService(new Intent("cc.abto.server"), new ServiceConnection(){

            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {

                iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {

            }
        }, BIND_AUTO_CREATE);
    }

    public void onClick(View view) {
        try {
            Toast.makeText(MainActivity.this, iMyAidlInterface.getName(), Toast.LENGTH_SHORT).show();
        } catch (RemoteException e){
            e.printStackTrace();
        }
    }
}
```

**3. 通过broadcast广播 intent。**

**4. 通过messenger通信\(最方便\)demo**

**service 构建messenger**

```text
public class MyService extends Service {

    public final static String TAG = "MyService";

    public final static int SERVICEID = 0x0001;
    private Messenger messenger = new Messenger(new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if (msg.arg1 == SERVICEID) {
                  //接受从客户端传来的消息
                Log.d(TAG, "客服端传来的消息===>>>>>>");
                String str = (String) msg.getData().get("content");
                Log.d(TAG, str);

                //发送数据给客户端
                Message msgTo = Message.obtain();
                msgTo.arg1 = 0X0002;
                Bundle bundle = new Bundle();
                bundle.putString("content", "我是从服务器来的字符串");
                msgTo.setData(bundle);
                try {
                    //注意，这里把数据从服务器发出了
                    msg.replyTo.send(msgTo);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    });

    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(TAG, "onCreat");
    }

    @Override
    public IBinder onBind(Intent intent) {
        return messenger.getBinder();
    }
}
```

**activity 构建messager**

```text
public class MainActivity extends AppCompatActivity {

     public final static String TAG = "MainActivity";
     public final static int ACTIVITYID = 0X0002;
     //客户端的Messnger
     private Messenger aMessenger = new Messenger(new Handler() {
         @Override
         public void handleMessage(Message msg) {
             super.handleMessage(msg);
             if (msg.arg1 == ACTIVITYID) {
              //客户端接受服务端传来的消息
                 Log.d(TAG, "服务端传来了消息=====>>>>>>>");
                 String str = (String) msg.getData().get("content");
                 Log.d(TAG, str);
             }
         }
     });

    //服务端传来的Messenger
     Messenger sMessenger;

     ServiceConnection serviceConnection = new ServiceConnection() {
         @Override
         public void onServiceConnected(ComponentName name, IBinder service) {
             sMessenger = new Messenger(service);

             Message message = Message.obtain();
             message.arg1 = 0x0001;
             //注意这里，把Activity的Messenger赋值给了message中，当然可能你已经发现这个就是Service中我们调用的msg.replyTo了。
             message.replyTo = aMessenger;

             Bundle bundle = new Bundle();
             bundle.putString("content", "我就是Activity传过来的字符串");
             message.setData(bundle);

             try {
               //消息从客户端发出
                 sMessenger.send(message);
             } catch (RemoteException e) {
                 e.printStackTrace();
             }
         }

         @Override
         public void onServiceDisconnected(ComponentName name) {
             Log.e(TAG, "连接Service失败");
         }
     };

     @Override
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_main);
         startAndBindService();
     }

     private void startAndBindService() {
         Intent service = new Intent(MainActivity.this, MyService.class);
         startService(service);
         bindService(service, serviceConnection, Context.BIND_AUTO_CREATE);
     }

 }
```

**5. Messenger 和 AIDL的对比**

a. Messenger适合于请求回复的场景。 b. AIDL除了请求回复，还具有推送功能，AIDL service端使用RemoteCallbackList管理client，能更好的管理client。

## BroadcastReceiver

### 分类

#### 根据注册方式：

1.代码注册 2.manifest文件注册

#### 按是否有序：

1.有序广播，如短信，电话。 Android:priority = "1000" \(-1000 ~ 1000\) 由高到低 可以拦截广播。 2.无序广播，无优先级。

> 注：广播启动activity必须加入FLAG\_ACTIVITY\_NEW\_TASK标记。

#### 根据发送范围

1.全局 2.定向 通过设置component定向发送广播

```text
Intent intent = new Intent();
intent.setComponent(new ComponentName(pkgName, className));
context.sendBroadcast(intent);
```

### 本地广播

#### 注册

```text
LocalBroadcastManager.getInstance(this).registerReceiver(receiver, intentFilter);
```

#### 取消注册

```text
LocalBroadcastManager.getInstance(this).unregisterReceiver(receiver);
```

#### 发送广播

```text
LocalBroadcastManager.getInstance(context).sendBroadcast(new Intent(MyCastReceiver.ACTION));
```

## 进程的优先级

### 1.前台进程

正交互的前台显示的进程。

### 2.可见进程

半透明的activity后面的activity。

### 3.服务进程

service

### 4.后台进程

按下Home键

### 5.Empty进程

按下back键

## android 8.0 查询的adj数值

activity: 前台：0 后台（按下home键）：11 退出（按下back键）：15

service： 前台：3 后台：8

应用无service广播开机自启动：13

标记 android:persistent 系统应用: -13 非系统应用: 没有任何影戏

## Android 中的一些时间

### 1. Toast  \(NotificationManagerService.java\)

```text
Toast.LENGTH_SHORT  2000ms // 2秒
Toast.LENGTH_LONG    3500ms // 3.5秒
```

### 2. ANR   \(ActivityManagerService.java\)

```text
KeyDispatchTimeout   KEY_DISPATCHING_TIMEOUT = 5 * 1000  // 5 秒
BroadcastTimeout     BROADCAST_FG_TIMEOUT = 10 * 1000     // 10 秒
ServiceTimeout       // 20 秒
```

## android中如何调用已经安装的某个应用

### 1.使用logcat过滤出 ActivityManager的log

```text
adb -s logcat | grep "ActivityManager"
```

### 2.点击应用icon 触发应用

```text
I/ActivityManager(  449): START u0 {act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10200000 cmp=com.android.calendar/.AllInOneActivity} from pid 5180  
cmp = com.android.calendar/.AllInOneActivity    是包名和启动类的名字
```

### 启动

#### 1. 根据包名和触发类的intent启动：

```text
intent = new Intent();  
intent.setClassName("com.android.calendar",  
        "com.android.calendar.AllInOneActivity");  
startActivity(intent);
```

#### 2. 根据包名使用activitymanager 启动：

```text
intent = this.getPackageManager().getLaunchIntentForPackage(  
        "com.android.calendar");  
if (intent != null) {  
    startActivity(intent);  
}
```

## 手机序列号和IMEI号的区别

### 手机序列号：

SN码是Serial Number的缩写，有时也叫SerialNo，也就是产品序列号，产品序列是为了验证“产品的合法身份”而引入的一个概念，它是用来保障用 户的正 版权益，享受合法服务的；一套正版的产品只对应一组产品序列号。别称：机器码、认证码、注册申请码等。

### IMEI:

IMEI（International Mobile Equipment Identity）是移动设备国际身份码的缩写。 IMEI码由GSM（Global System for Mobile Communications，全球移动通信协会）统一分配，授权BABT（British approvals Board of Telecommunications，英国通信认证管理委员会）审受。 IMEI 的主要用途是提供信息给网络系统，让系统知道哪个手机在收发信号。主要目的是防止被窃的手机登入网络并监听防止手机使用者任意干扰网络。

## getDimension\(\), getDimensionPixelOffset\(\), getDimensionPixelSize\(\)的区别

Resources中的这三个方法很让人迷惑。究竟有什么区别。

### 1. getDimension\(\)

```text
public float getDimension(int id) throws NotFoundException {  
    synchronized (mTmpValue) {  
        TypedValue value = mTmpValue;  
        getValue(id, value, true);  
        if (value.type == TypedValue.TYPE_DIMENSION) {  
            return TypedValue.complexToDimension(value.data, mMetrics);  
        }  
        throw new NotFoundException(  
                "Resource ID #0x" + Integer.toHexString(id) + " type #0x"  
                + Integer.toHexString(value.type) + " is not valid");  
    }  
}
```

最终会调用TypedValue.complexToDimension\(\)

```text
public static float complexToDimension(int data, DisplayMetrics metrics) {  
    return applyDimension(  
        (data>>COMPLEX_UNIT_SHIFT)&COMPLEX_UNIT_MASK,  
        complexToFloat(data),  
        metrics);  
}
```

### 2. getDimensionPixelOffset\(\)

```text
public int getDimensionPixelOffset(int id) throws NotFoundException {  
    synchronized (mTmpValue) {  
        TypedValue value = mTmpValue;  
        getValue(id, value, true);  
        if (value.type == TypedValue.TYPE_DIMENSION) {  
            return TypedValue.complexToDimensionPixelOffset(  
                    value.data, mMetrics);  
        }  
        throw new NotFoundException(  
                "Resource ID #0x" + Integer.toHexString(id) + " type #0x"  
                + Integer.toHexString(value.type) + " is not valid");  
    }  
}
```

最终会调用TypedValue.complexToDimensionPixelOffset\(\)

```text
public static int complexToDimensionPixelOffset(int data,  
        DisplayMetrics metrics) {  
    return (int)applyDimension(  
            (data>>COMPLEX_UNIT_SHIFT)&COMPLEX_UNIT_MASK,  
            complexToFloat(data),  
            metrics);  
}
```

### 3. complexToDimensionPixelSize\(\)

```text
public int getDimensionPixelSize(int id) throws NotFoundException {  
    synchronized (mTmpValue) {  
        TypedValue value = mTmpValue;  
        getValue(id, value, true);  
        if (value.type == TypedValue.TYPE_DIMENSION) {  
            return TypedValue.complexToDimensionPixelSize(  
                    value.data, mMetrics);  
        }  
        throw new NotFoundException(  
                "Resource ID #0x" + Integer.toHexString(id) + " type #0x"  
                + Integer.toHexString(value.type) + " is not valid");  
    }  
}
```

最终会调用TypedValue.complexToDimensionPixelSize\(\)

```text
public static int complexToDimensionPixelSize(int data,  
        DisplayMetrics metrics) {  
    final float value = complexToFloat(data);  
    final float f = applyDimension(  
            (data>>COMPLEX_UNIT_SHIFT)&COMPLEX_UNIT_MASK,  
            value,  
            metrics);  
    final int res = (int)(f+0.5f);  
    if (res != 0) return res;  
    if (value == 0) return 0;  
    if (value > 0) return 1;  
    return -1;  
}
```

### 结论

这3个方法在resources.Java中逻辑一致，只是最终调用的方法有所不同对比3个方法可以发现 getDimension\(\) 返回值是的float大小。 getDimensionPixelOffset\(\) 返回值是将 getDimension\(\) 强转成int 的值。（强转） getDimensionPixelSize\(\) 返回值是将getDimension\(\)+0.5 再强转成int的值。（4舍5入） 这3个方法的本质区别就是大小精确度的差别。想要大一点就用getDimensionPixelSize\(\), 想要精确点就用getDimension\(\), 想要小一点就用getDimensionPixelOffset\(\)

## @TargetApi的作用

### 官方文档描述

```text
As you can see, lint now has a database of the full Android API such that it knows precisely which version each API call was introduced in. If it detects that you are attempting to call a method which is not available in all versions you are trying to support, it will warn you with messages like the above.
```

正如你所看到的, lint现 在有完整的Android API的一个数据库,这样它清楚地知道哪个版本的哪一个API调用了。如果它检测到你正试图调用一个方法不是在所有版本都可用，那么它将提醒给你上面的消息。

```text
Of course, if you're building your project with the same version of Android as your oldest support version, the compiler will complain if it finds an API call it can't resolve. However, that approach has some downsides. In particular, you may want to access newer APIs on platforms where they are available. If you're compiling against the oldest version of the platform, you would need to use reflection to access all the new APIs, which can be really cumbersome, especially if you have a lot of code using the newer APIs.
The way most developers get around this is to actually use a newer build target, which means you can access all the new APIs directly. You then add some code checks to make sure that the code path which calls the newer APIs is only reached when running on a version which supports it.
```

当然,如果您编译的项目的版本和支持的最低支持的版本相同, 编译器会提示找不到相关API。然而,这种方法有一些缺点。特别是,您可能希望访问新的平台上可用的api。如果你对老版本的编译平台,您将需要使用反射来访问所有的新api,可真的麻烦,特别是如果你有很多的代码使用新的api。

实际上大多数开发人员解决这个问题的方法是使用更高的版本来编译这个项目,这意味着您可以直接访问所有的新api。然后添加一些代码检查,以确保调用的代码路径上运行时达成的新api只是一个版本支持它。

```text
However, when you do this you run the risk of accidentally calling newer APIs, and that's what Lint now helps you detect.
What about code where you are deliberately calling newer APIs from a class you know will only be loaded in the right circumstances? In that case you can "mark" the code as targeting a newer version of the API. Simply annotate the code with the new @TargetApiannotation:
```

然而,当你这么做了,那么在低版本上调用新的api可能会有崩溃的风险, 这就是lint检测提示的作用之所在。 那么如何保证你调用的新的api的代码的相关类只会在合适的版本下才加载? 在这种情况下你可以“标注”目标的代码的所在版本的API。简单的注释代码@TargetApi

```text
If the whole class is targeting newer APIs, you can place the annotation on the class instead:
```

如果整个类都是新的api的版本，你就可以把@Target标注移除掉。

### 总结

综上，@Target的作用在于提示：使用高编译版本的代码，为了通用性兼容运行此代码的低版本平台。要求程序员做出区分对待的加载。

## 任务调度的几种方法

### Timer

```text
public class Timer_ extends Entry {

    Timer timer = new Timer();
    long delay = 1000; // milliseconds
    long period = 1000; // milliseconds
    int count = 0;

    public void schedule() {
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                LogUtil.i("count =" + count++);
            }
        }, delay, period);

    }

    public void cancel() {
        timer.cancel();
    }

}
```

### ScheduledExecutorService

```text
public class ScheduledExecutorService_ extends Entry {

    ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
    long delay = 1000; // milliseconds
    long period = 1000; // milliseconds
    int count = 0;

    public void schedule() {
        service.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                LogUtil.i("count =" + count++);
            }
        }, delay, period, TimeUnit.MILLISECONDS);
    }

    public void cancel() {
        service.shutdownNow();
    }

}
```

### Cron4j

```text
public class Cron4j_ extends Entry {

    private Scheduler scheduler = new Scheduler();
    private String schedulingPattern1 = "30 10 * * *"; // 10:30  everyday
    private String schedulingPattern2 = "* * * * *"; // every minute
    private int count =0;

    public void schedule() {
        scheduler.schedule(schedulingPattern2, new Runnable() {
            @Override
            public void run() {
                LogUtil.i("count =" + count++);
            }
        });

        scheduler.start();
    }

    public void cancel() {
        scheduler.stop();
    }

}
```

### AlarmManager

```text
public class AlarmClock extends Entry {

    private static final int INTERVAL = 1000 * 60 * 60 * 24;// 24h

    public void setClock() {
        Intent intent = new Intent(getActivity(), AlarmClockReceiver.class);
        intent.setAction("abc");
        PendingIntent sender = PendingIntent.getBroadcast(getActivity(),
                1, intent, PendingIntent.FLAG_CANCEL_CURRENT);

        // Schedule the alarm!
        AlarmManager am = (AlarmManager) getActivity().getSystemService(Context.ALARM_SERVICE);
        Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.HOUR_OF_DAY, 17);
        calendar.set(Calendar.MINUTE, 52);
        calendar.set(Calendar.SECOND, 1);
        calendar.set(Calendar.MILLISECOND, 1);

        long time = calendar.getTimeInMillis();
        long t = System.currentTimeMillis();
        am.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP, time, INTERVAL, sender);
    }

}
```

## android 打包去掉log

代码混淆配置，发布release版本时关闭log输出

```text
-assumenosideeffects class android.util.Log {
    public static *** v(...);
    public static *** d(...);
    public static *** i(...);
    public static *** w(...);
    public static *** e(...);
}
```

使用这个配置时，一定要注意 -dontoptimize，配置。 don‘t optimize 不要优化；将会会关闭优化，导致日志语句不会被优化掉。所以不能有这个配置

## Log打印控制

推荐方法三，无须重新编译，且更加灵活，user版本也可控制打印

### 方法一：代码层标记控制

每次在发布之前都要手动去改这个DEBUG变量

```text
public class Log {
    private static final boolean DEBUG = true;

    public static void i(String tag, String msg) {
        if (DEBUG)
            android.util.Log.i(tag, msg);
    }

    public static void e(String tag, String msg) {
        if (DEBUG)
            android.util.Log.e(tag, msg);
    }

    public static void d(String tag, String msg) {
        if (DEBUG)
            android.util.Log.d(tag, msg);
    }

    public static void v(String tag, String msg) {
        if (DEBUG)
            android.util.Log.v(tag, msg);
    }

    public static void w(String tag, String msg) {
        if (DEBUG)
            android.util.Log.w(tag, msg);
    }
}
```

### 方法二：BuildConfig.DEBUG控制

DEBUG会根据Build类型自动设定 assembleRelease对应的DEBUG=true assembleDebug对应的DEBUG=false 从而实现不同包会自动过滤log信息

```text
public class Log {
    private static final boolean DEBUG = BuildConifg.DEBUG;

    public static void i(String tag, String msg) {
        if (DEBUG)
            android.util.Log.i(tag, msg);
    }

    public static void e(String tag, String msg) {
        if (DEBUG)
            android.util.Log.e(tag, msg);
    }

    public static void d(String tag, String msg) {
        if (DEBUG)
            android.util.Log.d(tag, msg);
    }

    public static void v(String tag, String msg) {
        if (DEBUG)
            android.util.Log.v(tag, msg);
    }

    public static void w(String tag, String msg) {
        if (DEBUG)
            android.util.Log.w(tag, msg);
    }
}
```

### 方法三：Log.isLoggable\(String tag, int level\)控制

1. isLoggable默认level为android.util.Log.INFO，level &gt;= INFO时isLoggable返回true
2. 动态设置（重启后失效）：可以通过setprop log.tag. 来改变log的默认level  适用系统版本（user/userdebug/eng）

```text
adb shell setprop log.tag.OTA VERBOSE
```

1. 静态设置（重启后有效）：按照log.tag.=D的形式，写入/data/local.prop中 适用系统版本（userdebug/eng）注：有些系统可能不会生效
2. tag的长度如果超过23个字符则会抛出IllegalArgumentException异常

   level如下：

```text
    /**
     * Priority constant for the println method; use Log.v.
     */
    public static final int VERBOSE = 2;

    /**
     * Priority constant for the println method; use Log.d.
     */
    public static final int DEBUG = 3;

    /**
     * Priority constant for the println method; use Log.i.
     */
    public static final int INFO = 4;

    /**
     * Priority constant for the println method; use Log.w.
     */
    public static final int WARN = 5;

    /**
     * Priority constant for the println method; use Log.e.
     */
    public static final int ERROR = 6;

    /**
     * Priority constant for the println method.
     */
    public static final int ASSERT = 7;
```

修改如下： 可用adb shell setprop log.tag.Setting D 动态开启debug模式，重启后失效

```text
public class Log {
    public static final String TAG = "OTA";

    public static void i(String tag, String msg) {
        if (android.util.Log.isLoggable(TAG, android.util.Log.DEBUG)) {
            android.util.Log.i(tag, msg);
        }
    }

    public static void e(String tag, String msg) {
        if (android.util.Log.isLoggable(TAG, android.util.Log.DEBUG)) {
            android.util.Log.e(tag, msg);
        }
    }

    public static void d(String tag, String msg) {
        if (android.util.Log.isLoggable(TAG, android.util.Log.DEBUG)) {
            android.util.Log.d(tag, msg);
        }
    }

    public static void v(String tag, String msg) {
        if (android.util.Log.isLoggable(TAG, android.util.Log.DEBUG)) {
            android.util.Log.v(tag, msg);
        }
    }

    public static void w(String tag, String msg) {
        if (android.util.Log.isLoggable(TAG, android.util.Log.DEBUG)) {
            android.util.Log.w(tag, msg);
        }
    }
}
```

## android log 分析

系统log保存地址 /data/log 关键字fatal、exception、No Response、crash、panic 1.程序异常退出（fatal） 2.程序强制关闭（fatal） 3.程序无响应（Application No Response--ANR） 分享anr文件/data/anr/traces.txt 关键字：ANR空格, DALVIK THREADS

```text
08-14 00:59:14.698 I/art     ( 4238): Wrote stack traces to '/data/anr/traces.txt'
08-14 00:59:14.702 E/ActivityManager(  562): ANR in com.baidu.launcher
08-14 00:59:14.702 E/ActivityManager(  562): PID: 1822
08-14 00:59:14.702 E/ActivityManager(  562): Reason: Broadcast of Intent { act=android.intent.action.TIME_TICK flg=0x50000014 (has extras) }
08-14 00:59:14.702 E/ActivityManager(  562): Load: 1.81 / 2.97 / 3.57
08-14 00:59:14.702 E/ActivityManager(  562): CPU usage from 746729ms to 0ms ago (2018-08-14 00:46:43.317 to 2018-08-14 00:59:10.047):
08-14 00:59:14.702 E/ActivityManager(  562):   58% 562/system_server: 35% user + 23% kernel / faults: 34475 minor 25 major
08-14 00:59:14.702 E/ActivityManager(  562):   57% 1822/com.baidu.launcher: 48% user + 9.1% kernel / faults: 547808 minor 117 major
08-14 00:59:14.702 E/ActivityManager(  562):   26% 231/surfaceflinger: 13% user + 12% kernel / faults: 13215 minor
08-14 00:59:14.702 E/ActivityManager(  562):   15% 248/mediaserver: 5.1% user + 10% kernel
08-14 00:59:14.702 E/ActivityManager(  562):   12% 240/audioserver: 7.7% user + 4.5% kernel / faults: 43 minor
08-14 00:59:14.702 E/ActivityManager(  562):   9.2% 2485/perfd: 3.8% user + 5.4% kernel / faults: 11314 minor
08-14 00:59:14.702 E/ActivityManager(  562):   9.2% 2407/adbd: 2.1% user + 7.1% kernel / faults: 36681 minor
```

4.native层：Tombstone crash 5.Kernel层：kernel panic

## SettingsProvider 读写

Secure：第三方APP有读权限没有写权限；系统应用\(system.ui.system或系统签名应用\)可读可写，需要加android.permission.WRITE\_SECURE\_SETTINGS权限可写 System：第三方APP有读权限没有写权限；系统应用\(system.ui.system 或 系统签名应用 或 内置应用（system/app或system/priv-app下的应用）\)可读可写，需要加android.permission.WRITE\_SETTINGS权限可写 Global：第三方APP有读权限没有写权限；系统应用\(system.ui.system或系统签名应用\)可读可写，需要加android.permission.WRITE\_SECURE\_SETTINGS权限可写

## 最简单的进程间通讯方式

contentprovider call方法

### 被其他应用调用访问的权限设置：

```text
    <permission
            android:name="com.xxx.xxx.permission.INTER_APP_COMMUNICATION"
            android:protectionLevel="signature"/>
```

### 其他应用调用这个应用的方式：

```text
    public static Bundle callOtherAppMethod(Context context, Uri uri, String method, String arg, Bundle bundle) {
        try {
            return context.getContentResolver().call(uri, method, arg, bundle);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```

### 被其他应用访问的下发逻辑：

```text
public class SettingsContentProvider extends ContentProvider {

    private static final HashMap<String, IMethodCallback> METHODS = new HashMap<>();

    @Override
    public Bundle call(@NonNull String method, @Nullable String arg, @Nullable Bundle extras) {
        if (getContext() != null) {
            return dispatchEvent(method, arg, extras);
        }
        return super.call(method, arg, extras);
    }

    private Bundle dispatchEvent(@NonNull String method, @Nullable String arg, @Nullable Bundle extras) {
        if (METHODS.containsKey(method)) {
            IMethodCallback callback = METHODS.get(method);
            if (callback != null) {
                callback.call(method, arg, extras);
            }
        }
        return null;
    }

    public static void registerMethod(IMethodCallback callback, String... methods) {
        for (String method : methods) {
            METHODS.put(method, callback);
        }
    }

    public static void unRegisterMethod(String... methods) {
        for (String method : methods) {
            METHODS.remove(method);
        }
    }

}
```

### 要点：

应用A 访问 应用B（提供contentprovider） 应用A（调用者）如果在主线程调用则主线程阻塞 应用B（被调用者）会单独开辟一个线程1执行call方法 如果B线程1阻塞状态 会新开一个线程2执行call 同时线程1sleep 最后会多出1个线程 也就是说阻塞次数越多 线程数就越多 线程名字为：12345678910abcdef 超过16个阻塞线程应用B会黑屏，应用A如果在主线程调用call会ANR

## AsyncTask

1. sdk &lt; 22（5.1） AsyncTask 的Handler使用的new Handler（），如果在非主线程中创建，则有可能不是mainlooper。 sdk &gt; 16\(4.1\) 在APP主线程ActivityThread的main函数里面，直接调用了AscynTask.init函数确保这个类是在主线程初始化的, 所以sk &lt; 16 必须在main线程中创建 AsyncTask。
2. sdk = 1.5 串行，  sdk = 1.6 并行，  sdk &gt;= 3.0 串行  串行的原因 是为了保证 多线程执行doinbackground（）方法里面并没有同步锁，当访问相同资源的时候会产生并发脏数据。

## apk 文件结构

1. AndroidManifest.xml
2. classes.dex      // 源码
3. res     // 资源文件
4. resource.arsc    // 资源映射信息
5. META-INFO

   5.1 MANIFEST.MF    // 文件摘要，所有文件的sha1哈希值列表

   5.2 CERT.SF     // 摘要签名，对MANIFEST.MF使用sha1-rsa算法用开发者私钥进行签名

   5.3 CERT.RSA    // 公钥信息

## Android 消息机制

### Looper

```text
threadlocal
loop msg 
recycle msg
```

### MessageQueue

```text
when排序单向链表
SyncBarrier同步屏障
idleHandler
```

### Handler

```text
getIMessenger
默认looper
Async handler
回调顺序
    1. msg callback
    2.  1) handler callback 返回值可以拦截handlermessage
        2) handlemessage
```

### Message

```text
Async message
缓存池，50链表
replyTo  Messenger
    1.service: new Messenger(handler).getBinder()提供binder
    2.client:  new Messenger(iBinder)获取messenger   message.replyTo = new Messager(hander);
    3.双方收消息都使用 new Messager(handler)方式;
```

## AsyncTask 原理

Handler + 线程池 1. SERIAL\_EXECUTOR 默认的Executor，串行执行 2. THREAD\_POOL\_EXECUTOR

```text
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE_SECONDS = 30;
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);


ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
```

核心线程数：（cpu\_count - 1） 保证在\[2, 4\]区间 最大线程数： cpu\_count \* 2 + 1 keep alive 时间： 30 秒 队列大小：128

默认 AsyncTask是串行的 多个任务执行会阻塞 使用并行的方式

```text
new MyAsyncTask(progressBar).executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR);
```

## Application onCreate 在多进程情况下多次执行

每个进程都会创建一个自己的application对象

```text
   @Override
    public void onCreate() {
        super.onCreate();
        String processName = getProcessName(this);
        if (processName!= null) {
            if(processName.equals("com.javalive09.demo")){
                //以包名为进程名，项目默认的进程
            } else if(processName.equals("com.javalive09.demo:api")){
                //com.javalive09.demo:api
            }else{

            }
        }
    }

    private String getProcessName(Context context) {
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningAppProcessInfo> runningApps = am.getRunningAppProcesses();
        if (runningApps == null) {
            return null;
        }
        for (ActivityManager.RunningAppProcessInfo proInfo : runningApps) {
            if (proInfo.pid == android.os.Process.myPid()) {
                if (proInfo.processName != null) {
                    return proInfo.processName;
                }
            }
        }
        return null;
    }
```

