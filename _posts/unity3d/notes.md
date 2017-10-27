## 物体

物体包括很多内容，如位置信息，外貌，物理属性等

### 物体的外貌

外貌由外壳与渲染构成

#### 外壳：

外壳就是形状（geometry），不过外壳这个词更准确，因为3d的这个模型只是构建了物体的外貌。

2d世界，两点决定一条直线，有限条线段能画出一个形状。3d里面，三个点决定一个平面，有限的平面的拼凑能拼出一个物体的外壳

unity里面通过Mesh Filter来描述外壳（为什么叫网格过滤器？google一下Mesh Filter的图片就知道）

#### 渲染render：

有了外壳，怎么显示？

unity里面用mesh render组件，mesh render包括Lighting 和 material

而material只有一个东西 shader！

这样说吧，render这个模型就是  光照到一个材质上  是怎样显示的



unity里面内置了很多shader，shader是可编程的，如StandardShader，是基于物理着色的。

基于物理着色（Physically Based Shading，PBS）就是遵循了Conservation of Energy（能量守恒定律）。只有这样，才能够真正让场景里面的光源依照能量传递的方式，产生一种自然和谐的光照效果



#### [rendering pipeline](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview)

The **Rendering Pipeline** is the sequence of steps that OpenGL takes when rendering objects. 



#### Shader

Shader（着色器）实际上就是一小段程序，它负责将输入的Mesh（网格）以指定的方式和输入的贴图或者颜色等组合作用，然后输出。绘图单元可以依据这个输出来将图像绘制到屏幕上。输入的贴图或者颜色等，加上对应的Shader，以及对Shader的特定的参数设置，将这些内容（Shader及输入参数）打包存储在一起，得到的就是一个Material（材质）。之后，我们便可以将材质赋予合适的renderer（渲染器）来进行渲染（输出）了。

Shader只是一段规定好输入（颜色，贴图等）和输出（渲染器能够读懂的点和颜色的对应关系）的程序。而Shader开发者要做的就是根据输入，进行计算变换，产生输出而已。



#### [Unity’s Rendering Pipeline](https://docs.unity3d.com/Manual/SL-RenderPipeline.html)

Shaders define both how an object looks by itself (its material properties) and how it reacts to the light. 



# ShaderLab: Pass Tags

Passes use tags to tell how and when they expect to be rendered to the rendering engine.



### Question

shader：
什么时候调用  谁调用？为什么输入参数可以自定义？

什么时候调用和谁调用这个可以分开两类shader分别讨论



首先，要记住pipeline工作的，也就是拆解成很多步骤（rendering state？）而每一个步骤其实都是对这个object的加工（vertex和fragment？），所以做法是，pass定义这段代码是那个步骤调用的。然后很多pass共同完成渲染？



[【Unity Shaders】Vertex & Fragment Shader入门](http://blog.csdn.net/candycat1992/article/details/40212735)



cg nvidia 出的书 "CgUsersManual"



https://developer.nvidia.com/cg-toolkit



https://en.wikibooks.org/wiki/Cg_Programming



http://www.unity.5helpyou.com/2830.html



unreal 用hlsl作为shader语言



各种缓冲区：

http://blog.sina.com.cn/s/blog_78c5ff950102vlf1.html