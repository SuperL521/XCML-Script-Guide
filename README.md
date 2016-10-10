#XCML Script教程

##简介
XCML Script是CM Launcher研发团队为开发安卓3D主题而开发的一种描述性脚本语言。CM Launcher的安卓3D主题与其他安卓主题类似，仅有资源文件和描述性的xml文件构成。随着功能的增多，其中描述3D主题的xml文件launcher_theme_3d_model.xml被添加了越来越多的功能，渐渐地成为了一套功能完备的简单语言。XCML Script文件会在安卓进程CM Launcher里程序里解析后在该进程中运行，换言之没有编译的过程，其实只是一种中间码。XCML Script的独特之处在于它可以用来描述3D模型的种种动画和脚本，这一切的底层是CM Launcher团队强大的GLView引擎，基于OpenGL ES。

###1. Zhe Script和Bin Script
XCML Script可以分为两部分：Zhe Script和Bin Script。Zhe Script是最初的版本，完全通过xml来进行所有的操作。Bin Script是Zhe Script的升级版，是将Zhe Script中的script部分多加了一层解析器，可以使用类C语言语法来编程。

**实质上，Zhe Script就是使用xml描述3D主题的空间布局，Bin Script描述的是布局下的各个组件对不同事件做出的反应来实现交互、动画等效果。**

###2. ISSUES
学习XCML Script最大的问题在于两点：
1. 由于历史原因，有很多API用处少、有bug且不规范，但是为了向上兼容，这些旧API又必须被保留，因此极容易与现有API混淆；
2. 由于最初需求、基础架构等等问题，目前的XCML Script的编程范式极为诡异：既面向对象又面向过程，既有事件驱动又有函数响应式，因而显得不伦不类，学习成本很高。

此外，由于XCML Script之前没有统一规范，以至于目前已有的Zhe Script代码可读性不佳。我们强烈建议您按照本网站提供的规范来写XCML Script以避免不必要的错误。XCML Script的解析在CM Launcher下完成，因此我们建议您保持你的CM Launcher时刻为最新版本以避免已被消除的bug和支持全部功能。

###3. STAFF

所有方： Cheetah Mobile

开发人员：王哲，唐际，李海池，陈志文，陈兆起，邵文彬，刘佳

文档编写和维护：邵文彬

##3D主题框架

如上文所述，3D主题是建立在一套安卓主题框架下。这一套框架具体有两大部分组成：资源文件（icons/weather/wallpaper/3d models）和描述性xml文件。本部分简略介绍这个框架的每个部分的作用。关于这个主题架构本教程只是简单介绍，具体的细节请参考CM Launcher的DIY主题教程。

###1. icons/app_theme_icons.xml

###2. weather

###3. wallpaper

###4. 3d models

###5. launcher_theme_config.xml

###6. app_theme_icons.xml

###7. launcher_theme_3d_model.xml

##XCML Script的基础框架和常用标签

Bin Script需要嵌套在Zhe Script中，Zhe Script可以单独使用。Zhe Script其实就是一种xml，基本语法也与xml等同。

###1. 从xml开始

作为xml的一种延伸，XCML Script也必须遵循xml的基本格式，必须这样开头：

```xml
<?xml version="1.0" encoding="UTF-8"?>
```


###2. 最外层标签：Theme和Wallpaper

Theme是XCML Script最外层标签，XCML Script必须强制从这个标签开始。举例：

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

###3. 基本单位Object

Object并不是一个标签或数据类型，而是一个基本概念，指XCML Script中的基本单位。Object的类型可以是3D模型也可以是一张图片也可以是float型变量。一个名为test的图片型Object的定义为：

```xml
<Image name="test" texture="testimg" />
```

之后就可以对test进行缩放和改变位置的操作如：

```xml
<CallMethod model="test" method="setScale" args="2" />
<CallMethod model="test" method="setX" args="0" />
```

显而易见，这里的Image就如同大部分面向对象语言的class，而这里的test就相当于名为Image的class的一个instance，声明test的过程就如同Image类的构造函数。常用的Object类型和函数调用的规则会在后面章节详细介绍。

XCML Script中，每个Object都有一些自己的变量和函数，但是只能通过Object来调用函数而不能直接访问变量。比如如果想获得test图片的宽度，就只能使用如下操作：

```xml
<CallMethod model="test" method="getWidth" args="2" />
```

相当于每个class的变量全部都是私有变量。

