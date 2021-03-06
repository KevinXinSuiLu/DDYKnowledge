> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.flutter123.net/Article/Detail/38

本文主要介绍 Flutter 布局中的 LimitedBox、Offstage、OverflowBox、SizedBox 四种控件，详细介绍了其布局行为以及使用场景，并对源码进行了分析。

1.  LimitedBox  
    A box that limits its size only when it's unconstrained.

1.1 简介  
LimitedBox，通过字面意思，也可以猜测出这个控件的作用，是限制类型的控件。这种类型的控件前面也介绍了不少了，这个是对最大宽高进行限制的控件。

1.2 布局行为  
LimitedBox 是将 child 限制在其设定的最大宽高中的，但是这个限定是有条件的。当 LimitedBox 最大宽度不受限制时，child 的宽度就会受到这个最大宽度的限制，同理高度。

1.3 继承关系  
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > LimitedBox  
1.4 示例代码  
Row(  
children: [  
Container(  
color: Colors.red,  
width: 100.0,  
),  
LimitedBox(  
maxWidth: 150.0,  
child: Container(  
color: Colors.blue,  
width: 250.0,  
),  
),  
],  
)  
1.5 源码解析  
const LimitedBox({  
Key key,  
this.maxWidth = double.infinity,  
this.maxHeight = double.infinity,  
Widget child,  
})  
1.5.1 属性解析  
maxWidth：限定的最大宽度，默认值是 double.infinity，不能为负数。

maxHeight：同上。

1.5.2 源码  
先不说其源码，单纯从其作用，前面介绍的 SizedBox、ConstrainedBox 都类似，都是通过强加到 child 的 constraint，来达到相应的效果。

我们直接看其计算 constraint 的代码

minWidth: constraints.minWidth,  
maxWidth: constraints.hasBoundedWidth ? constraints.maxWidth : constraints.constrainWidth(maxWidth),  
minHeight: constraints.minHeight,  
maxHeight: constraints.hasBoundedHeight ? constraints.maxHeight : constraints.constrainHeight(maxHeight)  
LimitedBox 只是改变最大宽高的限定。具体的布局代码如下：

if (child != null) {  
child.layout(_limitConstraints(constraints), parentUsesSize: true);  
size = constraints.constrain(child.size);  
} else {  
size = _limitConstraints(constraints).constrain(Size.zero);  
}  
根据最大尺寸，限制 child 的布局，然后将自身调节到 child 的尺寸。

1.6 使用场景  
使用场景是不可能清楚了，光是找例子，就花了不少时间。Flutter 的一些冷门控件，真的是除了官方的文档，啥材料都木有。谷歌说这个很有用，还是一脸懵逼。这种控件，也有其他的替代解决方案，LimitedBox 可以达到的效果，ConstrainedBox 都可以实现。

2.  Offstage  
    A widget that lays the child out as if it was in the tree, but without painting anything, without making the child available for hit testing, and without taking any room in the parent.

2.1 简介  
Offstage 的作用很简单，通过一个参数，来控制 child 是否显示，日常使用中也算是比较常用的控件。

2.2 布局行为  
Offstage 的布局行为完全取决于其 offstage 参数

当 offstage 为 true，当前控件不会被绘制在屏幕上，不会响应点击事件，也不会占用空间；  
当 offstage 为 false，当前控件则跟平常用的控件一样渲染绘制；  
另外，当 Offstage 不可见的时候，如果 child 有动画，应该手动停掉，Offstage 并不会停掉动画。

2.3 继承关系  
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Offstage  
2.4 示例代码  
Column(  
children: [  
new Offstage(  
offstage: offstage,  
child: Container(color: Colors.blue, height: 100.0),  
),  
new CupertinoButton(  
child: Text("点击切换显示"),  
onPressed: () {  
setState(() {  
offstage = !offstage;  
});  
},  
),  
],  
)  
当点击切换按钮的时候，可以看到 Offstage 显示消失。

2.5 源码解析  
const Offstage({Key key, this.offstage = true, Widget child})  
2.5.1 属性解析  
offstage：默认为 true，也就是不显示，当为 flase 的时候，会显示该控件。

2.5.2 源码  
我们先来看下 Offstage 的 computeIntrinsicSize 相关的方法：

@override  
double computeMinIntrinsicWidth(double height) {  
if (offstage)  
return 0.0;  
return super.computeMinIntrinsicWidth(height);  
}  
可以看到，当 offstage 为 true 的时候，自身的最小以及最大宽高都会被置为 0.0。

接下来我们来看下其 hitTest 方法：

@override  
bool hitTest(HitTestResult result, { Offset position}) {  
return !offstage && super.hitTest(result, position: position);  
}  
当 offstage 为 true 的时候，也不会去执行。

最后我们来看下其 paint 方法：

@override  
void paint(PaintingContext context, Offset offset) {  
if (offstage)  
return;  
super.paint(context, offset);  
}  
当 offstage 为 true 的时候直接返回，不绘制了。

