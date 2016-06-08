我们在编译和打包应用程序资源的过程中，会生成一个resources.arsc文件，这个文件记录了所有的应用程序资源目录的信息，包括每一个资源名称、类型、值、ID以及所配置的维度信息。我们可以将这个resources.arsc文件想象成是一个资源索引表，这个资源索引表在给定资源ID和设备配置信息的情况下，能够在应用程序的资源目录中快速地找到最匹配的资源。

resources.arsc格式？

http://blog.csdn.net/luoshengyang/article/details/8744683

http://blog.csdn.net/luoshengyang/article/details/8791064


在Android源代码工程环境中，Android系统提供的资源经过编译后，就位于out/target/common/obj/APPS/framework-res_intermediates/package-export.apk文件中，因此，在Android源代码工程环境中编译的应用程序资源，都会引用到这个package-export.apk

源ID是一个4字节的无符号整数，其中，最高字节表示Package ID，次高字节表示Type ID，最低两字节表示Entry ID。

PTEE

Package ID相当于是一个命名空间，限定资源的来源。Android系统当前定义了两个资源命令空间，其中一个系统资源命令空间，它的Package ID等于0x01，另外一个是应用程序资源命令空间，它的Package ID等于0x7f。所有位于[0x01, 0x7f]之间的Package ID都是合法的，而在这个范围之外的都是非法的Package ID。

关于资源ID的更多描述，以及资源的引用关系，可以参考frameworks/base/libs/utils目录下的README文件。
关于Android系统的资源覆盖（Overlay）机制，可以参考frameworks/base/libs/utils目录下的READ文件。