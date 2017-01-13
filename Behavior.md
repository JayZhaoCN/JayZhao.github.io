Behavior
========

###CoordinatorLayout

首先要说`CoordinatorLayout`.

`android`官方的说法：

    CoordinatorLayout is a super-powered FrameLayout.

    CoordinatorLayout is intended for two primary use cases:

    As a top-level application decor or chrome layout
    As a container for a specific interaction with one or more child view
  
 大致意思是说，`CoordinatorLayout`是一种`Super FrameLayout`,`FrameLayout`的加强版。
 
 它的两种主要用途是：
 
 1.作为顶层`Layout`.
 
 2.作为一个或多个子控件之间特殊交互的容器。
 
 `CoordinatorLayout`和`Behavior`的组合让你有机会以非侵入方式为`View`添加动态的依赖布局，和处理父布局(`CoordinatorLayout`)滑动手势的机会。
 
###FloatingActionButton
 
 举个例子，`fab`是浮动在界面之上的一个`button`，`fab`默认关联了一个`Behavior`:`FloatingActionButton.Behavior`
 
 使用如下方式添加关联：
 
 在类的开始添加：
 
 `@CoordinatorLayout.DefaultBehavior(FloatingActionButton.Behavior.class)`
 
 也可以在`xml`中添加：
 
 `app:layout_behavior=".MyBehavior"`
 
 `CoordinatorLayout`中的一个组件可以监听另一个组件的变化，并对该变化做响应。利用的就是`Behavior`。
 
###Behavior用法
 
 1.自定义一个`CustomBehavior`继承`Behavior`
 
 复写`layoutDependsOn`方法，该方法决定哪些组件是我需要去监听依赖的。比如说我现在要去监听一个`RecyclerView`的变化，可以这样写：
 
    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, FloatingActionButton child, View dependency) {
        return dependency instanceof RecyclerView;
    }
 
 `parent`是最上层的`CoordinatorLayout`，`child`是需要注册`Behavior`的组件，`dependency`是被监听的组件。
 
 我这里做的操作是只有当`dependency`是`RecyclerView`时，才去响应。
 
 2.筛选完了`dependency`,这时候就需要对`dependency`的变化做出响应了：
 
    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, FloatingActionButton child, View dependency) {
        //TODO
    }
   
 在`onDependentViewChanged`方法中做上述操作。
 
 有`parent`、`child`和`dependency`三者的引用，你可以做你想做的任何事情。
 
 需要注意的一点是：
 
    public MyBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    
 `Behavior`的构造函数一定不能少。
 
 具体原因可以追溯到`CoordinatorLayout`的源码中：
 
      Constructor<Behavior> c = constructors.get(fullName);
      if (c == null) {
          final Class<Behavior> clazz = (Class<Behavior>) Class.forName(fullName, true,
                  context.getClassLoader());
          c = clazz.getConstructor(CONSTRUCTOR_PARAMS);
          c.setAccessible(true);
          constructors.put(fullName, c);
      }
      return c.newInstance(context, attrs);
      
可以看到，在`parseBehavior`方法中,直接通过`Behavior`的类名反射调用了它的构造方法，如果没有写构造方法，肯定会报错。

到这里，基本就实现了Behavior的基础功能。

###仿知乎fab效果。

知乎android版本左下角有一个`fab`,上滑会消失，下滑又会出现。这个效果用`Behavior`来实现，再好不过。