到此，跟上面所说的布局行为对应上了。我们一定要清楚一件事情，Offstage 并不是通过插入或者删除自己在 widget tree 中的节点，来达到显示以及隐藏的效果，而是通过设置自身尺寸、不响应 hitTest 以及不绘制，来达到展示与隐藏的效果。

2.6 使用场景  
当我们需要控制一个区域显示或者隐藏的时候，可以使用这个场景。其实也有其他代替的方法，但是成本会高很多，例如直接在 tree 上插入删除，但是不建议这么做。

3.  OverflowBox  
    A widget that imposes different constraints on its child than it gets from its parent, possibly allowing the child to overflow the parent.

3.1 简介  
OverflowBox 这个控件，允许 child 超出 parent 的范围显示，当然不用这个控件，也有很多种方式实现类似的效果。

3.2 布局行为  
当 OverflowBox 的最大尺寸大于 child 的时候，child 可以完整显示，当其小于 child 的时候，则以最大尺寸为基准，当然，这个尺寸都是可以突破父节点的。最后加上对齐方式，完成布局。

3.3 继承关系  
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > OverflowBox  
3.4 示例代码  
Container(  
color: Colors.green,  
width: 200.0,  
height: 200.0,  
padding: const EdgeInsets.all(5.0),  
child: OverflowBox(  
alignment: Alignment.topLeft,  
maxWidth: 300.0,  
maxHeight: 500.0,  
child: Container(  
color: Color(0x33FF00FF),  
width: 400.0,  
height: 400.0,  
),  
),  
)  
OverflowBox 例子

当 maxHeight 大于 height 的时候，可以完全显示下来，当 maxHeight 小于 height 的时候，则不会会被隐藏掉

3.5 源码解析  
构造函数如下：

const OverflowBox({  
Key key,  
this.alignment = Alignment.center,  
this.minWidth,  
this.maxWidth,  
this.minHeight,  
this.maxHeight,  
Widget child,  
})  
3.5.1 属性解析  
alignment：对齐方式。

minWidth：允许 child 的最小宽度。如果 child 宽度小于这个值，则按照最小宽度进行显示。

maxWidth：允许 child 的最大宽度。如果 child 宽度大于这个值，则按照最大宽度进行展示。

minHeight：允许 child 的最小高度。如果 child 高度小于这个值，则按照最小高度进行显示。

maxHeight：允许 child 的最大高度。如果 child 高度大于这个值，则按照最大高度进行展示。

其中，最小以及最大宽高度，如果为 null 的时候，就取父节点的 constraint 代替。

3.5.2 源码  
OverflowBox 的源码很简单，我们先来看一下布局代码：

if (child != null) {  
child.layout(_getInnerConstraints(constraints), parentUsesSize: true);  
alignChild();  
}  
如果 child 不为 null，child 则会按照计算出的 constraints 进行尺寸的调整，然后对齐。

至于 constraints 的计算，则还是上面的逻辑，如果设置的有的话，就取这个值，如果没有的话，就拿父节点的。

3.6 使用场景  
有时候设计图上出现的角标，会超出整个模块，可以使用 OverflowBox 控件。但我们应该知道，不使用这种控件，也可以完成布局，在最外面包一层，也能达到一样的效果。具体实施起来哪个比较方便，同学们自行取舍。

4.  SizedBox  
    A box with a specified size.

4.1 简介  
比较常用的一个控件，设置具体尺寸。

4.2 布局行为  
SizedBox 布局行为相对较简单：

child 不为 null 时，如果设置了宽高，则会强制把 child 尺寸调到此宽高；如果没有设置宽高，则会根据 child 尺寸进行调整；  
child 为 null 时，如果设置了宽高，则自身尺寸调整到此宽高值，如果没设置，则尺寸为 0；  
4.3 继承关系  
Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > SizedBox  
4.4 示例代码  
Container(  
color: Colors.green,  
padding: const EdgeInsets.all(5.0),  
child: SizedBox(  
width: 200.0,  
height: 200.0,  
child: Container(  
color: Colors.red,  
width: 100.0,  
height: 300.0,  
),  
),  
)  
4.5 源码解析  
构造函数

const SizedBox({Key key, this.width, this.height, Widget child})  
4.5.1 属性解析  
width：宽度值，如果具体设置了，则强制 child 宽度为此值，如果没设置，则根据 child 宽度调整自身宽度。

height：同上。

4.5.2 源码  
SizedBox 内部是通过 RenderConstrainedBox 来实现的。具体的源码就不解析了，总体思路是，根据宽高值算好一个 constraints，然后强制应用到 child 上。

4.6 使用场景  
这个控件，很多场景可以使用。但是，可以替代它的控件也有不少，例如 Container、ConstrainedBox 等。而且 SizedBox 就是 ConstrainedBox 的一个特例。还是那句话，精简啊，多提供一些常用的，不要提供一大堆重复的，场景很少的控件。

5.  后话  
    笔者建了一个 Flutter 学习相关的项目，Github 地址，里面包含了笔者写的关于 Flutter 学习相关的一些文章，会定期更新，文章中的代码也在这个项目中，欢迎大家关注。
    
6.  参考  
    LimitedBox class  
    Offstage class  
    OverflowBox class  
    SizedBox class