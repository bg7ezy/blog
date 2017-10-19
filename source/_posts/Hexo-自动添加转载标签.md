---
title: Hexo 自动添加转载标签
copyright: true
date: 2017-09-04 14:46:55
tags: [Hexo,转载]
categories: 闲聊
zhuanzai:
---

##### 前言
最近在网上转载了很多技术类博文，处于对文章原创作者劳动成果的尊重，我在文章开头都会写上文章转载出处。但是每次都要手工添加一次虽然工作量不大就是复制粘贴再粘贴。
```html
转自: [http://xxx.com/xxx](http://xxx.com/xxx)
```
虽然很简单，但还是想有个什么方法再简化一下。
##### 修改代码
结合之前添加版权声明的修改方法，自己鼓捣了一个方案，效果达到了(PS:有更加简单直接的可以给我留言)。
修改 `themes/next/layout/_macro/post.swig` 文件,在
<!-- more -->
```html
          {% if post.description and (not theme.excerpt_description or not is_index) %}
              <div class="post-description">
                  {{ post.description }}
              </div>
          {% endif %}

        </div>
      </header>
    {% endif %}
    {#################}
    {### POST BODY ###}
    {#################}
```
在注释结尾前添加代码，标签样式你可以改成自己喜欢的样式。

```html
    {% if page.zhuanzai %}
      <p style="color: #FFA500;font-size=16px"><span>转自: </span><a href="{{ url_for(page.zhuanzai) }}" target="_blank">{{ page.zhuanzai }}</a></p>
    {% endif %}
```

添加好后的效果:
```html
          {% if post.description and (not theme.excerpt_description or not is_index) %}
              <div class="post-description">
                  {{ post.description }}
              </div>
          {% endif %}

        </div>
      </header>
    {% endif %}
    {% if page.zhuanzai %}
      <p style="color: #FFA500;font-size=16px"><span>转自: </span><a href="{{ url_for(page.zhuanzai) }}" target="_blank">{{ page.zhuanzai }}</a></p>
    {% endif %}
    {#################}
    {### POST BODY ###}
    {#################}
```
##### 转载设置
在 `POST BODY` 尾部添加转载信息，`page.zhuanzai` 中 `page` 表示的是页面模板，`zhuanzai` 引用的页面模板中的`zhuanzai` 字段的值，文件位置在`scaffolds/post.md` 。修改 `scaffolds/post.md` 添加 `zhuanzai` 字段。这样每次新建文章的时候就会自动加上转载标签了。如果这个值为空则不显示。这样在首页的摘要中不会显示转载信息，点开文章后才会显示。
