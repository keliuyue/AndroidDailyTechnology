这里主要分享Android日常使用的一些小技巧

#### 工具类

##### ADB Logcat

使用场景：当App已经签名打包。需要对App出现的问题进行分析处理。可以将电脑连接手机，通过命令行捕获相应的日志信息。然后可以输出到电脑，也可以输出到手机。然后对日志进行分析处理。
```
--"-s"选项 : 设置输出日志的标签, 只显示该标签的日志;

--"-f"选项 : 将日志输出到文件, 默认输出到标准输出流中, -f 参数执行不成功;

--"-r"选项 : 按照每千字节输出日志, 需要 -f 参数, 不过这个命令没有执行成功;

--"-n"选项 : 设置日志输出的最大数目, 需要 -r 参数, 这个执行 感觉 跟 adb logcat 效果一样;

--"-v"选项 : 设置日志的输出格式, 注意只能设置一项;

--"-c"选项 : 清空所有的日志缓存信息;

--"-d"选项 : 将缓存的日志输出到屏幕上, 并且不会阻塞;

--"-t"选项 : 输出最近的几行日志, 输出完退出, 不阻塞;

--"-g"选项 : 查看日志缓冲区信息;

--"-b"选项 : 加载一个日志缓冲区, 默认是 main, 下面详解;

--"-B"选项 : 以二进制形式输出日志;

    第一种事例：adblogcat > /sdcard/mylogcat.txt  保存log文件到sd卡中

    第二种事例：adblogcat > D:/Temp/1.txt
```

#### 四大组件相关

##### Activity
1. Activity成为对话框类型
```
在清单文件中添加
android:theme="@style/DialogActivtyTransparent"
```

2. 获取页面状态栏的高度
```
public static int getStatusHeight(Context context) {

        int statusHeight = -1;
        try {
            Class clazz = Class.forName("com.android.internal.R$dimen");
            Object object = clazz.newInstance();
            int height = Integer.parseInt(clazz.getField("status_bar_height").get(object).toString());
            statusHeight = context.getResources().getDimensionPixelSize(height);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return statusHeight;
 }
```
#### 常用控件

##### TextView

1. 设置自定义ttf字体
```
1.在assets目录下新建fonts目录，把ttf字体文件放到该目录下
2.正常使用
//实例化TextView
TextView textView = findViewById(R.id.textview);
//得到AssetManager
AssetManager mgr = getAssets();
//根据路径得到Typeface
Typeface tf = Typeface.createFromAsset(mgr, "fonts/pocknum.ttf");
//设置字体
textView.setTypeface(tf);
```
##### EditText

1. EditText在代码中设置最大输入长度并设置为密码输入框

```
//输入类型
editText.setInputType(InputType.TYPE_CLASS_NUMBER);
//最大输入长度
editText.setFilters(new InputFilter[]{new InputFilter.LengthFilter(6)});
//设置为密码输入框 EditText在代码中设置最大输入长度并设置为密码输入框
editText.setTransformationMethod(PasswordTransformationMethod.getInstance());
```

##### ListView

1.ListView中position的计算

```
// 在ListView中添加头部和尾部
mListView.addFooterView(footer);
mListView.addHeaderView(header);

例如上面这样的：ListView的position + 2
当position = 0时，为header
当position = mListView.size()-1，为footer
其他情况为正常ListView中列表项
```
##### TabLayout

1. 修改TabLayout下划线长度
```
public void setIndicator (TabLayout tabs,int leftDip,int rightDip){
   Class<?> tabLayout = tabs.getClass();
   Field tabStrip = null;
   try {
       tabStrip = tabLayout.getDeclaredField("mTabStrip");
   } catch (NoSuchFieldException e) {
       e.printStackTrace();
   }

   tabStrip.setAccessible(true);
   LinearLayout llTab = null;
   try {
       llTab = (LinearLayout) tabStrip.get(tabs);
   } catch (IllegalAccessException e) {
       e.printStackTrace();
   }

   int left = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, leftDip, Resources.getSystem().getDisplayMetrics());
   int right = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, rightDip, Resources.getSystem().getDisplayMetrics());

   for (int i = 0; i < llTab.getChildCount(); i++) {
       View child = llTab.getChildAt(i);
       child.setPadding(0, 0, 0, 0);
       LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(0, LinearLayout.LayoutParams.MATCH_PARENT, 1);
       params.leftMargin = left;
       params.rightMargin = right;
       child.setLayoutParams(params);
       child.invalidate();
   }
}
```

