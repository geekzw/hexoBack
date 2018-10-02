---
title: 记一次RocketMQ单机安装过程
date: 2018-04-29 13:15:24
categories: java
tags: java
---
网上关于centos7单机部署RocketMQ的文章挺多的，[官网][1]写的也比较清晰，但是奈何本人比较笨，还是没能很顺利的一次成功。既然没有一次成功，那么肯定是遇到坑了，所以还是记录下来好，说不定哪天还要回来看。
<!-- more -->

## 环境

 1. 系统:centos7 64(4G 1核虚拟机)
 2. jdk:1.8.0_161
 3. maven:3.5.3
 4. Git:1.8.3.1(这个无所谓的吧，不装都可以)
## 环境变量配置

    JAVA_HOME=/usr/java/jdk1.8.0_161
    CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
export MAVEN_HOME=/usr/local/maven/apache-maven-3.5.3
export PATH=${PATH}:${MAVEN_HOME}/bin

## RocketMQ安装

[下载][2]source压缩包,这里直接用wget下载
解压:压缩包是zip格式，所以要yum install unzip,然后直接解压就行了

     > unzip rocketmq-all-4.2.0-source-release.zip
     > cd rocketmq-all-4.2.0/
     > mvn -Prelease-all -DskipTests clean install -U
     > cd distribution/target/apache-rocketmq

我们需要的就是target目录下的apache-rocketmq,建议直接把这个目录mv到/usr/local目录下
接下来可以先配置一下环境变量了，我的配置

    
    export ROCKETMQ_HOME=/usr/local/apache-rocketmq
    export PATH=${PATH}:${ROCKETMQ_HOME}/bin

现在我们可以看下apache-rocketmq目录下的东西

 - benchmark
 - bin(重点操作对象)
 - conf(配置文件，后面再研究)
 - log(我创建的log日志目录)
当然还有几个文件，至少安装过程用不到，先不解释了
我们进入到bin目录，首先给这几个文件添加执行权限
chmod +x mqadmin mqbroker mqfiltersrv mqshutdown  mqnamesrv
然后就可以nohup mqnamesrv 1>/usr/local/apache-rocketmq/log/ng.log 2>/usr/local/apache-rocketmq/log/ng-err.log &
意思是启动nameserver,并把相应的日志写入日志文件，最后一个符号是后台启动的意思。
如果没报什么异常，就可以去ng.log看一下。应该会看到The Name Server boot success. serializeType=JSON
接着jps一下，就可以看到nameserver进程已经启动了
11510 NamesrvStartup
下面需要启动broker了，在此之前，我们要配置一下server的地址，我这里直接写在环境变量里了，也可以写到broker的配置里。

    export NAMESRV_ADDR=localhost:9876
这里说下，nameserver的默认端口是9876,不知道的话，当启动完server后，jps看进程id，然后netstat -nlep|grep xxxx就可以看到进程对应的端口了

很兴奋，接着赶紧nohupmqbroker>/usr/local/apache-rocketmq/log/mq.log &。意思跟上面差不多。这里我就坑了。
首先是报了一个jvm内存不足的异常。原来mq默认参数是8G，我的配置是4G的。这时候修改一下bin/runbroker.sh文件的
JAVA_OPT="${JAVA_OPT} -server -Xms2g -Xmx2g -Xmn521m"，根据自己的配置修改就行了。然后再次启动broker。网上说这时候应该没问题了，启动成功后，jps一下，应该看到
11542 BrokerStartup
这个我倒是看到了，但是还说日志里会有boot启动成功，还会打印server的地址，broker的地址，这个我死活都打印不出来。我一度认为是没启动成功，但是jps确实有这个进程了。那我就先不管了。试下官网的测试用例:sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
给我爆了个没有TopicTest这个topic的错误，那怎么办，加个topic呗。于是sh mqadmin updateTopic –n 192.168.1.23:9876 –c DefaultCluster –t TopicTest。相关命令可以[查看][3]。还是死活加不上。我还是怀疑broker的问题，查了一下集群状态，果然都是空，根本没有broker。这里就是诡异的问题了。就是broker怎么都注册不到server上。后来关机，重启，就好了。再次添加topic的时候，还是加不上，我-n后面试了127.0.0.1不行，localhost也不行，最后用了本机ip才加上。


  [1]: http://rocketmq.apache.org/docs/quick-start/
  [2]: https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip
  [3]: http://www.cnblogs.com/gmq-sh/p/6232633.html