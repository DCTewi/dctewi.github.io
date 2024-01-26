---
title: '【杂项】竞赛向 vimrc 配置小结'
date: 2019-08-06 12:15:47
tags: [vim,vimrc]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kagura-bigeye.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
竞赛向简洁风 vim 配置。

<!-- more -->

## 前言

~~俗话说，Vim 大法好。~~

前一段时间写题一直用的是 VS Code，VS Code 扩展丰富，功能强大，补全智能，跨平台。虽然打开的速度比较慢，但总体上来说用来写代码是真的舒服。

但可惜的是比赛中并没有 VS Code 可以拿来用。习惯了 IntelliCode 强大的基于文本和词法分析的补全功能之后，我发现自己连一行`ios::sync_with_stdio(false);`都不想写了。裸手代码能力在慢慢消退。

痛定思痛，我决定换回比赛中可以使用的 IDE 比如 Code::Blocks，但是在我的 Manjaro 笔记本的暗色微风主题下，CB 的界面充斥着各种暗色控件和亮色 Border ，变得特别丑，我还不如去用 Kate 或者 GEdit。这个时候，我突然回想起了在我开始用 Sublime Text 之前使用的神的编辑器 —— Vim。在花了大概一个多小时配置了一下 Vim之后，我已经完全爱上 Vim 啦！

这套配置遵守了以下的几个原则：

1. 不使用任何插件。因为比赛环境中是没有网络可以给 Vim 装插件的。
2. 尽量轻量，避免任何意义不大的配置。因为比赛中给选手用来调配置的时间并不多，应该在尽量快的时间内完成平常使用的配置。

本文不再记录 vim 的基本操作，请自行网上查询或者查看我[之前的文章](../vim-setup)。

## 逐条说明

话不多说，先逐条说明一下，在文章最后有单独的代码块：

```vimrc
if !exists("g:os")
    if has("win64") || has("win32") || has("win16")
        let g:os = "win"
    else
        let g:os = "unix"
    endif
endif
```

首先判断当前环境究竟是 Windows 还是 Linux，因为我平常会在两种环境下写代码，所以根据不同的情况需要进行不同配置。

### 基础设置：

```vimrc
set nocompatible              " 取消 vi 兼容模式
set showcmd                   " 右下角显示当前输入的命令
set mouse=a                   " 允许鼠标选中
set backspace=2               " 绑定 backspace 键到模式2
set encoding=utf-8            " 设定默认文件编码
set t_Co=256                  " 使用256色终端
filetype indent on            " 根据文件类型自动判断缩进类型
if g:os == "win"              " 将默认 yank 操作的寄存器设置为系统寄存器+/*
  set clipboard=unnamed       " 设置之后就可以用 y/p 操作系统剪切板了
else
  set clipboard=unnamedplus
endif
```

这里的 clipboard 设定需要 vim 的 X11 支持，请事先使用 `vim --version | grep clipboard`查看是否有 `+clipboard`。如果是减号，则需要安装 `vim-gtk`包或者 `gvim`包。

### 行为设置

```vimrc
set noswapfile                " 取消 swap 文件生成
set noerrorbells              " 取消声音提示
set autoread                  " 若文件被外部改动则自动更新内容
```

### 缩进设置

```vimrc
set cindent                   " 使用 C 系缩进方法
set tabstop=4                 " 设定 tab 宽度为 4
set shiftwidth=4              " 设定自动缩进宽度为 4
set expandtab                 " 将 tab 转换为空格
set softtabstop=4             " 将 tab 转换为 4 个空格
```

### 显示设置

```vimrc
syntax enable                 " 启用符号高亮
syntax on                     " 开启符号高亮
set number                    " 显示行号
set cursorline                " 高亮当前行
set showmatch                 " 高亮匹配括号以及高亮颜色设置
hi MatchParen cterm=bold ctermbg=none ctermfg=blue
```

### 搜索设置

```vimrc
set hlsearch                  " 高亮匹配结果
set incsearch                 " 逐字搜索
set ignorecase                " 不去别大小写
set smartcase                 " 若搜索关键字有大写则区别大小写
```

### 映射设置

