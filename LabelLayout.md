## LabelLayout

### 简介

这是一个给组件添加`Label`标签的`Layout`，应该是经常会遇见的需求。

借鉴了这位童靴在`GitHub`上的代码：[GitHub](https://github.com/luxiliu/LabelLayout) 

给了我灵感，并借鉴了实现方式。

之前一直在想，这种加`Label`的需求能否用一种通用的方式满足。看到这位童靴的仓库，茅塞顿开，立刻着手实现。

### 效果

先看下这个`Layout`跑起来的效果如何：


![Label Demo](http://wx4.sinaimg.cn/mw690/e1178d99gy1fbv11ovbkag20dw0oie8f.gif)

Label的位置有4种，`LEFT_TOP`, `LEFT_BOTTOM`, `RIGHT_TOP`, `RIGHT_BOTTOM`，顾名思义即可。

Label的高度以及距顶点的距离都是可以设置的。

Label Text的内容、大小、颜色也都可以设置。

还有一种Style:Dot:

![Dot Demo](http://wx4.sinaimg.cn/mw690/e1178d99ly1fbv1gwxqrij203b0393ya.jpg)

同样，红点有四种位置，和上述一致。

红点的颜色，半径，距离右上角（或右下角，左上角，左下角）都可以设置。

### 使用方式

在你需要设置Label的组件外围包含一层LabelLayout。就像下面这样：

      <com.jay.customview.widgets.LabelLayout
            app:dot_color="@color/classic_red"
            app:label_style="label"
            android:id="@+id/label_Layout"
            app:label_background_color="@color/bg_color_red_dark"
            app:label_y="70dp"
            app:text="爆款热卖"
            app:label_height="30dp"
            app:label_location="left_top"
            app:dot_margin_left="3dp"
            app:dot_margin_top="3dp"
            app:dot_radius="3dp"
            android:layout_margin="20dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
    
            <ImageView
                android:src="@drawable/icon_status_weight"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
    
        </com.jay.customview.widgets.LabelLayout>

解释下一些自定义属性的含义：

* dot_color: dot的颜色，很简单

* label_background_color: label的背景颜色，GIF图中不断变化的颜色就是。
 
* label_y: 怎么解释这个属性呢，label距离相对应顶点的最远距离。大概是这样，这个值越大，label距离顶点就越远。就是图中的Distance。
 
* text: 这个就不用解释了，就是Label上显示的文字。对，你写啥它就显示啥。你写"爆款热卖"还是"跳楼甩卖"，随你。
 
* label_height: 也很简单，Label的高度，GIF中调的Height就是。
 
* label_location: Label或Dot摆放的位置。四种选择，上面已经说过，不再赘述。

* dot_margin_left: dot和顶点水平方向的距离，恩，是这样的。
 
* dot_margin_top: dot和顶点垂直方向上的距离，恩，也是这样的。
 
* dot_radius: Dot的半径。
 
应该还有一些没有说到，不过顾名思义应该能看明白。


### 实现原理

其实就一句话，LabelLayout继承自FrameLayout。

也就是说，除了可以添加Label，其它和FrameLayout没有区别。

在dispatchDraw(Canvas canvas)方法中画label、text、dot。

PS:在自定义View时，我们会在onDraw方法中画东西，在自定义ViewGroup时，我们需要再dispatchDraw方法中画东西。

#### 画LabelBackground

画Label背景时，首先画一个平行于X轴的矩形，然后将canvas旋转45或-45度，就实现边角Label的效果。

需要注意的是，Canvas旋转之前要保存，旋转完再恢复Canvas：

    canvas.save();
    canvas.rotate(-45, 0, mLabelY);
    canvas.drawRect(getLabelBgRect(), mLabelBgPaint);
    canvas.restore();

#### 画LabelText

画Label上的文字时，需要用到一个方法：

    canvas.drawTextOnPath(mLabelText, mPath, mOffset[0], mOffset[1], mTextPaint);

在Path上画文字，Path将作为文字的BaseLine。那这时又有一个不胜其烦的问题，如何保证文字在Label上居中。

水平居中很简单，关键是垂直居中。

我这里偷了一点懒，直接拿到文字的高度，除以2，作为drawTextOnPath的第四个参数，vOffset。关于文字对齐的问题，有时间单开一篇细讲。

#### 画Dot

画`Dot`就很简单了，`paint.drawCircle(...)`，搞定。
