#XCML Script Tutorial

## Introduction
XCML Script is a descriptive scripting language developed by CM Launcher's development team to make Android 3D themes. CM Launcher's Android 3D theme is similar to other Android themes, with only resource files and descriptive XML files. Among these files, the “launcher_theme_3d_model.xml”was added more and more functions because of the increasing requirements. Finally, a new scripting language was created and was called as “XCML Script”.

XCML Script will work after being compiled in the Android process of CM Launcher. In another word, there is not linking or running process for this language. Actually, it is just a kind of middle codes. XCML Script is unique in that it can be used to describe a variety of 3D models of animation and scripting, all of these are benefited by the OpenGL ES-based GLView engine, developed and maintained by CM Launcher team.

### 1. Zhe Script and Bin Script
XCML Script can be divided into two parts: Zhe Script and Bin Script. Zhe Script is the original version and is just a kind of extension of XML. Bin Script is an upgraded version of Zhe Script and an alternative of the "Script" tag in Zhe Script. It can be programmed using the C-like syntax. There is a parser layer between Bin Script and Zhe script which will compile Bin Script codes to Zhe Script codes.

**In essence, Zhe Script is the use of XML to describe the spatial layout of 3D themes, Bin Script describes the various components of the layout of the response to different events to achieve interaction, animation and other effects.**

### 2. ISSUES
XCML Script is difficult to learn because of two reasons:
1. There were many APIs that are less useful, buggy, and non-standard, but for backward compatibility. These legacy APIs must be preserved, so they can easily be confused with existing APIs;
2. Because of the initial needs, infrastructure and so on, the current XCML Script programming paradigm is extremely strange: both object-oriented and process-oriented, both event-driven and function response type, which is neither fish nor fowl, high learning costs.

In addition, because XCML Script before there is no uniform programming style so that the existing Zhe Script code readability is poor. We strongly recommend that you write the XCML Script in accordance with the specifications provided on this site to avoid unnecessary errors. The XCML Script parsing is done under the CM Launcher, so we recommend that you keep your CM Launcher up-to-date to avoid bugs that have been eliminated and to support full functionality.

### 3. STAFF

Legal author: Cheetah Mobile

Developers: Wang Zhe, Tang Ji, Li Hai Chi, Chen Zhiwen, Chen Zhaoqi, Shao Wenbin, Liu Jia

Document maintenance: Shao Wenbin

## 3D Themes framework

As mentioned above, a 3D theme is built on a set of Andrews theme framework. The framework consists of two main components: the icons / weather / wallpaper / 3d models and the descriptive XML files. This section gives a brief introduction to the role of each part of the framework. About this topic architecture This tutorial is only a brief introduction, the specific details, please refer to the DIY CM Launcher theme tutorial.

###1. icons/app_theme_icons.xml

###2. weather

###3. wallpaper

###4. 3d models

###5. launcher_theme_config.xml

###6. app_theme_icons.xml

###7. launcher_theme_3d_model.xml

## XCML Script basic framework and commonly used tags

Bin Script needs to be nested in Zhe Script, Zhe Script can be used alone. Zhe Script is actually a kind of XML, the basic syntax is also equivalent to XML.

### 1. Start with XML

XCML Script must also follow the basic format of XML, and must begin with:

```xml
<? xml version = "1.0" encoding = "UTF-8"?>
```


### 2. Outermost tabs: "Theme" and "Wallpaper"

"Theme" is the XCML Script outermost label, XCML Script must be mandatory from the beginning of this label.

Which effect is defined in the CM Launcher cut the screen when the icon rotation effect, currently only supports "sphere".
"Wallpaper" is the most outside of the Theme label outside the label, the standardized wording is:

```xml
<? xml version = "1.0" encoding = "UTF-8"?>
<Theme effect = "sphere" version = "1">
  <Wallpaper name = "elementWallpaper">
    <! - your codes here ->
  </ Wallpaper>
</ Theme>
```

This is only one way.

In addition, there is a level with the Wallpaper label called ElementEffectCore. Because this label is likely to be abandoned, it is not recommended.

Basic Unit

Object is not a label or data type, but a basic concept, referring to XCML Script in the basic unit. Object type can be a 3D model, a picture or a float variable. A picture named Object named test is defined as:

```xml
<Image name = "test" texture = "testimg" />
```

The Object "test" can be scaled and change the location of the operation such as:

