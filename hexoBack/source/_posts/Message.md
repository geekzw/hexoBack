---
title: Message
date: 2016-11-01 15:10:26
categories: android
tags: handler
description: "message作为消息的载体，主要看下它是如何保存信息的"
---
message作为消息的载体，主要看下它是如何保存信息的
<!-- more -->
#### Message类关键属性
![message属性][1]

#### Message类关键方法
###### 1、获取消息
```java
public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```
因为Message采用链表的形式存储消息，当我们用obtaion获取消息实例的时候，首先就是遍历消息链表，如果链表中有消息实例，则直接返回，如果没有，则new一个出来，下面这些方法，都是首先调用obtain获取一个消息，然后把带来的参数赋给获取的实例
###### 2、获取消息重载方法
```java
public static Message obtain(Message orig)
public static Message obtain(Handler h)
public static Message obtain(Handler h, Runnable callback)
public static Message obtain(Handler h, int what)
public static Message obtain(Handler h, int what, Object obj)
public static Message obtain(Handler h, int what, int arg1, int arg2)
```
###### 3、回收消息
```java
void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```
这个方法写的很明白，就是把当前消息的属性重新初始化，然后放进复用池

[1]: http://ofy9dm2ii.bkt.clouddn.com/image/article/message.jpeg
