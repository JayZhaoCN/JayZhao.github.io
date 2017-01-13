

### 前言 

在调研`Material Design`时，首先是学习了很多`Library`，有`Google`官方的，也有`GitHub`上开源的。

这些库实现了非常好看的MD风格控件，布局和交互效果。

所以先列举一些比较好的`Library`，探究它们的使用方法和实现原理，并尝试自己实现。

之后是调研了采用`Material Design`风格的国产`App`。模仿其`UI`写一些`Demo`。

最后是自己实现了一个`GitHub`上一个开源库：`FloatingActionMenu`。

另外，期间踩过无数坑，也一并写出来，希望看过这篇分享的同学可以避开这些坑。


### Android Design Support Library

`Google I/O 2015`上发布了`Android Design Support Library`，向下支持到`Android 2.2`，非常良心。

提供了非常多MD风格的控件，布局：

##### Snackbar

###### 简介

`Snackbar`很类似于`Toast`，但和`Toast`不一样的一点是，`Snackbar`是可以点击的。大概是长这样的：

![Snackbar](http://wx4.sinaimg.cn/mw690/e1178d99ly1fbgpgr3w7jj21kw07cjxe.jpg)

左边是`Snackbar`显示的文字，右边是`Action`，`Action TextColor`是根据`colorAccent`来的。

一般显示在屏幕的底部，显示一段时间会自动消失。


###### 使用方法：

    Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                .setAction("Action", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        //TODO
                    }
                }).show();


###### Custom Snackbar

观察源码可以知道，`Snackbar`中显示的是一个`SnackbarLayout`，继承自`LinearLayout`。

通过`Snackbar.getView()`就可以拿到这个`Layout`，可以向里面添加其他组件，实现可定制的`Snackbar`。

但是不建议这样做，使用`Snackbar`的初衷就是提供一个非常轻量级的交互，如果在里面添加过多的内容，就失去了

`Snackbar`原有的意义。


#### FloatingActionButton

`FloatingActionButton`是一个悬浮于界面之上的`Button`，一般与`CoordinatorLayout`配合使用。