值得一提的是，不在RootGroup下的Object必须在drawWallpaper里进行dispatchDraw操作才能显示。这部分将在后面详述。

###4. Group和RootGroup

Group是一个Object的一个集合，一个Group标签里可以包含任意个数任意类型的Object。一个Group下也可以有多个Group。比如：

```xml
<Group name="group0">
  <Image name="test0" texture="img"/>
  <Image name="test1" texture="img"/>
  <Group name="group1">
    <Image name="test2" texture="img"/>
  </Group>
  <Image name="test3" texture="img"/>
</Group>
```

这里的group0就是group1的parent，而group1又是它里面的test2的parent。简而言之，Group就是描述不同Object间的父子层级关系。如Object一样，可以直接通过Group的名字来进行操作，如：

```xml
<CallMethod model="group0" method="setScale" args="2" />
<CallMethod model="group1" method="setRotationX" args="30" />
```

这样group0的所有children都会跟着放大到2倍，即test0, test1, test2, test3都会放大两倍。而group1的所有children都会随着旋转30度，即test2会旋转30度。无论Group的children类型是什么，Group可用的函数只有一些通常的基本的函数。Group实质上就是对多个Object进行批量化操作来简化代码量。
RootGroup是一种特殊的Group，它的声明为：

```xml
<RootGroup name="root"/>
```

RootGroup与Group的区别在于：
1. RootGroup下的Object支持“对象事件”，将于后面章节详细介绍；
2. RootGroup下的Object无需在drawWallpaper里进行dispatchDraw操作。


###5. Script
Script即脚本部分，只有在这个标签下可以进行以下两种操作：1. 调用Object的函数；2. 调用系统的事件函数。系统的事件函数是重要的6种事件，比如onTouchDown，将会在后面详述。调用的方法是在对应的Wallpaper标签或者Object标签内。

Script部分是XCML Script的重点所在。所谓Bin Script就是把Script标签下的内容替换成了一种新的写法。每一个Object都有一个Script，相当于一个自定义的成员函数，并且每个Object的成员函数仅仅作用于该Object，由于本语言不支持this指针，所以即使是在Object内的Script也必须使用该Object的名字来调用成员函数。

**我们强烈建议只在Wallpaper里使用Script。**

##Zhe Script基本语法

###1. 变量基本类型和数组

###2. 定义变量

###3. 定义方法

###4. 变量赋值

###5. 调用自定义的方法

###6. 逻辑判断

##常用Object和成员函数列表

###1. 基础Object和Group

###2. Model

###3. Image

##全局静态函数和静态变量##

###1. Math

###2. Calculate

###3. ThemeCommonUtils

###4. SensorConverter

###5. Product

###6. TimeData

###7. Setting

###8. 全局静态变量

##模型导入和屏幕适配##

###1. OBJ格式

XCML Script支持的obj格式为3ds max下导出的obj格式。因此如果您使用的是其他建模软件如maya、camera4d等等，请先导出一个3ds max可以支持的格式，再在3ds max中导出obj。目前主流建模软件直接导出的obj格式都可以支持，但是有时还会遇到一些问题。比如maya导出的obj必须要进行一次网格三角化操作，负责可能会出现部分区域不能显示的bug。模型的位置、大小、旋转中心、贴图UV等都会被保存，请在建模软件中调整好再导出。

**如果一个场景有多个模型，请现在建模软件里加载全部模型调整好大小比例和位置后再逐一导出。在XCML Script里调整会太过于繁琐并且可能会出现适配问题。**

###2. DAE格式

XCML Script支持的dae格式为3ds max、Maya下导出dae格式（勾选三角算法、单一矩阵）。目前导出的dae模型的动画只支持整个物体的旋转、缩放、平移，可能会出现个别节点没有解析的情况。同时支持模型节点为嵌套关系、支持模型没有纹理和法向量数据。请注意，场景中用不到的模型必须全部删除,之后再导出dae格式.不支持顶点颜色属性。不支持骨骼动画。

**如果一个场景有多个模型，请现在建模软件里加载全部模型调整好大小比例和位置后再逐一导出。在XCML Script里调整会太过于繁琐并且可能会出现适配问题。**

###3. 模型精度和贴图尺寸

由于模型文件会在一开始被一次性读入，过大的模型会导致读取较慢甚至out of memory崩溃。因此模型文件应当尽可能的小，请在导出时删除不必要的信息。考虑到低端安卓手机支持的图片尺寸有限制，在使用较大图片（大于1920*1080）时，请存储为较小尺寸再在程序中进行放大操作。

