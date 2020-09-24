---
title: Android判断点击区域
tags:
  - Android
abbrlink: ef258437
date: 2016-12-02 21:26:47
---

最近项目有这么个需求，后台返回一个图片URL，Android通过该URL加载网络图片，同时需要做到点击图片的不同区域，跳转到不同的Activity页面。此时就需要监听View的点击坐标点以及该坐标点所处的区域。

> 需要注意的是，Android中的`Rect`和iOS中的`CGRect`的概念有点不一样，iOS中的CGRect通过`CGRectMake(x, y, width, height)`方法进行创建，而Android中的Rect通过`public Rect(int left, int top, int right, int bottom)`初始化。iOS中的是`x`、`y`、`width`、`height`，对应的是视图的`x坐标点`、`y坐标点`、`视图的宽度值`、`视图的高度值`；而Android中的是`left`、`top`、`right`、`bottom`对应的是视图的`视图左边距坐标点`、`视图顶部边距的坐标点`、`视图右边距的坐标点`、`视图底部边距的坐标点`。Android中的Rect更类似于iOS中的UIEdgeInsets的概念。

## 下面附上源码

#### 布局文件
``` XML
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.wupincheng.myapplication.MainActivity">

    <Button
        android:id="@+id/main_button"
        android:layout_width="800dp"
        android:layout_height="200dp"
        android:text="Button"/>

</RelativeLayout>
```

#### Activity代码
``` Java
public class MainActivity extends AppCompatActivity {

    private int x;
    private int y;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.main_button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                /**
                // 水平N等分,通过将count设置为不同的数值就可达到将View按照水平方向平分为N分
                int count = 3;
                int averageWidth = view.getWidth() / count;
                int height = view.getHeight();

                for (int i=0; i<count; i++) {
                    int left = averageWidth * i;
                    Rect rect = new Rect(left, 0, left+averageWidth, height);
                    if (rect.contains(x, y)) {
                        System.out.println("点击按钮第"+i+"块区域");
                        break;
                    }
                }
                 */

                // 十字四等分
                int averageWidth = view.getWidth() / 2;
                int averageHeight = view.getHeight() / 2;
                for (int i=0; i<4; i++) {
                    int left = averageWidth * (i%2);
                    int top = averageHeight * (i/2);
                    Rect rect = new Rect(left, top, left+averageWidth, top+averageHeight);
                    if (rect.contains(x, y)) {
                        System.out.println("点击按钮第"+i+"块区域");
                        break;
                    }
                }
            }
        });

        findViewById(R.id.main_button).setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                if (motionEvent.getAction() == MotionEvent.ACTION_DOWN) {
                    x = (int)motionEvent.getX();
                    y = (int)motionEvent.getY();
                }
                return false;
            }
        });
    }
}
```
