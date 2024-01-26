---
title: '【捣鼓】WSL部署总结'
date: 2019-04-07 12:04:52
tags: [WSL,折腾,旧文归档]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---

Windows Subsystem for Linux 是微软在Windows 10和 Windows  Server 2019上添加的能够原生运行Linux二进制可执行文件的兼容层。

简单的记录了一下部署的过程。

<!-- more -->

## Windows Subsystem for Linux 简介

 Windows Subsystem for Linux 是微软在Windows 10和 Windows  Server 2019上添加的能够原生运行Linux二进制可执行文件的兼容层。

由于没有硬件仿真/虚拟化（与coLinux等其他项目不同），WSL直接使用主机文件系统（通过VolFS和DrvFS）和硬件的某些部分，例如网络（Web服务器，用于例如，可以通过主机上配置的相同接口和IP地址进行访问，并且对使用需要管理权限的端口或已经被其他应用程序占用的端口共享相同的限制），这保证了互操作性。

此子系统无法运行所有Linux软件（如32位二进制文件）或需要在WSL中未实现的特定Linux内核服务的软件。由于WSL中没有“真正的”Linux内核，因此无法运行内核模块（如设备驱动程序）。

可以通过在Windows（主机）环境（例如VcXsrv或Xming）中安装X窗口系统来运行一些图形（GUI）应用程序（例如Mozilla Firefox），尽管并非没有警告，例如缺乏音频支持或硬件加速（导致图形性能不佳）。当前还没有实施对OpenCL和CUDA的支持，尽管计划在将来的版本中使用。

~~**——以上说明复制自Wikipedia。**~~

总之，这是除了双系统之外，同时使用Windows和Linux的另一种**轻量级**的方法。虽然有局限性，但是一般的Linux使用还是没有什么问题的。对于日常学习，使用Linux而不想/不能离开Windows的同学来说有着很大的帮助。

## WSL的启用与安装

首先用管理员权限打开Windows Powershell。一种可能的打开方式是：`Windows徽标键 + X` -> `Windows Powershell(管理员)(A)`。

然后键入：

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

稍等片刻，会弹出按Y重启的通知。按Y之后会自动重启两到三次，再次进入桌面的时候，WSL就已经启用完毕了。