###4. 光照和材质

目前并不支持主流建模软件中的光照和材质，只能用GLSL shader来实现。关于shader，后面章节有详细介绍。

###5. 屏幕适配问题和方案

屏幕适配一直是安卓应用不可避免的大问题，由于安卓手机品类过多，不同品牌不同型号的手机屏幕尺寸和分辨率差距极大。而在3d场景中，不一样的尺寸和分辨率出现的效果可能完全不同。为了解决适配问题，请尽量避免使用以px为单位的操作而使用以dp为单位的操作。对Object进行的缩放操作，请使用上文提到的ThemeCommonUtils转换数值后再用作函数参数。为减少繁琐的操作，请不要使用过多的涉及到位置信息的动画。

##Bin Script语法##

Bin Script目前还在不断完善中。暂时还不支持大括号，因此请务必注意代码对齐。目前版本，Bin Script只能在Script标签内使用，还要在开始加上一个<![CDATA[标识，并在结束加上]]>标识。如：
```xml
<Script>
  <![CDATA[
    var test_your_codes_here = 0;
  ]]>
</Script>
```

在程序底层，Bin Script代码会被翻译成Zhe Script代码。

###1. 变量的类型和作用域

在Zhe Script中，Var是自定义的局部变量，GlobalVar是全局变量。 与Object不同，Var和GlobalVar不受Group约束。Bin Script中，它们对应的数据类型分别为var和gvar。目前Bin Script支持的变量类型只有三种：int，float和boolean。int和float都会在运算过程中被强制转换为float。三种变量的定义方法为：

```
var test_int = 1;
var test_float = 0.5f;
var test_boolean = true;
```

float型变量必须以f结尾。boolean型必须用"true"或者"false"来定义和赋值，不支持其他赋值方式也不支持与其他类型的强制转换。不支持long或double等数据类型，最大只支持4个字节，请注意不要产生变量溢出。

var变量的作用域为变量所在的标签或函数内。换言之，如果按照要求把所有脚本都写在wallpaper后的script下的事件函数之外，则所有的变量均可以作用于全局。与JS类似，var可以先使用再声明。gvar是全局变量，声明后可在整个xml文件内作用。

**我们建议在变量声明时务必赋予一个初始值**

###2. 数组

###3. 事件函数

由于不支持大括号，事件函数开始也需要用分号隔开，结束时需要添加endfunction标识。如：

```
function onDrawStart();
 var test_your_codes_here = 0;
endfunction;
```

###4. 运算

与其他语言不同，Bin Script中调用一个运算式需要加上exp标识，目前只支持四则运算，不支持位运算等其他运算。在exp的括号内写入算式，如：

```
var test0 = exp(1+1);
var test1 = exp(test0*10);
var test2 = exp(test0/test1);
```

###5. if/else/elseif/endif

Bin Script中的逻辑判断语句，如：

```
var test0 = true;
var test1 = false;
var test_your_value = 0;

if(test0);
  test_your_value = 1;
endif;

if(test0);
  test_your_value = 1;
else;
  test_your_value = 2;
endif;

if(test0);
  test_your_value = 1;
else if(test1);
  test_your_value = 2;
else;
  test_your_value = 3;
endif;
```

同样的，需要每行以;结束，条件判断结束时必须有endif。目前支持的条件运算符有：>, <, !=, >=, <=。目前尚不支持取反运算符和位运算符。float和int在比较时会被强制转换为float。boolean型不能和float或int进行比较。if后紧跟的括号内可以加入逻辑判断式。

```
var test0 = 0;
var test1 = 1;

if(test0<test1);
  test0 = 2;
endif;
```

###6. 函数调用

通过Object的名字调用函数，只需要像java一样加上一个.即可。值得一提的是，目前并不支持原子运算，也是就是说函数的参数必须是一个变量而不能是一个运算式。对于需要有运算式的情况，需要先自定义一个变量储存运算结果然后再调用。正确的函数调用：

```
test.setScale(2);
var scale = exp(2*3);
test.setScale(scale);
```

##事件驱动编程

