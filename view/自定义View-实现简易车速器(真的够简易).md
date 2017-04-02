# 自定义View-实现简易车速器（真的够简易）

　　学习自定义View挺久了,好久没用都快忘了,这里实现一个简易的车速器算是一个回顾，项目比较简单，代码较少，但自定义View的流程基本都涉及到了.本文不是一篇讲解自定义View基础的文章,而是一个小的实战,如果想看讲解自定义View的文章,强烈推荐博客:http://blog.csdn.net/aigestudio,强烈=_=.    
　　效果如下图:  

![](https://github.com/whyrookie/SpeedometerView/blob/master/images/SpeedometerView.gif)  

　　接下来是代码部分:  
　　attrs:  
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="SpeedometerView">
        <attr name="centerCircleWidth" format="dimension"/>
        <attr name="outCircleWidth" format="dimension"/>
        <attr name="whiteScaleWidth" format="dimension"/>
        <attr name="redScaleWidth" format="dimension"/>
        <attr name="numberWidth" format="dimension"/>
        <attr name="pointerWidth" format="dimension"/>
        <attr name="outCircleColor" format="color"/>
        <attr name="innerCircleColor" format="color"/>
        <attr name="numberColor" format="color"/>
        <attr name="normalScaleColor" format="color"/>
        <attr name="multipleOfTenScaleColor" format="color"/>
    </declare-styleable>
</resources>
```
　　attrs文件包含自定义的属性,可以在布局文件或者代码中使用.  
　　colors:  
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#3F51B5</color>
    <color name="colorPrimaryDark">#303F9F</color>
    <color name="colorAccent">#FF4081</color>
    <color name="color_out_arc">@android:color/holo_red_dark</color>
    <color name="color_full_ten_number">@android:color/holo_red_dark</color>
    <color name="color_inner_arc">@android:color/white</color>
    <color name="color_full_ten_indicator">@android:color/holo_red_dark</color>
    <color name="color_not_full_ten_indicator">@android:color/white</color>
</resources>

```

  布局文件:  
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:SpeedometerView="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/darker_gray"
    tools:context="com.example.why.speedometerview.MainActivity">

   <com.example.why.speedometerview.SpeedometerView
       android:background="@android:color/black"
       android:id="@+id/speedometer_view"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       SpeedometerView:outCircleWidth="5dp"
       />
</FrameLayout>

```

　　接下来是Java代码:　　
　　SpeedometerView:
```Java
/**
 * Created by why on 17-3-9.
 * @author why
 * 简易View实现类
 */

public class SpeedometerView extends View {

    private static final String TAG = "SpeedometerView";

    private Paint mInnerArcPaint; // 内部弧线画笔
    private Paint mOutArcPaint; // 外部弧线画笔
    private Paint mNotFullTenIndicatorPaint; // 非整10刻度画笔
    private Paint mFullTenIndicatorPaint; // 整10加粗刻度画笔
    private Paint mNumberPaint; // 数字画笔
    private Paint mIndicatorPathPaint; // 刻度指针画笔,计可旋转的线条

    private int mInnerArcLineWidth; // 内部弧线线条宽度
    private int mOutArcLineWidth; // 外部弧线线条宽度
    private int mNormalIndicatorWidth; // 刻度线条宽度
    private int mFullTenScaleWidth; // 整10加粗刻度线条宽度
    private int mFullTenScaleLen ; // 非整10加粗线条长度
    private int mNotFullScaleLen; // 白色刻度线条长度
    private int mNumberWidth; // 数字线条刻度
    private int mIndicatorPathLineWidth;//
    private float mNumberTextSize; // 数字字体大小
    private int mCenterCircleRadius; // 内圆半径
    private int mOutCircleRadius; // 外圆半径

    private  RectF mOutRectF; // 外弧线的基准矩形
    private  RectF mCenterRectF; //内弧线基准矩形

    // 这两个值是当view的width和height属性为wrap_content时,属性不起作用配置的默认值，详情间onMeasure();
    private int mViewDefaultWidth; // 默认的view的宽度
    private int mViewDefaultHeight; // 默认的view的高度,

    private int[] mWindowSize = new int[2]; // 保存屏幕的宽/高
    private Path mIndicatorPath; // 刻度指针,用path实现
    private int mScaleIndicatorLen; // 刻度指针的长度
    private int mIndicatorX; // 刻度指针指尖的X坐标
    private int mIndicatorY; // 刻度指针指尖的Y坐标

    private int mOutArcColor = Color.RED; // 外部弧线默认颜色
    private int mNumberColor =Color.RED; // 刻度数字默认颜色
    private int mInnerArcColor = Color.WHITE; //内部弧线默认颜色
    private int mFullTenIndicatorColor = Color.RED; // 整10刻度颜色
    private int mNotFullTenIndicatorColor = Color.WHITE; // 非整10刻度颜色

    public SpeedometerView(Context context) {
        super(context);
        init(context,null,0);
    }

    public SpeedometerView(Context context, AttributeSet attrs) {
        super(context,attrs);
        init(context,attrs,0);
    }

    public SpeedometerView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context,attrs,defStyleAttr);
        init(context,attrs,defStyleAttr);
    }

    /**
     * 初始化默认数值
     */
    private void initSize(){
        mWindowSize = getWindowSize();
        mViewDefaultWidth = mWindowSize[0];
        mViewDefaultHeight = mWindowSize[1];
        mOutCircleRadius = mWindowSize[0]/2-20;
        mNumberTextSize = mOutCircleRadius / 13;
        mInnerArcLineWidth = mOutCircleRadius/150;
        mCenterCircleRadius = mOutCircleRadius / 8;
        mOutArcLineWidth = mOutCircleRadius /150;
        mNormalIndicatorWidth =  mOutCircleRadius /150;
        mFullTenScaleWidth = mOutCircleRadius / 150;
        mFullTenScaleLen = mOutCircleRadius /6;
        mNotFullScaleLen = mOutCircleRadius / 7;
        mIndicatorPathLineWidth = mOutCircleRadius/100;
        mNumberWidth =  mOutCircleRadius/100;
    }

    /**
     * 初始化默认颜色
     */
    private void initColor() {
        Context context = getContext();
        mOutArcColor = ContextCompat.getColor(context, R.color.color_out_arc);
        mNumberColor = ContextCompat.getColor(context, R.color.color_full_ten_number);
        mInnerArcColor = ContextCompat.getColor(context, R.color.color_inner_arc);
        mFullTenIndicatorColor = ContextCompat.getColor(context, R.color.color_full_ten_indicator);
        mNotFullTenIndicatorColor = ContextCompat.getColor(context,R.color.color_not_full_ten_indicator);
    }

    /**
     * 初始化
     * @param context
     * @param attrs
     * @param defStyleAttr
     */
    private void init(Context context,AttributeSet attrs, int defStyleAttr){

        initSize(); //为各个属性设置默认的值
        initColor(); //设置默认颜色
        //从xml获取设置的属性，注意：这里只提供颜色属性
        if (attrs != null) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs,R.styleable.SpeedometerView,defStyleAttr,0);
            mOutArcColor = typedArray.getColor(R.styleable.SpeedometerView_outCircleColor,mOutArcColor);
            mNumberColor = typedArray.getColor(R.styleable.SpeedometerView_numberColor, mNumberColor);
            mInnerArcColor = typedArray.getColor(R.styleable.SpeedometerView_innerCircleColor, mInnerArcColor);
            mNotFullTenIndicatorColor = typedArray.getColor(R.styleable.SpeedometerView_normalScaleColor,                                                       mNotFullTenIndicatorColor);
            mFullTenIndicatorColor = typedArray.getColor(R.styleable.SpeedometerView_multipleOfTenScaleColor,                                                   mFullTenIndicatorColor);
            typedArray.recycle();
        }

        //设置内弧线画笔
        mInnerArcPaint = new Paint();
        mInnerArcPaint.setStyle(Paint.Style.STROKE);
        mInnerArcPaint.setColor(mInnerArcColor);
        mInnerArcPaint.setAntiAlias(true); //抗锯齿
        mInnerArcPaint.setStrokeWidth(mInnerArcLineWidth);

        //设置外弧线画笔
        mOutArcPaint = new Paint();
        mOutArcPaint.setStyle(Paint.Style.STROKE);
        mOutArcPaint.setColor(mOutArcColor);
        mOutArcPaint.setAntiAlias(true);
        mOutArcPaint.setStrokeWidth(mOutArcLineWidth);

        //设置非整十刻度画笔
        mNotFullTenIndicatorPaint = new Paint();
        mNotFullTenIndicatorPaint.setStyle(Paint.Style.FILL);
        mNotFullTenIndicatorPaint.setColor(mNotFullTenIndicatorColor);
        mNotFullTenIndicatorPaint.setAntiAlias(true);
        mNotFullTenIndicatorPaint.setStrokeWidth(mNormalIndicatorWidth);
        //设置整十刻度画笔
        mFullTenIndicatorPaint = new Paint();
        mFullTenIndicatorPaint.setStyle(Paint.Style.FILL);
        mFullTenIndicatorPaint.setColor(mFullTenIndicatorColor);
        mFullTenIndicatorPaint.setAntiAlias(true);
        mFullTenIndicatorPaint.setStrokeWidth(mFullTenScaleWidth);

        //设置数字画笔
        mNumberPaint = new Paint();
        mNumberPaint.setStyle(Paint.Style.FILL);
        mNumberPaint.setColor(mNumberColor);
        mNumberPaint.setAntiAlias(true);
        mNumberPaint.setStrokeWidth(mNumberWidth);
        mNumberPaint.setTextSize(mNumberTextSize);
        mNumberPaint.setTextAlign(Paint.Align.CENTER);

        //设置刻度指针画笔
        mIndicatorPathPaint = new Paint();
        mIndicatorPathPaint.setAntiAlias(true);
        mIndicatorPathPaint.setStyle(Paint.Style.STROKE);
        mIndicatorPathPaint.setColor(Color.RED);
        mIndicatorPathPaint.setStrokeWidth(mIndicatorPathLineWidth);

        mScaleIndicatorLen = mOutCircleRadius*8/13;//指针的长度
        mIndicatorX = -mScaleIndicatorLen;//因为指针一开始是放在-180°的位置
        mIndicatorY = 0; //指针的起止坐标为内圆的圆心(onDraw()中会把原点坐标平移到屏幕的中心,方便画图)，所以y为0
        mIndicatorPath = new Path();
        mOutRectF = new RectF();
        mCenterRectF = new RectF();

    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        //通过MeasureSpec获取各自的mode和size
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

        /**
         * 这里这么做的原因是自定义View的时,width或者height的属性wrap_content不起作用，效果和match_parant相同,这里有牵扯到自
         * View扥绘制流程这里暂时不表,建议看看《Android开发艺术探索》中自定义View篇章或者看官网和相应的博客，这里只是当属性为
         * wrap_content时设置默认的mViewDefaultWidth,mViewDefaultHeight;
         */
        if (widthSpecMode == MeasureSpec.AT_MOST && heightMeasureSpec == MeasureSpec.AT_MOST) {
            setMeasuredDimension(mViewDefaultWidth, mViewDefaultHeight);
        }else if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode != MeasureSpec.AT_MOST) {
            setMeasuredDimension(mViewDefaultWidth, heightSpecSize);
        }else if (widthMeasureSpec != MeasureSpec.AT_MOST && heightMeasureSpec == MeasureSpec.AT_MOST){
            setMeasuredDimension(widthSpecSize, heightMeasureSpec);
        }else {
            setMeasuredDimension(widthSpecSize, heightSpecSize);
        }

    }

    /**
     * 在onDraw(Canvas canvas)中画图,每次调用重绘方法invalidate()都会再次调用onDraw(Canvas canvas);
     * @param canvas
     */
    @Override
    protected void onDraw(Canvas canvas) {
        canvas.translate(mWindowSize[0]/2, mWindowSize[1]/2);//canvas坐标平移到屏幕中心,这样容易操作
        //画外层的弧形,默认红色
        drawOutArc(mOutRectF, canvas);
        //画内层的弧形,默认白色
        drawInnerWhiteArc(mCenterRectF, canvas);
        //画整十刻度以及整10数字
        drawFullTenIndicatorAndNumber(canvas);
        //接下来画普通的刻度,即未加粗的刻度
        drawNotFullTenIndicator(canvas);
        //接下来画指示图标,注意这里使用path的drawPath()方法画line,每次画之前path调用path.lineTo(mIndicatorX, mIndicatorY)
        //连接内圆心和该点
        drawIndicator(mIndicatorPath, canvas);

    }

    /**
     * 获取触摸的坐标
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = MotionEventCompat.getActionMasked(event);
        float posX;
        float posY;
        double scale;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                posX  = event.getX();
                posY = event.getY();
                //如果触摸的坐标不在弧线的基准矩形中，那就不做处理，直接return
                if (!mOutRectF.contains(posX-mWindowSize[0]/2,posY - mWindowSize[1]/2) || posY - mWindowSize[1]/2 > 0){
                    return true;
                }

                /**
                 * 由于刻度指针的长度是固定的，但是手指触摸的点不是固定的，不能直接将指针的终点坐标设置为获取的x,y,但可以先算出
                 *手指触摸的点到内圆中心的距离，然后除于指针的长度,得到比例,根据相似三角形的性质可知,这个比例对于三角形的三条边都是一样的
                 *注意：这里的三角形是假设圆心,触摸点,以及X坐标相连获得的直角三角形，于是我们可以根据比例算出在点与内圆心的距离为默认的指针
                 *长度时，此时另外两条直角边的长度，即可求出,x,y的值
                 */
                scale = Math.sqrt((posX-mWindowSize[0]/2)*(posX-mWindowSize[0]/2)+ (posY - mWindowSize[1]/2)*(posY -                                        mWindowSize[1]/2))/mScaleIndicatorLen;
                mIndicatorX = rawXToIndicatorX(posX, scale);
                mIndicatorY = rawXToIndicatorY(posY, scale);
                break;
            case MotionEvent.ACTION_MOVE:

                //移动过程中不断的获取触摸点的坐标，并按比例转换为相应的指针终点坐标
                posX = event.getX();
                posY = event.getY();
                if (!mOutRectF.contains(posX-mWindowSize[0]/2,posY - mWindowSize[1]/2)|| posY - mWindowSize[1]/2 > 0){
                    return true;
                }
                Log.d(TAG,"x的="+posX+"--y的值=" +
                        ""+posY);
                scale = Math.sqrt((posX-mWindowSize[0]/2)*(posX-mWindowSize[0]/2)+ (posY - mWindowSize[1]/2)*(posY -                                        mWindowSize[1]/2))/mScaleIndicatorLen;

                //或最终的指针终点坐标,onDraw()根据mIndicatorX.mIndicatorY画指针的终点
                mIndicatorX = rawXToIndicatorX(posX, scale);
                mIndicatorY = rawXToIndicatorY(posY, scale);
                break;
        }
        //重新绘制view,onDraw()会被调用
        invalidate();
        //return true表示自己处理触摸事件,parent的onTouchEvent()方法不会被调用了
        return true;
    }

    /**
     * 按边长比例算出最终的X
     * @param posX
     * @param scale
     * @return
     */
    private int rawXToIndicatorX(double posX, double scale) {
        return (int)((posX-mWindowSize[0]/2) / scale);
    }

    /**
     * 按边长比例算出最终的Y
     * @param posY
     * @param scale
     * @return
     */
    private int rawXToIndicatorY(double posY, double scale) {
        return (int)((posY - mWindowSize[1]/2) / scale);
    }

    /**
     * 画出外弧线
     * @param rectF
     * @param canvas
     */
    private void drawOutArc(RectF rectF, Canvas canvas) {
        rectF.set(-mOutCircleRadius,  - mOutCircleRadius, mOutCircleRadius, mOutCircleRadius);
        canvas.drawArc(mOutRectF,0,-180,false,mOutArcPaint);
    }
    //画出内弧线
    private void drawInnerWhiteArc(RectF rectF, Canvas canvas) {
        rectF.set(-mCenterCircleRadius, -mCenterCircleRadius,mCenterCircleRadius, mCenterCircleRadius);
        canvas.drawArc(mCenterRectF, 0, -180, false, mInnerArcPaint);
    }

    /**
     * 画整10的刻度,因为数字对应整时，可在同一个循环中画出
     * @param canvas
     */
    private void drawFullTenIndicatorAndNumber(Canvas canvas) {

        //整十刻度终点离内圆心的长度
        int distance = mOutCircleRadius - mFullTenScaleLen;
        //接下来画10的倍数的刻度
        for (int i = -180; i<=0; i += 15) {
            double radian = ((double)i/180)*Math.PI;//弧度
            double cosValue = Math.cos(radian);//三角函数值
            double sinValue = Math.sin(radian);

            //根据弧度算出起始点和终点坐标
            float startX = (float)(distance*cosValue);
            float startY = (float)(distance*sinValue);
            float endX = (float)(mOutCircleRadius*Math.cos(radian));
            float endY = (float)(mOutCircleRadius*Math.sin(radian));
            canvas.drawLine(startX,startY,endX,endY,mFullTenIndicatorPaint);

            //刻度数字标识的坐标，
            float numberX =(float)((distance-50)*cosValue);
            float numberY =(float)((distance-50)*sinValue);
            canvas.drawText(""+Math.abs(i+180)*2/3,numberX, numberY, mNumberPaint);
        }
    }

    /**
     * 画非整10刻度
     * @param canvas
     */
    private void drawNotFullTenIndicator(Canvas canvas) {
        float normalIntervalRadian = (float)Math.PI / 60;
        int  normalIntervalRadianCount = (int)(Math.PI/normalIntervalRadian);
         int distance = mOutCircleRadius - mNotFullScaleLen;
        for (int i = 0; i <= normalIntervalRadianCount; i++) {
            if (!(i%5 == 0) ){
                float cosValue = (float)Math.cos(-normalIntervalRadian*i);
                float sinValue = (float)Math.sin(-normalIntervalRadian*i);
                float startX = (distance*cosValue);
                float startY = (distance*sinValue);
                float endX = (float)((mOutCircleRadius-10)*Math.cos((-normalIntervalRadian*i)));
                float endY = (float)((mOutCircleRadius-10)*Math.sin((-normalIntervalRadian*i)));
                canvas.drawLine(startX,startY,endX,endY,mNotFullTenIndicatorPaint);
            }
        }
    }

    /**
     * 画刻度指针
     * @param path
     * @param canvas
     */
    private void drawIndicator(Path path, Canvas canvas) {
        path.lineTo(mIndicatorX, mIndicatorY);
        canvas.drawPath(path,mIndicatorPathPaint);
        path.reset();
    }

    /**
     * 获取屏幕的宽,高,按屏幕的比例设置view的各个属性
     * @return
     */
    private int[] getWindowSize() {
        int[] size = new int[2];
        Resources resources = getContext().getResources();
        DisplayMetrics displayMetrics = resources.getDisplayMetrics();
        size[0] = displayMetrics.widthPixels;
        size[1] = displayMetrics.heightPixels;
        return size;
    }

    //这里只提供修改颜色的方法
    public int getOutCircleColor() {
        return mOutArcColor;
    }

    public void setOutCircleColor(int outCircleColor) {
        this.mOutArcColor = outCircleColor;
    }

    public int getNumberColor() {
        return mNumberColor;
    }

    public void setmNumberColor(int numberColor) {
        this.mNumberColor = numberColor;
    }

    public int getInnerCircleColor() {
        return mInnerArcColor;
    }

    public void setInnerCircleColor(int innerArcColor) {
        this.mInnerArcColor = innerArcColor;
    }

    public int getmOutArcColor() {
        return mOutArcColor;
    }

    public void setmOutArcColor(int outArcColor) {
        this.mOutArcColor = outArcColor;
    }

    public int getmNumberColor() {
        return mNumberColor;
    }

    public int getmInnerArcColor() {
        return mInnerArcColor;
    }

    public void setmInnerArcColor(int innerArcColor) {
        this.mInnerArcColor = innerArcColor;
    }

    public int getmFullTenIndicatorColor() {
        return mFullTenIndicatorColor;
    }

    public void setmFullTenIndicatorColor(int fullTenIndicatorColor) {
        this.mFullTenIndicatorColor = fullTenIndicatorColor;
    }

    public int getmNotFullTenIndicatorColor() {
        return mNotFullTenIndicatorColor;
    }

    public void setmNotFullTenIndicatorColor(int notFullTenIndicatorColor) {
        this.mNotFullTenIndicatorColor = notFullTenIndicatorColor;
    }
}

```

```Java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        SpeedometerView speedometerView = (SpeedometerView)findViewById(R.id.speedometer_view);
    }
}

```

注释还是很清楚的,源码地址https://github.com/whyrookie/SpeedometerView
