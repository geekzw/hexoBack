---
title: 深入理解Handler
date: 2016-11-01 16:43:27
categories: android
tags: handler
---
handler是发送消息和最终处理消息的类。从发送消息的方式看，有两种，一种是handler.post(Runnable)，另一种是handler.sendMessage()。
<!-- more -->
## 关联类
### <font color="#47C4EA">[Message](https://geekzw.github.io/2016/11/01/Message/index.html)</font>
### <font color="#47C4EA">[MessageQueue](https://geekzw.github.io/2016/11/01/MessageQueue/index.html)</font>
### <font color="#47C4EA">[Looper](https://geekzw.github.io/2016/11/01/Looper/index.html)</font>

## 概述
handler是发送消息和最终处理消息的类。从发送消息的方式看，有两种，一种是handler.post(Runnable)，另一种是handler.sendMessage()。

## handler流程解析
### handler.post()方式
handler.post()接受一个Runnable对象，run方法中的代码执行在UI线程也就是主线程。跟进post方法可以发现，其实还是把Runnable通过getPostMessage方法包装成一个Message对象，其中Runnable实例赋值给Message对象的callback成员变量。最后调用
```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
这个方法很简单，就是对与handler绑定的msgqueue判空，不为空则把当前包装了Runnable的msg放入messageQueue。此处调用的enqueueMessage(queue, msg, uptimeMillis);在<font color="#47C4EA">[MessageQueue](https://geekzw.github.io/2016/11/01/MessageQueue/index.html)</font>类中有详细解释至此，算是把一个Runnable对象放入了消息队列中，那么什么时候会执行这个Runnable呢？那就要看<font color="#47C4EA">[Looper](https://geekzw.github.io/2016/11/01/Looper/index.html)</font>类了，我们知道，Looper类的主要作用就是从消息队列中取出消息，最后发送给目标handler。我们在分析Looper类的时候，分析过一个loop（）方法，它就是用来循环取出消息队列中的消息的方法，其中当取出一个有效的消息后，会执行msg.target.dispatchMessage(msg);这行代码。msg当然就是Message的一个实例，在<font color="#47C4EA">[Message](https://geekzw.github.io/2016/11/01/Message/index.html)</font>类中，我们知道有一个target的成员变量，作用是保存这个消息最终到达的handler的实例。那么这行代码调用的就是handler.dispatchMessage()这个方法。
```java
public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
看到这里，终于是看到了我们最初post的那个Runnable对象了，它其实就是msg.callback。当它不位空的时候调用handleCallback(msg)这个方法更是简单，就一行代码
```java
private static void handleCallback(Message message) {
        message.callback.run();
    }
```
看到这里就恍然大悟，为什么post的Runnable对象，run里面的代码是跑在UI线程的了。因为就是在UI线程中在一个合适的时间把run方法当做一个普通方法调用了，没有把它放入一个Thread中当做子线程处理。这里容易理解错是因为我们平时使用Runnable都是放入一个子线程中执行，不会手动调用run方法。平时开发中，使用postDelayed比较多，用于延时执行一段代码。
### handler.sendMessage()方式
sendMsg方法有很多重载，但是都不难理解，只是指定的参数不一样，有些参数不想系统默认。如果调用的sendMessage（）不是message实例，则系统会调用Message.obtain获取一个消息，来包装传入的参数，obtain方法在Message中有详细分析。后面流程与post方式一致，压入消息列表，loop循环取出，调用msg.target.dispatchMessage(msg);这次callback为空。则走else流程。首先判断mCallback是否为空。Callback是一个接口，里面就一个handleMessage(msg)方法，handler中的Callback是我们平时经常用的标准的回调模式。如果mCallback不为空，则回调handleMessage，否则调用handleMessage，sdk对这个方法的注释说，子类必须重写这个方法，用来处理接受到的消息。
### 对handler机制的一个疑问
当app启动的时候，会执行ActivityThread类中的main方法。main方法代码如下
```java
public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
我们只关心跟handler有关的代码，其实主要也就是创建looper和handler。21行可以看到调用了Looper.prepareMainLooper();这个方法在Looper类分析中有做解释。就是创建一个mainLooper。接着创建一个sMainThreadHandler，然后调用Looper.loop();这是标准的在一个线程中创建handler的方式，我们自己使用的时候，也应该这么创建。看到这里我就有疑问了。主线程中调用了loop方法，我们都知道，loop方法里面是一个无限循环，并且带阻塞的。那为什么app还能继续执行，并没有因此而阻塞在这里。后来发现，原来我对app的运行流程一直理解有误。app大部分时间确实是处于阻塞状态。现在我们来想一想，线程是什么东西。线程的作用就是执行一块代码，当代码运行完了，它也就结束了。但是想想我们的app，它可是可以长期运行，不会因为线程执行结束而死掉的。那我们平时要想让一个线程一直运行下去，做法是什么呢？就是在线程中写个死循环。这么一想，也算是大致理解了mainLooper这个东西。当创建了mainLooper之后，它就会进入死循环不断的从MessageQueue中取消息，那么在app启动过程中，系统会系统还会创建必要的服务进程，这些进程会通过binder发送给主线程事件，比如aty的create start等等。这时我们可以看一下MainThreadHandler这个东西了。MainThreadHandler类型为H，就是ActivityThread的内部类，重写handMessage方法
```java
public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case RELAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
                    ActivityClientRecord r = (ActivityClientRecord)msg.obj;
                    handleRelaunchActivity(r);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case PAUSE_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handlePauseActivity((IBinder) args.arg1, false,
                            (args.argi1 & USER_LEAVING) != 0, args.argi2,
                            (args.argi1 & DONT_REPORT) != 0, args.argi3);
                    maybeSnapshot();
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case PAUSE_ACTIVITY_FINISHING: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handlePauseActivity((IBinder) args.arg1, true, (args.argi1 & USER_LEAVING) != 0,
                            args.argi2, (args.argi1 & DONT_REPORT) != 0, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case STOP_ACTIVITY_SHOW: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStop");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleStopActivity((IBinder) args.arg1, true, args.argi2, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
```
代码过多，不全部贴出来了。从这个方法中可以看出来了。系统发的消息都在这里处理了，比如app启动，重启等消息。app的启动和运行就是不停的执行各种消息的过程，当暂时没有消息的时候，app就会阻塞在当前状态。这个时候还有一个疑问，一旦创建了mainLooper，调用了loop方法，主线程就处于无限循环和阻塞状态，理论上着会占用很大的cup。但是实际上，当前没有可取的消息的时候，线程将会被挂起。再有消息到来的时候再唤醒，这个流程采用的是epllo模式，这个模式在分析MessageQueue的时候有做解释。
### 使用handler需要注意的地方
1、从looper的源码中可以知道，每个线程只能有一个looper，也就是不能在同一个线程中多次调用prepare方法。
2、在子线程中使用handler，要按Looper.prepare,new handler,looper.loop顺序调用，其中原理已经分析过了
3、使用handler时要注意内存泄露。当消息队列中还有待处理的消息时，如果aty被销毁，因为消息中持有aty实例，导致内存泄露。此时我们可以采用静态内部类+对aty的弱引用解决。
