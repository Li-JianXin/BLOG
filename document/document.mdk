## QuartzCore
### Core Animation
在使用Core Animation 开发动画的本质是将CALayer中的内容转化为位图从而供硬件操作。
CALayer很多属性在修改时可以形成动画，隐式动画属性。

### <font face="微软雅黑" color=#000000>CALayer</font>
> Layers Provide the Basis for Drawing and Animations（Layers是绘图和动画的基础）

UIView继承自UIResponder，其主要任务是响应触摸事件。显示通过组合，将责任委托给成员变量BackingLayer。

![](https://github.com/Li-JianXin/ILEImage/blob/master/UIView.png?raw=true)


#### Layer Tree
1、模型树Model Tree，直接创建的或者通过UIView获得的view.layer用于显示的图层树。模型树的背后还存在两份图层树的拷贝，一个是呈现树Presentation Tree，一个是渲染树Render Tree。

2、呈现树Presentation Tree，可以通过普通layer.presentationLayer获得。模型树则可以通过modelLayer属性获得.模型树的属性在其被修改的时候就变成了新的值,这个是可以用代码直接操控的部分;呈现树的属性值和动画运行过程中界面上看到的是一致的.

3、渲染树Render Tree，渲染树是私有的，无法访问到，,渲染树是对呈现树的数据进行渲染,为了不阻塞主线程,渲染的过程是在单独的进程或线程中进行的,所以你会发现Animation的动画并不会阻塞主线程。

Layer是基于bitmap的，它会捕获View要呈现的内容，然后cache在一个bitmap中，这个bitmap可以看作一个对象。这样每次进行操作，例如平移旋转等，只是bitmap的矩阵运算。基于Layer的动画过程如图。


#### 为CALayer提供内容的三种方式：
##### 把一个图像对象直接赋值给contents属性
屏幕上看到的东西其实是一张张图，由CALayer的contens提供。
```
@property(nullable, strong) id contents;
```
只有传入CGImage的内容，才可以正常显示。


##### 设置delegate，让代理绘制layer的内容
```
- (void)displayLayer:(CALayer *)layer;
```
如果实现了这个方法，会绘制一个bitmap，然后赋值给contents属性。
```
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```
如果实现了这个方法，Core Animation提供一个context来生成bitmap，自己需要把想要的内容绘制到context。
这两个方法至少实现两个代理方法其中的一个，如果都实现，则调用displayLayer。

###### 通过Core Graphics自定义绘图到CALayer的Backing Store中

> CALayer会根据contentsScale创建Backing Store，并且根据contentScale设置Context的CTM（Concurrent Transform Matrix）。例如，Layer的尺寸为10 Point x 10 Point，当contentsScale为2.0时，最后生成的Backing Store大小为20 Pixel x 20 Pixel，并且在将创建的Graphics Context传入drawLayer:InContext:前，会调用CGContextScaleCTM(context, 2, 2)进行放大，这样会使生成的内容自动满足屏幕的要求。


##### 继承自CALayer，重写绘制方法，来提供layer内容


```
- (void)drawRect:(CGRect)rect;
```

这个方法是对 drawlayer:inContext的包装。没有默认实现，如果不需要自定义的绘制，就不要创建这个方法，会造成CPU资源和内存浪费。

> 内存hekt：图层宽*图层高*4字节，宽高的单位均为像素

#### 属性
##### drawsAsynchronously
iOS 6中，苹果为CALayer引入了这个令人好奇的属性，drawsAsynchronously属性对传入-drawLayer:inContext:的CGContext进行改动，允许CGContext延缓绘制命令的执行以至于不阻塞用户交互。

它与CATiledLayer使用的异步绘制并不相同。它自己的-drawLayer:inContext:方法只会在主线程调用，但是CGContext并不等待每个绘制命令的结束。相反地，它会将命令加入队列，当方法返回时，在后台线程逐个执行真正的绘制。
根据苹果的说法。这个特性在需要频繁重绘的视图上效果最好（比如我们的绘图应用，或者诸如UITableViewCell之类的），对那些只绘制一次或很少重绘的图层内容来说没什么太大的帮助。

#### 应用问题
一、使用UIBezierPath作为路径，在drawRect中直接使用其
```
- (void)stroke;
```
方法，存在严重的性能问题，如果连续绘制，UIBezierPath中会记录一条很长的路径，重绘的工作越来越多，界面会越来越卡。

二、使用Quartz2D(苹果提供的一套二维绘图引擎,API为Core Graphics框架的一部分)，每次在drawRect中的时候仅仅绘制一条线段。

三、解决内存点用的方法，可以使用CAShapleLayer绘制矢量图形。

四、renderInContext 方法。

在非drawRect方法中调用 renderInContext方法，或者path stroke等方法。会导致警告：


```

Mar 29 12:16:22  Undercover[3702] <Error>: CGContextDrawImage: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
Mar 29 12:16:22  Undercover[3702] <Error>: CGContextRestoreGState: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
Mar 29 12:16:22  Undercover[3702] <Error>: CGContextRestoreGState: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
Mar 29 12:16:22  Undercover[3702] <Error>: CGContextSaveGState: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
Mar 29 12:16:22  Undercover[3702] <Error>: CGContextSaveGState: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
```


