#背景
1. 万表网是电商，电商讲求**强运营**，经常要发布新内容，如新的活动。
2. 快速开发，上线！是很重要的能力
2. 修bug
3. app大到一定的程度，编译都要很久，这时候要考虑**插件化**。

对上面的情况，我们提出，app必须要有动态的能力。所谓动态的能力是指，突破重新安装新app才能修改"内容"的限制，下面就说说几种实现方式

#实现
##一些实现方式简介
1. 模板，配置
2. html
3. 脚本，如lua
4. 热部署，如插件，osgi

###模板，配置
这个应该是大多数强调强运营都具有的能力。譬如首页提供几种写死的展示模板，运营人员在后台配置不同的图片，文字，然后app每次打开之前都先请求服务器，然后显示最新内容。

* 优点：实现简单，写好一次后，不容易出bug
* 缺点：表现能力有限，紧限于那几种模板，要添加，修改新模板，要重新安装。

###html
就是混合开发了，这也是很多电商app选择的方式，特别是商品详情页。京东，天猫也选用这种方式。

* 优点：html的开发比原生快很多，而且有bug可以马上修复
* 缺点：网速，网速，网速。在网速差的情况下，就是一整屏的白屏，而交互，html肯定比不上原生，html和原生交互需要联合测试

但是对于缺点，也是有解决方法的。譬如网速慢，可以用spdy协议，又譬如第一屏用原生，以下的内容才用html

###脚本语言
游戏是经常要修改内容的，而他们用的是lua，但是他们是引擎带了解释器。而android原生方面，android使用lua，几个开源框架都没有很多的支持，所以还是不敢用于商业项目。而且团队没有人会lua，意味着，要么招人，要么自己学习，需要时间上手。

###插件化，热部署
这个是已有实现方案的，把这些方案分为两类

1. 360的DroidPlugin
2. 其他通过classloader来导入其他包的类来实现动态

360的很有趣，可以不安装就能执行。他们是hook掉了ActivityThread的Handler等一系列处理Intent的来实现不安装就能执行的

而classloader能实现热部署，是因为java语言的类"连接"机制是发生在运行阶段，不像c，是编译时间。简单说一下，java代码通过javac编译后，只是包含了自己的类代码，并没有连接引用类的代码。那依赖的类是怎样寻找（连接）的呢？就是通过运行的时候，通过classloader来把依赖的类代码连接过来的。而且尽管load同一个类，用不同classloader来load，是被判断为不是同一个类的！

###其他
其实阿里还有一个热修复框架[AndFix](https://github.com/alibaba/AndFix)，但是热修复和插件化不是相同的事物

##万表的实现
万表选择使用混合开发加插件化。为什么选择这样的方案？其实问题是什么才是最好的方案？当然是最适合团队的方案。这根每个公司业务特点，人员配置，技术特长有关。

###混合开发
混合开发就是js和原生代码能通讯，上网搜一下jsbridge就能找到，或者研究微信的js代码。其实就是建立js向java发送信息的通道（iframe.src或者alert），java向js发送信息（webview.loadurl("javascript:")）的信息通道。两边建立消息队列。

然后java定义向js提供的接口。

但是webview是提供的callback是很有限的。譬如Document什么时候load完，是没有提供的。但是js可以通知java什么时候document已经load完了。而java需要通知html，onPause,onStop,onResume等这些事件。

####原生向js提供什么接口？

* 分享
* 支付
* 网络访问
* header定制
* 其他一些本地的控件，如loading
* 等等

注意，html有域的问题。所以如果能用原生的网络访问，那就更自由了，而且可以复用原生的连接。而且可以统一管理。譬如2g 3g的情况下访问不同质量的图片，或者启用SPDY协议（WebView好像在4.0以后才支持SPDY）。做法就是`shouldInterceptRequest`这个方法，但是这个方法返回的是`WebResourceResponse`,不存在回调，也就是网络访问的时候必须阻塞！我们的做法是，如果resource不在缓存中，返回占位图，然后异步访问资源，再通过JavaScript通知重新设置资源,暂时只用于图片。譬如图片url如下
	
	<img id="img1" src="/image/watch-tissot?intercept=true&id=img1">

在`shouldInterceptRequest`中检查uri，如果有intercept=true,就马上返回一个占位图片，获取id，然后用okHttp或者其他的，当网络回调，通过js 
	
	document.getElementById("img1").src='/image/watch-tissot?intercept=true&cached=true'

这样就又到`shouldInterceptRequest`这次就返回已经获取到的图片。注意，不能直接src='file:///'这种，因为存在跨域问题。

而万表中，没有用到上面这种。而是把部分html页面本地化。首先，我们是这样的

* 前后端分离，也就是HTML页面是显示框架，具体数据动态获取（SEO不友好）
* 只把HTML缓存在本地，动态获取的资源由前端人员去做，因为这样比较简单，ios，android，手机网页端，可以共用一套代码。

这样做的好处是，html放在本地，能很快读取到，不受网络影响，页面框架先出来了。当然，第二点也是可以用intercept的方法去做，只是因为万表没那么多人手，所以没有这么去做


####js向原生提供什么
* document load完的通知，因为onPageFinish()这个方法。。。经常很久都不能返回

jsbridge图解：
![jsbridge图解](https://raw.githubusercontent.com/k-lam/blog/gh-pages/image/jsbridge.jpg)

另外，如果要移动站和app端用同一套代码，那关于怎么做到seo友好，但是又静态页面加载动态内容，这一块真的不了解。

###组件化
我们是打算做成组件，各组件之间是通过Intent或EventBug来通讯。不直接进行代码调用，这样组件间就能解耦了。暂时没有做进程隔离。

我们对代码进行如下归类：

1. wbSDK
2. MainFrame
3. 组件1，组件2...

把MainFrame和wbSDK合并。如果是这两部分进行修改，需要重新发布新版本。各个组件依赖于wbSDK。如果团队有人力，可以研究编译时，怎么把组件的wbSDK代码不要打包到dex里面。

知道通过classloader来load组件apk或dex的类后，还不够，难道每次用到这个类都通过classLoader吗？这显然对组件的编程人员是很痛苦的，而且有些类，如Activity是通过系统去新建的，虽然通过代理模式可以解决问题，但是也是编程起来很麻烦。有没有办法告诉系统，这个class就是从这个组件apk里读取了？显然是有这样的办法。就是osgi！而android里面其实不需要这么麻烦，因为我们发现Application里面有一个classLoader，而这个classLoader是PathClassLoader！我们注意到，MultiDex这个东西，是google提供的，因为默认一个dex只能有65536个方法，为了突破这个限制，google搞出了multiDex这东西，具体[链接](http://developer.android.com/tools/building/multidex.html).我们可以参照MultiDex的源码实现。其实就是把组件的dex路劲添加进PathClassLoader.这样，组件中的java类就能被解析到了。

对于Resource的解决方法，网上很多，也是把路径写入AssetManager里面。这个上网搜索一下就好了。

另外，每个组件都应该有版本号！！！这个版本号通常是给后台接口和html用的。