![FloatingActionButton](http://wx2.sinaimg.cn/mw690/e1178d99ly1fbgqb4euiaj207i07l746.jpg)

`Fab`可能会跟随界面的变化而变化，例如界面上滑`Fab`显示，下滑`Fab`消失，

或`Snackbar`显示时，`Fab`上移为其腾出位置等，这些都可以使用`Behavior`实现。

在上一篇分享[Behavior--一种为View添加动态依赖的优雅方式](https://dzn.huami.com/?p=108)中有详细叙述。

#### TabLayout

点击`Tab`切换`View`其实不是新概念了，但`TabLayout`是`Google`做的一个规范。一般`TabLayout`会和`ViewPager`结合来使用。

    ViewPager pager = (ViewPager) findViewById(R.id.container);
    MyPagerAdapter adapter = new MyPagerAdapter(getSupportFragmentManager());
    pager.setAdapter(adapter);

    TabLayout layout = (TabLayout) findViewById(R.id.tab_layout);
    layout.setupWithViewPager(pager);

    for(int i=0; i<3; i++) {
        TabLayout.Tab tab = layout.getTabAt(i);
        if(tab != null) tab.setCustomView(adapter.getTabView(i));
    }

使用`TabLayout.setupWithViewPager(ViewPager);`将`TabLayout`和`ViewPager`关联起即可。

![FloatingActionButton](http://wx1.sinaimg.cn/mw690/e1178d99ly1fbgrce9hr5j20zy0qogni.jpg)

每个Tab显示的内容可以通过如下两种方式设置：

    TabLayout.Tab tab = tablayout.getTabAt(i);
    tab.setText("text");
    tab.setIcon(R.drawable.some_icon);
    
    //自定义布局
    TabLayout.Tab tab = tablayout.getTabAt(i);
    tab.setCustomView(customView);

还有很多控件，类似`NavigationView`等，就不再一一叙述了。


### Material Menu

![Material Menu](http://cms.csdnimg.cn/article/201411/21/546f0b8672e44.jpg)

这个库实现了一个非常酷炫的菜单、返回、删除以及检查按钮变形，完全控制动画，

并为开发者提供了两种`MaterialMenuDrawable`包装。

[GitHub地址](https://github.com/balysv/material-menu)

我自己也尝试模仿该库写了一个[Material Menu](https://dzn.huami.com/?p=106)，但提供的变形动画没有原版丰富，有时间会考虑增加动画效果。


### FloatingActionMenu

![FloatingActionMenu](https://github.com/futuresimple/android-floating-action-button/raw/master/screenshots/menu.gif)


[GitHub地址](https://github.com/futuresimple/android-floating-action-button)

实现了`FloatingActionButton`的扩展。

折叠时和普通的`Fab`没有区别，但展开后，可以提供更多用户操作接口。

这种控件在很多`App`上都有使用，知乎、印象笔记等都有类似的控件。

但感觉这个库提供的接口不太友好，所以自己动手写了一个`PopLayout`，功能上基本类似（踩了好多坑）:


限于篇幅，还有看过的很多好的MD库就不一一介绍了。


### 仿知乎首页的一个`Demo`

国产`App`中，遵循`Material Design`的不多，知乎是一个。

对比一下真实的知乎和`Demo`的效果：


#### 真实效果

![zhihu](http://ww4.sinaimg.cn/mw690/e1178d99gw1fbgw9u6yqvg20b40jlhdu.gif)


#### Demo效果

![Demo](http://ww4.sinaimg.cn/mw690/e1178d99gw1fbgzg8uw0wg20b40jl7wh.gif)


###### 分析

知乎主界面其实不太复杂。

主体是一个可以滑动的控件，可以用`RecyclerView`实现。

上方是一个标题栏，可以用`AppBarLayout`嵌套`Toolbar`实现。

可以看到，`RecyclerView`上滑，标题栏会隐藏；下滑，标题栏又会出现。

为`Toolbar`设置属性：`app:layout_scrollFlags="scroll|enterAlways"`

表示标题栏可以滚动出屏幕，且向下滑动就会出现。

右下方是一个`FloatingActionButton`，`RecyclerView`上滑`fab`消失，下滑`fab`出现。可以为`fab`设置自定义的`Behavior`实现。

下方直接使用一个`LinearLayout`来做。上滑隐藏，下滑出现同样使用`Behavior`实现。


### PopLayout


这是我仿照开源库`FloatingActionMenu`写的一个控件。上面仿知乎Demo中右下角的控件就是用`PopLayout`实现的。

未展开时，和普通的`FloatingActionButton`一样，展开后，会显示更多信息。

###### 未展开：

![PopLayout](http://wx3.sinaimg.cn/mw690/e1178d99ly1fbh4ngj3jcj204v05lmx0.jpg)

###### 展开：

![PopLayout](http://wx4.sinaimg.cn/mw690/e1178d99ly1fbgvuu4r2sj20bx0ijt97.jpg)

提供两种展开动画，一种是向上弹出，一种是GIF上展示的，向两边展开。（脑补一下）


#### 写MD风格控件需要注意的几点：

###### 1.阴影

MD为组件引入了海拔（`Evelation`）的概念。之前一个控件在屏幕上展示的位置由X、Y坐标决定，现在相当于加入了Z轴的概念。

一个组件Z坐标，也就是高度，虽然不能决定其显示位置，但会决定这个控件在屏幕上投影的大小。

高度越大，投影就越大。我现在要做的这个控件，学名叫`FloatingActionMenu`，它是漂浮在界面之上的，是高于屏幕的。

或者准确点说，是高于其父布局的。所以必然会产生一个投影，绘制时，是需要考虑这个投影的。

这个投影如何画出来，我目前采用的方法是为`Paint`画笔设置一个`ShadowLayer`，绘制时会产生一个阴影层，类似投影的效果，如下：

    mShadowPaint.setShadowLayer(10, 0, 5, ContextCompat.getColor(mContext, R.color.black_30_percent));
    
效果很好：

![Shadow](http://ww3.sinaimg.cn/mw690/e1178d99gw1fbgzzfzlarj203c031t8l.jpg)

可以看到下方有一层淡淡的阴影。

使用这种方法在`Api 11`以上需要禁用硬件加速：

     if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        setLayerType(LAYER_TYPE_SOFTWARE, null);
     }

`GitHub`上有相关的库，也是使用这种方法设置阴影的。

###### 2.Ripple Effect（水波纹效果）

系统自带控件是默认具有水波纹效果的，但`CustomView`就要自己实现了。

其实也很简单，只需要在点击控件时启动一个`ValueAnimator`去画一个半径逐渐增大的圆即可。

需要注意的一点是，画的圆不能超过`CustomView`的边界。所以要用到`PrterDuff.Mode.SRC_ATOP`。

(PS: 关于`PrterDuff`，要说清楚，可能单独讲。)

`GitHub`上有专门的库实现MD阴影和`Ripple Effect`。其实更好的写法是写一个`View`，实现这两种效果，需要有这些效果的继承自这个`View`即可。

###### 踩过的坑

`View`的显示位置：

有一个公式：

    x = left + translationX

    y = top + translationY


`left`, `top`, `right`, `bottom`是`View`的真实位置，`x`, `y`是`View`的显示位置，`translationX`和`translationY`是平移距离。

在`ViewGroup`的`onLayout()`方法中，可能的一种情景是某个`ChildViewA`的位置在`ChildViewB`的左（右）边多少`dp`，

这时需要拿到`ChildViewB`的位置，来计算`ChildViewB`的位置。

应该使用的是`ChildViewB.getLeft()`或`getRight()`方法而不能使用`ChildViewB.getX()`方法，后者将平移距离也计算在内了。

当时遇到的`Bug`是，每次开关屏幕都会导致`PopLayout`里面的`ChildView`显示位置不正确。

因为关屏再开屏会导致`ViewGroup`的一次`invalidate`，这时`onLayout()`会再执行一遍，

因为我执行过平移了，所以`x`和`left`已经不相等了，这时再使用`x`来布局，就会出现问题。


### 最后

代码已上传至`UIComponents`，`Demo`在我的`GitHub`上：[CoordinatorDemo](https://github.com/JayZhaoCN/CoordinatorDemo)




