# Activity启动过程

## **根Activity启动过程中涉及的进程**

![](.gitbook/assets/veqyvt.png)

1. launcher调用startActivity\(\) -&gt; instrumentation.execStartActivity\(\) 通过binder调用ams的startActivity\(\)。
2. ams调用Process.start\(\)并通过socket通讯调用Zygote进程。
3. Zygote进程接收到socket消息后调用Zygote.

## **Launcher请求AMS过程**

![](.gitbook/assets/veb67d.png)

## **AMS调用APP过程**

![](.gitbook/assets/veby0h.png)

## AMS和APP通讯

![](.gitbook/assets/vebdxd.png)

## **APP进程中ActivityThread启动Activity的过程**

![](.gitbook/assets/veqcku.png)

{% hint style="info" %}
[Android深入四大组件（六）Android8.0 根Activity启动过程（前篇）](https://liuwangshu.cn/framework/component/6-activity-start-1.html)

[Android深入四大组件（七）Android8.0 根Activity启动过程（后篇）](https://liuwangshu.cn/framework/component/7-activity-start-2.html)
{% endhint %}
