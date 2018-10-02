---
title: MessageQueue
date: 2016-11-01 16:36:38
categories: android
tags: handler
description: "MessageQueue的作用是操作message，因为message本身就是链表结构，所以MessageQueue不必用LinkList之类的数据结构存储msg。只需要持有一个message实例就行了。此方法中有许多native方法，不过多分析。主要分析几个关键的方法，有助于理解MessageQueue是如何管理msg的。"
---
MessageQueue的作用是操作message，因为message本身就是链表结构，所以MessageQueue不必用LinkList之类的数据结构存储msg。只需要持有一个message实例就行了。此方法中有许多native方法，不过多分析。主要分析几个关键的方法，有助于理解MessageQueue是如何管理msg的。
<!-- more -->
## 概述
MessageQueue的作用是操作message，因为message本身就是链表结构，所以MessageQueue不必用LinkList之类的数据结构存储msg。只需要持有一个message实例就行了。此方法中有许多native方法，不过多分析。主要分析几个关键的方法，有助于理解MessageQueue是如何管理msg的。
## 关键方法
#### 1、初始化
```java
MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }
```
初始化很简单，mPtr相当于一个指针，指向nativeInit（）方法返回的东西，那么就看一下nativeInit（）的实现
```c++
static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) {
    NativeMessageQueue* nativeMessageQueue = new NativeMessageQueue();
    if (!nativeMessageQueue) {
        jniThrowRuntimeException(env, "Unable to allocate native queue");
        return 0;
    }

NativeMessageQueue::NativeMessageQueue() :
        mPollEnv(NULL), mPollObj(NULL), mExceptionObj(NULL) {
    mLooper = Looper::getForThread();
    if (mLooper == NULL) {
        mLooper = new Looper(false);
        Looper::setForThread(mLooper);
    }
}

static struct {
    jfieldID mPtr;   // native object attached to the DVM MessageQueue
    jmethodID dispatchEvents;
} gMessageQueueClassInfo;
```
可以看到，原来native层也有一个MessageQueue类，java层MessageQueue的mPtr就是指向native层的MessageQueue对象。native层的结构体中的变量mPtr指向java的Mess对象，这样，java层和native层就关联起来了。同时又可以看到，native层也有Looper对象，也是保存在当前线程的。看下native层Looper的创建方法
```c++
Looper::Looper(bool allowNonCallbacks) :
        mAllowNonCallbacks(allowNonCallbacks), mSendingMessage(false),
        mPolling(false), mEpollFd(-1), mEpollRebuildRequired(false),
        mNextRequestSeq(0), mResponseIndex(0), mNextMessageUptime(LLONG_MAX) {
    mWakeEventFd = eventfd(0, EFD_NONBLOCK);
    LOG_ALWAYS_FATAL_IF(mWakeEventFd < 0, "Could not make wake event fd.  errno=%d", errno);

    AutoMutex _l(mLock);
    rebuildEpollLocked();
}
```
其中第5行很重要，保存对管道的读写描述符，然后创建一个epoll实例。
也就是说，java层和native层的Looper和MessageQueue是一一对应，协同工作的，创建顺序是java:Looper->java:MessageQueue->native:MessageQueue->native:Looper
## epoll
在消息循环过程中，如果没有新的消息要处理，线程就会睡眠在管道的读端文件描述符上，直到有消息处理为止；当其他线程向这个线程发送消息的时候，其他线程就会通过这个管道的写端文件描述符写数据从而将线程唤醒。epoll机制就是为了同时监听多个文件描述符的IO读写时间而设计的，它是一个多路复用的IO接口。类似于Linux的select机制。select就是监听众多的文件描述符接口，但是每次发生事件，它都要遍历所有的监听，找到匹配的发送事件。epoll则是select的加强版，在监听大量IO读写，但只有少量是活跃的时候，能显著减少cpu使用率。[关于epoll机制的详细信息和实现][2]
#### 2、取消息
```java
 Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```