```xml
<CallMethod model = "test" method = "setScale" args = "2" />
<CallMethod model = "test" method = "setX" args = "0" />
```

Obviously, the "Image" here is like most of the object-oriented language class, and here the "test" is equivalent to an instance of a class named "Image". Common Object types and their member functions call rules will be described in detail later.

In XCML Script, each Object has some of their own variables and member functions, but only through the Object to call the function can not directly access the variable. For example, if you want to get the width of "test", you can only use the following operations:

```xml
<CallMethod model = "test" method = "getWidth" target="testwidth" />
```

It can be understood as all the variables of one class are all private variables.

Except the Objects in the RootGroup, all Objects should call "dispatchDraw" method in the "drawWallpaper" function in the "Wallpaper" tag. This will be described in detail later.

### 4. Group and RootGroup

Group is a collection of Objects, a Group tag can contain any number of arbitrary types of Object. There can be multiple Goups in one Group. For example:

```xml
<Group name = "group0">
  <Image name = "test0" texture = "img" />
  <Image name = "test1" texture = "img" />
  <Group name = "group1">
    <Image name = "test2" texture = "img" />
  </ Group>
  <Image name = "test3" texture = "img" />
</ Group>
```

Where group0 is the parent of group1, and group1 is the parent of test2 inside it. In short, Group is to describe the different parent-child relationship between Object. Such as Object, you can operate one Group directly through its name:

```xml
<CallMethod model = "group0" method = "setScale" args = "2" />
<CallMethod model = "group1" method = "setRotationX" args = "30" />
```

All the children of group0 will follow to enlarge to 2 times, that is, test0, test1, test2, test3. And all the children of group1 will rotate with 30 degrees, that test2 will rotate 30 degrees. Regardless of the Group's children types, the Group's available functions have only a few basic functions. Group is essentially a number of Object to bulk operations to simplify the amount of code.
RootGroup is a special group, its statement is:

```xml
<RootGroup name = "root" />
```

RootGroup and Group are different in:
1. RootGroup under Object support "object events" will be described in detail in the following chapters;
2. RootGroup under the Object does not need to dispatchDraw drawWallpaper operation.


### 5. Script
"Script" is the script part, only in this tag can be the following two operations: 1. Call the member functions of one Object; 2. Call the system event functions. The system event functions contain six events functions, such as onTouchDown, which will be discussed later. Calling the method is in the corresponding Wallpaper label or Object tags.

Script section is the core of XCML Script. The so-called Bin Script is just the Script tag content replaced by a new wording. Each Object can have one Script tag, which is equivalent to a custom member function, and each member of the Object function only on the Object, because this language does not support "this" pointer, even in the Object of the Script must also use the Object name to call the member function.

**We strongly recommend using Script only in Wallpaper.**

## Zhe Script Basic syntax

### 1. Variables Basic types and arrays

### 2. Define variables

### 3. Define the method

### 4. Variable assignment

### 5. Call the custom method

### 6. Logical judgment

## Common Object and member function list

### 1. Basic Object and Group

### 2. Model

### 3. Image

## Global Static Functions and Static Variables ##

###1. Math

###2. Calculate

###3. ThemeCommonUtils

###4. SensorConverter

###5. Product

###6. TimeData

###7. Setting

### 8. Global static variables

## Model Import and Screen Adaptation ##

### 1. OBJ format

XCML Script supports the OBJ format exported by 3DS MAX. So if you are using other modeling software such as Maya, Camera4D, etc., please export a 3ds max can support the format, then export obj in 3ds max. 
Obj files exported by mainstream modeling software can be supported, but sometimes there will also be some problems. For example, Maya-exported obj must be triangulated, or there could be a bug that some areas cannot be displayed. The information about the position, size, rotation center, texture UV, etc. of the model will be saved. Please adjust these in the modeling software and export it.

**If a scene has more than one model, now in the modeling software to load all the models to adjust the size and position and then export one by one. In the XCML Script in the adjustment will be too cumbersome and may be the problem of adaptation.**

### 2. DAE format

XCML Script supports DAE format for 3DS MAX, Maya under the exported DAE format (tick triangulation algorithm, a single matrix). The animation of the currently derived DAE model supports only the rotation, scaling, translation of the whole object, and there may be instances where individual nodes are not parsed. The support model has nested relationships, and the support model has no texture and normal vector data. Note that all models that are not used in the scene must be deleted before exporting the DAE format. The vertex color attribute is not supported. Skeletal animation is not supported.

