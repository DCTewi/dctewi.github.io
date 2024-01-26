---
title: "Template"               # 标题
layout: post                    # 默认布局
date: 2024-01-27 00:05:27       # 创建时间
updated: 2024-01-27 00:05:52    # 更新时间
comments: true                  # 开启评论
permalink: template.html        # 覆盖文章的永久链接, 永久链接应该以 / 或 .html 结尾
excerpt: "Details"              # 纯文本的页面摘要
disableNunjucks: false          # 禁用标签渲染功能
lang: Zh-CN                     # 自定义本页面语言
published: false                # 文章是否发布
tags:                           # 标签 (没有顺序和层次)
- tag1
- tag2
categories:                     # 分类 (有顺序和层次)
- 生活
- 杂谈      # 获得了 生活->杂谈
- [旅游, 南京]
- [感想]    # 获得了 旅游->南京 和 感想 (并列分类) 
cover: /images/cover.jpg        # 封面
coverWidth: 1600                # 封面宽
coverHeight: 900                # 封面高
reprint: false                  # 是否是转载
---


Gallery:

| ![6ade6c2a3205d620306de34d8f5bc295.png](https://i.dawnlab.me/6ade6c2a3205d620306de34d8f5bc295.png)      | ![](https://i.dawnlab.me/b4bea1206475acb925968b76148b0e0a.jpg) | ![a29d9534105a19909dbc1fbece7b0621.png](https://i.dawnlab.me/a29d9534105a19909dbc1fbece7b0621.png) |
| ----------- | ----------- | ----------- |


{% gallery %}
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192753.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192754.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192755.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192756.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192757.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192530.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192531.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192532.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192533.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192534.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192535.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192415.jpg)
![1](https://cdn.jsdelivr.net/gh/nexmoe/image@latest/20210207192416.jpg)
{% endgallery %}


Links:

{% links shuffle %}
[
 {
  "title": "折影轻梦",
  "link": "https://nexmoe.com",
  "img": "https://www.gravatar.com/avatar/c7fd185f8c967dec20c29c75a40b9e09",
  "des": "为热爱战斗着，努力学着变得勇敢"
 }
]
{% endlinks %}

