---
title: hexo接入valine评论采坑
date: 2020-11-17 18:20:40
tags: hexo
---

配置过程很简单
1、 注册leanCloud，创建应用
2、 按照教程配置好AppId和AppKey

然后问题来了：
```
Code 401: 未经授权的操作，请检查你的AppId和AppKey.
```       
 <!--more-->
折腾了一个下午，还是没有找到问题所在，后来注册了leanCloud国际版重试了一遍，好了

{% note warning %}
划重点：一定要注册 {% label danger@leanCloud国际版 %}
{% endnote %}




