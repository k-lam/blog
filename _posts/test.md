Reference:

[在Android Studio中进行单元测试和UI测试](http://www.jianshu.com/p/03118c11c199)

[Building Instrumented Unit Tests](https://developer.android.com/intl/in/training/testing/unit-testing/instrumented-unit-tests.html)

[Automating User Interface Tests](https://developer.android.com/intl/in/training/testing/ui-testing/index.html)

[Testing Support Library](https://developer.android.com/intl/in/tools/testing-support-library/index.html)

[Testing UI for a Single App](https://developer.android.com/intl/in/training/testing/ui-testing/espresso-testing.html)

[Activity Testing](http://developer.android.com/intl/in/tools/testing/activity_testing.html)


Junit:

逐个test开头的调用，

对于测试多线程，需要回调的，有生命周期的，用线程阻塞（但必须在同一个test方法中）

如

	public void testDiv() throws Exception {
        final CountDownLatch lock = new CountDownLatch(1);
        new Thread(){
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                assertEquals(10 / 2f, 5f);
                lock.countDown();
            }
        }.start();
        // lock.await(10, TimeUnit.SECONDS);
        lock.await();
    }
    
    
    
####[Testing Fundamentals](https://developer.android.com/intl/in/tools/testing/testing_android.html)