```vimrc
if g:os == "win"
    map <F5> :w <CR> :!g++ "%" -o "%<.exe" -std=c++14 -O2 -Wall -DDEBUG ; "./%<.exe" <CR>
else
    map <F5> :w <CR> :!g++ "%" -o "%<.exe" -std=c++14 -O2 -Wall -DDEBUG && "./%<.exe" <CR>
endif

map <C-F5> :w <CR> :!python3 % <CR>
```

上边是一键<F5>编译 C++ 单文件的映射，其中`%`表示文件名，`%<`表示去掉扩展名的文件名，`:w`保存，`<CR>, <cr>`代表换行，`:!`表示在外部 Shell 中运行指令。

最后一行是按<Ctrl + F5> 运行 python 脚本的映射，可以根据设置将 python3 改成 python。

### 补全设置

```vimrc
inoremap ( ()<ESC>i
inoremap [ []<ESC>i
inoremap ' ''<ESC>i
inoremap " ""<ESC>i
inoremap { {}<ESC>i
inoremap {<CR> {<CR>}<ESC>O

inoremap <F8> #include<bits/stdc++.h><CR>using namespace std;<CR>typedef long long ll;<CR>
```

前几个是自动补全符号，其中如果只输入大括号则单行补全，大括号后按一下回车就会将大括号展开。最后一行是 F8 补全标准文件头，竞赛向嘛，竞赛向。

### GVim 设置

```vimrc
if has("gui_running")
    set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
    set fileencoding=utf-8       " 显示编码
    set guioptions-=m            " 隐藏菜单，工具栏和滚动条
    set guioptions-=T
    set guioptions-=r
    set lines=35 columns=120     " 设置大小
    colorscheme torte            " 设置配色
    if g:os == "win"             " 设置字体
        set guifont=Microsoft_Yahei_Mono:h11
    else
        set guifont=Fira\ Mono\ 12
    endif
endif
```

这一段是当使用 vim GUI 模式也就是 GVim 的时候进行的简单配置。比赛的时候可以无视，主要是平时在 Windows 上使用 Vim 的时候用到的。

## 汇总

```vimrc
" Vim Configuration File
" Author: dctewi@dctewi.com
" Description: For minimal icpc use
"

" System Detect
if !exists("g:os")
    if has("win64") || has("win32") || has("win16")
        let g:os = "win"
    else
        let g:os = "unix"
    endif
endif

" Properties
set nocompatible
set showcmd
set mouse=a
set backspace=2
set encoding=utf-8
set t_Co=256
filetype indent on
if g:os == "win"
  set clipboard=unnamed
else
  set clipboard=unnamedplus
endif

" Util
set noswapfile
set noerrorbells
set autoread

" Indent
set cindent
set tabstop=4
set shiftwidth=4
set expandtab
set softtabstop=4

" Layout
syntax enable
syntax on
set number
set cursorline
set showmatch
hi MatchParen cterm=bold ctermbg=none ctermfg=blue

" Search
set hlsearch
set incsearch
set ignorecase
set smartcase

" Compile
if g:os == "win"
    map <F5> :w <CR> :!g++ "%" -o "%<.exe" -std=c++14 -O2 -Wall -DDEBUG ; "./%<.exe" <CR>
else
    map <F5> :w <CR> :!g++ "%" -o "%<.exe" -std=c++14 -O2 -Wall -DDEBUG && "./%<.exe" <CR>
endif
map <C-F5> :w <CR> :!python3 % <CR>

" Auto complete
inoremap ( ()<ESC>i
inoremap [ []<ESC>i
inoremap ' ''<ESC>i
inoremap " ""<ESC>i
inoremap { {}<ESC>i
inoremap {<CR> {<CR>}<ESC>O

" Header
inoremap <F8> #include<bits/stdc++.h><CR>using namespace std;<CR>typedef long long ll;<CR>

" GVim
if has("gui_running")
    set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
    set fileencoding=utf-8
    set guioptions-=m
    set guioptions-=T
    set guioptions-=r
    set lines=35 columns=120
    colorscheme torte
    if g:os == "win"
        set guifont=Microsoft_Yahei_Mono:h11
    else
        set guifont=Fira\ Mono\ 12
    endif
endif
```

文章中 vimrc 不随着我的更新而更新。我当前使用的 vimrc 可在[我的Github上](https://github.com/DCTewi/My-Algorithm-Templates/blob/master/.vimrc)找到。

以上，Happy Vimming Everyday！

