开发中经常需要自定义ProgressBar，这里是有了自定义View和ClipDrawable实现简单的ProgressBar
自定义View效果:


```java

public class CustomProgressBar extends View {

    private Paint mBgtPaint;//底部背景画笔
    private Paint mProgressPaint;//progress画笔
    private int startX;//起始X坐标，保持不变
    private int endX;//终点坐标,保持不变
    private int currentX;//当前坐标,progress换算而来，不断增大

    private int mProgressBgHeight = 60;
    private int mProgressHeight = 50;
    private int mProgressBgColor = Color.BLACK;
    private int mProgressColor = Color.BLUE;

    public CustomProgressBar(Context context){
        this(context, null);
    }

    public CustomProgressBar(Context context, @Nullable AttributeSet attrs){
       this(context, attrs, 0);

    }

    public CustomProgressBar(Context context,  @Nullable AttributeSet attrs, int defStyleAttr){
        super(context, attrs, defStyleAttr);

        TypedArray a = context.getTheme().obtainStyledAttributes(attrs,
                R.styleable.myProgressBar, defStyleAttr, 0);

        mProgressBgHeight = a.getInteger(R.styleable.myProgressBar_progress_background_height, mProgressBgHeight);
        mProgressHeight = a.getInteger(R.styleable.myProgressBar_progress_height,mProgressHeight);

        mProgressBgColor = a.getColor(R.styleable.myProgressBar_progress_background_color,mProgressBgColor);
        mProgressColor = a.getColor(R.styleable.myProgressBar_progress_color, mProgressColor);

        a.recycle();

        init();
    }


    private void init() {

        //初始化画笔
        mBgtPaint = new Paint();
        mBgtPaint.setAntiAlias(true);//抗锯齿
        mBgtPaint.setStyle(Paint.Style.STROKE);
        //注意：这是笔尖样式的属性，Paint.Cap.ROUND设置笔尖为圆形，默认方形，
        mBgtPaint.setStrokeCap(Paint.Cap.ROUND);
        mBgtPaint.setColor(mProgressBgColor);
//        mBgtPaint.setColor(Color.parseColor("#1F1F26"));//设置画笔颜色，这里是深灰色
        mBgtPaint.setStrokeWidth(60);

        mProgressPaint = new Paint();
        mProgressPaint.setAntiAlias(true);
        mProgressPaint.setStyle(Paint.Style.STROKE);
        mProgressPaint.setStrokeCap(Paint.Cap.ROUND);
        mProgressPaint.setColor(mProgressColor);
//        mProgressPaint.setColor(Color.parseColor("#54FC00"));
        mProgressPaint.setStrokeWidth(50);//这里的线条宽度比背景线条窄一些。

    }


    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        //这里根据屏幕的宽高获取坐标，可以设置成自定义属性，让用户设置
        startX = getWidth()/2-getWidth()/3;
        endX = getWidth()/2 + getWidth()/3;
        currentX = startX;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //绘制底部背景线条(深灰色，区别于View背景色)
        canvas.drawLine(startX, getHeight() / 2, endX,
                getHeight() / 2, mBgtPaint);

        //绘制当前的progress
        canvas.drawLine(startX+1, getHeight() / 2, currentX,
                getHeight() / 2, mProgressPaint);

    }

    //更新进度
    public void updateProgress(int progress) {

        //这里progress的长度分成100个单位，再计算坐标，其实progress可以是实时的下载进度，
        //计算下载量百分比，然后再计算坐标，会相对精确
        if (progress <= 100){
            currentX = startX + (int)((progress * 1.0)/100 * (endX - startX));
            invalidate();//重新绘制才能生效
        }
    }
}

```

使用到了自定义的属性,可以在values/下添加attrs.xml（文件可以为其他名字）,res/values/attrs.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="myProgressBar">

        <attr name="progress_background_color" format="color"/>
        <attr name="progress_background_height" format="integer"/>
        <attr name="progress_color" format="color"/>
        <attr name="progress_height" format="integer"/>
    </declare-styleable>
</resources>
```

代码中使用：

```java
public class CustomActivity extends AppCompatActivity {


    private CustomProgressBar mCustomProgressBar;

    private int mProgress=0;

    private Handler mHandler = new Handler();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_custom);

        mCustomProgressBar = (CustomProgressBar)findViewById(R.id.progress_bar_custom);

        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {

                mCustomProgressBar.updateProgress(++mProgress);
                if (mProgress < 100) {
                    mHandler.postDelayed(this, 500);
                }
            }
        }, 500);

    }
}

```
这样大部分的自定义的ProgressBar都可以画出来，有些可能需要添加动画。如果有些ProgressBar本身的样式
太过复杂，使用自定义view画出来有较大的难度，那么还有更简单的方法，让ui切出ProgressBar完整的图片，然后使用ClipDrawable根据
进度的更新不断地显示，使用起来其实挺简单的，效果图:


先定义一个clipDrawable
```xml
<?xml version="1.0" encoding="utf-8"?>
<clip xmlns:android="http://schemas.android.com/apk/res/android"
    android:clipOrientation="horizontal"
    android:drawable="@drawable/gakki_alone"
    android:gravity="left"/>

```
第一个属性：裁剪方向，这里是水平方向。第二个参数：图片，第三个参数：开始裁剪的位置，这里是从左开始

xml布局中使用，和普通的drawable使用方法一样

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"     
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.why.traing2.ClipDrawableActivity">

    <ImageView
        android:id="@+id/img_clip"
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:scaleType="centerCrop"
        android:background="@color/colorAccent"
        android:src="@drawable/clip"
        android:layout_centerInParent="true"/>

</FrameLayout>

```
最后是代码中的的引用

```java
public class ClipDrawableActivity extends AppCompatActivity {

    private Handler mHandler = new Handler();
    private int mLevel;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_clip_drawable);

        ImageView imageView = (ImageView)findViewById(R.id.img_clip);

        final ClipDrawable clipDrawable = (ClipDrawable)imageView.getDrawable();

        //level取值:0-10000, 0:不可见，10000:完全显示
        mLevel = clipDrawable.getLevel();

        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                clipDrawable.setLevel(mLevel);
                mLevel += 100;

                if (mLevel <= 10000) {
                    mHandler.postDelayed(this, 500);
                }
            }
        }, 500);

    }
}

```

这样复杂的ProgressBar, 可以直接使用图片按比例显示读取进度,节省代码量.
