---
title: springboot集成dubbo经历
date: 2017-07-30 22:17:57
tags: java
---

springboot集成dubbo，dubbo-admin的配置和使用
<!-- more -->
# dubbo介绍
dubbo的介绍，网上有很多，让我这个新手讲，也讲不出什么东西。献上个链接[dubbo详解][1]，自己参详
# springboot-dubbo-server

### pom

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0       http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.gzw</groupId>
<artifactId>server</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>

<name>server</name>
<description>Demo project for Spring Boot</description>

<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>1.5.6.RELEASE</version>
<relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
<java.version>1.8</java.version>
</properties>

<dependencies>

<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-test</artifactId>
<scope>test</scope>
</dependency>
<!-- Spring Boot Dubbo 依赖 -->
<dependency>
<groupId>io.dubbo.springboot</groupId>
<artifactId>spring-boot-starter-dubbo</artifactId>
<version>1.0.0</version>
</dependency>

</dependencies>

<build>
<plugins>
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
</plugins>
</build>


</project>
这个是我的服务端pom文件，很简单，引入dubbo-starter就行了。集成dubbo有两种方式，这是一种，还有一种就是直接引dubbo，然后用配置文件的形式。这里既然说了用springboot，springboot的宗旨就是消灭xml，那么另外一种就先不说了。

### properties

spring.dubbo.application.name=provider
spring.dubbo.registry.address=zookeeper://127.0.0.1:2181
spring.dubbo.registry.group=gzw
spring.dubbo.protocol.name=dubbo
spring.dubbo.protocol.port=20880
spring.dubbo.scan=com.gzw.dubbo.service.impl
server.port=8081
这是配置文件，简单集成，配置很少，第一行是名称，第二行是zookeeper的地址，然后是分组信息，协议地址，协议端口，要扫面的服务接口实现，最后为了不和8080端口冲突，自己改了个。
### service
在你要扫描的包下，创建一个服务接口，比如HelloService，里面暂时就一个方法：String getHi();然后写个接口的实现类，实现类有一点需要注意，

@Service(version="1.0")
public class HelloServiceImpl implements HelloService {
@Override
public String getHi() {
return "hello";
}
}
Service注解要用dubbo，不要用springboot的，不然里面的version属性找不到，version是定义服务接口的版本的，客户端调用相应服务的时候，要标明版本。然后。。。。没有然后了，一个简易的dubbo服务已经配置完成。我当时碰到的坑，其实怪自己，我参考了注解方式和xml方式两种，写到同一个项目里了，然后就各种奇葩错，如果直接按这个来，应该一点问题没有。
# springboot-dubbo-client
客户端的pom就不贴了，因为我没有专门写一个demo，引的依赖比较多，对于dubbo而言，就跟服务端一样，只引入一个dubbo-starter就行了，另外，springboot的优势就是各种starter，它会帮你做很多事，比如自动读取properties，自动引一些需要的依赖，dubbo-starter就自动把zookeeper相关的也包含进来了，我们不用单独再引zookeeper依赖。
### properties

dubbo:
application:
name: consumer
registry:
address: zookeeper://127.0.0.1:2181
scan: com.gzw.dubbo.service.serviceImpl
客户端关于dubbo的简单配置，其实就一行最主要，就是注册中心的地址，其他的跟服务端也一样

### service
service这里看实现方式。第一种，我们可以直接引入dubbo服务项目，这样就直接掉服务里的服务接口就行了。第二种，我们可以把服务接口单独抽出来，让客户端和服务项目共同引用接口项目。第三种，服务项目写一个服务接口，客户端掉服务接口的时候，在客户端写个一模一样的接口，我这里使用的是第三种，第一种的话，客户端和服务耦合，不好，第二种挺好，但是我只是写个非常简单的集成一下dubbo，不想建3个项目。这里需要注意的是，客户端写的接口要和服务端一样，然后实现类里注入接口的时候要加@Reference(version = "1.0")，完整的就是

@Service
public class HelloServiceImpl implements HelloService{
@Reference(version = "1.0")
HelloService helloService;
@Override
public String getHi() {
return helloService.getHi();
}
}
不要忘记客户端配置里也有扫描路径，这个service要在扫描路径下
然后。。。又没有然后了，这就完了，写个简单的controller掉下这个service，应该可以看到结果了。现在看看真的是很简答，不知道为什么我当时遇到各种错，不过错也挺好，如果没有遇到错，可能我到这里就结束了，因为出错让我学到更多。刚才我们看到，服务端和客户端都注册了zookeeper这个东西，这个东西是什么呢？简单说就是个注册中心，dubbo把服务注册到这里，然后客户端在这里找已经注册的服务。具体深入的话，同样，我等渣渣目前的功力还解释不了，献上链接[zookeeper详解][2]

