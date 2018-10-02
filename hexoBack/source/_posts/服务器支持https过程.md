---
title: 服务器支持https过程
date: 2017-07-19 15:44:31
tags: java
---
最近接触了java后台开发，对后台与前端的加密解密比较感兴趣，碰巧又读了一篇浅显易懂的关于https的文章，就自己试着捣鼓了一下。
<!-- more -->
## 简介
说道加密解密，https。那内容就很多了，首先加解密，就分好多种，哈希，对称，非对称等等。http和https更是能说太多的内容。这里我肯定不会展开来说，想详细了解某一个技术细节的，可以自行google，这里贴一篇文章，讲的非常的通俗易懂，如果对https和加密解密不了解的，相信看完这篇文章，一定会醍醐灌顶，反正我是这样，链接在此[一个故事讲完https][1]。

## 加密过程
其实加密过程还是挺简单的，至少说简单的集成起来很简单，说的有点绕了。加密算法我们用到了对称加密和非对称加密。网上很多相关的工具类，可以选择一个封装的好的使用，当然，用我demo的，我也不介意。除了这些工具类外，还可以用openssl生成。
## 本地apache配置https
本人用的是mac系统，其他系统类似
#### 生成CA证书
为了统一管理生成的文件，最好先创建一个文件夹，可以叫ssl，CA之类的，然后进入文件夹，命令行进入openssl

 1. 密钥 genrsa -des3 -out ca.key 4096
 2. 证书 req -new -x509 -days 365 -key ca.key -out ca.crt
 3. 创建服务器私钥 genrsa -out server.key 4096
 4. 生成证书请求文件CSR req -new -key server.key -out server.csr
 5. 自己作为CA签发证书 ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -days 365




这里我是用自己的本地服务器，自己签证书，如果有域名，有外网服务器，那么只需要去申请就行了，自签证书是不被信任的。需要注意的是，Common Name应该与域名保持一致
如果此时碰到错误：I am unable to access the ./demoCA/newcerts directory ./demoCA/newcerts: No such file or directory
执行以下几条命令即可：
mkdir -p ./demoCA/newcerts
touch demoCA/index.txt
touch demoCA/serial
echo 01 > demoCA/serial

#### apache配置文件修改

 - /etc/apache2/httpd.conf
 去掉以下三行的注释：
LoadModule ssl_module libexec/apache2/mod_ssl.so
Include /private/etc/apache2/extra/httpd-vhosts.conf
Include /private/etc/apache2/extra/httpd-ssl.conf
 - /etc/apache2/extra/httpd-ssl.conf
 去掉以下两项注释并检查是否与之前安装私钥和证书的路径一致
可以把生成的两个文件放到下面的路径，ssl是我自己创建的文件夹
SSLCertificateFile "/etc/apache2/extra/ssl/server.crt"
SSLCertificateKeyFile "/etc/apache2/extra/ssl/server.key"
 - 编辑/etc/apache2/extra/httpd-vhosts.conf文件
 在最下方添加：

    < VirtualHost *:443>
    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /private/etc/apache2/server.crt
    SSLCertificateKeyFile /private/etc/apache2/server.key
    ServerName 127.0.0.1 
    DocumentRoot "/Library/WebServer/Documents"
< / VirtualHost>

到这里算是配置完了，我遇到了2个问题，第一个是apache2.4跟以前的2.2配置有所不同，需要把:/etc/apache2/httpd.conf的
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so取消注释
第二个问题，当引入httpd-vhosts后，会报找不到documentRoot的路径，当然这只是个警告，具体影响我也没有关注，但还是改掉了，改个存在的文件夹就行了
还有个不是问题的问题，但是花了我好长时间，就是引入httpd-vhosts后，这个配置文件最底部有作者，版权之类的东西，竟然没有注释掉，然后我重启apache的时候，报错，开始没有仔细看错误，只是看到一坨。后来实在找不到怎么解决。就回头认证看了下错误描述，说是可能拼写有误。我就在config文件中找，仔细核对，最后血都要喷出来了，把最下面的版权信息注释掉就好了。
好了，这个时候重启apache：sudo apachectl restart。如果正常启动，那么用https访问本地应该是可以访问的。
## 安装根证书
上述步骤做完，可以访问，但是浏览器提示不安全，因为没有加入我们自己的证书，双击前面生成的ca安装，跳到钥匙串界面，右键我们的证书，简介，永久信任，ok。

  [1]: https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513779&idx=1&sn=a1de58690ad4f95111e013254a026ca2&chksm=80d67b70b7a1f26697fa1626b3e9830dbdf4857d7a9528d22662f2e43af149265c4fd1b60024#rd
