# **使用window实现画中多画的位移动画效果**
android原生SurfaceAnimator 的 AnimationLeash框架支持位移动画，通过一些特殊设置可以达到画中多画的位移动画效果，注：原生框架位移动画的开始结束位置是配置到文件中的不可随意动态变化。具体实现如下：动画 起点 （0，0），  终点（900，300）
# 一，window styel配置
```
<style name="dialogWindowAnim" parent="android:Animation" >
    <item name="android:windowEnterAnimation">@anim/translate_enter</item>
    <item name="android:windowExitAnimation">@anim/translate_exit</item>
 
    <!-- clear dialog style -->
    <item name="android:windowFrame">@null</item>
    <item name="android:windowIsFloating">true</item>
    <item name="android:windowIsTranslucent">false</item>
    <item name="android:windowNoTitle">true</item>
    <item name="android:background">@android:color/darker_gray</item>
    <item name="android:windowBackground">@null</item>
    <item name="android:backgroundDimEnabled">false</item>
    <item name="android:backgroundDimAmount">0</item>
</style>
```
# 二，位移动画配置
## 1，进入动画
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:interpolator="@android:interpolator/decelerate_quad">
    <translate
        android:fromXDelta="-900"
        android:fromYDelta="-300"
        android:toXDelta="0"
        android:toYDelta="0" />
 
</set>
```
## 2，退出动画
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:interpolator="@android:interpolator/decelerate_quad">
    <translate
        android:fromXDelta="0"
        android:toXDelta="-900"
        android:fromYDelta="0"
        android:toYDelta="-300"/>
</set>
```
# 三，代码实现
```
public void winAnim(CodeActivity a) {
    int toX = 900;
    int toY = 300;
    final Dialog dialog = new Dialog(a, R.style.dialogWindowAnim);
    TextView textView = new TextView(a);
    textView.setText("Window anim============================== \n " +
            "========================== \n " +
            "============================");
    dialog.setContentView(textView);
    dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
    dialog.getWindow().setWindowAnimations(R.style.dialogWindowAnim);
    WindowManager.LayoutParams wmlp = dialog.getWindow().getAttributes();
    wmlp.setTitle("custom dialog===========");
    wmlp.gravity = Gravity.START | Gravity.TOP;
    wmlp.x = toX;
    wmlp.y = toY;
    dialog.show();
    textView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            dialog.dismiss();
        }
    });
```