# zookeeper安装配置
我刚开始也看了好几个关于zookeeper的安装配置的博客，总结起来其实很简单，下载zookeeper，解压，然后conf目录下有个叫zoo_sample.cfg的文件，把它改成zoo.cfg就行了，这些链接里都有，只是真的很简单，如果只是安装配置，看我这几句话足以。
# 中结
像我上面说的，如果我没有碰到错误，那么到这里估计我就会结束了。当时配置好之后，客户端死活掉不到dubbo服务，各种检查配置，都没问题，先说结果。我前面说了，参考了注解和xml两种方式，结果傻逼的用混了，pom里没用starter，要自己写xml导入配置，但是我又是用属性的方式配置的，结果导致根本没读到配置。这是结果。我检查完配置后，就想，会不会是zookeeper出问题了，根本没有注册进去。这个时候就找到了dubbo-admin了。有同学问了，想看有没有注册进去，去zookeeper里看下不就行了，这个可以，而且简单，但是我看到dubbo-admin的时候，就想用这种方式，以后也方便操作。
# dubbo-admin
### 编译
网上有dubbo-admin编译好的war，放到tomcat下就行，但是很不幸，因为每个人的环境不同，导致很多war下下来不能用，还是自己打包靠谱。我们先到[这里][3]下载包好dubbo-admin的源码，然后进入dubbo-admin目录然后执行mvn package -Dmaven.skip.test=true 命令打包，当然这个时候，你的jdk，maven肯定是已经装好了。打包过程中如果没有出现错误，那只能说你太幸运了，环境跟源码一致，我肯定是报错了，首先是说Could not find artifact com.alibaba:dubbo:jar:2.5.4-SNAPSHOT in nexus-aliyun，这个原因很简单，maven仓库中最新的猜2.5.3，这里用的是2.5.4，两种方法，一种是在dubbo-admin同级别目录打包，就是全量编译，这样就能用刚打好的2.5.4了，还有一种，直接打开dubbo-admin里的pom，把2.5.4改成2.5.3就行了。然后再打包，这个时候又报错 Error creating bean with name 'uriBrokerService'，这个错是因为我用的jdk1.8，这里有三种方法了，第一，把jdk1.8换成1.7，第二种其实dubbo已经有3了，更新后的支持1.8，下那份源码，可以打1.8下的包，第三种，就是改pom，这里给个链接，里面说的比较详细[看这里][4]。这个处理完，就应该能正常打出包了。
### tomcat运行
打完包，将war放入tomcat/webapps下，运行tomcat后，会自动解压war包，这个时候停掉tomct，到tomcat/webapps下找到解压后的文件夹dubbo-admin-2.5.4-SNAPSHOT。进入到WEB-INF下，有个dubbo-properites文件，打开里面有三项内容，zookeeper地址，这里主要看端口，是不是zookeeper暴露的端口，一般不用改，下面两行是root的密码和guest的密码，下面会用。这个时候启动tomcat，访问http://localhost:8089/dubbo-admin-2.5.4-SNAPSHOT/。
如果一切正常，浏览器就会弹出需要输入用户名密码的对话框，我们输入root，root。就进入了管理页面。大功告成。但是问题又来了，我明明把dubbo的服务注册到zookeeper了，客户端也成功请求到了，为什么这个后台上显示服务数为0，搜也搜不到。网上看了下，原来是分组的问题。
### 分组
我们看上面服务项目的配置，有个分组spring.dubbo.registry.group=gzw。如果不设置，默认是dubbo组。但是我们的dubbo-admin没有设置分组信息，所以找不到。我们来到tomcat7/webapps/dubbo-admin/WEB-INF/classes/META-INF/spring这个目录，里面有个dubbo-admin.xml文件，打开可以看到有一行`<dubbo:registry address="${dubbo.registry.address}" check="false" file="false" />`
这里只有address没有分组信息，改为` <dubbo:registry group="${dubbo.registry.group}"address="${dubbo.registry.address}" check="false" file="false" />`然后到tomcat7/webapps/dubbo-admin/WEB-INF这个目录下，有个dubbo.properties文件，打开，加入分组信息dubbo.registry.group=gzw。然后重启tomcat，对了，最好去tomcat的server.xml中，把端口改掉，防止和8080冲突。这个时候，我们应该可以正常看到后台显示有一个服务注册到zookeeper上了。
#总结
自己做完之后，感觉其实接起来挺简单的，但是第一遍，总是有各种坑，有的是自己粗心，自己给自己挖的，有的因为环境等之类的原因。dubbo这个项目貌似已经不维护了，所以随着jdk等环境的不断升级，可能会遇到各种问题。上面我接入过程中遇到的，也只是部分问题，大家在接入的时候，可能还会碰到各种五花八门的错误。耐心点，好好看错误提示，好好google，总会解决的。有一个事实是，如果你现在开始接，你前面已经有无数个人接过了，你碰到的问题，其他人基本也碰到了，所以，我们这种就很幸运了，坑已经被别人填平了。比如还有个问题，加入热部署依赖后，dubbo会报接口不可见的错，这个时候，简单粗暴的方法是把热部署干掉，但是你如果实在需要热部署,去dubbo的git的issue里看看，已经有人给出方案了。这只是极简的接入dubbo然后配置了下dubbo-admin，要学的还很多，小白继续努力，以后深入研究了，再来续写更好的文章。



[1]: http://shiyanjun.cn/archives/325.html
[2]: https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/
[3]: https://github.com/alibaba/dubbo
[4]: https://github.com/alibaba/dubbo/issues/50