##### 普通消息
第5行，mPtr指向native中的MessageQueue，是java层和native联系的纽带。第11行变量nextPollTimeoutMillis是设置休眠时间的：如果赋值为0，则表示无论消息队列中是否有消息，线程都不进入休眠状态，如果赋值为-1，则表示无论什么情况，线程都进入休眠状态。接下来就进入消息循环中了，看13行，如果此时线程需要进入休眠状态，那么，就执行Binder.flushPendingCommands();sdk的解释为：Flush any Binder commands pending in the current thread to the kernel driver.  This can be useful to call before performing an operation that may block for a long time, to ensure that any pending object references have been released in order to prevent the process from holding on to objects longer than it needs to.大意就是执行binder通信请求，防止等待时间过长。接下来就是调用nativa方法nativePollOnce
```c++
//MessageQueue
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,
        jlong ptr, jint timeoutMillis) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->pollOnce(env, obj, timeoutMillis);
}

//MessageQueue
void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);
    mPollObj = NULL;
    mPollEnv = NULL;

    if (mExceptionObj) {
        env->Throw(mExceptionObj);
        env->DeleteLocalRef(mExceptionObj);
        mExceptionObj = NULL;
    }
}
//Looper
int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData) {
    int result = 0;
    for (;;) {
        while (mResponseIndex < mResponses.size()) {
            const Response& response = mResponses.itemAt(mResponseIndex++);
            int ident = response.request.ident;
            if (ident >= 0) {
                int fd = response.request.fd;
                int events = response.events;
                void* data = response.request.data;
#if DEBUG_POLL_AND_WAKE
                ALOGD("%p ~ pollOnce - returning signalled identifier %d: "
                        "fd=%d, events=0x%x, data=%p",
                        this, ident, fd, events, data);
#endif
                if (outFd != NULL) *outFd = fd;
                if (outEvents != NULL) *outEvents = events;
                if (outData != NULL) *outData = data;
                return ident;
            }
        }

        if (result != 0) {
#if DEBUG_POLL_AND_WAKE
            ALOGD("%p ~ pollOnce - returning result %d", this, result);
#endif
            if (outFd != NULL) *outFd = 0;
            if (outEvents != NULL) *outEvents = 0;
            if (outData != NULL) *outData = NULL;
            return result;
        }

        result = pollInner(timeoutMillis);
    }
}
```
首先看第4行，前面说过，java层的mPtr就是指向native层MessageQueue的指针，此处创建了一个native层的MessageQueue的指针，并且把mPtr赋值给它，那么它此时代表的就是native层与java层绑定的MessageQueue对象。
然后调用这个对象的pollOnce方法去取消息。然后看到第12行，pollOnce方法里其实是调用了native层的Looper实例的pollOnce方法去取消息。接下来就可以看到Looper的pollOnce方法了，首先在while循环中，经过各种条件的判断，如果有新消息，那么就返回。（中间各种epoll相关各种赋值不做研究）如果没有新消息，那么就调用pollInner方法，这个方法比较长，也是各种epoll的操作，不做分析，主要作用就是监听管道的读写时间，从而使线程能从休眠中唤醒或者进入休眠状态。
接下来，如果在native层中拿到了消息，那么java层的mMessage成员变量就不为空，24-30行则是从消息队列中取出一个非同步消息。32行是判断此消息是否需要立即处理，如果不需要，则设置等待时间，如果需要，进入else，else中有个mBlocked变量，它的作用是标记线程是否处于阻塞状态。那么当mMessage为空的时候，则nextPollTimeoutMillis为-1，表示进入休眠。

##### 特殊消息
IdleHandler表示一种特殊的消息。这种消息会在线程空闲的时候被处理，能充分利用cup空闲时间。只要实现了IdleHandler接口，就可以被注册到空闲消息列表。
第10行pendingIdleHandlerCount表示的是注册到空闲消息列表的消息处理器的个数，这里赋值为-1。然后看到62行，因为pendingIdleHandlerCount初始值为-1，所以当没有消息，或者消息时间大于当前时间的时候满足条件，此时如果空闲消息列表有事件，则会在80出执行，如果没有事件，则在66行处让线程休眠。如果有空闲事件，执行完之后，在99行会重新给pendingIdleHandlerCount赋值为0.作用就是每次进入next方法后，之后执行一次空闲消息，后期线程被唤醒，继续消息循环，不会执行空闲消息。我们还可以看到在103行给nextPollTimeoutMillis 赋值为0.应为在处理空闲消息的时候，有可能有新的消息发来，所以当再次调用nextPollTimeoutMillis的时候不会进入休眠状态。
#### 2、存消息
```java
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
把消息加入到消息队列的逻辑相对简单，我们从27行看起，if的条件是新加入的消息目标时间小于消息队列的队头消息，则把当前消息放入队头。否则的话进入else，else的for循环是为了寻找和新消息的目标时间最相近的消息，然后把消息插入到队列中去。当我们把消息放在队头的时候，如果此时为阻塞状态，那么肯定需要唤醒线程，告诉它有新消息，也就是needWake =mBlocked;当插入到队中的时候，一般不需要唤醒线程，只有在线程是阻塞状态，并且我们插入的消息是队列中第一个异步消息。也即是 needWake = mBlocked && p.target == null && msg.isAsynchronous();

[2]: http://blog.csdn.net/xiajun07061225/article/details/9250579
