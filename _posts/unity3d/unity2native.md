# 概述

### [IL2CPP](https://zhuanlan.zhihu.com/p/19972689)



http://www.jianshu.com/p/fa133b61fdf8



# Android

[官网：JAR plug-ins](https://docs.unity3d.com/Manual/AndroidJARPlugins.html)

[5、与iOS、Android的交互 实践篇——主动调用](http://www.jianshu.com/p/83c5736007f6)

[Unity3D研究院之与Android相互传递消息（十九）](http://www.xuanyusong.com/archives/676)

### 1. android向unity发送信息

`UnityPlayer.UnitySendMessage(String, String, String)` 参数1表示发送游戏对象的名称，参数2表示对象绑定的脚本接收该消息的方法，参数3表示本条消息发送的字符串信息

返回值：无
函数类型：静态函数，直接调用即可（Java中包含在com.unity3d.player.UnityPlayer中）
参数1：场景内某GameObject物体的名称
参数2：该GameObject物体上组件所挂载的函数名
参数3：调用该函数时传入的参数



### 2. unity调用android的方法

通过jni调用

### 3. unity直接调用ndk build的so库

[传送门](https://docs.unity3d.com/Manual/AndroidNativePlugins.html)



# IOS

https://docs.unity3d.com/Manual/PluginsForIOS.html





测试：

1. unity 作为原生的一个插件，尽量少导入
2. 原生调用unity unity调用原生
3. unity多sense，原生切换到后台 unity是否能恢复





后续：

1. 工程化：现在每次修改unity，都要输出一次代码项目，原生再导入，build  相当耗时。对于andriod，现在  gradle可以有模板 manifest可以有模板 UnityPlayerActivity可以自定义，放到unity中

   android对这个最好的做法是，插件！unity build一个app，作为一个插件app。

2. Debug.log会输出到native的log（android确定，连函数栈都有）

3. unity可以直接调用原生的库，so，jar。但是打包的时候  必须不重复打入。