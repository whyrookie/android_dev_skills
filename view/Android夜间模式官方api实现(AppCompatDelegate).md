## Android夜间模式官方api实现(AppCompatDelegate)

Android夜间模式可以通过手动设置不同的Theme来实现，也有第三方框架可拿来用，Api 23.0.0后可以使用AppCompatDelegate来实现夜间模式切换。

首先我们需要在style(res/values/style.xml)中生成我们需要主题:

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.DayNight">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>

```
这里我们的主题继承了Theme.AppCompat.DayNight，这个主题是一个family,还包含一些额外的主题：
Theme.AppCompat.DayNight.DarkActionBar、Theme.AppCompat.DayNight.NoActionBar等等.都可以使用在夜间模式中.一个夜间，一个白天，显然需要两套资源方案，我们这里设置的只是日间模式(不知道这个表达对不对)下的主题，包括颜色也是日间模式所需要的(\res\values\colors.xml):

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#3F51B5</color>
    <color name="colorPrimaryDark">#303F9F</color>
    <color name="colorAccent">#FF4081</color>

    <color name="textColor">#f45d5d</color>
</resources>
```

接下来需要提供一套夜间模式的资源，可以在/res/下生成一个values-night资源文件夹,
当我们使用代码设置夜间模式时，系统会自动引用这个文件夹下的资源.该文件夹下的资源文件名和/res/values/中的文件名保持一致，包括资源Id也需一致，如下：

/res/values-night/style.xml:
```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.DayNight">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>

```
/res/values-night/color.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#141212</color>
    <color name="colorPrimaryDark">#423737</color>
    <color name="colorAccent">#2e41ea</color>
    <color name="textColor">#4861ea</color>
</resources>

```
其他资源文件的设置也是如此，系统会在特定的模式下，自动识别,这里的颜色我是自己配置的，其实更好的方式是使用系统自带的颜色方案:?android:attr/textColor、?android:attr/textColorPrimary等，避免自己配置的颜色不统一，比如文字颜色和背景色相近，文字就看不清楚了，接下来就是代码的动态设置。

AppCompatDelegate:

AppCompatDelegate有四种模式可以设置:

MODE_NIGHT_YES:直接指定夜间模式

MODE_NIGHT_NO：直接指定日间模式

MODE_NIGHT_FOLLOW_SYSTEM:根据系统设置决定是否设置夜间模式

MODE_NIGHT_AUTO:根据当前时间自动切换模式

如果要在app启动的时候就设置夜间模式，可以将代码写在Application中：

```java
public class MyApplication extends Application {

    //也可以直接使用代码块直接设置
    //    static {
  //                AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_ NIGHT_YES);
  //    }
    @Override
    public void onCreate() {
        super.onCreate();

        //根据app上次退出的状态来判断是否需要设置夜间模式,提前在SharedPreference中存了一个是否是夜间模式的boolean值
        boolean isNightMode = NightModeConfig.getInstance().getNightMode(getApplicationContext());
        if (isNightMode) {
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);
        }else {
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO);
        }
    }
}

```

一般我们都是给用户提供一个选项来选择是否要启用夜间模式，写一个Activity，提供设置功能,布局:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/tv_message"
        android:layout_marginLeft="20dp"
        android:layout_marginTop="50dp"
        android:gravity="center"
        android:layout_alignParentLeft="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我的消息"
        android:textColor="@color/colorAccent"
        android:textSize="25sp"/>

    <TextView
        android:id="@+id/tv_book"
        android:layout_alignLeft="@id/tv_message"
        android:layout_below="@id/tv_message"
        android:layout_marginLeft="20dp"
        android:layout_marginTop="50dp"
        android:gravity="center"
        android:layout_alignParentLeft="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/colorAccent"
        android:text="我的书籍"
        android:textSize="25sp"/>

    <TextView
        android:id="@+id/tv_money"
        android:layout_alignLeft="@id/tv_message"
        android:layout_below="@id/tv_book"
        android:layout_marginLeft="20dp"
        android:layout_marginTop="50dp"
        android:gravity="center"
        android:layout_alignParentLeft="true"
        android:layout_width="wrap_content"
        android:textColor="@color/colorAccent"
        android:layout_height="wrap_content"
        android:text="我的金币"
        android:textSize="25sp"/>

    <TextView
        android:id="@+id/tv_market"
        android:layout_alignLeft="@id/tv_message"
        android:layout_below="@id/tv_money"
        android:layout_marginLeft="20dp"
        android:layout_marginTop="50dp"
        android:gravity="center"
        android:layout_alignParentLeft="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/colorAccent"
        android:text="图书市场"
        android:textSize="25sp"/>

    <RelativeLayout
        android:layout_alignLeft="@id/tv_message"
        android:layout_below="@id/tv_market"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="50dp">
        <TextView
            android:id="@+id/tv_mode"
            android:gravity="center"
            android:layout_alignParentLeft="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="夜间模式"
            android:textColor="@color/colorAccent"
            android:textSize="25sp"/>

    </RelativeLayout>
</RelativeLayout>

```

