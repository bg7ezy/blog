---
title: squid透明代理之应用层广告过滤
date: 2017-09-18 11:19:57
tags: [squid,过滤,代理]
categories: [Linux,squid]
zhaunzai: https://qiananhua.com/6584
---

透明代理的意思是客户端不需要知道有代理服务器的存在，也不需要有设置代理的操作，它改变你的request fields(报文)，并会传送真实IP。
##### 什么时候用透明代理？

squid正向代理其实我一直都在用，在局域网做个缓存服务器常用网站的加载速度会有明显提升，我主要还是为了在应用层过滤广告，可是移动设备设置代理又比较麻烦，一个网段下的其他人也不能受益，这种情况就可以搞起 “透明代理了“ 。
<!-- more -->

当然如果想了解局域网内的其他人都上了哪些不可告人的网站，刷朋友圈浏览了哪些照片。这时squid相当于是个中间人，对于裸奔的http请求，那绝对毫无隐私可言啊，结合一些日志分析工具譬如awstats，可以很直观的了解局域网内设备的浏览行文，当然这一切对于客户端是透明的无感知的。这也是为什么现在如此推崇ssl的原因了。对于https squid 其实也是可以解析的，就是比较麻烦这个后面再说。
##### 准备物料
树莓派，我这用的树莓派2b+；

路由器，华为pro；
##### Squid
###### squid安装过程不表仅贴一下编译参数，具体可参见官方文档
```
    ./configure --prefix=/usr/local/squid \
    --enable-gnuregex \
    --disable-carp \
    --enable-async-io=240 \
    --with-pthreads \
    --enable-storeio=ufs,aufs,diskd \
    --disable-wccp \
    --enable-icmp \
    --enable-kill-parent-hack \
    --enable-cachemgr-hostname=Raspi \
    --with-maxfd=65535 \
    --enable-poll \
    --enable-Linux-netfilter \ #透明代理必需
    --enable-large-cache-files \
    --disable-ident-lookups \
    --enable-default-hostsfile=/etc/hosts \
    --with-dl \
    --with-large-files \
    --enable-delay-pools \
    --enable-snmp \
    --disable-internal-dns \
    --enable-underscore \
    --enable-arp-acl
```
###### SSL中间人
当然你如果想让squid承担中间人重新加密的任务（就是解析https数据拉），就需要增加以下两个参数了，让squid支持sslbump和动态证书生成，但是这种方式仅支持正向代理。
```
    --with-openssl
    --enable-ssl-crtd
```
具体可以参照以下官方文档：
[http://wiki.squid-cache.org/Features/SslBump](http://wiki.squid-cache.org/Features/SslBump)
[http://wiki.squid-cache.org/Features/DynamicSslCert](http://wiki.squid-cache.org/Features/DynamicSslCert)
[http://wiki.squid-cache.org/Features/MimicSslServerCert](http://wiki.squid-cache.org/Features/MimicSslServerCert)

###### squid装好后有几个关键配置
```
    vim /usr/local/squid/etc/squid.conf
    http_port 3128 transparent #开启透明代理支持
    cache_mem 128 MB
    logformat combined %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh #combined格式输出日志，后面awstats统计用
    access_log /data/logs/squid/access.log combined #后面awstats统计要用到
    cache_log /data/logs/squid/cache.log
     
    #广告过滤
    error_directory /usr/local/squid/share/errors/en-us/ #指定错误信息语言，我这里是英文（广告请求拦截后的转发页面也放这里）
    acl deny_url dstdom_regex "/usr/local/squid/etc/acl/deny_url.acl" #广告url列表，github上有很多如 https://github.com/vokins/yhosts
    deny_info 410:ERR_URL deny_url #ERR_URL是error_directory目录下面的错误文件，可以是文本或者html文件。 http_access deny deny_url
```
###### squid常用命令

    /usr/local/squid/sbin/squid -z #重建缓存
    /usr/local/squid/sbin/squid #启动
    /usr/local/squid/sbin/squid -k reconfigure #重载配置文件

如果一切顺利此时squid的透明代理已经支持。
##### DHCP

但是要让整个局域网使用透明代理，还需让客户端的网关地址是squid服务器的地址，网关地址在普通路由器上是没法自定义的。
你可以关闭路由器的dhcp功能，路由器就当ap来使用，在树莓派上搭建一个dhcp服务，这里用的是isc-dhcp-server

apt-get直接安装
```
    apt-get install isc-dhcp-server
```
编辑配置文件
```
    vim /etc/dhcp/dhcpd.conf
    subnet 10.0.0.0 netmask 255.255.255.0 {
      range 10.0.0.10 10.0.0.100;
      option domain-name-servers 180.76.76.76;
      option domain-name "qiananhua.com";
      option routers 10.0.0.2; #squid主机地址
      #option broadcast-address 10.0.0.255;
      default-lease-time 86400;
      max-lease-time 172800;
    }
    service isc-dhcp-server restart
```
断开wifi重新连接即可。
##### iptables

最后一步让经过squid服务器网卡的http 80端口流量用squid服务的3128端口处理

转发
```
    iptables -t nat -A PREROUTING -s 10.0.0.0/24 -p tcp --dport 80 -j REDIRECT --to-port 3128
```
如果一切顺利此时打开 www.163.com 你看到的页面应该是这样的

![](https://qiananhua.com/wp-content/uploads/2017/08/WechatIMG9-262x300.jpeg)

##### awstats

安装比较简单如图所示

![](https://qiananhua.com/wp-content/uploads/2017/08/WechatIMG2111.jpeg)

配置文件需要指定LogFile为squid日志文件地址
```
    vim /etc/awstats/awstats.squid.conf
    LogFile="/data/logs/squid/access.log"
```
手动生成下结果页面-dir=指定生成到nginx配置的虚拟主机根目录
```
    /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=squid -lang=cn -dir=/data/htdocs/awstats -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl
```
当然也可以吧这条命令放到.sh文件里添加到crontab自动执行，比如每30分钟执行一次统计
```
    */30 * * * * /data/shell/awstats.sh > /dev/null 2>&1
```
接着nginx添加一个awstats的虚拟主机
```
    vim /usr/local/nginx/conf/vhosts/awstats.conf
    server {
        listen 80;
        server_name awstats;
        access_log /data/logs/nginx/awstats.log;
        root /data/htdocs/awstats;
            index awstats.squid.html;
     
        location ~ ^/icon/{
                root /usr/local/awstats/wwwroot;
        }
    }
```
注意本地修改下host，不出意外你会看到以下页面（最后一张图http状态吗410就是被过滤的广告链接了，占比还挺大）
![](https://qiananhua.com/wp-content/uploads/2017/08/WechatIMG2102-300x271.jpeg)

![](https://qiananhua.com/wp-content/uploads/2017/08/WechatIMG2101-300x271.jpeg)

![](https://qiananhua.com/wp-content/uploads/2017/08/WechatIMG2103-300x271.jpeg)
