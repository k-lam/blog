注意，如果两个Activity位于不同task，切换时，跟处于同一个task的两个Activity切换的动画效果是不同的

singleInstance

4.2后，切换动画是像iphone的由下至上 

A1->A2(singleInstance)->A3

A3按back->A1(A2被跳过了)

但是如果A3是singleInstance，
A1->A2->A3
back是：A3->A2->A1

singleInstance给人的感觉就像一个流程，中间插入的打断一样。譬如购物，可能判断到你未登录，所以在购物流程中，插入一个登录的交互操作，当然，这个例子中用startActivityForResult。

singleTop:
A1 is singleTop
A1->A1,onNewIntent is call
singleTop的可以有多个实例，只是当本来就在task top的，再次startActivity，还是这个Activity时，就不再创建一个Activity了，只是把Intent传给onNewIntent 

用setFlag设置为FLAG_ACTIVITY_NEW_TASK，与在Activity的launchMode中设为singleTask，实际上表现出来的行为不一样。所以要看看，这究竟在源码实现时，是怎么一回事

singleTask

用getTaskId,发现，并没有新开一个task

A2是singleTask，
A1-A2-A3-A4-A2,这时back，A2->A1

A2 A4 是singleTask,A3 Ax是FLAG_ACTIVITY_NEW_TASK

(A1 Ay) (A2 A3) (A4 Ax),同一括号里的taskAffinity一样

A1-A2-A3-A4-Ax-Ay

Ay返回，会直接到A1

再由A1进入A2，注意了，A3 destroy，A2 newIntent。。。。下面是操作结果记录

	04-10 15:50:40.316: D/debug(26323): class backStack.A1-->onCreate
	04-10 15:50:40.322: D/debug(26323): class backStack.A1-->onResume
	04-10 15:50:47.533: D/debug(26323): class backStack.A2-->onCreate
	04-10 15:50:47.535: D/debug(26323): class backStack.A2-->onResume
	04-10 15:50:48.924: D/debug(26323): class backStack.A3-->onCreate
	04-10 15:50:48.926: D/debug(26323): class backStack.A3-->onResume
	04-10 15:50:50.520: D/debug(26323): class backStack.A4-->onCreate
	04-10 15:50:50.521: D/debug(26323): class backStack.A4-->onResume
	04-10 15:50:51.809: D/debug(26323): class backStack.Ax-->onCreate
	04-10 15:50:51.810: D/debug(26323): class backStack.Ax-->onResume
	04-10 15:50:53.018: D/debug(26323): class backStack.Ay-->onCreate
	04-10 15:50:53.019: D/debug(26323): class backStack.Ay-->onResume
	04-10 15:50:55.073: D/debug(26323): class backStack.Ay-->onBackPressed
	04-10 15:50:55.113: D/debug(26323): class backStack.A1-->onResume
	04-10 15:50:55.382: D/debug(26323): class backStack.Ay-->onDestroy
	04-10 15:50:56.837: D/debug(26323): class backStack.A3-->onDestroy
	04-10 15:50:56.858: D/debug(26323): class backStack.A2-->onNewIntent
	04-10 15:50:56.859: D/debug(26323): class backStack.A2-->onResume
	04-10 15:50:58.038: D/debug(26323): class backStack.A3-->onCreate
	04-10 15:50:58.039: D/debug(26323): class backStack.A3-->onResume
	04-10 15:50:59.727: D/debug(26323): class backStack.Ax-->onDestroy
	04-10 15:50:59.743: D/debug(26323): class backStack.A4-->onNewIntent
	04-10 15:50:59.744: D/debug(26323): class backStack.A4-->onResume
	04-10 15:51:01.174: D/debug(26323): class backStack.Ax-->onCreate
	04-10 15:51:01.175: D/debug(26323): class backStack.Ax-->onResume
	04-10 15:51:02.462: D/debug(26323): class backStack.Ay-->onCreate
	04-10 15:51:02.463: D/debug(26323): class backStack.Ay-->onResume
	04-10 15:51:04.116: D/debug(26323): class backStack.Ay-->onBackPressed
	04-10 15:51:04.152: D/debug(26323): class backStack.A1-->onResume
	04-10 15:51:04.418: D/debug(26323): class backStack.Ay-->onDestroy
	04-10 15:51:06.519: D/debug(26323): class backStack.A1-->onBackPressed
	04-10 15:51:06.554: D/debug(26323): class backStack.Ax-->onResume
	04-10 15:51:07.120: D/debug(26323): class backStack.A1-->onDestroy
	04-10 15:51:19.864: D/debug(26323): class backStack.Ax-->onBackPressed
	04-10 15:51:19.902: D/debug(26323): class backStack.A4-->onResume
	04-10 15:51:20.168: D/debug(26323): class backStack.Ax-->onDestroy
	04-10 15:51:23.582: D/debug(26323): class backStack.A4-->onBackPressed
	04-10 15:51:23.612: D/debug(26323): class backStack.A3-->onResume
	04-10 15:51:24.168: D/debug(26323): class backStack.A4-->onDestroy
	04-10 15:51:24.963: D/debug(26323): class backStack.A3-->onBackPressed
	04-10 15:51:24.992: D/debug(26323): class backStack.A2-->onResume
	04-10 15:51:25.259: D/debug(26323): class backStack.A3-->onDestroy
	04-10 15:51:28.226: D/debug(26323): class backStack.A2-->onBackPressed
	04-10 15:51:29.533: D/debug(26323): class backStack.A2-->onDestroy

