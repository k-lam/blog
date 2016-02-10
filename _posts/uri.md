
	Uri uri = new Uri.Builder().scheme("dabiaoge").authority("homepage").fragment("promotion").path("uu").query("user=kl").build();

	//结果：dabiaoge://homepage/uu?user%3Dkl#promotion


Uri :

Scheme://authority/[path]['?'query]['#'fragment]

其中authority是[ userinfo '@' ] host [ ':' port ]


###Intent
Intent可以直接设置Uri

	Intent intent = new Intent();
    Uri uri = new Uri.Builder().scheme("dabiaoge").authority("homepage").build();
    intent.setData(uri);
    startActivity(intent);
    
Intent filter:

		<intent-filter>
            <action android:name="push_open"/>
            <category android:name="android.intent.category.DEFAULT"/>
             <data
                 android:scheme="dabiaoge"
                 android:host="homepage" />
        </intent-filter>
        
注意:因为要用startActivity打开，所以必须加上` <category android:name="android.intent.category.DEFAULT"/>
`

>Note: In order to receive implicit intents, you must include the CATEGORY_DEFAULT category in the intent filter. The methods startActivity() and startActivityForResult() treat all intents as if they declared the CATEGORY_DEFAULT category. If you do not declare it in your intent filter, no implicit intents will resolve to your activity.

这种通过Uri跳转的方式和传统的方式，一个重大的区别在于参数方面，Uri是通过queryParams来传递参数的如
	
	dbg://homepage/watch?brand=tissot&gender=male
	
通过query来传参是没有类型的，而intent.put的方式是有类型的。解决方案是，在接收方用两套方法去获取参数，有Uri的情况（intent.getData()!= null）和没uri的情况

###Intentfilter
android中怎么用Uri对应应用的跳转呢？用IntentFilter.
这样的好处是，解耦，不用在Uri中指定明确地Activity的名字！

譬如我mainframe下面的一个叫promtion的tab，我们可以做如下约定的Uri
dbg://homepage/promtion



####[#黑科技# 跳出浏览器](http://zhuanlan.zhihu.com/andlib/19848910)





