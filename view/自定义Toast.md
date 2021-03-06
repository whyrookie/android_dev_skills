## 自定义Toast

平时一般只用默认的Toast，使用Toast.makeTest()方法调用，默认的风格是白字半透明灰框，经常与app的主题颜色不符，所以需要自定义Toast．效果图：

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/toast.jpg)


显示定义需要的布局文件:layout/custom_toast.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/custom_toast_container"
            android:orientation="horizontal"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:padding="8dp"
            android:background="#DAAA"
            >
  <ImageView android:src="@drawable/droid"
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:layout_marginRight="8dp"
             />
  <TextView android:id="@+id/text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="#FFF"
            />
</LinearLayout>

```

注意root view 的Ｉ必须设置id(接下来代码用到,此时为custom_toast_container)

接下来是kotlin代码(根据android官网，手动将Java代码改为Kotliin代码)

```Java

class ToastActivity:AppCompatActivity(){


  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      var layoutInflater:LayoutInflater = layoutInflater


      //这里需要一个安全类型转换as?, 不然编译无法通过，因为ViewGroup是not null类型，
      //而findViewById(R.id.custom_toast_container)可能为null,所以不能直接用as
      var layout: View = layoutInflater.inflate(R.layout
              .custom_toast, findViewById(R.id.custom_toast_container) as? ViewGroup)

      val text:TextView = layout.findViewById(R.id.text) as TextView
      text.setText("This is a custom toast")

      //这个如果是java语法，则需要调用getApplicationContext,
      val toast = Toast(applicationContext)
      toast.setGravity(Gravity.CENTER_VERTICAL, 0, 300)//设置位置
      toast.duration = Toast.LENGTH_LONG
      toast.view = layout//java:toast.setView(layout);
      toast.show()
  }
}

```
代码都是官方文档拿过来的，里面是Java代码：
https://developer.android.com/guide/topics/ui/notifiers/toasts.html#CustomToastView