实际上，如果我希望一个task（包含几个Activity），作为一个执行完就结束的，不可back的task，单单用Activity是做不到的，应该把用Activit代替Task，用Fragment代替Activity，那样我们就可以简单的用Activity.finish来结束一个'task'

另外一点，值得注意而且很重要的，如果新打开了一个task，那你作为launcher那个Activity结束后，可能不会回到launch，而是跳到那个task的top（如果这个task没有结束掉的话）。


而如果在A1.startActivity中的Intent.setflag(FLAG_ACTIVITY_NEW_TASK);得到的结果和singleTask不一样，表现出来的结果是standard


对于singleTask和FLAG_ACTIVITY_NEW_TASK，官网有错
根据http://disanji.net/2011/08/26/%E8%A7%A3%E5%BC%80android%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E7%BB%84%E4%BB%B6activity%E7%9A%84singletask%E4%B9%8B%E8%B0%9C/里面说

1. 设置了”singleTask”启动模式的Activity，它在启动的时候，会先在系统中查找属性值affinity等于它的属性值 taskAffinity的任务存在；如果存在这样的任务，它就会在这个任务中启动，否则就会在新任务中启动。因此，如果我们想要设置 了”singleTask”启动模式的Activity在新的任务中启动，就要为它设置一个独立的taskAffinity属性值。

2. 如果设置了”singleTask”启动模式的Activity不是在新的任务中启动时，它会在已有的任务中查看是否已经存在相应的Activity实 例，如果存在，就会把位于这个Activity实例上面的Activity全部结束掉，即最终这个Activity实例会位于任务的堆栈顶端中。
Activity中的launchMode和setFlag，会出现在哪里？

会保存在ActivityInfo中。

对这个类的解释，官网`Information you can retrieve about a particular application activity or receiver. This corresponds to information collected from the AndroidManifest.xml's <activity> and <receiver> tags.`

而当startActivity时，实际上：
	
startActivity最终会调用以下这个方法。

	final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType, IBinder resultTo,
            String resultWho, int requestCode, int startFlags, String profileFile,
            ParcelFileDescriptor profileFd, WaitResult outResult, Configuration config,
            Bundle options, int userId) {
        // Refuse possible leaked file descriptors
        if (intent != null && intent.hasFileDescriptors()) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }
        boolean componentSpecified = intent.getComponent() != null;

        // Don't modify the client's object!
        intent = new Intent(intent);

        // Collect information about the target of the Intent.
        ActivityInfo aInfo = resolveActivity(intent, resolvedType, startFlags,
                profileFile, profileFd, userId);
		......

	}

而resolveActivity：

	ActivityInfo resolveActivity(Intent intent, String resolvedType, int startFlags,
            String profileFile, ParcelFileDescriptor profileFd, int userId) {
        // Collect information about the target of the Intent.
        ActivityInfo aInfo;
        try {
            ResolveInfo rInfo =
                AppGlobals.getPackageManager().resolveIntent(
                        intent, resolvedType,
                        PackageManager.MATCH_DEFAULT_ONLY
                                    | ActivityManagerService.STOCK_PM_FLAGS, userId);
            aInfo = rInfo != null ? rInfo.activityInfo : null;
        } catch (RemoteException e) {
            aInfo = null;
        }
		... ...
	} 