### 3. Model accuracy and texture size

Because the model file is initially read in at once, an oversized model can cause slower reads or even out of memory crashes. Therefore, the model file should be as small as possible, in the export to delete unnecessary information. Taking into account the low-end Andrews phone supports image size is limited, in the use of larger images (greater than 1920 * 1080), please store the smaller size and then zoom in the program operation.

### 4. Lighting and materials

Currently does not support the mainstream modeling software, lighting and materials, can only be achieved with GLSL shader. About the shader, the latter chapters are described in detail.

### 5. Screen adaptation issues and scenarios

Screen adaptation Android application has always been an inevitable big problem, as Android mobile phones vary too much, different brands of different models of mobile phone screen size and resolution gap. In 3d scenes, the effect of different sizes and resolutions may be completely different. In order to solve the adaptation problem, please try to avoid using px as the unit of operation and use dp as the unit of operation. To scale an Object, use the ThemeCommonUtils mentioned above to convert the values ​​and use them as function arguments. To reduce tedious operations, do not use too many animations that involve location information.

## Bin Script Syntax ##

Bin Script is currently being refined. Curly braces are not supported, so be sure to pay attention to code alignment. The current version, Bin Script can only be used within the Script tag and must start with a "<! [CDATA [" and at end with a "]]>". Such as:

```xml
<Script>
<! [CDATA [
  var test_your_codes_here = 0;
]]>
</ Script>
```

In the bottom of the program, Bin Script code will be translated into Zhe Script code.

### 1. The type and scope of the variable

In Zhe Script, Var is a custom local variable, GlobalVar is a global variable. Unlike Object, Var and GlobalVar are not constrained by Group. Bin Script, they correspond to the data types are var and gvar. Bin Script currently supports only three types of variables: int, float and boolean. Both int and float are cast to float during operation. Three variables are defined as follows:

```
var test_int = 1;
var test_float = 0.5f;
var test_boolean = true;
```

The float variable must end in "f". Boolean type must use "true" or "false" to define and assign values, does not support other assignment methods and does not support other types of coercion. Do not support long or double and other data types, maximum support only 4 bytes, please be careful not to produce a variable overflow.

The scope of a "var" variable is the tag or function within which the variable is located. In other words, if all scripts are written as required outside the event function under the script in the Wallpaper tag, all variables will be global. Like JS, "var" can be re-declared first. "gvar" is a global variable that can be declared in the entire XML file.

**We recommend that you assign an initial value to the variable declaration.**

### 2. The array

### 3. Event Functions

Because they do not support curly brackets, event functions need to be separated by semicolons, and the endfunction flag needs to be added at the end. Such as:

```
function onDrawStart ();
var test_your_codes_here = 0;
endfunction;
```

### 4. Operations

Unlike other languages, Bin Script calls an expression with the "exp", currently only supports four operations, does not support computing and other operations. In the brackets in "exp" to write the formula, such as:

```
var test0 = exp (1 + 1);
var test1 = exp (test0 * 10);
var test2 = exp (test0 / test1);
```

### 5. if / else / elseif / endif

Bin Script in the logic of judgment, such as:

```
var test0 = true;
var test1 = false;
var test_your_value = 0;

if (test0);
test_your_value = 1;
endif;

if (test0);
test_your_value = 1;
else;
test_your_value = 2;
endif;

if (test0);
test_your_value = 1;
else if (test1);
test_your_value = 2;
else;
test_your_value = 3;
endif;
```

Similarly, the need for each line to; end, conditional judgment must endif. Currently supported conditional operators are:>, <,! =,> =, <=. The negation operator and the bitwise operator are not currently supported. "float" and "int" will be cast to "float" when compared. "boolean" type can not be compared with "float" or "int". If followed by the parentheses can be added to the logic of judgment.

```
var test0 = 0;
var test1 = 1;

if (test0 <test1);
test0 = 2;
endif;
```

### 6. Function Calling

Call the function through the name of the Object, only need to add the same as in Java. It is worth mentioning that the atomic operation is not supported at the moment, that is to say, the function parameters must be a variable and not an expression. For cases where an expression is required, you must first define a variable to store the result of the operation and then call it. Correct function calling:

```
test.setScale (2);
var scale = exp (2 * 3);
test.setScale (scale);
```

## Event-driven programming