代码设置，点击文字"夜间模式"的时显示效果
```java
package com.example.why.traing2;

import android.content.Intent;
import android.content.res.Configuration;
import android.support.v4.text.TextDirectionHeuristicCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.app.AppCompatDelegate;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class FirstActivity extends AppCompatActivity {

    private TextView mNightModeTv;
    private Button mNextBtn;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_first);

        //AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);，如果直接在onCreate()中调用，FirstActivity直接生效，无需调用recreate()
        mNextBtn = (Button)findViewById(R.id.btn_next);

        mNightModeTv = (TextView)findViewById(R.id.tv_mode);

        mNightModeTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //获取当前的模式，设置相反的模式，这里只使用了，夜间和非夜间模式
                int currentMode = getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
                if (currentMode != Configuration.UI_MODE_NIGHT_YES) {
                   //保存夜间模式状态,Application中可以根据这个值判断是否设置夜间模式
                    AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);

                    //ThemeConfig主题配置，这里只是保存了是否是夜间模式的boolean值
                    NightModeConfig.getInstance().setNightMode(getApplicationContext(),true);
                } else {
                    AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO);
                    NightModeConfig.getInstance().setNightMode(getApplicationContext(),false);
                }

                recreate();//需要recreate才能生效
            }
        });

    }
}

```
currentMode取值:

```java
int currentNightMode = getResources().getConfiguration().uiMode
        & Configuration.UI_MODE_NIGHT_MASK;
switch (currentNightMode) {
    case Configuration.UI_MODE_NIGHT_NO://夜间模式未启用
    case Configuration.UI_MODE_NIGHT_YES://夜间模式启用
    case Configuration.UI_MODE_NIGHT_UNDEFINED://不确定是哪种模式，假设不是夜间模式
}

```

```java
package com.example.why.traing2;

import android.content.Context;
import android.content.SharedPreferences;

public class NightModeConfig {
    private SharedPreferences mSharedPreference;
    private static final String NIGHT_MODE = "Night_Mode";
    public static final String IS_NIGHT_MODE = "Is_Night_Mode";

    private boolean mIsNightMode;

    private  SharedPreferences.Editor mEditor;

    private static NightModeConfig sModeConfig;

    public static NightModeConfig getInstance(){

        return sModeConfig !=null?sModeConfig : new NightModeConfig();
    }

    public boolean getNightMode(Context context){

        if (mSharedPreference == null) {
            mSharedPreference = context.getSharedPreferences(NIGHT_MODE,context.MODE_PRIVATE);
        }
        mIsNightMode = mSharedPreference.getBoolean(IS_NIGHT_MODE, false);
        return mIsNightMode;
    }

    public void setNightMode(Context context, boolean isNightMode){
        if (mSharedPreference == null) {
            mSharedPreference = context.getSharedPreferences(NIGHT_MODE,context.MODE_PRIVATE);
        }
        mEditor = mSharedPreference.edit();

        mEditor.putBoolean(IS_NIGHT_MODE,isNightMode);
        mEditor.commit();
    }
}

```

这样就可以根据用户的喜好来设置是否启用夜间模式,注意点:

1. AppCompatDelegate类中还有一个方法：setLocalNightMode(int mode)，这个方法作用在当前组件，Activity中使用 getDelegate().setLocalNightMode(AppCompatDelegate.MODE_NIGHT_YES)设置。比如在Application中设置为日间模式，而在当前Activity中调用getDelegate().setLocalNightMode(AppCompatDelegate.MODE_NIGHT_YES),该Activity会显示为夜间模式,覆盖原来的模式。
2.  AppCompatDelegate.setDefaultNightMode(int mode);只作用于新生成的组件，对原先处于任务栈中的Activity不起作用。如果直接在Activity的onCreate()中调用这句代码，那当前的Activity中直接生效，不需要再调用recreate(),但我们通常是在监听器中调用这句代码，那就需要调用recreate()，否则对当前Activity无效(新生成的Activity还是生效的)。当然可以提前在onSaveInstanceState(Bundle outState, PersistableBundle outPersistentState)中保存好相关属性值,重建时使用。

参考:
https://medium.com/@chrisbanes/appcompat-v23-2-daynight-d10f90c83e94

http://wuxiaolong.me/2016/07/12/appcompatDayNight/
