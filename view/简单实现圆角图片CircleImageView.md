## 简单实现圆角图片CircleView

首先继承ImageView,然后使用RoundRectShape在onDraw()中画出圆角矩形，就搞定了

CircleImageView代码：

```java
public class CircleImageView extends AppCompatImageView {

  public CircleImageView(Context context) {
      this(context, null);
  }

  public CircleImageView(Context context, AttributeSet attrs) {
      this(context, attrs, 0);
  }

  public CircleImageView(Context context, AttributeSet attrs, int defStyleAttr) {
      super(context, attrs, defStyleAttr);
  }

  private Paint mPaint;
  private RoundRectShape mRroundRectShape;

  @Override
  protected void onDraw(Canvas canvas) {
  Drawable drawable = getDrawable();

      if (drawable == null) {
          return;
      }

      //获取设置src的bitmap
      Bitmap bitmap = ((BitmapDrawable)drawable).getBitmap();

      //剪切成和ImageView一样的width和height
      Bitmap scaleBitmap = Bitmap.createScaledBitmap(bitmap, getWidth(), getHeight(), true);

      mPaint = new Paint();
      mPaint.setAntiAlias(true);//抗锯齿,更平滑

      //设置BitmapShader,指定了区域需要填充的Bitmap和填充方式，这里是CLAMP，拉伸式
      mPaint.setShader(new BitmapShader(scaleBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP));

      //设置两层图像叠加的模式,这里SRC_IN：显示两层图像交集的上层，这里上层是canvas绘制的RoundRect
      //下层即ImageView设置的src图片，而mPaint.setShader指定了RoundRect被scaleBitmap填充，因此最终看到的被scaleBitmap填充的RoundRect.
      mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));

      //画出RoundRect，填充的内容有mPaint决定
      canvas.drawRoundRect(new RectF(0, 0, getWidth(), getHeight()),getWidth()/2, getHeight()/2, mPaint);

  }
}
```

  mPaint.setShader(new BitmapShader(scaleBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP))：
shader指定scacleBitmap将要覆盖的模式，REPEAT, MIRROR, 或者CLAMP,其他


整个原理就是先获取ImageView:src设置的Drawable，然后利用Bitmap.createScaledBitmap()剪切合适的大小，将其设置给BitmapShader,
用以填充到RoundRect中，再设置Xfermode,取二层图像交集的上层，调用canvas.drawRoundRect(new RectF(0, 0, getWidth(), getHeight()),getWidth()/2, getHeight()/2, mPaint)画出RoundRect, 由于RoundRect区域小于下层的区域(ImageView宽和高决定),效果就是只显示
被scaleBitmap填充的RoundRect。

使用，xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.why.traing2.MainActivity">

        <com.example.why.traing2.CircleImageView
            android:id="@+id/img_circle"
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:scaleType="fitCenter"
            android:src="@drawable/gakki"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>

```


Activivty:

```java

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CircleImageView circleImageView = (CircleImageView)findViewById(R.id.img_circle);
    }
}
```
PorterDuffXfermode模式效果官网链接:
https://developer.android.com/reference/android/graphics/PorterDuff.Mode.html
