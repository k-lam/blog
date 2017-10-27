目标：自己写一个生成代码的插件



[参考链接](http://www.infoq.com/cn/articles/android-in-depth-gradle)



Gradle中，每一个待编译的工程都叫一个Project。每一个Project在构建的时候都包含一系列的Task。比如一个Android APK的编译可能包含：**Java源码编译Task、资源编译Task、JNI编译Task、lint检查Task、打包生成APK的Task、签名Task等**。



一个Project到底包含多少个Task，其实是由编译脚本指定的插件决定。插件是什么呢？插件就是用来定义Task，并具体执行这些Task的东西。



Gradle是一个框架，作为框架，它负责定义流程和规则。而具体的编译工作则是通过插件的方式来完成的。比如**编译Java有Java插件，编译Groovy有Groovy插件，编译Android APP有Android APP插件，编译Android Library有Android Library插件**



每一个Library和每一个App都是单独的Project。根据Gradle的要求，每一个Project在其根目录下都需要有一个build.gradle。build.gradle文件就是该Project的编译脚本



在Gradle中，这叫**Multi-Projects Build**。把posdevice改造成支持Gradle的Multi-Projects Build很容易，需要：

- 在posdevice下也添加一个build.gradle。这个build.gradle一般干得活是：配置其他子Project的。比如为子Project添加一些属性。这个build.gradle有没有都无所属。
- 在posdevice下添加一个名为settings.gradle。这个文件很重要，名字必须是settings.gradle。它里边用来告诉Gradle，这个multiprojects包含多少个子Project。



```
//通过include函数，将子Project的名字（其文件夹名）包含进来  
include  'CPosSystemSdk' ,'CPosDeviceSdk' ,  
       'CPosSdkDemo','CPosDeviceServerApk','CPosSystemSdkWizarPosImpl'  
```

**问题：include是什么？闭包？函数？**

Groovy中函数调用的时候还可以不加括号。比如：

```
println("test") ---> println "test"
```

另外：Groovy支持函数调用的时候通过  参数名1:参数值2，参数名2：参数值2 的方式来传递参数，如

```
apply plugin: 'com.android.application'
```

另外，settings.gradle除了可以include外，还可以设置一些函数。这些函数会在gradle构建整个工程任务的时候执行，所以，可以在settings做一些初始化的工作。



**问题：setting.gradle 和root project的build.gradle里面的执行顺序**





- **每一个Project都必须设置一个build.gradle文件。至于其内容，我们留到后面再说**。
- **对于multi-projects build，需要在根目录下也放一个build.gradle，和一个settings.gradle**。
- **一个Project是由若干tasks来组成的，当gradle xxx的时候，实际上是要求gradle执行xxx任务。这个任务就能完成具体的工作**。



## **gradle命令介绍**

* gradle projects



Task和Task之间往往是有关系的，**这就是所谓的依赖关系。比如，assemble task就依赖其他task先执行，assemble才能完成最终的输出**。







### 4.3  Gradle工作流程

Gradle的工作流程其实蛮简单，用一个图26来表达：

![img](http://cdn3.infoqstatic.com/statics_s1_20161220-0322/resource/articles/android-in-depth-gradle/zh/resources/image028.png)

 图26告诉我们，Gradle工作包含三个阶段：

- 首先是初始化阶段。对我们前面的multi-project build而言，就是执行settings.gradle
- Initiliazation phase的下一个阶段是Configration阶段。
- Configration阶段的目标是解析每个project中的build.gradle。比如multi-project build例子中，解析每个子目录中的build.gradle。在这两个阶段之间，我们可以加一些定制化的Hook。这当然是通过API来添加的。
- Configuration阶段完了后，整个build的project以及内部的Task关系就确定了。恩？前面说过，一个Project包含很多Task，每个Task之间有依赖关系。Configuration会建立一个有向图来描述Task之间的依赖关系。所以，我们可以添加一个HOOK，即当Task关系图建立好后，执行一些操作。
- 最后一个阶段就是执行任务了。当然，任务执行完后，我们还可以加Hook。





Gradle基于Groovy，Groovy又基于Java。所以，Gradle执行的时候和Groovy一样，会把脚本转换成Java对象。Gradle主要有三种对象，这三种对象和三种不同的脚本文件对应，在gradle执行的时候，会将脚本转换成对应的对端：

- Gradle对象：当我们执行gradle xxx或者什么的时候，gradle会从默认的配置脚本中构造出一个Gradle对象。在整个执行过程中，只有这么一个对象。Gradle对象的数据类型就是Gradle。我们一般很少去定制这个默认的配置脚本。
- Project对象：每一个build.gradle会转换成一个Project对象。
- Settings对象：显然，每一个settings.gradle都会转换成一个Settings对象。

注意，对于其他gradle文件，除非定义了class，否则会转换成一个实现了Script接口的对象。这一点和3.5节中Groovy的脚本类相似





Task是Gradle中的一种数据类型，它代表了一些要执行或者要干的工作。不同的插件可以添加不同的Task。每一个Task都需要和一个Project关联。

Task的API文档位于**https://docs.gradle.org/current/dsl/org.gradle.api.Task.html**。关于Task，我这里简单介绍下build.gradle中怎么写它，以及Task中一些常见的类型



关于Task。来看下面的例子：

[build.gradle]

```
//Task是和Project关联的，所以，我们要利用Project的task函数来创建一个Task  
task myTask  <==myTask是新建Task的名字  
task myTask { configure closure }  
task myType << { task action } <==注意，<<符号是doLast的缩写  
task myTask(type: SomeType)  
task myTask(type: SomeType) { configure closure } 
```

上述代码中都用了Project的一个函数，名为task，注意：

- 一个Task包含若干Action。所以，Task有doFirst和doLast两个函数，用于添加需要最先执行的Action和需要和需要最后执行的Action。Action就是一个闭包。
- Task创建的时候可以指定Type，通过**type:名字**表达。这是什么意思呢？其实就是告诉Gradle，这个新建的Task对象会从哪个基类Task派生。比如，Gradle本身提供了一些通用的Task，最常见的有Copy 任务。Copy是Gradle中的一个类。当我们：**task myTask(type:Copy)**的时候，创建的Task就是一个Copy Task。
- 当我们使用 **task myTask{ xxx}**的时候。花括号是一个closure。这会导致gradle在创建这个Task之后，返回给用户之前，会先执行closure的内容。
- 当我们使用**task myTask << {xxx}**的时候，我们创建了一个Task对象，同时把closure做为一个action加到这个Task的action队列中，并且告诉它“最后才执行这个closure”（**注意，<<符号是doLast的代表**）。





*Task是一个类*  [link](https://docs.gradle.org/2.14.1/userguide/custom_tasks.html)





每一个build.gradle文件都会转换成一个Project对象。在Gradle术语中，Project对象对应的是**Build Script**。

Project包含若干Tasks。另外，由于Project对应具体的工程，所以需要为Project加载所需要的插件，比如为Java工程加载Java插件。**其实，一个Project包含多少Task往往是插件决定的。**



- 加载插件。
- 不同插件有不同的行话，即不同的配置。我们要在Project中配置好，这样插件就知道从哪里读取源文件等
- 设置属性。



```
apply plugin: 'com.android.application'  <==如果是编译Android APP，则加载此插件
```

除了加载二进制的插件（上面的插件其实都是下载了对应的jar包，这也是通常意义上我们所理解的插件），还可以加载一个gradle文件

**问题来了，插件究竟是什么？**



BuildScript又是什么  和Groovy怎么关联起来？

build.gradle就是BuildScript



![img](http://cdn3.infoqstatic.com/statics_s1_20161220-0322/resource/articles/android-in-depth-gradle/zh/resources/image037.png)

 你看，subprojects、dependencies、repositories都是SB。那么SB到底是什么？它是怎么完成所谓配置的呢？

仔细研究，你会发现SB后面都需要跟一个花括号，而花括号，恩，我们感觉里边可能一个Closure。由于图34说，这些SB的Description都有“Configure xxx for this project”，**所以很可能subprojects是一个函数，然后其参数是一个Closure。是这样的吗？**

Absolutely right。只是这些函数你直接到Project API里不一定能找全。不过要是你好奇心重，不妨到**https://docs.gradle.org/current/javadoc/**，选择**Index**这一项，然后**ctrl+f**，输入图34中任何一个Block，你都会找到对应的函数。





build.gradle中，如android{} dependence{}这些只是调用函数也就是configurate的时候执行，然后具体的excute过程，是参与不到？