![](https://s2.ax1x.com/2019/04/07/AfHruD.png)

接着，需要选择一款你想要的Linux发行版。进入Microsoft Store，搜索Linux，进入Linux发行版专题。早些时候只有Ubuntu可以选择，但是现在除了Ubuntu之外还有Debian，Kali等多个发行版可以选择。当然作为只用的来Ubuntu的菜鸡，我选择了Ubuntu并下载。

![](https://s2.ax1x.com/2019/04/07/AfHsDe.png)

安装完成之后，你的WSL就已经初步部署完毕了！

有两种方法可以进入WSL：

1. 通过Powershell指令`bash`进入WSL。
2. 直接点击Ubuntu图标进入WSL。

打开之后，键入`lsb_release -c`，显示`Codename:        bionic`。说明我们安装了Ubuntu18.04 LTS。

## WSL的初步配置

下载完成之后官方的数据源仓库是ubuntu仓库，由于是境外网站，而且WSL不能走SOCKS的原因，所以最好把数据源更换成国内镜像站。这里推荐清华的镜像。

首先备份原来的数据源配置文件:

```sh
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

然后修改数据源配置文件，可以通过自带的`vim`。如果不会用的话，可以手动用Windows上的notepad++之类打开`%UbuntuPath%\LocalState\rootfs\etc\apt\sources.list`修改。

全选，删除，修改成以下内容：

```sh
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main multiverse restricted universe

```

**注意：**这仅限于Ubuntu 18.04，即bionic版本。如果你`lsb_release -c`后显示的结果不是bionic，那么请将以上数据源地址中所有的`bionic`替换成你的版本代号。

修改保存之后，更新配置并且升级（可能会占用较长的时间，可以起身转一转）：

```sh
sudo apt update
sudo apt upgrade
```

到此，WSL已经初步配置完成啦。

## WSL的本地化

虽说日常使用用英文也没什么大碍，但是既然有中文本地化，就没理由补折腾一番啦。

首先安装中文本地化包：

```sh
sudo apt install language-pack-zh-hans
```

然后更新本地化设置：

```sh
sudo update-locale LANG=zh_CN.UTF-8
```

接着安装Noto CJK包来防止中文被显示为日文字形，导致的全员小方框乱码故障：

```sh
sudo apt install fonts-noto-cjk
```

重启WSL，现在已经是中文本地化模式了。

![](https://s2.ax1x.com/2019/04/07/AfHgUA.png)

## WSL配合Xming显示GUI

WSL默认是没有安装图形窗口系统的（毕竟只有300MiB大），所以要想显示GUI，必须要再自己安装。

X窗口系统，是一种以位图方式显示的软件窗口系统。好像Windows自带的X.Org就是一种X窗口系统，Xming这个工具是通过给WSL提供端口来实现图形化界面的显示的。

官方的Xming已经年经失修，推荐一个魔改版本VcXsrv。下载地址：[SourceForge](https://sourceforge.net/projects/vcxsrv/)，[百度云](https://pan.baidu.com/s/1y3AHoNbCiLJj-AndnlUQsw)(提取码`hm4o`)。

安装VcXsrv之后，启动XLaunch，开放0端口。然后打开WSL，输入：

```sh
echo "export DISPLAY=127.0.0.1:0">>~/.bashrc
```

这样就把默认的显示配置完成了。

### 安装Ubuntu Desktop (可选) 

Ubuntu-Desktop不是Ubuntu桌面，只是安装了Ubuntu GUI默认安装的图形工具，比如gedit之类。比较大，有2G那么大，而且我的Windows上也不怎么需要Ubuntu Desktop，所以我就没安装。

直接键入：

```sh
sudo apt install ubuntu-desktop
```

就完事了。有2G大，估计会持续很长时间。

### 安装Gnome-terminal

如果你安装了`ubunutu-desktop`可能可以跳过这一步，因为`ubuntu-desktop`包中应该已经安装好了`Gnome-ternimal`的依赖项。当然打一行指令也没啥。

```sh
sudo apt install dbus-x11
```

然后就是安装了，没啥好说的。

```sh
sudo apt install gnome-terminal
```

好了，安装完成。保持后台Xming服务开启，终端输入:

```sh
gnome-terminal
```

然后应该就弹出终端了。

![](https://s2.ax1x.com/2019/04/07/AfHyHH.png)

### 安装Geany

Geany是ICPC World Final 2019上，华沙大学队伍使用的编辑器。轻量，五脏俱全，支持C, C++, Java, Python以及各种前端语言的语法高亮和自动补全。~~我寻思这个支持的语言列表，难道是面向ACM开发的嘛……~~众所周知，Code::Blocks在Ubuntu上经常闪退，而赛场上的vim肯定没什么时间深度配置所以也不太好用，CLion要收费，所以这个Geany安装之后玩了一会儿就爱上了。

添加ppa，更新包列表，安装一气呵成：

```sh
sudo add-apt-repository ppa:geany-dev/ppa
sudo apt-get update
sudo apt-get install geany
```

然后在终端键入`geany`，不出意外的话就会弹出来啦！

![](https://s2.ax1x.com/2019/04/07/AfHcEd.png)

在首选项里配置一番就开始玩吧！当然，别忘了安装相关语言的环境：

```sh
# C/C++
sudo apt install build-essential
# JDK (Unconfirmed)
sudo apt install default-jre
sudo apt install default-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/bin/
export JRE_HOME=$JAVA_HOME/jre
source /etc/environment
# Python
sudo apt install python3
```

**注意：**Java的环境变量地址请根据自己的安装路径调整，查看安装地址可以使用：

```sh
sudo update-alternatives --config java
```

## WSL要知道的

- Windows和WSL中的路径对应关系：

  - 磁盘根目录对应：

    ```sh
    # Windows
    C:\
    # WSL
    /mnt/c/
    ```

  - WSL根目录对应：

    ```sh
    # Windows
    C:\Users\%USER_NAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_%VERSION_HASH%\LocalState\rootfs\
    # WSL
    /
    ```
    **注意！请不要直接从C盘访问该地址修改内容，会导致权限混乱。Windows10-1903以上版本请在WSL运行时访问`\\$wsl\`**

- 卸载WSL

  Powershell：`lxrun /uninstall /full`

- 一个更好的wsl终端，可以用来显示32位色：

  [WSL-terminal on github](https://github.com/goreliu/wsl-terminal)

  ![](https://s2.ax1x.com/2019/04/07/AfHBjO.png)

## 参考资料

[Windows Subsystem for Linux Installation Guide for Windows 10](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

[适用于 Linux 的 Windows 子系统 ——维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E9%80%82%E7%94%A8%E4%BA%8E_Linux_%E7%9A%84_Windows_%E5%AD%90%E7%B3%BB%E7%BB%9F)

[Issues with dbus-launch and x11 forwarding and /etc/machine-id? ——Github](https://github.com/Microsoft/WSL/issues/2016)

[[Asking] WSL, Ubuntu 18.04, DBUS fix updated instruction? ——Reddit](https://www.reddit.com/r/bashonubuntuonwindows/comments/9l46br/asking_wsl_ubuntu_1804_dbus_fix_updated/)

[How to Install Geany IDE on Ubuntu 18.04 & 16.04 LTS ——TecAdmin](https://tecadmin.net/install-geany-ide-ubuntu/)

[How To Install Java with apt on Ubuntu 18.04 ——DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-18-04)

[如何正确为 Noto Sans CJK 配置 fontconfig 使中文不会被显示为日文字型？——知乎](https://www.zhihu.com/question/47141667)

## 总结

以上就是本次总结的全部内容了。

人人都应该试一试Linux！玩得愉快！

**冻葱Tewi** 于 **2019-04-07**