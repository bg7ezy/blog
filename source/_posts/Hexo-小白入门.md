---
title: Hexo-小白入门
date: 2017-08-28 15:56:55
tags: [Hexo,入门]
categories: 闲聊
copyright: true
---

##### 一、搭建网站
[GitHub](https://github.io)为我们提供了免费的域名空间，So从网上扒了几个教程，开干！
&emsp;&emsp;参考的教程有：
1. [[干货]如何在一天之内搭建以你自己名字为域名且具备cool属性的个人博客](http://www.jianshu.com/p/99665608d295)
2. [手把手教从零开始在GitHub上使用Hexo搭建博客教程(一)-附GitHub注册及配置](http://www.jianshu.com/p/f4cc5866946b)

###### 个人认为写的不错，既有指导思想，又有实际案例。参考教程已经写的很清楚了，这里就不细说了，有不懂的请找度娘开小灶。下面说几个本人部署的时候几个小坑：

1.  GitHub注册好新建repository的时候，刚开始我填的是 `用户名/blog.github.io`，结果无法打开网页，后来重新按教程里的`用户名/用户名.github.io`成功了。
2.  倒腾主题的时候发现页面效果的更改需要一定时间才能生效，刚开始我还以为配置出错了。
3.  我用的Linux,把部署命令设置了alias, `alias='hexo clean && hexo generate && hexo deploy'`

<!-- more -->

##### 二、配置Next主题
###### Next主题简洁明快，大爱。具体配置可参考[官方文档](http://theme-next.iissnan.com/getting-started.html)。启用菜单项的时候，如果还没创建过文章或者文件里面没有包含分类、标签时即分类、标签字段为空则访问这些页面时看到的是404。


##### 三、发布文章
###### 发布文章的命令为`hexo new '文章名称'`，如文章名称包含空格需加字符界定符'' 或者 "" 。Hexo文章采用的是MarkDown语法，你可以使用MarkDown编辑器离线生成，或者直接编辑`source/_posts/`下面的文章。
```javascript
---
title: 文章名
date: 创建名称
categories: 分类
tags: 标签
---
```
###### 开头的这段代码是对文章的描述，默认没有categories:字段，可手动添加。设置多个标签的方法为: `tags: [tags1,tags2,tags3]` 。要使用摘要功能可在适当的位置添加 `<-- more -->`。好了，发个测试博文试试。

```javascript
---
title: test
date: 2017-08-28 14:42:43
tags: 测试
categories: 闲聊
---

# 你好
这是我的第一篇文章:)
<!-- more -->
```
<div align=center>
![](http://img.hb.aicdn.com/ece4ab84d43531901be8923203bb7415307fd5e5536e-UlG9eX_fw658)
</div>