我们用`RecyclerView`来做主界面的滑动组件。这时候需要去监听`RecyclerView`的滑动，需要复写`Behavior`的两个方法：

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View directTargetChild, View target, int nestedScrollAxes) {
        //true if the Behavior wishes to accept this nested scroll
        return true;
    }
    
    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
    }

 解释一下这两个方法。第一个要`return true`，表示`Behavior`接受了这次滑动事件。
 
 第二个方法的前三个参数不用赘述，和前面提到的一样。后四个参数分别指的是
 
 `dxConsumed`：`x`方向滑动距离
 
 `dyConsumed`：`y`方向滑动距离
 
 `dxUnConsumed`：`x`方向未消费距离
 
 `dyUnConsumed`：`y`方向未消费距离
 
 （其实我也不知道这个未消费是什么意思，总的滑动距离减去已经滑动的距离？但了解前面两个已经可以做我们的需求了。）
 
 （这里做一个补充，2016.12.22在做仿知乎Demo时，发现。未消费指的是这段滑动距离没有体现在滑动控件上，比如上滑、下滑到头了，这时候
 
  滑动的距离是不能反映在滑动控件上的，故用UnConsumed来表示。）
 
 `dyConsumed`大于0，表示上滑，这时需要隐藏fab；`dyConsumed`小于0，表示下滑，这时需要展现`fab`。原理理清了，接下来就是实现了：
 
    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed);

        if(dyConsumed > 0) {
            //up disappear
            if(!isHided) {
                //total translationX distance
                animatorWidth = coordinatorLayout.getWidth() - child.getX();
                //ObjectAnimator
                mAnimator = ObjectAnimator.ofFloat(child, "translationX", 0, animatorWidth);
                mAnimator.setDuration(500);
                mAnimator.setInterpolator(new DecelerateInterpolator(3));
                mAnimator.start();
                isHided = true;
            }
        } else {
            //down appear
            if(isHided) {
                mAnimator = ObjectAnimator.ofFloat(child, "translationX", animatorWidth, 0);
                mAnimator.setInterpolator(new DecelerateInterpolator(3));
                mAnimator.setDuration(500);
                mAnimator.start();
                isHided = false;
            }
        }
    }
 
 大功告成。
 
###实现主界面滑动时，Toolbar的展示与消失。

这也是`Android`知乎的一个效果。主界面是一个`RecyclerView`或其他的可滑动的`View`，当`RecyclerView`上滑，`Toolbar`消失，下滑，`Toolbar`显示。

当然，真实的要比上述效果复杂很多。

原理就是将`RecyclerView`的滑动信息传递给`AppBarLayout`，这中间的媒介就是`CoordinatorLayout`。

1.首先，为RecyclerView添加Behavior：

    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    
`@string/appbar_scrolling_view_behavior`是`support library`中一个特殊的字符串，它和`AppBarLayout.ScrollingViewBehavior`相匹配，用来通知

`AppBarLayout`这个特殊的`view`何时发生了滚动事件，这个`behavior`需要设置在触发事件（滚动）的`view`之上。

PS:这里有一点我很难理解，在做`recyclerView`滑动事件引起fab显示和隐藏时，`Behavior`是设置在`fab`上，为什么这里`Behavior`要设置
在`RecyclerView`（触发事件的`View`）上？先占个坑。

做到这里还不够，还需要在需要响应`RecyclerView`滑动的组件上添加如下标识：

    app:layout_scrollFlags="scroll|enterAlways"
    
这里就是`Toolbar`。

解释一下这几个标志位：

`enterAlways`: 一旦向上滚动这个`view`就可见。

`enterAlwaysCollapsed`: 顾名思义，这个`flag`定义的是何时进入（已经消失之后何时再次显示）。

假设你定义了一个最小高度（`minHeight`）同时`enterAlwaysCollapsed`也定义了，那么`view`将

在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。

`exitUntilCollapsed`: 同样顾名思义，这个`flag`时定义何时退出，当你定义了一个`minHeight`，

这个`view`将在滚动到达这个最小高度的时候消失。

`scroll`：只有设置了这个标志位，`View`才可以滚动出屏幕，否则`View`将一直固定在一个位置。

当`CoordinatorLayout`发现`RecyclerView`设置了属性appbar_scrolling_view_behavior，

会遍历所有child view，找到AppBarLayout中设置了scrollFlags属性的控件。

`AppBarLayout.ScrollingViewBehavior`描述了`RecyclerVie`w与`AppBarLayout`之间的依赖关系。

`RecyclerView`的任意滚动事件都将触发`AppBarLayout`或者`AppBarLayout`里面`view`的改变。

