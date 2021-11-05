# **Android 10 VS Android 9 优势在哪里?**


|功能 |android9 | android10 |
|-|-|-|
|多屏|在 Android 9 及更低版本中，SurfaceFlinger 和 DisplayManagerService 假设最多存在两个物理屏幕，其硬编码 ID 分别为 0 和 1。 如过要在Android 9上实现多屏，需要底层配合支持。|从 Android 10 开始，SurfaceFlinger 可以利用 Hardware Composer (HWC) API 生成稳定的屏幕 ID，使其能够管理任意数量的物理屏幕|
|多输入|没有多输入的概念，输入管道是单通道，不支持多屏幕同时输入，会有丢事件的问题。|Android 10上入引入了输入路由的概念，实现了屏幕和输入的绑定，这样多屏就可以实现同时输入，简单来讲就是有原来android9上的输入单通道变成了多通道，互不干扰。 |
|多用户| 目前没有开机启动多用户的策略，但是目前这部分可以实现。|支持开机启动2个用户，user0和user10，user0用来做后台管理，user10是前台用户，目前这部分谷歌也有说明。|
|双桌面|没有第二个桌面的概念，目前的实现方式就是将桌面改个包名，去掉home属性，直接startActivity到副屏上。|Android 10 引入了SECONDRY HOME，当开机启动的副屏会启动SECONDRY HOME，主屏还是启动默认的 CATEGORY_HOME。|
|双壁纸|不支持双壁纸|在 Android 10 及更高版本中，辅助屏幕可支持壁纸，在 Android 10 中，IWallpaperConnection#attachEngine() 和 IWallpaperService#attach() 接口接受 displayId 参数来创建与每个屏幕之间的连接。 这部分目前还没看代码，还未输出文档。|
|低内存探测器|对此部分没有涉及，实现需要重构AMS，基本上无法实现。|Android 10对AMS进行了重构，增加了LowMemDetector，利用内核PSI机制，实时上报内存压力值，根据不同的内存压力值，调整trimMemoryLevel，达到应用减少内存占用的目的。|
|App内存Compaction| 对此部分没有涉及，实现需要重构AMS，基本上无法实现。|Android 10中AMS单独抽出一个类进行oomadj的相关逻辑实现，根据应用oomadj的值，当应用进入后台之后，来进行相关内存页的精简和回收，减少app内存占用，并且当应用回到前台的时候不会影响性能。|
|分流广播队列|目前没有这个机制，但是可以实现。|Android 10 为了解决广播派发的性能问题，在原有前台广播队列/后台广播队列的基础上，增加了分流广播队列，可以将特定的广播塞到这个队列中，减少排队的现象。|

