---
title: mvp+rx+retrofit
date: 2017-01-04 10:14:02
categories: android
tags: android
description: "感觉Rxjava+retrofit是现在android中最火的网络框架组合了。火肯定是有原因的，确实好用。mvp模式是mvc的升级版，更适合移动端。如果使用mvc的话，往往导致activity很尴尬，因为它好像是v+c的组合，导致无法好好的解耦，mvp就不一样了，真正实现了view与data的解耦，view专注于绘制界面，获取数据则有p来间接获取。今天说的这三个东西，不准备详细讲，只是说下用法，详细起来，每一个都得单独来一篇，后面有机会再说吧。"
---
感觉Rxjava+retrofit是现在android中最火的网络框架组合了。火肯定是有原因的，确实好用。mvp模式是mvc的升级版，更适合移动端。如果使用mvc的话，往往导致activity很尴尬，因为它好像是v+c的组合，导致无法好好的解耦，mvp就不一样了，真正实现了view与data的解耦，view专注于绘制界面，获取数据则有p来间接获取。今天说的这三个东西，不准备详细讲，只是说下用法，详细起来，每一个都得单独来一篇，后面有机会再说吧。
<!-- more -->
感觉Rxjava+retrofit是现在android中最火的网络框架组合了。火肯定是有原因的，确实好用。mvp模式是mvc的升级版，更适合移动端。如果使用mvc的话，往往导致activity很尴尬，因为它好像是v+c的组合，导致无法好好的解耦，mvp就不一样了，真正实现了view与data的解耦，view专注于绘制界面，获取数据则有p来间接获取。今天说的这三个东西，不准备详细讲，只是说下用法，详细起来，每一个都得单独来一篇，后面有机会再说吧。
![目录结构][1]
## MVP
上图就是我写的一个小demo，首先我们先看mvp部分。mvp部分有两个根接口BasePresenter和BaseView。BasePresenter是给具体的presenter实现的，BaseView是给具体的view接口实现的。我们看到fragment目录下，首先我们创建一个MainContract接口。
![MainContract][2]
这个接口的作用主要是把presenter和view接口整合起来，当我们在具体实现的时候，只需要继承MainContract.presenter就行了，看起来比较整洁，使用起来也比较方便。还有一点，整合之后，我们不必再为不同的Presenter和View接口起名字了有没有，只需要给Contract起个名字就行了。别小看名字，有时候很让人头疼的。接下来看MainPresenter。它就是对presenter的实现
![MainPresenter][3]
这里实现了MainContract.presenter。有设置view的方法，获取网络数据的方法。获取数据后，调用view的showMainView方法显示，这个方法就是具体的view实现的MainContract.presenter接口，比如我们的MainFragment
![MainFragment][4]
总结一下：首先数据层和view层是不直接交互的，中间用Presenter代理。Presenter层会持有对应view的实例，activity持有presenter的实例。在aty中需要操作view的时候，都可以用presenter来代理。
## Rxjava+Retrofit
这两个的教程，网上好多，就不多解释了，直接看用法。
再看到上面的第一张图，我们的Retrofit部分都在data包里
![HttpUtil][5]
我们先看到HttpUtil。我们利用单例模式，创建一个Retrofit的实例，其中设置了host，Rxjava，Gson，这些都不用说，标配。下面的client，如果不设置的话，默认是用Okhttp，我们当然也是用的ok，只是对ok做了一点配置，看到下面获取ok实例的方法，其实就是设置了一下自定义日志拦截器，方便我们看到请求和返回值。等等，好像Gson工厂也是我们自己实现的。确实是，看data目录下，有GsonConvertFactory，GsonRequestConvert，GsonResponseConverter。这三个都与有GsonConvertFactory有关。作用是把一个实体类转换为RequestBody和把ResponseBody转换为实体类。
![server][6]
这里就是定义api的地方了，后面注释写的很清楚，不过多说了。我们看到第一个，post请求，我们只需要往方法里扔一个对应的实体类，然后返回的时候用另一个实体类接收，就完成参数的传递和接收了，超方便，实体类就定义在entry目录下。具体的转换逻辑就是上面说的GsonConvertFactory。那么准备工作做完，怎么用呢，其实上面已经贴过了，看MainPresenter类，获取网络数据。通过api我们知道，请求网络返回的是一个Observable。那么就可以利用rxjava的优势，链式操作，中间各种过滤，转换，切线程操作，最后获取正确的数据刷新view。这里面的可操作性很强，用到什么程度，就看对Rxjava的理解了。


Demo地址[点此跳转][7]
不善言辞，写的不好，主要留给自己回顾吧，若能帮到你，实属万幸


  [1]: http://ofy9dm2ii.bkt.clouddn.com/image/article/mvp+rx1.png
  [2]: http://ofy9dm2ii.bkt.clouddn.com/image/article/mvp+rx2.png
  [3]: http://ofy9dm2ii.bkt.clouddn.com/image/articlemvp+rx3.png
  [4]: http://ofy9dm2ii.bkt.clouddn.com/image/articlemvp+rx4.png
  [5]: http://ofy9dm2ii.bkt.clouddn.com/image/articlemvp+rx5.png
  [6]: http://ofy9dm2ii.bkt.clouddn.com/mvp+rx6.png
  [7]: https://github.com/geekzw/mvp-retrofit-rxjava
