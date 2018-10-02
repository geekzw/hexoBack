---
title: Looper
date: 2016-11-01 16:41:40
categories: android
tags: handler
description: "Looper这个类的主要功能是做消息的分发，并没有很复杂的逻辑"
---
Looper这个类的主要功能是做消息的分发，并没有很复杂的逻辑
<!-- more -->
## 概述
Looper这个类的主要功能是做消息的分发，并没有很复杂的逻辑

## Looper类关键属性
![looper属性][1]
## Looper类关键方法
###### 1、创建Looper
```java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```
这两个方法都是创建looper的方法，第一个是在一般线程中创建使用，第二个是创建mainLooper的时候调用。
主要区别在于quitAllowed参数，第一个方法默认接收的是true，表示此looper关联的MessageQueue允许停止，而mainLooper关联的MessageQueue是不允许停止的。从方法的实现可以看出，每个线程只允许一个Looper的存在。
######2、遍历取消息
```java
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```
下面就来分析下这个方法做的事情：
首先第二行是获取当前线程的Looper实例，接着是在looper中获取MessageQueue。下面这两行就如注释所说，保证当前线程是在本地运行并持续跟踪。具体想了解其中原理和机制可以[点这里查看][3]，这两行不理解，并不耽误整体流程的理解。
接下来就是进入一个死循环，循环中不断的从queue中取出消息，然后验证消息的各种属性，判断是否是一个有效的消息。其中有一行msg.target.dispatchMessage(msg);这行代码的意思就是把消息分发出去，最后让属于此msg的handler来处理。最后，会调用msg.recycleUnchecked();如果看了Message这个类，可以知道，这个方法就是对消息重新初始化，然后扔进复用池，而且在这里调用的是强制回收方法。

[1]: http://ofy9dm2ii.bkt.clouddn.com/image/article/looper.png
[3]: http://www.th7.cn/Program/java/201603/774109.shtml
