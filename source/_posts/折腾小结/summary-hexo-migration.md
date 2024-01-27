---
title: Hexo迁移小结
date: 2024-01-27 15:39:19
cover: /images/bg.jpg
coverWidth: 1280
coverHeight: 726
categories:
- [折腾小结]
tags:
- 折腾
- Hexo
- 博客
---

转眼间又很久很久很久（三年）没有管博客了，为了不再荒废，痛定思痛，把博客迁移到了Hexo。

<!--more-->


这么多年过去了，博客也是转来转去折腾了好几次了。从一开始的Wordpress到静态博客，从Gridea到现在的Hexo。俺寻思，好久不写博客一定是因为Gridea的图形界面不好用（心虚）！另外，Gridea把各种密钥也一起放在了配置、文章目录下，使得前端和后端的仓库必须分离而且保持private，实在不方便我这种很多台设备不断切换的玩法。所以，把整个博客迁移到了Hexo。

为了庆祝迁移完工，也是给恢复更新开个头，简单记录一下这次的折腾过程。

## 安装和配置Hexo

安装Hexo其实挺傻瓜式的，基本按着[官方教程](https://hexo.io/zh-cn/docs/)走下来就ok了。

```shell
HEXO_PATH=~/Applications/Hexo
mkdir $HEXO_PATH && cd $HEXO_PATH

npm install -g hexo-cli
hexo init
npm install
```

就万事大吉了，就非常的傻瓜式。

接下来就是配置一下_config.yml和package.json了，根据需求照着改就OK。我的一些自定义：

```yaml
# 为了让_post文件夹内用于分类的子文件夹不影响URL的生成, 换成了:name
permalink: ':year/:month/:name/'

# 想了下, Github Page在国内某些运营商网络下可能比较慢, 所以镜像了Gitee Page
# 这就需要生成相对路径而不是绝对路径
relative_link: true

# 和主题配合设置代码高亮
syntax_highlighter: highlight.js
highlight:
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: true
prismjs:
  preprocess: true
  line_number: true
  tab_replace: ''

# 部署相关配置
deploy:
- type: git
  repo: https://github.com/DCTewi/dctewi.github.io
  branch: hexo
  message: '[update] hexo updated at {{ now(''yyyy-MM-dd HH:mm:ss'') }}'
- type: git
  repo: https://gitee.com/dctewi/dctewi.gitee.io
  branch: hexo-mirror
```

配置好之后直接`hexo clean && hexo g && hexo s --debug`就可以在本地4000端口预览了。

## 更换主题、配置插件

原版主题有种2016年Wordpress默认主题的味道，在[主题商店](https://hexo.io/themes/)翻了半天，找到了这个叫做[NexMoe](https://github.com/theme-nexmoe/hexo-theme-nexmoe)的主题，感觉还不错。

依旧是傻瓜式安装：

```shell
npm install hexo-theme-nexmoe hexo-renderer-inferno
hexo config theme nexmoe
hexo g && hexo s
```

进行一次hexo gen之后，会在根目录生成一个_config.nexmoe.yml，这就是主题的配置文件了。这个主题的配置还挺多的，配置了半天。

```yaml
# 直接使用了source/images/下的文件，毕竟都上传到Github了，什么图床也没有本地靠谱
avatar: /images/avatar.png # 网站 Logo

# 使用jsdelivr的开源CDN功能缓存文章内的图片
imageCDN: # 图片 CDN 功能
  enable: true # 开启该功能  
  # 查看源码发现这里其实是一个在文章渲染后进行全局匹配的正则
  # 使用前向匹配来避免匹配到不应该匹配的东西
  # 比如默认的../../images就会把非img标签内的字符串匹配掉
  # 这样，md内的![]()图片就会被替换到cdn去，达到编辑时和部署后的路径同步
  origin: (?<=<img.*?src=")../../images/ 
  to: https://cdn.jsdelivr.net/gh/DCTewi/dctewi.github.io@hexo/images/

# 在文章底部添加MathJax来提供Latex支持
slotFooter: |
  <script src="https://cdn.jsdelivr.net/npm/mathjax@latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
  <script>MathJax.Hub.Config({
      tex2jax: {
          inlineMath: [ ['$','$'], ["\\(","\\)"] ],
          processEscapes: true,
          skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
  });
  MathJax.Hub.Queue(function() {
      var all = MathJax.Hub.getAllJax(), i;
      for(i=0; i < all.length; i += 1) {
          all[i].SourceElement().parentNode.className += ' has-jax';
      }
  });</script>

# Giscus配置
slotComment: | 
  <script src="https://giscus.app/client.js"
          data-repo="DCTewi/dctewi.github.io"
          data-repo-id="你的Giscus配置"
          data-category="Giscus"
          data-category-id="你的Giscus配置"
          data-mapping="pathname"
          data-strict="0"
          data-reactions-enabled="1"
          data-emit-metadata="0"
          data-input-position="top"
          data-theme="noborder_light"
          data-lang="zh-CN"
          crossorigin="anonymous"
          async>
  </script>
```
之后把所需要的几个插件安装一下：

```shell
npm install hexo-generator-feed
npm install hexo-generator-json-content
npm install hexo-generator-sitemap
npm install hexo-word-counter
```

并且在_config.yml中进行配置：

```yaml
# hexo-generator-feed
feed:
  enable: true
  type: atom
  path: atom.xml
  limit: 20
  hub: null
  content: null
  content_limit: 140
  content_limit_delim: ' '
  order_by: '-date'
  icon: icon.png
  autodiscovery: true
  template: null

# hexo-generator-json-content
jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: false
    path: true
    text: true
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: false

# hexo-word-counter
symbols_count_time:
  symbols: true
  time: true
  total_symbols: false
  total_time: false
  exclude_codeblock: true
  awl: 4
  wpm: 275
  suffix: ''

# hexo-generator-sitemap
sitemap:
  path: sitemap.xml
  # template: ./sitemap_template.xml
  rel: true
  tags: false
  categories: false
```

到这里就大功告成了，剩下的就是迁移内容。因为都是Markdown，只需要把两个博客框架的front-matters统一一下就可以直接使用了。整体把文章移过去，收拾了一下关于、友链等几个页面，博客终于焕然一新了。之后最重要的还是保持更新吧，经常写点博客也确实是个好习惯。

## 其他的一些配置

### 配置[Giscus](https://giscus.app/zh-CN)

按照官网配下来就可以，本地配置在上方有写。

### 友链增强

原版主题其实就带友情链接的模板，但是是静态随机的。简单的搞了下动态随机。

```html
<!--https://github.com/DCTewi/dctewi.github.io/blob/hexo-backend/source/friends.md?plain=1-->
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
<script>
// 这里注入了一个Fisher–Yates Shuffle算法用来洗牌
jQuery.prototype.shuffle = function () {
    var res = this;
    for (var i = res.length - 1; i >= 0; i--) {
        var rnd = Math.floor(Math.random() * (i + 1));
        var ind = res[rnd]; res[rnd] = res[i]; res[i] = ind;
    }
    return res;
}
// 打乱一下顺序
var items = $(".nexmoe-py > ul > li").shuffle(); $(".nexmoe-py > ul").empty().append(items);
</script>

```

