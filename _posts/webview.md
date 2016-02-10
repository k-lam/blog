#WebView

[webkit包](http://developer.android.com/reference/android/webkit/package-summary.html)

[Building Web Apps in WebView](http://developer.android.com/guide/webapps/webview.html)

[WebView](http://developer.android.com/reference/android/webkit/WebView.html)

##onPageStarted  shouldOverrideUrlLoading  onPageFinished  onLoadResource  onReceivedTitle

一般情况下（没有重定向，手机正常，操作系统是原生）
loadUrl(url_abc)
执行顺序

onPageStarted ->

onLoadResource(先是这个url的，再是这个url_abc指向的html下的各个资源) ->

onReceivedTitle (onLoadResource url_abc之后，所有resources之前)->

onPageFinished

但是个别抽风的手机，如华为荣耀6，onPageFinish会多次被调用，这是手机自己改了浏览器webview的问题！

存在重定向的情况：
url_abc重定向到url_def

onPageStarted (url_abc)->

onPageStarted (url_def)->

shouldOverrideUrlLoading(url_def)->

之后一样
但是这时在onPageStarted (url_def)之后，getUrl就变成url_def，但是可以通过getOriginalUrl获得url_abc

##onProgressChanged
多用来显示进度条，当progress = 100时，调用onPageFinished，但某些修改了webview的机，如华为，还是会像多次调用onPageFinished那样，多次progress = 100，抽风！

##load layout prase render
layout 应该是view的layout，就是android view 框架那套
而显示一个网页，其实包括load，prase  render这些步骤，但是，google都没有给出回调！！！

直到api23（6.0）多了下面这些接口

	WebViewClient；
	onPageCommitVisible
	postVisualStateCallback

不提供回调，但必须注意到有这些过程的存在，因为如果html写得不好，这些prase  render是很耗时间的。而且注意到，onPageStart,onPageFinish,onProgressChanged,onLoadResource,指的是load过程！而且onPageStart,onPageFinish指的是main frame的load。正是prase慢，用户会有明显的延迟感（如果不是都不用做原生了，我们这些做原生的可以去吃屎啦）

##history stack
这个stack是浏览器的内容，就是一个list，和一个current指针。

而且这个stack不能用编程方式write，只能read。

goBack和goForward操作只是改变current指针，不会删除list，每一次跳转都会增加一个item。

goBack和goForward操作相当于loadUrl，但是很多机型的onReceiveTitle不会被再次调用，其他callback应该和原来的无差别

重定向的originalURL不会放到这个list中

##InterceptRequest

	            @Override
            public WebResourceResponse shouldInterceptRequest(WebView 		view, String url) {

	                if(url.contains("http://img213.wbiao.co/up/	201509/09/144176474237082.jpg")){
   	                 try {
	  	                 WebResourceResponse resourceResponse = new 	WebResourceResponse("image/jpeg","utf-8",getAssets().open("inter.jpg"));
                        return resourceResponse;
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                return super.shouldInterceptRequest(view, url);
            }
