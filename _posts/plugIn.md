##360 DroidPlugIn
###Hook
com.morgoo.hook
在Application 的onCreate中就会installHook所有Hook

	HookFactory.java:
	public final void installHook(Context context, ClassLoader classLoader) throws Throwable {
        installHook(new IClipboardBinderHook(context), classLoader);
        //for INotificationManager
        installHook(new INotificationManagerBinderHook(context), classLoader);
        installHook(new IMountServiceBinder(context), classLoader);
        installHook(new IAudioServiceBinderHook(context), classLoader);
        installHook(new IContentServiceBinderHook(context), classLoader);
        installHook(new IWindowManagerBinderHook(context), classLoader);
        if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP_MR1) {
            installHook(new IGraphicsStatsBinderHook(context), classLoader);
        }
        if (VERSION.SDK_INT >= VERSION_CODES.KITKAT) {
            installHook(new IMediaRouterServiceBinderHook(context), classLoader);
        }
        if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
            installHook(new ISessionManagerBinderHook(context), classLoader);
        }
        if (VERSION.SDK_INT >= VERSION_CODES.JELLY_BEAN_MR2) {
            installHook(new IWifiManagerBinderHook(context), classLoader);
        }

        if (VERSION.SDK_INT >= VERSION_CODES.JELLY_BEAN_MR2) {
            installHook(new IInputMethodManagerBinderHook(context), classLoader);
        }

        installHook(new IPackageManagerHook(context), classLoader);
        installHook(new IActivityManagerHook(context), classLoader);
        installHook(new PluginCallbackHook(context), classLoader);
        installHook(new InstrumentationHook(context), classLoader);
        installHook(new LibCoreHook(context), classLoader);

        installHook(new SQLiteDatabaseHook(context), classLoader);
    }

####ProxyHook
用代理的方法，代替hook掉的实现

###进程管理
####RunningProcesList
正在运行的进程列表
####MyActivityManagerService
进程管理服务
问题：registerApplicationCallback方法

####com.morgoo.droidplugin.pm.PluginManager
插件包管理服务的客户端实现。

#####mPluginManager
实现是IPluginManagerImpl（下文）

####com.morgoo.droidplugin.pm.IPluginManagerImpl

此服务模仿系统的PackageManagerService，提供对插件简单的管理服务。

####IActivityManagerHook
在onInstall的

	FieldUtils.writeStaticField(cls, "gDefault", proxiedActivityManager);
	
这样hook了系统的gDefault，因为这个field是static的单例！