
# WPF 源代码 从零开始写一个 UI 框架

需要知道 WPF 是一个 UI 框架，作为一个 UI 框架，最主要的就是交互。也就是 UI 框架需要有渲染显示和处理用户输入的功能。
如果直接告诉大家 WPF 里面有哪些类，估计没有几位小伙伴会听下去，要么就是讲的类太简单，看过去我也就知道了，要么就是这个类可能我一直都不会用到他，即使可能会用到也早就忘了。
本文不会直接告诉大家 WPF 的源代码是如何写的，而是从零开始一起来写一个 UI 框架，在写的过程就会了解到为什么 WPF 可以这样写，为什么需要这样写，和 WPF 这样写的好处。
本文适合 WPF 的开发者同样也适合其他语言希望自己写一个 UI 框架的小伙伴。

<!--more-->


<!-- csdn -->
<!-- 不发布 -->

这个故事的开始是有一天，前端的小伙伴在问我桌面端可以做的界面能否在前端也做出来。熟悉的小伙伴都认识我，我是不会前端的。于是我就向他请教，在前端里面有没有调用一个函数就可以做到在某个起点开始画圆？调用函数在某个起点画线段？调用函数在某个起点画点？画文字？画几何图形？画图片？ 他说有啊，有一个叫 Canvas 的控件，可以在里面做这些。我说那很棒，基本都可以做到。

前端小伙伴问那难不难，我就再问他，有没有一个东西，这个东西里面支持画点画线画文字这些，然后这个东西可以被画到 Canvas 的任何一个地方？虽然这句话比较饶，大概的意思就是 Canvas 可以嵌套 Canvas 类似的东西不？被嵌套的 Canvas 能否在任意的坐标开始画。解释清楚之后，前端小伙伴说可以啊。

于是我就回答他，那我可以写出一个框架，这样其他开发者就可以简单的进行开发了。

然后他问我，那么一个框架大概要写多久。我说三年……

……

当然，一个UI框架写三年速度是十分快的。好在本文是 WPF 的源代码，而不是手把手教大家如何写一个 UI 框架，所以本文不会写三年。为什么我会询问前端的小伙伴这些问题？因为我问的是绘制原语，只要能满足绘制原语，就可以做出一个 UI 框架的渲染显示部分。

更多的小伙伴关注的是渲染显示而不是输入层，实际上在渲染显示框架做好了之后，输入层也差不多完成了。本文的顺序就是先开始渲染显示框架是如何做的，然后在告诉大家输入层是如何做的。

刚才说道了绘制原语，需要解释一下，不知有没小伙伴不知道拼音的？如果学会了拼音，就可以使用拼音拼出普通话。电脑知道绘制原语就画出界面。从计算机图形学上，支持绘制原语就基本会画出所有的界面。

能知道在任意坐标，画出任意颜色的点，理论上就可以画出任何的界面。如果还可以在任意的坐标，画出任意颜色的几何，几何包括填充或描线两个方式，就可以高效画出任何界面。至于其他的画圆、画文字、画图片这些，如果有，开发起来会更加简单。如果没有原生的支持，那么想要做一个高性能的UI框架是很难的。

本文不会告诉大家如何通过只能画点封装出来画圆、画图片这些。先假设底层已经支持了绘制原语，现在开始封装一个 UI 框架。

在开始之前，需要引用画布的概念。假设一个界面就是一个画布，这个画布的左上角就是(0,0)的点，坐标就是从左向右 x 变大，从上到下 y 变大。具体请看下图。

<!-- ![](image/WPF 源代码 从零开始写一个 UI 框架/WPF 源代码 从零开始写一个 UI 框架0.png) -->

