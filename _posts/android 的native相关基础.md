###最终运行
而c\c++和java不同，java是运行在虚拟机。而c\c++写成的代码，最终由java机调用，但是直接由cpu运行，因为c\c++  build出来的是机器可以识别的机器码。


###交叉编译
首先，要理解android开发是交叉编译的。c的代码开发也是。最终运行在android的智能手机上面。


###编译连接
c\c++的编译过程包括：单文件的编译->连接->最终output,参考[这里](http://www.cnblogs.com/hongfenglee/archive/2012/02/18/2356808.html)

#####由于交叉编译、c/c++运行和java的区别和c/c++编译特性，决定了android用ndk开发时，开发环境中需要头文件和所有实现代码，在真机中，只需要有.so文件，给虚拟机调用就可以了。

###头文件和定义文件
可以的话，直接参考`<ndk>/docs`,非要参考头文件和源码的，The headers corresponding to a given API level are now located under $NDK/platforms/android-/arch-arm/usr/include

具体可以参考`docs/STABLE-APIS.html`

几个重要的库

* [STL](http://baike.baidu.com/subview/332356/10428592.htm?fr=aladdin) : `<ndk>/sources/cxx-stl`




###c/c++知识补充

####静态库、共享库、动态库
函数库可以看做是事先编写的函数集合，它可以与主函数分离，从而增加程序开发的复用性。Linux中函数库可以有3种使用的形式：静态、共享和动态。

1. 静态库(.a、.lib)的代码在编译时就已连接到开发人员开发的应用程序中；
2. 而共享库(.so)只是在程序开始运行时才载入；
3. 动态库也是在程序运行时载入，但与共享库不同的是，动态库使用的库函数不是在程序运行使开始载入，而是在程序中的语句需要使用该函数时才载入。动态库可以在程序运行期间释放动态库所占用的内存，腾出空间供其他程序使用。


###原生代码的执行是不受虚拟机控制的