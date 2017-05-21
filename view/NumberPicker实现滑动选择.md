# NumberPicker使用,实现滑动选择(可实现省市联动)

  最近使用了一个控件NumberPicker,该控件可以实现省市区联动以及其他各种滑动选择，这里只是简单的记录下如何使用。
效果图:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/Peek%202017-01-14%2009-51.gif)

  先是xml文件:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.why.numberpickertest.MainActivity">
   <NumberPicker
       android:layout_gravity="center"
       android:id="@+id/number_picker"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"></NumberPicker>
</FrameLayout>
```

接下来是Activity的代码:

```java
public class MainActivity extends AppCompatActivity implements NumberPicker.Formatter{

    private NumberPicker mNumberPicker;
    private String[] mCities  = {"北京","上海","广州","深圳","杭州","青岛","西安"};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setTheme(R.style.AppTheme);
        setContentView(R.layout.activity_main);
        mNumberPicker = (NumberPicker)findViewById(R.id.number_picker);
        mNumberPicker.setDisplayedValues(mCities);//设置需要显示的数组
        mNumberPicker.setMinValue(0);
        mNumberPicker.setMaxValue(mCities.length - 1);//这两行不能缺少,不然只能显示第一个，关联到format方法
        setPickerDividerColor();

        setNumberPickerTextColor(mNumberPicker,Color.RED);
    }

    //这个方法是根据index 格式化先生的文字,需要先 implements NumberPicker.Formatter
    @Override
    public String format(int value) {
        return mCities[value];
    }

    /**
     * 通过反射改变分割线颜色,
     */
    private void setPickerDividerColor() {
        Field[] pickerFields = NumberPicker.class.getDeclaredFields();
        for (Field pf : pickerFields) {
            if (pf.getName().equals("mSelectionDivider")) {
                pf.setAccessible(true);
                try{
                    pf.set(mNumberPicker,new ColorDrawable(Color.BLUE));
                }catch (IllegalAccessException e) {
                    e.printStackTrace();
                }catch (Resources.NotFoundException e) {
                    e.printStackTrace();
                }catch (IllegalArgumentException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 过反射改变文字的颜色
     * @param numberPicker
     * @param color
     * @return
     */
    public static boolean setNumberPickerTextColor(NumberPicker numberPicker, int color)
    {
        final int count = numberPicker.getChildCount();
        for(int i = 0; i < count; i++){
            View child = numberPicker.getChildAt(i);
            if(child instanceof EditText){
                try{
                    Field selectorWheelPaintField = numberPicker.getClass()
                            .getDeclaredField("mSelectorWheelPaint");
                    selectorWheelPaintField.setAccessible(true);
                    ((Paint)selectorWheelPaintField.get(numberPicker)).setColor(color);
                    ((EditText)child).setTextColor(color);
                    numberPicker.invalidate();
                    return true;
                }
                catch(NoSuchFieldException e){
                    Log.w("setTextColor", e);
                }
                catch(IllegalAccessException e){
                    Log.w("setTextColor", e);
                }
                catch(IllegalArgumentException e){
                    Log.w("setTextColor", e);
                }
            }
        }
        return false;
    }
}
```

以上就是NumberPicker的简单使用了，可以用它实现省市区联动(3个NumberPicker)，也可以实现充值打折金额滑动选择(比如充100(实付:90))等等.通过反射改变分割线的方法是在stackoverflow上看到的：
改变分割线颜色: http://stackoverflow.com/questions/24233556/changing-numberpicker-divider-color
改变文字颜色: http://stackoverflow.com/questions/22962075/change-the-text-color-of-numberpicker
