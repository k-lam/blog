###理论基础
[Exception handling wiki](https://en.wikipedia.org/wiki/Exception_handling)

###java 异常基础

####Throwable
是ERROR和Exception的父类，一定要实现这个接口，才能被throw
>An Error is a subclass of Throwable that indicates serious problems that a reasonable application should not try to catch. Most such errors are abnormal conditions.

#####分类
>There are really three important subcategories of Throwable:

>**Error** - Something severe enough has gone wrong the most applications should crash rather than try to handle the problem,
Unchecked 

>**Unchecked exception** (aka RuntimeException) - Very often a programming error such as a NullPointerException or an illegal argument. Applications can sometimes handle or recover from this Throwable category -- or at least catch it at the Thread's run() method, log the complaint, and continue running.

>**Checked Exception** (aka Everything else) - Applications are expected to be able to catch and meaningfully do something with the rest, such as FileNotFoundException and TimeoutException...


基于以上分类，Error是不处理的，Exception(RuntimeException)是要处理（handle or recover）,而对于Checked Exception也是要处理，try catch 处理

>"If it's so good to document a method's API, including the exceptions it can throw, why not specify runtime exceptions too?" Runtime exceptions represent problems that are the result of a programming problem, and as such, the API client code cannot reasonably be expected to recover from them or to handle them in any way. Such problems include arithmetic exceptions, such as dividing by zero; pointer exceptions, such as trying to access an object through a null reference; and indexing exceptions, such as attempting to access an array element through an index that is too large or too small.

>Runtime exceptions can occur anywhere in a program, and in a typical one they can be very numerous. Having to add runtime exceptions in every method declaration would reduce a program's clarity. Thus, the compiler does not require that you catch or specify runtime exceptions (although you can).

引用自:[Unchecked Exceptions — The Controversy](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)


####How uncaught exceptions are handled in jvm
 When an uncaught exception occurs, the JVM does the following:

it calls a special private method, dispatchUncaughtException(), on the Thread class in which the exception occurs;
it then terminates the thread in which the exception occurred1.

[How uncaught exceptions are handled](http://www.javamex.com/tutorials/exceptions/exceptions_uncaught_handler.shtml)


####阶段总结1：
对于 Checked Exception,用`try...catch...`解决，对于Unchecked Exception,用通过提供一个自定的`Thread.UncaughtExceptionHandler`来解决

按异常发生的线程作不同处理 

* 主线程
* 一般工作线程（以后可能要区分io和运算密集型）


###Android的unchecked exception
*这节如无特别说明，所提及的exception都是unchecked exception*

####Activity中抛异常
Activity1->Activity2->Activity3->throw exception
所有对象都会被销毁

打印log发现，是异常退出后，是重新开启了一条新的**进程**，包括application都会重新生成

Activity1->throw exception->crash
是真的退出了，不会重新开启一条新进程
(已经在多台手机测试)

下一步要看源码了，看看是怎样重启了进程的，和有没有办法MainFrame崩，也返回MainFrame，还有测试用fragment的情况

####方案一
由以上，得出以下几点
1. android 在非生命周期函数中的主线程异常，会退出进程变且重启，而且会重启进程，进入到上次操作的activity。但如果只有一个Activity，不会重启进程
2. saveInstance会保留，也就是上一个进程Activity的信息能通过saveInstance传递到重启进程
3. 在生命周期函数中异常，不会重启进程
4. 想通过saveInstance传数据给重启进程，但是在onSaveInstance之外的地方直接修改ActivityClientRecord的state，无效

####android主线程crash的源码处理过程

android自带的Thread.UncaughtExceptionHandler在RuntimeInit.java中

    /**     * Use this to log a message when a thread exits due to an uncaught     * exception.  The framework catches these for the main threads, so     * this should only matter for threads created by applications.     */    private static class UncaughtHandler implements Thread.UncaughtExceptionHandler {        public void uncaughtException(Thread t, Throwable e) {            try {                // Don't re-enter -- avoid infinite loops if crash-reporting crashes.                if (mCrashing) return;                mCrashing = true;                if (mApplicationObject == null) {                    Clog_e(TAG, "*** FATAL EXCEPTION IN SYSTEM PROCESS: " + t.getName(), e);                } else {                    StringBuilder message = new StringBuilder();                    message.append("FATAL EXCEPTION: ").append(t.getName()).append("\n");                    final String processName = ActivityThread.currentProcessName();                    if (processName != null) {                        message.append("Process: ").append(processName).append(", ");                    }                    message.append("PID: ").append(Process.myPid());                    Clog_e(TAG, message.toString(), e);                }                // Bring up crash dialog, wait for it to be dismissed                ActivityManagerNative.getDefault().handleApplicationCrash(                        mApplicationObject, new ApplicationErrorReport.CrashInfo(e));            } catch (Throwable t2) {                try {                    Clog_e(TAG, "Error reporting crash", t2);                } catch (Throwable t3) {                    // Even Clog_e() fails!  Oh well.                }            } finally {                // Try everything to make sure this process goes away.                Process.killProcess(Process.myPid());                System.exit(10);            }        }    }

ActivityManagerNative.getDefault().handleApplicationCrash(                        mApplicationObject, new ApplicationErrorReport.CrashInfo(e));在
	 /**     * Used by {@link com.android.internal.os.RuntimeInit} to report when an application crashes.     * The application process will exit immediately after this call returns.     * @param app object of the crashing app, null for the system server     * @param crashInfo describing the exception     */    public void handleApplicationCrash(IBinder app, ApplicationErrorReport.CrashInfo crashInfo) {        ProcessRecord r = findAppProcess(app, "Crash");        final String processName = app == null ? "system_server"                : (r == null ? "unknown" : r.processName);        handleApplicationCrashInner("crash", r, processName, crashInfo);    }    /* Native crash reporting uses this inner version because it needs to be somewhat     * decoupled from the AM-managed cleanup lifecycle     */    void handleApplicationCrashInner(String eventType, ProcessRecord r, String processName,            ApplicationErrorReport.CrashInfo crashInfo) {        EventLog.writeEvent(EventLogTags.AM_CRASH, Binder.getCallingPid(),                UserHandle.getUserId(Binder.getCallingUid()), processName,                r == null ? -1 : r.info.flags,                crashInfo.exceptionClassName,                crashInfo.exceptionMessage,                crashInfo.throwFileName,                crashInfo.throwLineNumber);        addErrorToDropBox(eventType, r, processName, null, null, null, null, null, crashInfo);        crashApplication(r, crashInfo);    }
        crashApplication最重要一句

     Intent appErrorIntent = null;        synchronized (this) {            if (r != null && !r.isolated) {                // XXX Can't keep track of crash time for isolated processes,                // since they don't have a persistent identity.                mProcessCrashTimes.put(r.info.processName, r.uid,                        SystemClock.uptimeMillis());            }            if (res == AppErrorDialog.FORCE_QUIT_AND_REPORT) {                appErrorIntent = createAppErrorIntentLocked(r, timeMillis, crashInfo);            }        }        if (appErrorIntent != null) {            try {                mContext.startActivityAsUser(appErrorIntent, new UserHandle(r.userId));            } catch (ActivityNotFoundException e) {                Slog.w(TAG, "bug report receiver dissappeared", e);            }        }
        
源码也是做了两件事
1.log and report error
2.reopen app if appErrorIntent != null

而android重开进程后默认是回退到上一个界面。我们可以简单处理为回退到mainframe，但是welcome中的初始化代码要注意了
（回退到之前程序出错的上一个Activity，程序处理会比较复杂，需要saveInstance！恢复saveInstance，而且还要获取到Activity的Task栈！）



