首先，要能观察到task的情况，命令如下：

adb shell dumpsys activity activities | sed -En -e '/Running activities/,/Run #0/p'

然后参考这篇文章：[[解开Android应用程序组件Activity的"singleTask"之谜](http://blog.csdn.net/luoshengyang/article/details/6714543)](http://blog.csdn.net/luoshengyang/article/details/6714543)

感谢罗升阳大神，android文档骗人！



最后要知道，如果要结束一个task的所有Activity,然后打开新的Activity，用FLAG_ACTIVITY_CLEAR_TASK，一个坑是

​	Intent intent = new Intent(ActivityA.this, ActivityB.class);

必须要：

​	intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK);



这样是不行的：                 

​	intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

​	intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK );

呵呵呵，这个语句的结果是，结束和B相同Task的所有activity！怎么放到同一个task？看罗升阳大神上面那个链接。taskAffinity并不能通过编程设置，除非你hook掉。。。