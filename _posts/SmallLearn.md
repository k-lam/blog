
###加载插件

Small.setUp这个方法就是加载插件的，后台线程LoadBundleThread加载。具体看代码

通过apktool反编译，可以看到，在打包时，譬如package name是 com.a.b.c,Small会在/lib/armeabi/目录下生成一个libcom_a_b_c.so的apk，然而这个apk貌似是没有证书的

也就是说，就是第一次打包，Small也是打插件包。

插件包的class，只有自己的类，compile project的没有放到自己的classes.dex中（反编译看过了）

###aapt
[查看apk包的内容](http://blog.csdn.net/sodino/article/details/6122665)

	aapt dump resources xxx.apk


###dexdump

	如：dexdump classes.dex | grep 'Class descriptor'  //查看所有class名字
	
	
	
	1. 自定义transform，剥离AAR (知识点，看资料花了很长时间。。AppPlugin.groovy#91)
2. hook ProGuard task,  加入混淆配置，加入公共库引用（难点，看源码+猜测，用了一些私有API，AppPlugin.groovy#294）
3. 延迟分离R.class （坑点。。AppPlugin.groovy#298)

第一点，可以看下@杭州-区长 的这篇文章，哈哈 http://blog.csdn.net/sbsujjbcy/article/details/50839263




一个app plugin  如果依赖了appframework  就会出错

setUp放在一个application.onCreate的Handler.post里  就ok

最新情况，一个hybird不会，但是加上test，且依赖appframework就会有问题！

hybird + test(不依赖appframework) 无问题
这时候应该对hybird和test进行横向对比。注意，test本来无多少代码和内容，从test入手比较简单
空的test也有问题，只要依赖了

resource问题！！！！！！！

Maybe https://github.com/wequick/Small/issues/67

aar resource 合并 r.id的生成！


加载完依赖appframework的插件后
design包string not found
support v7 layout not found
appframework not found
app not found

如果没有依赖appframework  是ok的
用R.string.app_name 做测试

全部都有问题！

DevSimple 可以在onComplete前后都找到support包的资源  自己包的也能找到