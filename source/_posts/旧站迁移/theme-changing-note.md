---
title: '【旧文归档/捣鼓】更换主题小记'
date: 2019-01-10 23:03:49
tags: [折腾,旧文归档]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
用<a href="https://www.cssplus.org/" target="_blank" rel="noreferrer noopener" aria-label="（在新窗口打开）">路易大佬</a>的<a rel="noreferrer noopener" aria-label="（在新窗口打开）" href="https://www.cssplus.org/wordpress-theme-inspire.html" target="_blank">inspire</a>好几个月了，于是就闲得发慌脑子一抽没事找事想要换一个主题——

<!-- more -->

接着我就发现：**路易nb！**

因为inspire已经集成了许多方便实用的nb功能在里面，包括且不限于音乐播放、评论回复、友链模板、登陆保护、SMTP、统计代码、百度SEO等等功能。失去了inspire的庇护之后，发现这些东西全都需要自己搞。我捣鼓了几个小时之后才勉勉强强搞到能用的样子。在这里把一些关键步骤记录一下。

## 一、新评论自动通知评论作者

这个功能首先需要SMTP支持并且正确设置SMTP服务器。

在安装了WP插件商店中的某个SMTP插件并且配置完成之后，在主题的`functions.php`中代码最后（不是文件最后）添加如下代码即可：

```php
function comment_mail_notify($comment_id) {
	$comment = get_comment($comment_id);
	$parent_id = $comment->comment_parent ? $comment->comment_parent : '';
	$spam_confirmed = $comment->comment_approved;
	if (($parent_id != '') && ($spam_confirmed != 'spam')) {
		$wp_email = 'admin@' . preg_replace('#^www\.#', '', strtolower($_SERVER['SERVER_NAME']));//admin@改成可用的账户
		$to = trim(get_comment($parent_id)->comment_author_email);
		$subject = '您在 [' . get_option("blogname") . '] 的留言有了回应';
		$message = '&lt;div style="border-right:#666666 1px solid;border-radius:8px;color:#111;font-size:12px;width:702px;border-bottom:#666666 1px solid;font-family:微软雅黑,arial;margin:10px auto 0px;border-top:#666666 1px solid;border-left:#666666 1px solid">&lt;div class="adM">
&lt;/div>&lt;div style="width:100%;background:#666666;min-height:60px;color:white;border-radius:6px 6px 0 0">&lt;span style="line-height:60px;min-height:60px;margin-left:30px;font-size:12px">您在&lt;a style="color:#00bbff;font-weight:600;text-decoration:none" href="' . get_option('home') . '" target="_blank">' . get_option('blogname') . '&lt;/a> 上的留言有回复啦！&lt;/span> &lt;/div>
&lt;div style="margin:0px auto;width:90%">
&lt;p>' . trim(get_comment($parent_id)->comment_author) . ', 您好!&lt;/p>
&lt;p>您于' . trim(get_comment($parent_id)->comment_date) . ' 在文章《' . get_the_title($comment->comment_post_ID) . '》上发表评论: &lt;/p>
&lt;p style="border-bottom:#ddd 1px solid;border-left:#ddd 1px solid;padding-bottom:20px;background-color:#eee;margin:15px 0px;padding-left:20px;padding-right:20px;border-top:#ddd 1px solid;border-right:#ddd 1px solid;padding-top:20px">' . nl2br(get_comment($parent_id)->comment_content) . '&lt;/p>
&lt;p>' . trim($comment->comment_author) . ' 于' . trim($comment->comment_date) . ' 给您的回复如下: &lt;/p>
&lt;p style="border-bottom:#ddd 1px solid;border-left:#ddd 1px solid;padding-bottom:20px;background-color:#eee;margin:15px 0px;padding-left:20px;padding-right:20px;border-top:#ddd 1px solid;border-right:#ddd 1px solid;padding-top:20px">' . nl2br($comment->comment_content) . '&lt;/p>
&lt;p>您可以点击 &lt;a style="color:#00bbff;text-decoration:none" href="' . htmlspecialchars(get_comment_link($parent_id)) . '" target="_blank">查看回复的完整內容&lt;/a>&lt;/p>
&lt;p>感谢对 &lt;a style="color:#00bbff;text-decoration:none" href="' . get_option('home') . '" target="_blank">' . get_option('blogname') . '&lt;/a> 的关注，若有任何疑问，欢迎在博客留言，我会一一解答&lt;/p>&lt;p>(此邮件由系统自动发出，请勿回复。)&lt;/p>&lt;/div>&lt;/div>';
		$from = "From: \"" . get_option('blogname') . "\" &lt;$wp_email>";
		$headers = "$from\nContent-Type: text/html; charset=" . get_option('blog_charset') . "\n";
		wp_mail( $to, $subject, $message, $headers );
	}
}

add_action('comment_post', 'comment_mail_notify');
```

中间一大坨主要是邮件的格式，除了那一堆东西之外，东西不是太多。

## 二、登陆地址跳转

