---
title: '【捣鼓】Ubuntu及vim、gcc配置笔记'
date: 2018-12-08 18:36:11
tags: [折腾,旧文归档,笔记]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
最近突然发觉ICPC比赛中的机器都是Ubuntu环境，而我却从来都没有用过Linux操作系统。所以便打算在实验室的电脑上捣鼓一番，熟悉一下比赛中使用的环境。

<!--more-->

## 二、Ubuntu的安装及简单配置

我选择的操作系统是Ubuntu 18.04.1 LTS。从Ubuntu的官网下载系统iso文件之后，用Rufus刻录到一个新买的优盘当作启动盘（其实2G就足够了，但是学校京东竟然已经没有这么小的优盘了）。然后在开机时进入BIOS选择从U盘启动，选择直接覆盖安装Ubuntu，接着就是一顿傻瓜式安装引导，Ubuntu的安装就完成了。

在设置里稍微设置了一下可以自定义的选项。值得一提的是，18.04版本的Ubuntu在安装的时候如果选择了中文，那么他会将/home/下的文件夹也建立成中文的，这会在终端使用过程中导致一系列问题。要解决这个问题，需要在区域与语言设置里，先将系统语言设置成中文并重启，在开机完成后会提示是否要替换成英文文件夹。替换成英文文件夹后再换成中文环境并重启，这次重启后选择保留原样即可保留英文文件夹名称。

## 三、Code::blocks的安装及简单配置

一开始我是想继续使用code::blocks这样的IDE。但是code::blocks在半个小时之内崩溃了三次让我身心俱疲，于是就进入了vim的大坑。这里还是简单地记录一下code::blocks的安装方法:

（以下安装过程在终端中完成）

```shell
sudo apt-get install build-essential
	//build-essential是一些基础的编译环境的集成包，GCC等工具都被包含在内
sudo apt-get install codeblocks
	//安装codeblocks的基础包
sudo apt-get install codeblocks codeblocks-contrib
	//安装codeblocks的插件包（自动补全、自动对齐、自动缩进等）
```

CB默认将Xterm作为调试终端，如果想要美化一下，使用Ubuntu的Gnome终端的话，则只需要在CB的环境设置里将Terminal to launch console programs选项栏填写成：`gnome-terminal-t $TITLE -x`即可。

到此，CB已经基本安装完成，简单设置之后就可以使用辽。但可能是由于兼容性还是什么的问题，我在Ubuntu上使用codeblocks的半小时内codeblocks崩溃了三次。让我十分恐惧，所以就想到用另一个据说会用之后十分好用的编译套装：`vim + gcc + gdb`。

## 四、vim的安装及配合gcc的简单应用。

很早就听说vim是一个会用之后运笔如飞的文本编辑器，今天终于有机会来使用一波这个广受好评（？）的文本编辑器辽。

vim 的安装与其他软件包无异,没有安装 vim 的机器在终端键入 `sudo apt-get install vim` 再 y一下即可完成安装。安装完成后,键入 vim ,就可以看到 vim 的主页面 (?) 辽:

在 Linux 中, Ctrl + C 可以强制中断进程, Ctrl + Z 可以将进程挂起。当在 vim 里出不去的时候可以用这两个快捷键找出出路(大雾)。

**vim 有三种模式:命令模式,输入模式和底线命令模式。**

命令模式就是刚启动 vim 的模式。此时的键盘操作会被 vim 识别为命令而不是字符。大小写的i , a , o 分别可以以不同的光标位置进入编辑模式。  
输入模式则是编辑文件的模式。此时的键盘操作是正常的操作文件。 Esc 键可以退出到命令模式。  
底线命令模式,是在命令模式下输入英文冒号 : 后进入的模式。此模式下,可以使用单个或者多字符命令的模式，可用的命令非常多。 Enter 输入命令后或者 Esc 键可以返回命令模式。

**要用 vim 舒服的编辑 cpp 文件,则需要简单的配置一下:**

在终端输入 `vim ~/.vimrc` 就会进入 vim 的配置文件。如果是去参加比赛，不适合仔仔细细地一个一个配置，所以简单地撰写以下内容基本就可以很舒服的写代码了:

```vimrc
syntax on # 语法高亮
set mouse=a # 允许鼠标点击
set cindent # 使用 C 缩进
set tabstep=4 # 一个 tab 是 4 个空格(也可以写成 set ts=4 )
set shiftwidth=4 # 自动缩进是 4 个空格(也可以写成 set sw=4 )
set nu # 显示行号(没想明白为啥是 nu 这两个字母)
```

**vim 常用的快捷键很多,但是比赛时需要用到的命令其实也没有太多:**

```vimrc
i 从当前光标进入编辑模式
I 从当前行第一个非空格进入编辑模式
a 从当前光标的下一个字符进入编辑模式
A 从当前行最后一个字符处进入编辑模式
o 在当前行下方另起一行进入编辑模式
O 从当前行上方另起一行进入编辑模式
x 删除光标处的字符
X 删除光标前的字符

dd 删除( delete )当前行( 5dd 删除当前行向下 5 行)
yy 复制( yank )当前行( 5yy 复制当前行向下 5 行)
p 粘贴( paste )到当前行下
P 粘贴( paste )到当前行上
u 撤销( undo )
Ctrl + r 重做( redo )
. (英文句号)继续做上次操作

j k 上下移动光标
h l 左右移动光标

:w 写入( write )即保存
:q 退出( quit )
:q! 强制退出
:wq 保存并退出
  另外, :w 可以在后边加入文件名参数来另存为
/word 查找当前光标下的第一个 word
?word 查找当前光标上的第一个 word
:%s/oldword/newword/g 将文件中的所有 oldword 替换成 newword
:%s/oldword/newword/gc 将文件中的所有 oldword 替换成 newword ,在替代前让用户确认
:a,bs/oldword/newword/g 将 a 到 b 行中的所有 oldword 替换成 newword ,当然也有对应的 gc 版本,其中若 b 为 $ 则表示文件末尾。
n 正向重复上次搜寻
N 反向重复上次搜寻
```

**编写、编译运行一个 cpp 文件的一般流程一般是这样的:**

```shell
1. 用终端进入想要存放代码的位置( cd 或者直接在指定文件夹打开终端)。

2. 使用 vim filename.cpp 命令来打开一个 cpp 文件(若不存在则创建)。

3. 编写代码。

4.1 在本终端窗口编译代码, :wq 保存并退出 vim 就可以进行操作 5 。
4.2 Ctrl + Shift + N 新建终端窗口编译代码,需要 :w 保存后新建终端窗口执行操作 5 。

5. 终端中键入 g++ filename.cpp -o filename -std=c++11 -O2 来编译可执行文件。
上句命令即为 g++ 的编译指令,
第一个参数是代码文件名, 
-o 制定目标名称为 flename (缺省值为 filename.out ),
-std=c++11 来指定 c++11 标准, 
-O2 开启氧气优化。
若要使用 gdb 调试,则最好加上 -g 编译选项来更好地支持调试功能。

6.1 可以使用 ./filename 来直接运行编译出的可执行文件。
6.2.1 也可以使用 gdb 工具: gdb filename 来进行 gdb 调试:
6.2.2 r 运行程序。
更多的调试命令可以查看紫书 ( 刘汝佳 著《算法竞赛入门经典 第 2 版》 )P463 附表。
```

到此,在 Ubuntu 上的编译环境已经基本配置完成,然后就可以在 ICPC 中徜 pa 徉 xing 了。本文为冻葱 Tewi 记录回顾之用,若能够对你的学习工作进程有所帮助,不胜荣幸。

以上。
