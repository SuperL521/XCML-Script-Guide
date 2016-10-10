#哲彬语言教程#

哲彬语言是CM Launcher研发团队为开发安卓3D主题而开发的一种描述性脚本语言。CM Launcher的安卓3D主题与其他安卓主题类似，仅有资源文件和描述性的xml文件构成。随着功能的增多，其中描述3D主题的xml文件launcher_theme_3d_model.xml被添加了越来越多的功能，渐渐地成为了一套功能完备的简单语言。哲彬语言文件会在安卓进程CM Launcher里程序里解析后在该进程中运行，换言之没有编译的过程，其实只是一种中间码。哲彬语言的独特之处在于它可以用来描述3D模型的种种动画和脚本，这一切的底层是CM Launcher团队强大的GLView引擎，基于OpenGL ES。

哲彬语言可以分为两部分：哲语言和彬语言。哲语言是最初的版本，完全通过xml来进行所有的操作。彬语言是哲语言的升级版，是将哲语言中的script部分多加了一层解析器，可以使用类C语言语法来编程。

学习哲彬语言最大的问题在于两点：1. 由于历史原因，有很多API用处少、有bug且不规范，但是为了向上兼容，这些旧API又必须被保留，因此极容易与现有API混淆；2. 由于最初需求、基础架构等等问题，目前的哲彬语言的编程范式极为诡异：既面向对象又面向过程，既有事件驱动又有函数响应式，因而显得不伦不类，学习成本很高。

此外，由于哲彬语言之前没有统一规范，以至于目前已有的哲语言代码可读性不佳。我们强烈建议您按照本网站提供的规范来写哲彬语言以避免不必要的错误。哲彬语言的解析在CM Launcher下完成，因此我们建议您保持你的CM Launcher时刻为最新版本以避免已被消除的bug和支持全部功能。

开发人员：王哲，唐继，李海池，陈志文，邵文彬，刘佳

##3D主题框架##
如上文所述，3D主题是建立在一套安卓主题框架下。这一套框架具体有两大部分组成：资源文件（icons/weather/wallpaper/3d models）和描述性xml文件。本部分简略介绍这个框架的每个部分的作用。关于这个主题架构本教程只是简单介绍，具体的细节请参考CM Launcher的DIY主题教程。

icons/app_theme_icons.xml

weather

wallpaper

3d models

launcher_theme_config.xml

app_theme_icons.xml

launcher_theme_3d_model.xml

##哲彬语言的基础框架和常用标签##

彬语言需要嵌套在哲语言中，哲语言可以单独使用。哲语言其实就是一种xml，基本语法也与xml等同。

1.从xml开始

作为xml的一种延伸，哲彬语言也必须遵循xml的基本格式，必须这样开头：

```xml
<?xml version="1.0" encoding="UTF-8"?>
```


2.最外层标签：Theme和Wallpaper

Theme是哲彬语言最外层标签，哲彬语言必须强制从这个标签开始。举例：

其中effect定义的是CM Launcher中切屏时的图标旋转效果，目前仅支持sphere即圆球滚动。
Wallpaper是仅次于Theme标签以外的最外层标签，标准化写法为：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Theme effect="sphere" version="1">
  <Wallpaper name="elementWallpaper">
     <!--your codes here-->
  </Wallpaper>
</Theme>
```

有且只有这一种写法。

除此之外还有一种与Wallpaper平级的标签ElementEffectCore，但是这个标签很可能要被弃用，因此不推荐使用。

3.基本单位Object

Object并不是一个标签或数据类型，而是一个基本概念，指哲彬语言中的基本单位。Object的类型可以是3D模型也可以是一张图片也可以是float型变量。一个名为test的图片型Object的定义为：

之后就可以对test进行缩放和改变位置的操作如：

显而易见，这里的Image就如同大部分面向对象语言的class，而这里的test就相当于名为Image的class的一个instance，声明test的过程就如同Image类的构造函数。常用的Object类型会在后面章节详细介绍。

哲彬语言中，每个Object都有一些自己的变量和函数，但是只能通过Object来调用函数而不能直接访问变量。比如如果想获得test图片的宽度，就只能使用如下操作：

相当于每个class的变量全部都是私有变量。

不在RootGroup下的Object必须在drawWallpaper里进行dispatchDraw操作才能显示：

这部分将在后面详述。

4.Group和RootGroup

Group是一个Object的一个集合，一个Group标签里可以包含任意个数任意类型的Object。一个Group下也可以有多个Group。比如：


这里的group0就是group1的parent，而group1又是它里面的test2的parent。简而言之，Group就是描述不同Object间的父子层级关系。如Object一样，可以直接通过Group的名字来进行操作，如：

这样group0的所有children都会跟着放大到2倍，即test0, test1, test2, test3都会放大两倍。而group1的所有children都会随着旋转30度，即test2会旋转30度。无论Group的children类型是什么，Group可用的函数只有一些通常的基本的函数。Group实质上就是对多个Object进行批量化操作来简化代码量。
RootGroup是一种特殊的Group，它的声明为：


RootGroup与Group的区别在于：1. RootGroup下的Object支持“对象事件”，将于后面章节详细介绍；2. RootGroup下的Object无需在drawWallpaper里进行dispatchDraw操作。


5.Script
Script即脚本部分，只有在这个标签下可以进行以下两种操作：1. 调用Object的函数；2. 调用系统的事件函数。系统的事件函数是重要的6种事件，比如onTouchDown，将会在后面详述。调用的方法是在对应的Wallpaper标签或者Object标签内，举例：

Script部分是哲彬语言的重点所在。所谓彬语言就是把Script标签下的内容替换成了一种新的写法。每一个Object都有一个Script，相当于一个自定义的成员函数，并且每个Object的成员函数仅仅作用于该Object，由于本语言不支持this指针，所以即使是在Object内的Script也必须使用该Object的名字来调用成员函数。
NOTE：
我们强烈建议您只在Wallpaper里使用Script。
