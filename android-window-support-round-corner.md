# **Android Window支持圆角显示**
# 一. Window的contentview为Android GUI框架内的View

可通过配置style及圆角背景来支持
```
<item name="android:windowClipToOutline">true</item>

 <item name="android:windowBackground">@drawable/round_corners_background</item>
```
示例代码如下：
## 1.src/main/res/values/styles.xml
```
<style name="AppTheme1" parent="Theme.AppCompat.Light.DarkActionBar">
      <!-- Customize your theme here. -->
      <item name="colorPrimary">@color/colorPrimary</item>
      <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
      <item name="colorAccent">@color/colorAccent</item>
 
 
      <!-- window 圆角 start-->
      <item name="android:windowBackground">@drawable/round_corners_background</item>
      <item name="android:windowClipToOutline">true</item>
      <!-- window 圆角 end-->
  </style>
```

## 2.src/main/res/drawable/round_corners_background.xml

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
 android:shape="rectangle">
 <!-- rectangle表示为矩形 -->
 
 <!-- 填充的颜色 -->
 <solid android:color="#ffffff" />
 
 <!-- 边框的颜色和粗细 -->
 <stroke
 android:width="1dp"
 android:color="#00000000"
 />
 
 <!-- android:radius 圆角的半径 -->
 <corners
 android:radius="100dp"
 />
 
</shape>
```
# 二. Window的contentview为SurfaceView
surfaceView 不属于gui框架内的view，它的canvas是独立的，因此不支持以上配置。

可通过如下2项设置可达到圆角效果

## 1.surfaceview设置
```
SurfaceView.setZOrderOnTop(true);

SurfaceHolder.setFormat(PixelFormat.TRANSLUCENT);
```
## 2.canvas clippath
```
Path mPath = new Path();

float[] mRadius = new float[]{radius, radius, radius, radius, radius, radius, radius, radius};

RectF mRectF = = new RectF();

mPath.addRoundRect(mRectF, mRadius, Path.Direction.CW);

mCanvas.clipPath(mPath);
```

示例代码如下：
```
private static class SurfaceViewDemo extends SurfaceView implements SurfaceHolder.Callback, Runnable {
 
    private SurfaceHolder mSurfaceHolder;
    //绘图的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;
 
    private int h;
 
    private int w;
 
    private Path mPath;
 
    private RectF mRectF;
 
    private float[] mRadius;
 
    Paint mPaint;
 
    public SurfaceViewDemo(Context context) {
        super(context);
        init();
    }
 
    private void init() {
        mPath = new Path();
        mSurfaceHolder = getHolder();
        mSurfaceHolder.addCallback(this);
        setFocusable(true);
        setKeepScreenOn(true);
        setFocusableInTouchMode(true);
        mRectF = new RectF();
        float radius = 100f;
        mRadius = new float[]{radius, radius, radius, radius, radius, radius, radius, radius};
        mPaint = new Paint();
        mPaint.setColor(Color.BLUE);
        mPaint.setTextSize(30);
    }
 
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        setZOrderOnTop(true);
        getHolder().setFormat(PixelFormat.TRANSLUCENT);
        new Thread(this).start();
 
    }
 
    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
 
    }
 
    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }
 
    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
            SystemClock.sleep(16);
        }
    }
 
    private void draw() {
        h = Math.max(h, getMeasuredHeight());
        w = Math.max(w, getMeasuredHeight());
        mRectF.set(0, 0, w, h);
        mPath.addRoundRect(mRectF, mRadius, Path.Direction.CW);
 
        try {
            mCanvas = mSurfaceHolder.lockCanvas();
            mCanvas.save();
            mCanvas.clipPath(mPath);
            mCanvas.drawText("地图surfaceView", 30, 100, mPaint);
            mCanvas.restore();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (mCanvas != null) {
                //释放canvas对象并提交画布
                mSurfaceHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}
```