如上文所说，复杂的编程范式是XCML Script的难点所在。事件驱动编程是游戏开发中常用的编程范式，它指对不同的事件执行对应的程序。也就是说，绝大部分的脚本都是在特定的事件下运行。在特定的事件外，只能进行一些变量的声明等操作。代码模块之间的逻辑关系也就是不同事件下的逻辑关系。XCML Script目前支持以下6种类型的事件函数。

调用事件函数的通用模板为：

```
var test_your_value = 0;

function onDrawStart();
  test_your_value = 1;
endfunction;
```

这里的onDrawStart就是6种事件函数之一，将它替换成其他名字即是调用其他事件函数。由于只有这固定的6种事件函数，所有函数也没有参数表和返回值。暂时还不支持自定义函数。

**除此6种事件外，其实还有一种不标准的写法。在script标签内直接写入一些脚本，在某些情况下，这些脚本可能会在初始化时运行一次，即效果如同写在init下。我们强烈建议放弃这种写法。**

###1. init/drawWallpaper

每一个Object都有一个init方法来完成一些初始化操作。如上文所说，大部分属性都可以在Object声明时进行定义，但是由于声明时不支持变量，因此有一些操作必须在init下完成，比如让一个名为test的宽度为540的Image全屏。举例：

```
function init();
  var CanvasWidth = ThemeVariable.getCanvasWidth();
  var scaleUnit = exp(CanvasWidth/540f);
  test.setScale(scaleUnit);
endfunction;
```

通常，初始化都是进行一些屏幕适配操作。drawWallpaper与init类似，也是进行一些初始化操作，不同的是drawWallpaper只能在特殊标签Wallpaper内调用。如果按照我们的建议只在Wallpaper里使用Script，则drawWallpaper和init的功能等同。drawWallpaper还有一个常用的特殊功能就是上文提到过的dispatchDraw。下面的代码展示将一个名为group的Group使用dispatchDraw，如果不使用该函数会导致该group内的所有Object无法显示。

```
function drawWallpaper();
  group.dispatchDraw();
endfunction;
```

**我们建议把所有初始化操作都放在drawWallpaper里完成而不要再使用init**

###2. onDrawStart

onDrawStart是XCML Script中最核心的事件，绝大部分操作都在onDrawStart中进行。相当于很多其他语言中的update函数，onDrawStart是逐帧调用的，这意味着通常这个函数是每秒要被调用60次。作为核心，onDrawStart通常会是3D主题造成卡顿的罪魁祸首。onDrawStart执行的操作越多就会带来越多的性能损耗，因此应当技巧性地避免在onDrawStart里进行过多的操作。

###3. onDesktopEffectStart/onDesktopEffectEnd

这两个函数是指在桌面滑屏开始和结束时调用。大多数情况下，需要配合一个boolean型变量来对一个3D模型进行控制，例如：

```
var isDesktopEffectRunning = false;
function onDesktopEffectStart();
  isDesktopEffectRunning = true;
endfunction;
function onDesktopEffectEnd();
  isDesktopEffectRunning = false;
endfunction;
```

###4. onIconStartDrag/onIconEndDrag

类似onDesktopEffectStart/onDesktopEffectEnd，这一对函数是在拖动桌面图标开始和结束时调用。

###5. onTouchMove/onTouchDown/onTouchUp

如同它们的名字那样，这个是响应的触摸的函数。
onTouchDown在触摸开始的时候调用
onTouchUp在触摸结束时调用
onTouchMove在触摸移动时调用，可以用来捕捉触摸轨迹，在触摸开始和结束时也会被调用。通常用作实现手势跟随效果。
对应的有onTouchDown_x/onTouchDown_y, onTouchUp_x/onTouchUp_y, onTouchMove_x/onTouchMove_y，三对全局静态变量，在对应的事件下调用对应的变量来实现要求的功能。

举例：


###6. onSensorChanged

在sensor改变时被调用，最常用的就是用来获得当前陀螺仪的旋转角度。在我们的测试过程中，sensor被证明是最耗电的，所以如果出于省电考虑，可以不使用sensor。在底层，sensor是每3帧调用一次，这也是为了节省电量，因此仔细观察会有轻微的不流畅。由于目前引擎并没有camera类，大部分3D主题的立体感都依靠响应sensor来实现。与触摸事件函数类似，该事件函数也有专属的静态变量。
举例：


##特殊的Object

###1. ValueInterpolate

###2. WaveInterpolate

###3. Timer

###4. Others

##动画类

##高级类

##粒子系统相关类

##GLSL SHADER

##常用模板