![](http://image.acmx.xyz/lindexi%2F2018111919201102)

再引入元素的概念，元素的边框就是一个矩形，元素将可以在自己的矩形之内使用绘制原语画出元素。元素的概念属于框架级的，也就是原生是没有这个概念，原生只有绘制原语的概念。

有了元素之后，一个 UI 框架的最简单的实现就已经完成了。但是这样的元素还无法做到灵活的画出界面，只是基本要求可以满足。虽然说简单，这部分的代码还是需要讲一下。

下面的代码是对应到 WPF 的布局和 UIElement 的 OnRender 方法，在看完本文就知道 UIElement 为什么需要 OnRender 的设计，以及 OnRender 设计的好处。

元素声明自己的坐标，只要不添加布局元素就可以不声明自己的宽度和高度。所有的在元素内部的绘制都是相对于元素自身的左上角坐标。

刚才说道的简单的绘制原语里面是在任意的坐标画出，而不是特定坐标，如果想要依靠元素的左上角，就需要再做一层封装。

还记得刚才的第二个问题，是否存在某个东西，这个东西可以在上面绘制，然后这个东西本身也可以被绘制到画布的任何坐标的问题。这个问题就是询问原生是否有支持在设置绘制原语的坐标的左上角为元素的左上角的东西，然后根据元素所在画布的坐标，画出这个东西。

如果有的话，就可以少封装一些内容，如果没有自己写也是可以的。于是先来写出这个东西的封装，一旦封装了这个东西，就需要同时封装了整个绘制原语。封装有一个好处，如果某个实现的原生框架不支持某个绘制原语还可以通过这一层进行实现。想要实现他就需要给一个命名，在 WPF 里面这个叫 DrawingContext 于是这里来实现一套。

需要注意本文下面实现的是简单版的 DrawingContext 里面包含了 WPF 布局的部分代码

```csharp
    public class DrawingContext
    {
        /// <inheritdoc />
        public DrawingContext(Point elementPoint, Board board)
        {
            ElementPoint = elementPoint;
            Board = board;
        }

        public Point ElementPoint { get; }

        public Board Board { get; }

        public void DrawEllipse(Point point, double radiusX, double radiusY, Color? pen, Color? brush)
        {
            
        }
    }
```

从上面代码可以看到，这个 DrawingContext 包含了以下属性。这里是为了提供为其他语言的小伙伴可以知道设计 DrawingContext 需要哪些内容 

 - 元素所在上一层的 x y 坐标
 - 画布本身

和提供了绘制原语的方法，大家也不会想看到每个的实现，所以我就使用画椭圆为例子。

这个 DrawingContext 里的属性都是注入的，因为现在的 UI 框架只有画布和元素两个，所以注入 DrawingContext 就需要在画布中做。

定义一个画布 Board 这个画布里面拥有直接调用原生的绘制原语的方法，画布里面可以包含元素。定义的代码请看下面

```csharp
    public class Board
    {
        public List<Element> ElementList { get; } = new List<Element>();

        public IPlatformVisual PlatformVisual { get; set; }

        public void InvalidateVisual()
        {
            foreach (var temp in ElementList)
            {
                var dc = new DrawingContext(temp.Point, this);
                temp.OnRender(dc);
            }
        }
    }
```

画布包含的属性列表是

 - 元素集合

 - 原生的绘制类

画布现在就包含一个方法

 - 渲染方法 调用这个方法就会触发渲染

这里的原生的绘制的类，是需要根据不同的平台来做的，有一些平台，如 OPG 是只有调用方法，于是就需要自己封装一个类包含这些方法，进行注入。

从上面的代码可以看到，画布的渲染方法 InvalidateVisual 需要被调用才可以绘制，实际的 WPF 框架也是这样，在 WPF 是通过 dx 的垂直同步或者 `WM_Paint` 消息进行绘制的。在 WPF 可以通过监听 CompositionTarget.Rendering 事件获得 WPF 进行渲染。

因为使用了元素，为了写出画布的渲染方法需要先告诉大家元素的定义。

```csharp
    public class Element
    {
        public Point Point { get; set; }

        public virtual void OnRender(DrawingContext dc)
        {
        }
    }
```

元素的定义属性只有元素的坐标，元素的方法也只有渲染方法

现在开始填充画板的渲染方法代码，和元素的渲染方法代码

```csharp
    public interface IPlatformVisual
    {
        void DrawEllipse(Point point, double radiusX, double radiusY, Color? pen, Color? brush);
    }

    public class Board
    {
        public void DrawEllipse(Point point, double radiusX, double radiusY, Color? pen, Color? brush)
        {
            PlatformVisual.DrawEllipse(point, radiusX, radiusY, pen, brush);
        }
    }
```

元素的渲染是通过 DrawingContext 渲染，在 DrawEllipse 方法需要将元素给的画出来的坐标加上元素的坐标，然后就直接调用 Board 的方法

```csharp
    public class DrawingContext
    {

        public void DrawEllipse(Point point, double radiusX, double radiusY, Color? pen, Color? brush)
        {
            point = new Point(ElementPoint.X + point.X, ElementPoint.Y + point.Y);
            Board.DrawEllipse(point, radiusX, radiusY, pen, brush);
        }
    }

```



现在一个最简单的 UI 框架已经完成，虽然还没有写输入的代码，验证这个 UI 框架是否可行，可以使用真实的代码跑一下。

我使用 win2d 作为原生的绘制方法，除了 win2d 其他的代码都是我自己写的。

第一步就是封装一下 win2d 的代码，这样 win2d 的概念在下面也就不会提及了。即使有提交也只是 win2d 的 [CanvasCommandList](https://lindexi.gitee.io/post/win2d-CanvasCommandList-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.html) 刚才封装的画板的渲染方法，需要支持元素在某个坐标绘制写了很多代码，而在 win2d 因为存在了[CanvasCommandList](https://lindexi.gitee.io/post/win2d-CanvasCommandList-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.html)只需要使用很少量的代码就可以做到，因为[CanvasCommandList](https://lindexi.gitee.io/post/win2d-CanvasCommandList-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.html)支持在里面绘制，然后在 Canvas 的任意坐标画出[CanvasCommandList](https://lindexi.gitee.io/post/win2d-CanvasCommandList-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.html) 这样就可以完成画布的渲染方法。

这里的 IPlatformVisual 就是平台的元素渲染接口，本文使用 win2d 需要继承这个接口实现一个类

```csharp
   public class DrawVisual : IPlatformVisual
    {
        /// <inheritdoc />
        public void DrawEllipse(Point point, double radiusX, double radiusY, Color? pen, Color? brush)
        {
            DrawVisualList.Add(ds =>
            {
                if (pen != null)
                {
                    ds.DrawEllipse((float) point.X, (float) point.Y, (float) radiusX, (float) radiusY, pen.Value);
                }

                if (brush != null)
                {
                    ds.FillEllipse((float) point.X, (float) point.Y, (float) radiusX, (float) radiusY, brush.Value);
                }
            });
        }

        public List<Action<CanvasDrawingSession>> DrawVisualList { get; } = new List<Action<CanvasDrawingSession>>();
    }
```

这个 DrawVisual 类的实现是将调用的方法暂时放在列表，等待调用才画出。在不同的平台可以使用不同的实现，只要调用了对应的方法就可以在界面画出就可以

第二步是创建一个元素继承元素，创建的元素就叫椭圆，这个元素就是画出椭圆。

```csharp

```

第三步尝试在画板画出元素

```csharp

```

运行一下，已经可以看到最简单的UI框架完成。





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。