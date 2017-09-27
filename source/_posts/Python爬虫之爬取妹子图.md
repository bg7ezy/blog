---
title: Python爬虫之爬取妹子图
date: 2017-08-30 10:59:53
tags: [python,爬虫]
categories: Python
copyright: true
---

##### 工具介绍
1.  Python版本为python3.4,不选2.7版是因为蛋疼的编码问题，你懂的。
2.  Requests获取网页源码。
3.  XPACH获取需要采集的内容，本文使用的规则很简单，都是些入门的知识。
4.  Chrome用来分析、查看网页源码。
5.  Pycharm(非必须)，Python代码编写调试。

##### 准备工作
######  分析网页，找到入口
首先打开Chrome,我们来到 [妹子图首页](http://www.mzitu.com),随便瞄了几下发现网页很有规则的样子，当我点开[每日更新](http://www.mzitu.com/all/)时，激动的热泪盈眶啊，站长真是好人啊！所有数据都做成了列表，也一目了然。
<!-- more -->

![](http://img.hb.aicdn.com/998d88bbfa1b9d319fc1cdb96edd64c4f965febccb23-ax6pVv_fw658)

##### 爬取数据
查看源码发现页面地址都包含在 `<p class="url">` 标签下面对应的XPATH规则可以这样写：`'//p[@class="url"]/a/@href'`

![](http://img.hb.aicdn.com/e97c3030d906de122d258a9a3316a095c37991cbb942-cZoLRJ_fw658)

运行下代码试试
```python
#! /usr/bin/env python3
# coding:utf-8

import requests
from lxml import html

# 获取主页列表
def getPage():
    baseUrl = 'http://www.mzitu.com/all'
    selector = html.fromstring(requests.get(baseUrl).content)
    urls = []
    for i in selector.xpath('//p[@class="url"]/a/@href'):
        urls.append(i)
        #print(i)
    return urls
page_url = getPage()
for i in page_url:
    print(i)
```
得到预期的结果：

![](http://img.hb.aicdn.com/a883cf9bf9719521c3be9d6fb5f2fb277f18922d5fe6-JBIUGC_fw658)

下面我们就开始爬图片了。这里有个小坑就是每一页只有一个张图片。我们可以先获取总页数，然后循环爬取图片。

![](http://img.hb.aicdn.com/a77ddd9df3dd21cc92a685c00af87a39373b58e5a1523-OfTkBX_fw658)

页码部分的代码为：
```html
<div class="pagenavi">
         <a href='http://www.mzitu.com/101256'><span>&laquo;上一页</span></a><a href='http://www.mzitu.com/101256'><span>1</span></a><span>2</span><a href='http://www.mzitu.com/101256/3'><span>3</span></a><a href='http://www.mzitu.com/101256/4'><span>4</span></a><a href='http://www.mzitu.com/101256/5'><span>5</span></a><span class='dots'>…</span><a href='http://www.mzitu.com/101256/65'><span>65</span></a><a href='http://www.mzitu.com/101256/3'><span>下一页&raquo;</span></a>      </div>
      <div class="main-tags"><span>相关专题:</span><a href="http://www.mzitu.com/tag/ugirls/" rel="tag">Ugirls(尤果网)</a><a href="http://www.mzitu.com/tag/tangxiruo/" rel="tag">唐溪若</a><a href="http://www.mzitu.com/tag/xingganneiyi/" rel="tag">性感内衣</a><a href="http://www.mzitu.com/tag/xinggan/" rel="tag">性感美女</a><a href="http://www.mzitu.com/tag/leg/" rel="tag">美腿</a><a href="http://www.mzitu.com/tag/meitun/" rel="tag">美臀(翘臀)</a><a href="http://www.mzitu.com/tag/youhuo/" rel="tag">诱惑</a><a href="http://www.mzitu.com/tag/heisi/" rel="tag">黑丝</a></div>
```
图片地址的代码：
```html
<div class="main-image"><p><a href="http://www.mzitu.com/101256/3" ><img src="http://i.meizitu.net/2017/08/27c02.jpg" alt="清纯少女唐溪若充满灵气 透视蕾丝释放你的肾上腺素" /></a></p>
</div>
```
对应图片数量的XPATH规则为 `'//div[@class="pagenavi"]'//a/span/text()`
图片地址为 `//div[@class="main-image"]/img/@src`
标题为 `//div[@class="main-image"]/img/@alt`
整理一下对应的获取图片地址的代码就是:
```python
urls_ = 'http://www.mzitu.com/101256'
r = html.fromstring(requests.get(urls_).content)
title = r.xpath('//div[@class="main-image"]//img/@alt')[0].replace(' ','')
if os.path.exists(title):
    print(u'文件夹已经建立，无需重建。')
else:
    os.mkdir(title)

total = r.xpath('//div[@class="pagenavi"]/a/span/text()')[4]
for jpg_num in range(1,int(total)+1):
    print(u'[', title, ']', u'总共有', total, u'张图片,正在下载第', jpg_num, u'张图片。')
    jpg_url_ = urls_ + '/' + str(jpg_num)
    p = html.fromstring(requests.get(jpg_url_).content)
    jpg_url = p.xpath('//div[@class="main-image"]//img/@src')[0]
    jpgfile = str(jpg_num) + '.jpg'
    filename = os.path.join(os.path.abspath('.'),title,jpgfile)
    with open(filename,'wb') as jpg_file:
        jpg_f = requests.get(jpg_url).content
        jpg_file.write(jpg_f)
```
运行一下代码瞧瞧，看看妹子到碗里来没？

![](http://img.hb.aicdn.com/696d028e73280fbe46f3797635a6245c29110ee44968-ifs6WO_fw658)

WTF,什么鬼？看来网管做了反爬虫处理。requests页面时加个headers参数试试。
```python
def header(referer):
    headers = {
        'Host': 'i.meizitu.net',
        'Pragma': 'no-cache',
        'Accept-Encoding': 'gzip, deflate',
        'Accept-Language': 'zh-CN,zh;q=0.8,en;q=0.6',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
        'User-Agent': 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.112 Safari/537.36',
        'Accept': 'image/webp,image/apng,image/*,*/*;q=0.8',
        'Referer': '{}'.format(referer),
    }
    return headers
```
修改下获取图片部分代码 `p = html.fromstring(requests.get(jpg_url_).content)` 为 `p = html.fromstring(requests.get(jpg_url_,headers=header(jpg_url_)).content)`
再试试看效果，妹子果然到碗里来了。

![](http://img.hb.aicdn.com/59a903c7a8e6c675ce9269e7a73f1df00f079eca1bf4b-ieBhWX_fw658)
![](http://img.hb.aicdn.com/96c7eb86407602e34249bc0f6756f083114329ccea5c1-1OCjb9_fw658)

完整的代码：
```python
#! /usr/bin/env python3
# coding:utf-8

import requests
from lxml import html
import os
import time

## 设置henders参数模拟浏览器访问
def header(referer):
    headers = {
        'Host': 'i.meizitu.net',
        'Pragma': 'no-cache',
        'Accept-Encoding': 'gzip, deflate',
        'Accept-Language': 'zh-CN,zh;q=0.8,en;q=0.6',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
        'User-Agent': 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.112 Safari/537.36',
        'Accept': 'image/webp,image/apng,image/*,*/*;q=0.8',
        'Referer': '{}'.format(referer),
    }
    return headers

## 获取主页列表
def getPage():
    baseUrl = 'http://www.mzitu.com/all'
    selector = html.fromstring(requests.get(baseUrl).content)
    urls = []
    for i in selector.xpath('//p[@class="url"]/a/@href'):
        urls.append(i)
    return urls

## 获取图片地址，并保存图片到对应文件夹。
def getJpg(jpg_):
    for urls_ in jpg_:
        print(urls_)
        r = html.fromstring(requests.get(urls_).content)
        title = r.xpath('//div[@class="main-image"]//img/@alt')[0].replace(' ','')
        if os.path.exists(title):
            print(u'文件夹已经建立，无需重建。')
        else:
            os.mkdir(title)
        total = r.xpath('//div[@class="pagenavi"]/a/span/text()')[4]
        for jpg_num in range(1,int(total)+1):
            print(u'[', title, ']', u'总共有', total, u'张图片,正在下载第', jpg_num, u'张图片。')
            jpg_url_ = urls_ + '/' + str(jpg_num)
            p = html.fromstring(requests.get(jpg_url_).content)
            jpg_url = p.xpath('//div[@class="main-image"]//img/@src')[0]
            jpgfile = str(jpg_num) + '.jpg'
            filename = os.path.join(os.path.abspath('.'),title,jpgfile)
            with open(filename,'wb') as jpg_file:
                jpg_f = requests.get(jpg_url,headers=header(jpg_url)).content
                jpg_file.write(jpg_f)

page_url = getPage()
getJpg(page_url)
```