As mentioned above, the complex programming paradigm is the XCML Script difficult. Event-driven programming is commonly used in game programming paradigm, it refers to the implementation of the corresponding procedures for different events. In other words, most of the script is in a specific event. In addition to a specific event, only a number of variables such as statements of operation. The logical relationship between the code modules is the logical relationship under different events. XCML Script currently supports the following six types of event functions.

The general template for calling the event function is:

```
var test_your_value = 0;

function onDrawStart ();
test_your_value = 1;
endfunction;
```

Here "onDrawStart" is one of the six event functions, replacing it with other names is to call other event functions. Since there are only six fixed event functions, all functions do not have a parameter table or return value. Custom functions are not yet supported.

**In addition to these six kinds of events, in fact, there is a non-standard wording. In the script tag directly into some script, in some cases, these scripts may be run once in the initialization, that effect as written in init. We strongly recommend that this waiver is abandoned.**

### 1. Init / drawWallpaper

Each Object has an init method to do some initialization. As mentioned above, most of the properties can be defined in the Object declaration, but because the declaration does not support variables, so some operations must be done in the init, for example, let a test width of 540 full-screen Image. For example:

```
function init ();
var CanvasWidth = ThemeVariable.getCanvasWidth ();
var scaleUnit = exp (CanvasWidth / 540f);
test.setScale (scaleUnit);
endfunction;
```

Typically, the initialization is to carry out some screen adaptation operation. "drawWallpaper" is similar to "init", but also for some initialization, the difference is that only in the special label Wallpaper drawWallpaper call. If we follow the recommendations in the use of Script in Wallpaper, the function of the same drawWallpaper and init. DrawWallpaper There is also a special feature that is commonly used in the above mentioned "dispatchDraw". The following code shows the use of a group called group dispatchDraw, if you do not use the function will lead to the group of all Object can not be displayed.

```
function drawWallpaper ();
group.dispatchDraw ();
endfunction;
```

**We recommend that all initialization operations are done in the drawWallpaper instead of using "init".**

### 2. OnDrawStart

"onDrawStart" in XCML Script is the core of the incident, the vast majority of operations are carried out onDrawStart. Equivalent to many other languages ​​in the update function, onDrawStart is called by frame, which means that usually this function is to be called 60 times per second. As the core, onDrawStart is usually the culprit of the 3D theme caused by Caton. OnDrawStart performs more operations will bring more performance loss, it should be skillfully avoided in the onDrawStart in too many operations.

### 3. OnDesktopEffectStart / onDesktopEffectEnd

These two functions are called at the start and end of the desktop slideshow. Most of the time, you need to work with a boolean variable to control a 3D model, for example:

```
Var isDesktopEffectRunning = false;
Function onDesktopEffectStart ();
IsDesktopEffectRunning = true;
Endfunction;
Function onDesktopEffectEnd ();
IsDesktopEffectRunning = false;
Endfunction;
```

### 4. OnIconStartDrag / onIconEndDrag

Similar to onDesktopEffectStart / onDesktopEffectEnd, this pair of functions are called at the beginning and end of the drag-and-drop desktop icon.

### 5. OnTouchMove / onTouchDown / onTouchUp

As with their names, this is a function of the touch of the response.
OnTouchDown Called at the beginning of the touch
OnTouchUp Called at the end of touch
OnTouchMove Called when the touch moves, can be used to capture the touch track, at the beginning and end of the touch will be called. It is typically used to implement gesture-following effects.
The corresponding onTouchDown_x / onTouchDown_y, onTouchUp_x / onTouchUp_y, onTouchMove_x / onTouchMove_y, three pairs of global static variables, the corresponding events in the corresponding call to achieve the required variables.

For example:


### 6. OnSensorChanged

When the sensor is changed, the most commonly used is to get the rotation angle of the current gyroscope. In our test process, sensor proved to be the most power, so if the consideration for power, you can not use the sensor. In the bottom, sensor is called once every 3, which is to save power, so careful observation will be slightly not smooth. As the current engine and no camera class, most of the 3D theme of the three-dimensional sense to rely on the sensor to achieve. Similar to the touch event function, the event function also has a dedicated static variable.
For example:


## Special Object

### 1. ValueInterpolate

### 2. WaveInterpolate

### 3. Timer

### 4

## Animation classes

## Advanced classes

## Particle system related classes

## GLSL SHADER

## Common templates