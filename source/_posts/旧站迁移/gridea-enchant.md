---
title: '【折腾】Gridea Enchant 总结'
date: 2020-03-13 18:13:32
tags: [折腾,前端]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kano-smile-p2.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
Gridea 是一个类似于 Hexo 的静态博客管理框架。看中了它的颜值，但相比于 Hexo，有些功能不够全面。所以凑今天进行了一波魔改，将过程记录了下来。

<!-- more -->

注：本文中所有的`..`路径指`库/文档/Gridea`路径。

## 友情链接

换成 Gridea 之后，友链一直简单地在用[生成的](https://github.com/DCTewi/Links-mdGenerator)用 markdown 硬搓的 html/js，不够优雅，而且更新链条比较长（编辑json -> 跑py -> 复制markdown），所以自己魔改了一波 Gridea 和主题。

首先在`../themes/你的主题/config.json`文件中的`customConfig`列表中添加一个新对象：

```json
{
      "name": "friends",
      "label": "友链",
      "group": "友链",
      "type": "array",
      "value": [],
      "arrayItems": [
        { "label": "名称", "name": "siteName", "type": "input", "value": "" },
        { "label": "链接", "name": "siteLink", "type": "input", "value": "" },
        { "label": "Logo", "name": "siteLogo", "type": "input", "value": "" },
        { "label": "描述", "name": "description", "type": "textarea", "value": "" }
      ],
      "note": ""
    }
```

接着重启 Gridea，可以看到已经有了配置页。进行一波填写之后，进行接下来的操作。

![Gridea Config](https://s1.ax1x.com/2020/03/13/8KpixH.png)

将`../templates/`下的`post.ejs`复制一份，命名为你想要的地址，如`links.ejs`。接着进行一波编辑，将有关`post`的内容都删去，编辑一下页面名称，把正文的位置(一般都是`{<%- post.content %>}`)替换成友链内容，再随便加个洗牌，实例：

```html
<div class="post-content yue flex flex-wrap">
          
          <% if (site.customConfig.friends) { %>
            <% site.customConfig.friends.forEach(function(friend) { %>
              <div class="friend-box">
                <img class="friend-avatar" src="<%= friend.siteLogo %>">
                <div class="flink-info">
                  <a href="<%= friend.siteLink %>" target="_blank"><%= friend.siteName %></a>
                  <br/>
                  <%= friend.description %>
                </div>
              </div>
            <% }); %>

            <script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
            <script>
              // 这里注入了一个Fisher–Yates Shuffle算法用来洗牌
              jQuery.prototype.shuffle = function() {
                var res = this;
                for (var i = res.length - 1; i >= 0; i--) {
                  var rnd = Math.floor(Math.random() * (i + 1));
                  var ind = res[rnd];

                  res[rnd] = res[i];
                  res[i] = ind;
                }
                return res;
              }
              // 打乱一下顺序
              var items = $(".friend-box").shuffle();
              $(".post-content").empty().append(items);
          </script>
          <% } %>

        </div>
```

在`../themes/你的主题/assets/styles/main.less`中加点样式和动画：

```css
.friend-box {
  float: left;
  width: calc(50% - 20px);
  margin: 15px 10px;
  img.friend-avatar{
    width: 70px;
    height: 70px;
    border-radius: 50% !important;
    float: left;
    margin:0 15px 0 0 !important;
    transition:0.5s;
    -webkit-transtion:1.5s;
  }
  img.friend-avatar:hover{
    transform:rotate(360deg);
		-webkit-transform:rotate(360deg);
  }
  .flink-info{
    height:70px;
    overflow: hidden;
    line-height: 24px;
  }
}
```

就完事了，效果：[https://blog.dctewi.com/links/](https://blog.dctewi.com/links/)

## SEO 

Gridea 的主题大都不在网页标题加 title，SEO真的爆炸。Google Analytics 天天发邮件，一般在`../templates/post.ejs`里加个参数就可以了：

```html
<head>
    <%- include('./includes/head', { siteTitle: `${post.title} | ${themeConfig.siteName}` }) %>
</head>
```

百度提交和谷歌统计的 js 都放进`../templates/includes/head.ejs`合适位置就行了。

## 参考资料

- [三步创建 Gridea 友情链接页 - 林小沐](https://i.immmmm.com/gridea-templates-friends/)
- [jQuery如何对div进行排序 - SegmentFault](https://segmentfault.com/q/1010000000141873)
- [Flex 布局教程：语法篇 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

## 最后

没了，就这。

我的前端知识全是被逼出来的（悲）。