因为众所周知的原因，讲WP默认的wp-login.php更改是基本操作。基本上有两种比较简单的方法来达到目的：

### 1.更改php文件的名字

将站点内的wp-login.php重命名，然后将php内的几个wp-login.php都替换成新名字，就完事儿了。目前看来好像没什么bug的样子？

### 2.要求访问添加键值

在主题的functions.php内追加：

```php
function login_protec()
{
	if ($_GET['login'] != 'please') header('Location: https://baidu.com/');
}
add_action( 'login_enqueue_scripts' , 'login_protec' );
```

保存之后，就必须访问 dctewi.com/wp-login.php?login=please 才可以进入登陆界面，否则将会跳转到baidu.com。当然，键值对中的key和value可以随意更改，网址也一样。

## 三、友链页面及友链管理

换主题之后有一种怅然若失的感觉。突然发现是因为原来的<a rel="noreferrer noopener" aria-label="友链界面（在新窗口打开）" href="https://dctewi.com/links" target="_blank">友链界面</a>不管用了，不仅如此，原来仪表盘内的链接管理项也不存在了。慌的一批的我东凑西凑才找到了解决办法：

### 1.开启wp隐藏起来的链接管理器

熟悉的functions.php内追加：

```php
add_filter('pre_option_link_manager_enabled','__return_true');
```

然后就可以发现仪表盘内出现了链接管理器！是不是很简单（）？

### 2.建立页面模板

页面模板原则上可以放在wp路径内的任何一个位置。复制一个主题带的page.php。重命名副本后，在文件内添加：

```php
<?php
/*
Template Name: template-name
*/
?>
```

然后wp就可以将这个php识别为一个页面模板文件。在新建页面的时候就可以看到，高级（指gutenberg编辑器，经典编辑器在选项卡中部）选项卡内有了一个名字叫做template-name的模板。通过这个模板创建的页面会用模板php来排版而不是默认的page.php。

### 3.一个看起来海星的友链模板

这个模板是我到处凑出来的，在接连的修改中解决了一些本来存在的问题（比如li标签没有厚度，footer跑到友链上方，高度不协调导致形状感人等等）。

```php
<?php get_header(); ?>

<?php
/*
Template Name: 友情链接
*/
?>
<style>
	.link-content li
	{
		display: inline-block; //让li标签有宽度
		vertical-align: top; //让li标签以顶部对齐
		text-align: center;
		width: 100px;
		height: 150px; //让li标签的高度固定
		font-size:14px;
		margin-bottom:10px;
		list-style-type: none;
	}
	.link-content li img
	{
		border-radius:100%;
		margin-bottom:5px;
		width:64px;
		height=64px;
		transition:0.5s;
		-webkit-transtion:1.5s; //设置变换时间
	}
	.link-content li span
	{
		display:block; //让span成为块元素
	}
	.link-content li:hover img
	{
		transform:rotate(360deg); //简单的hover旋转
		-webkit-transform:rotate(360deg);
	}
</style>

<section id="content">

<?php the_post(); ?>
		
<article class="sp">

<hgroup class="p_lt p_a">

	<header class="p_t">
		<h2><a href="<?php the_permalink(); ?>">
			<?php the_title(); ?>
		</a></h2>
	</header> 

</hgroup>

<div class="sp_c">
<div>
 <?php
//获取友链并且随机排序
 $bookmarks = get_bookmarks('title_li=&orderby=rand'); 
//打印格式
 if ( !empty($bookmarks) ){
 	echo '<ul class="link-content clearfix">';
 	foreach ($bookmarks as $bookmark) {
 		$friendimg = $bookmark->link_image;
 		if(empty($friendimg)){
 			echo '<li><a href="' . $bookmark->link_url . '" title="' . $bookmark->link_description . '" target="_blank" >'. get_avatar($bookmark->link_notes,64) . '<span class="sitename">'. $bookmark->link_name .'</span></a></li>';
 		} else {
 			echo '<li><a href="' . $bookmark->link_url . '" title="' . $bookmark->link_description . '" target="_blank" >'. '<img src="'. $bookmark->link_image. '" />' . '<span class="sitename">'. $bookmark->link_name .'</span></a></li>';
 		}
 	}
 echo '</ul>';
 }
//打印完毕
 ?>
</div>
<?php the_content(); ?>
</div>

</article>

<?php comments_template( '', true ); ?>

</section>

<?php get_footer(); ?>
```

然后把我原来的传送门页面套用这个模板就欧凯了，页面会自动获取连接管理器内的链接。连接图片优先获取链接的 图像地址 项，如果没有填写图像地址，则会以 备注 中的邮箱来获取Gravatar头像，如果还是不存在则使用默认头像。具体效果可以看我的<a href="https://dctewi.com/links/" target="_blank" rel="noreferrer noopener" aria-label="（在新窗口打开）">友链界面</a>。

---

以上，做了一些微小的工作，让我的站点得以换个外观。但是我还是想说：坐等路易新主题pix，路易nb！

<del>这么搞了搞感觉自己的php水瓶有所提高了（大雾）。</del>