**使用**

```
tab.post(new Runnable() {
       @Override
       public void run() {
           setIndicator(tab,60,60);
       }
});
```

参考： [android-tablayout-custom-indicator-width](https://stackoverflow.com/questions/45715737/android-tablayout-custom-indicator-width)

2. 取消TabLayout与ViewPager嵌套时页面切换动画
```
mTabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {

        @Override
        public void onTabSelected(TabLayout.Tab tab) {
            // 默认切换的时候，会有一个过渡动画，设为false后，取消动画，直接显示
            mViewPager.setCurrentItem(tab.getPosition(), false);
        }

        @Override
        public void onTabUnselected(TabLayout.Tab tab) {

        }

        @Override
        public void onTabReselected(TabLayout.Tab tab) {

        }
 });
```
##### WebView
1.添加WebView加载动画

**页面正常加载**

```
webView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                //返回值是true的时候控制去WebView打开，为false调用系统浏览器或第三方浏览器
                view.loadUrl(url);
                return true;
            }

            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                // 在页面打开时调用，这里用户可以使用自己的加载动画
                LoadingDialogUtils.createLoadinGdialog(LoanApplyQualificationActivity.this);//开始加载动画
                LoadingDialogUtils.showDialog();
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
                // 当加载结束时移除动画
                LoadingDialogUtils.dismissDialog();
            }
});
```

**使用加载进度条**

需要在xml布局中定义ProgressBar或者其他加载进度相关的控件
```
webView.setWebChromeClient(new WebChromeClient() {
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                if (newProgress == 100) {
                    // 网页加载完成，隐藏进度条
                    mPbProgress.setVisibility(View.GONE);
                } else {
                    // 加载中，设置加载进度
                    mPbProgress.setProgress(newProgress);
                }
                super.onProgressChanged(view, newProgress);
            }
 });
```

##### TextInputLayout

1. TextInputLayout使用注意

```

1.setHint();设置提示语

2.setError();设置错误显示信息

3.setErrorEnabled();设置错误信息是否显示。true显示，false不显示。

4.getEditText();得到EditText的控件实例。
```

##### RatingBar

1. 使RatingBar无法通过点击修改具体显示的星值
```
1.屏蔽星级选择  ,通过监听OnTouchListener
holder.rbGrade.setOnTouchListener(new OnTouchListener() {
   @Override
   public boolean onTouch(View arg0, MotionEvent arg1) {
       // TODO Auto-generated method stub
       return true;
   }
});
2.通过设置isIndicator属性
android:isIndicator="true"
```
##### 其他

1. 只允许一个子控件的容器控件
```
1.ScrollView
2.NestedScrollView
3.TextInputLayout
```

#### 网络请求

#### 存储

##### SharedPreferences

1. SharedPreferences中commit()和apply()异同

```
1. apply没有返回值而commit返回boolean表明修改是否提交成功
2. apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。
3. apply方法不会提示任何失败的提示。
由于在一个进程中，SharedPreferences是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。
```

#### 数据处理

#### 布局使用

1. 添加水波纹效果

**简单使用**
```
// 有界水波纹，可以设置背景，也可以设置前景
android:foreground="?android:attr/selectableItemBackground"
android:background="?android:attr/selectableItemBackground"
// 无界水波纹，可以设置背景，也可以设置前景
android:foreground="?android:attr/selectableItemBackgroundBorderless"
android:background="?android:attr/selectableItemBackgroundBorderless"
```

**自定义效果**

需要在drawable目录下，创建ripple节点，然后进行自定义处理。这里只使用很简单的方式
```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android" android:color="@color/colorAccent">

</ripple>
```

2. Android阻尼效果的实现
```
增加这个效果，方法如下：
1、开启overScrollMode为always
在布局中 android:overScrollMode="always"
或在代码中 setOverScrollMode(View.OVER_SCROLL_ALWAYS);
2、继承listview 覆盖overScrollBy方法，并且利用反射机制修改阴影效果为透明
```

#### 工具

1. MarkDown转换为pdf
```
https://www.typora.io/
http://www.markdowntopdf.com/
```

2.doc文件转换为pdf
```
1.https://smallpdf.com/cn/word-to-pdf
```