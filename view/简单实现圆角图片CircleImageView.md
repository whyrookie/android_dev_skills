## 简单实现圆角图片CircleView

效果:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/gakki.png)

CircleImageView代码：

```java
public class CircleImageView extends AppCompatImageView {

    private Path mPath;
    private RectF mRectF;
    private Paint mPaint;
    public CircleImageView(Context context) {
        this(context, null);
    }

    public CircleImageView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }


    private void init() {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(15);

        mPath  = new Path();

    }
    @Override
    protected void onDraw(Canvas canvas) {
        //设置外框的矩形区域，不可再init()初始化，构造器中width和height还未确定，可在onMesure()中获取并设置
        mRectF = new RectF(0,0, getWidth(),getHeight());
        //path划出一个圆角矩形，容纳图片,图片矩形区域设置比红色外框小，否则会覆盖住外框，随意控制
        mPath.addRoundRect(new RectF(10, 10, mRectF.right-10,mRectF.bottom-10), 50, 50, Path.Direction.CW);

        canvas.drawRoundRect(mRectF, 50, 50, mPaint); //画出红色外框圆角矩形

        canvas.clipPath(mPath);//将canvas裁剪到path设定的区域，往后的绘制都只能在此区域中，

        //这一句应该放在canvas.clipPath(path)之后,canvas.clipPath(path)只对裁剪之后的绘制起作用，
        // 这个方法在ImageView中会画出xml设置的Drawable,落在刚才设置的path中
        super.onDraw(canvas);

    }
}
```



&emsp;&emsp; 整个原理就是用Path划出一个圆角矩形区域，调用super.onDraw(canvas)就可以让Drawable 落在那个区域。

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
           android:src="@drawable/gakki"
           android:scaleType="centerCrop"
           app:layout_constraintTop_toTopOf="parent"
           android:layout_marginTop="8dp"
           app:layout_constraintBottom_toBottomOf="parent"
           android:layout_marginBottom="8dp"
           android:layout_marginLeft="8dp"
           app:layout_constraintLeft_toLeftOf="parent"
           app:layout_constraintVertical_bias="0.501"
           android:layout_marginRight="8dp"
           app:layout_constraintRight_toRightOf="parent"
           />

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

看了些参考资料，发现上篇写错了，发现思路比较简单的，记录下。也可以用Shader, Xfermode实现,参考:https://enggm.wordpxress.com/tag/android-custom-image-view-in-circular-shape/
