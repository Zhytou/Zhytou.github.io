---
title: "搭建自己的生产环境"
date: 2023-07-03T10:01:07+08:00
draft: false
---

随着Coding的时间越来越长，我接触到的工具也越来越多，也逐渐有了一套自己的工具包。这篇博客就记录了我对WSL、Docker、Shell、SSH、TMux和Vim等工具的学习和使用笔记以及一些环境配置的方法。

## Virtual Machine & Containerization

首先，介绍一下最基础也是最常见的虚拟化工具。由于我们的电脑一般是Windows，但课程实验往往要求Unix/Linux的开发环境，此时我们就需要使用虚拟化技术来模拟目标环境。

使用虚拟化技术的好处如下：

- 更高效利用物理服务器资源。一个物理服务器可以运行多个虚拟机,更充分地利用CPU、内存和存储资源。
- 更高的弹性和可扩展性。通过克隆和删除虚拟机,可以快速实现应用程序的弹性伸缩。
- 更好的隔离性。虚拟机相互隔离,不会相互影响。如果一个虚拟机宕机,也不会影响其他虚拟机。
- 更方便的测试和开发。可以快速部署虚拟机用于软件测试和开发。

而虚拟机（Virtual Machine，VM）和容器化（Containerization）是两种重要的虚拟化技术。二者区别如下：

![vm vs container](https://zhytou.github.io/post/2023-7-3/vm_vs_container.png)

下面介绍最常用的两种虚拟化工具WSL和Docker。

### WSL

**Introduction**：

适用于Linux的Windows子系统（Windows Subsystem for Linux，WSL）是一个为了在Windows操作系统上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。

**Setup**：

简单来说，可以把WSL想象成Windows自带的Linux虚拟机，但是它并不是默认开启的，需要先检测是否支持，然后再将其在设置中打开。具体步骤如下：

- 重启电脑按住F10进入BIOS，确认开启虚拟化功能；
- 控制面板-程序-启用或关闭Windows功能-开启“适用于Linux的Windows子系统”功能；
- 前往微软商店，选择Linux发行版安装即可。

### Docker

**Introduction**：

Docker 是一种轻量级的虚拟化解决方案，可以将应用程序和其依赖项打包成一个可移植的容器，然后在不同的环境中运行。Docker 容器包含了运行应用程序所需的所有组件，包括代码、运行时、系统工具、系统库等，因此可以确保应用程序在不同的环境中都能够稳定运行。

**Notion**：

镜像（Image）是一个只读的模板，用于创建Docker容器。它包含了一个容器需要的所有内容，包括代码、运行时、库、环境变量和配置文件等。

容器（Container）是Docker运行时的实例。它和镜像的关系就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

仓库（Repository）是用于存储Docker镜像的地方，可以理解为一个代码库。

**Setup**：

### Other Virtualization Tool

除了上面提到的WSL和Docker两种最重要的虚拟化工具，我在这里补充一些其他常用的软件或工具：

**BusyBox**：嵌入式 Linux 的瑞士军刀

**Virtual Box**：虚拟机软件

## Shell

接着，必须要介绍的工具就是Shell。它是Unix/Linux操作系统下传统的用户和电脑的交互界面，是一种典型的命令行接口（Command Line Interface，CLI）程序。常见的Shell程序有Bourne Shell(sh)、Bourne Again Shell(bash)、Z Shell(zsh)等。

此外，我们有时也以Shell代指Shell编程语言（Shell Programming Language）。它是一门 “能够把用户指令翻译成系统调用” 的编程语言，而由Shell Language所编写的程序则被称为Shell脚本（Shell Scripts）。因此，Shell也可以理解成控制系统的脚本语言的解释器。

补充：可以阅读这篇[实验指南](https://jyywiki.cn/OS/2022/labs/M4)来了解如何实现一个类似Python Shell的交互式Shell。

### Basic Functions

Shell提供了很多功能和特性，下面将着重介绍其中最重要和常见的几个。

**命令执行 Commands Excution**：

首先，Shell最基础也是最重要的功能——执行命令。一般来说，Shell中可执行命令分成Shell函数、Shell内置命令和普通可执行程序三类。

其中，最常见的就是普通可执行程序，以我们所常用的ls、grep、mv等来自[GNU Coreutils](https://www.gnu.org/software/coreutils/)核心工具集的命令为例，它们实际上就是一组默认安装在Linux系统的程序。更多有关GNU Tools的介绍可以阅读我的另一篇博客[一文了解GNU Tools](https://zhytou.top/post/2023-6-27/gnu-tools/)。

除此之外，我们自己编写的可执行程序也可以使用Shell来执行，方法就是使用绝对路径或相对路径调用该程序。此外，以cd、pwd、alias等为代表的Shell内置命令也是非常常见。

值得注意的是，有时候使用Shell执行命令也会出现错误。其中，除了名称或参数调用错误之外，最常见的就是未赋予可执行权限和无法查找到指定命令。前者可以使用chmod命令赋予相应权限，后者则涉及到Shell执行命令的流程。一般来说，Shell执行命令的大致流程是:

- 将用户输入的命令解析为命令名称和参数。
- 查找命令名称对应的可执行文件名。
- 根据找到的可执行文件名，使用相应的路径执行命令。

因此，无法查找到指定命令涉及到Shell的又一个重要特点，即：环境变量。

**环境变量 Environment Variables**：

Shell环境变量一些可用于Shell脚本和命令执行中的信息。常见的Shell环境变量有：

- PATH:可执行文件的搜索路径
- HOME:用户主目录
- SHELL:当前使用的Shell程序
- USER:当前用户名称

此外，我们可以使用export或env命令查看或设置当前Shell的环境变量。

值得注意的是，使用export设置Shell环境变量只会对当前Shell进程有效。若希望长期修改某环境变量，则应该去修改相应Shell配置文件，或者修改操作系统的环境变量。因为，Shell在初始化时会继承操作系统的所有环境变量。

**重定向 Redirections**：

Shell重定向是Shell的一种强大特性，它可以将一个命令的输出重定向到另一个文件或者另一个命令的输入。

```bash
# 将cmd的标准输出写入file.txt
cmd > file.txt

# 将cmd的标准输出追加到file.txt
cmd >> file.txt

# 将cmd的错误输出写入file.txt
cmd 2> file.txt

# 将file.txt作为cmd的输入
cmd < file.txt
```

> 标准输出（stdout）是程序默认输出的流，它通常指向控制台或屏幕，在程序运行时会将输出信息打印到屏幕上。
>
> 标准输入（英文缩写为stdin）是程序默认从中读取输入的流，它通常指向键盘，在程序运行时会等待用户输入数据。
>
> 错误输出（英文缩写为stderr）是程序默认输出错误信息的流，它也通常指向控制台或屏幕，在程序运行时会将错误信息打印到屏幕上，以便程序员进行调试和故障排除。

**管道 Pipeline**：

Shell管道是一种特殊的重定向，它可以将一个命令的输出重定向到另一个命令的输入。

```bash
# 将cmd1的输出作为cmd2的输入
cmd1 | cmd2
```

值得注意的是，Shell管道其实就是依靠操作系统中进程通信方式之一的管道实现的。

**别名 Aliases**：

Shell别名允许我们使用更简短或者更容易记忆的命令代替完整的命令。

```bash
# 将cmd命名为new_cmd
alias new_cmd='cmd'
```

此外，还有两种与Shell别名类似的命令我想特别强调一下。首先是使用mv命令重命名文件，它和alias的不同之处在于mv 真正地改变文件的名字，而alias不会修改原文件。

其次是使用ln命令符号链接文件，它和alias的不同之处在于ln 创建了一个实际的链接文件，其它程序都可以读取和使用，而alias定义的别名只对Shell有效。

此外，Shell作为一门编程语言，也类似其他语言一样提供了运算符、参数、变量、函数、流控制等功能。下面介绍一些Shell作为一门编程语言的相关特性。

**变量和参数 Variables & Parameters**：

**操作符 Operators**：

```txt
# Control operators:
& && ( ) ; ;; | || <newline>

# Redirection operators:
< > >| << >> <& >& <<- <>
```

**函数 Functions**：

**流控制 Flow Control**：

```bash
# if else fi
if xxx
then xxx
elif xxx
then xxx
else xxx 
fi

# while
while xxx
do xxx
done

# for

```

更多更详细的介绍可以使用指令`man sh`来阅读sh的man page。

### Simple Usage

下面我会写一些Shell脚本来演示上述功能。

```bash
```

### Prettifying

作为一个开发者，一个美观的Shell不仅能提高工作效率，还能让我们保持愉悦的心情，所以Shell的美化是很有必要的。

对于我而言，我主要使用Oh-My-Zsh和Oh-My-Posh这两种插件来分别美化Zsh和Powershell两个最常用的Shell。下面来介绍一下两者的配置方法。

**Setup For Oh-My-Zsh**：

- 首先，使用以下命令`sh -c "$(wget -O- https://gitee.com/pocmon/mirrors/raw/master/tools/install.sh)"`安装Oh-My-Zsh；
- 接着，修改.zshrc配置文件，更换zsh主题，具体方法如下：`ZSH_THEME="你想要的主题名称"`；
- 最后，重启终端使设置生效。
- 推荐一个还不错的主题Powerlevel10k。

**Setup For Oh-My-Posh**：

- 首先，在微软商店中下载Oh-My-Posh；
- 其次，编辑powershell配置脚本（类似.bashrc或.bash_profile），在该脚本中添加以下指令`oh-my-posh init pwsh | Invoke-Expression`；
- 接着，保存退出，使用`. $PROFILE`命令执行该脚本；
  - 若PowerShell报错不允许执行任何脚本，则需要在管理员模式下执行以下命令`set-ExecutionPolicy RemoteSigned`解除紧张执行脚本的设置；
  - 接着，使用`. $PROFILE`命令再次执行；
- 然后，使用`Get-PoshThemes`命令，获取Oh-My-Posh支持的所有主题，
- 选择一个你喜欢的主题，修改$PROFILE为以下内容`oh-my-posh init pwsh --config 'C:\Users\[Your Name]\AppData\Local\Programs\oh-my-posh\themes\[Theme Name].omp.json'| Invoke-Expression`；
- 最后，再次执行$PROFILE，使配置生效。
- 此外，可能还需要安装字体（可以前往[这里](https://www.nerdfonts.com/)寻找字体），才能保证显示正常。安装完成后，在终端设置-默认值-外观-字体中选择新下载的字体，即可正常显示。

### Remote Shell(SSH)

除了在本地使用Shell进行项目开发和调试，我们还常常需要在远程服务器上进行项目部署和运维。远程Shell（Remote Shell）也应运而生。

而SSH(Secure Shell)正是一种实现安全远程Shell的方式。SSH是一种能够提供安全的远程登录和命令执行功能的安全协议，它的出现代替了Telnet以及其他非安全远程Shell。

**Asymmetric Cryptography**：

SSH使用非对称加密（Asymmetric Cryptography）来保证安全连接，与之相对的则是对称加密（Symmetric Cryptography）。二者的区别在于，前者加密和解密使用的密钥不同，加密使用公钥，解密则使用私钥；而后者则是加密解密使用同一密钥。

除了SSH使用了非对称加密技术之外，还有用于实现互联网的HTTPS和电子邮件的SMTPS等安全协议也基于该加密技术。

**Login Steps**：

- 登录者向主机发送一个登录请求，附带登录者自己的公钥。
- 主机收到请求后，会检查登录者的公钥是否在主机的authorized_keys文件中。
- 如果在authorized_keys中，则登录成功；反之，则登录失败。

**Password Login vs Public Key Login**：

使用SSH协议登录远程主机一般分成口令登录（Password Login）和公钥登录（Public Key Login）。其中，前者依靠口令，也就是密码验证登录者的身份；后者则是利用登录者公钥验证身份。

具体来说，公钥登录就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录，不再要求密码。

对于口令登录来说，它的麻烦之处就是需要每次输入密码。特别是，当你第一次登录远程主机时，系统甚至会给出下面的提示：

```txt
　　The authenticity of host 'host (12.18.429.21)' can't be established.

　　RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.

　　Are you sure you want to continue connecting (yes/no)?
```

这段话的意思是，无法确认host主机的真实性，只知道它的公钥指纹，问你还想继续连接吗？

当你成功连接一次之后，这个警告就不再出现了。因为这台远程主机的信息已经被保存在了文件$HOME/.ssh/known_hosts之中。下次再连接这台主机，系统就会认出它的公钥已经保存在本地了，从而跳过警告部分，直接提示输入密码。

**Simple Usage**：

```bash
# 生成密钥对
# -t rsa/dsa 指定要创建的密钥类型。
# -C comment 提供注释
ssh-keygen -t rsa -C "xxx@gmail.com"

# 将公钥传送到远程主机上
ssh-copy-id username@remote_host

# 登录
ssh username@remote_host
```

除了可以使用Linux中的ssh命令来实现远程Shell之外，还可以使用一些SSH客户端，比如：PuTTY、WinSCP等。

## Terminal & Job Control

终端（Terminal）早期是指一种允许工程师向计算机发送指令并观察其运行结果的硬件设备，但现在我们一般所说的终端其实都是终端模拟器（Terminal Emulator），即使用软件模拟文本输入输出的界面。

### Use Windows Terminal

**Shortcuts**:

![windows_terminal](https://zhytou.github.io/post/2023-7-3/windows_terminal.png)

- 打开/关闭终端：Ctrl+Alt+T 或 Win+Shift+Enter
- 创建新的标签页：Ctrl+Shift+T
- 关闭标签页：Ctrl+Shift+W
- 切换标签页：Ctrl+Tab 或 Ctrl+Shift+Tab
- 切换全屏模式：F11
- 缩放终端：Ctrl+鼠标滚轮 或 Ctrl++/-
- 复制文本：Ctrl+Shift+C
- 粘贴文本：Ctrl+Shift+V
- 打开命令面板：Ctrl+Shift+P 或 F1
- 改变字体大小：Ctrl+鼠标滚轮
- 以管理员身份运行当前终端：Ctrl+Shift+Enter

### Use TMux to Multliplex

### Dotfiles

点文件（Dotfiles）指Unix/Linux系统用户主目录下以点开头的纯文本配置文件。因为它们默认是隐藏文件，所以ls并不会显示它们。

对于bash来说，在大多数系统下，我们可以通过编辑.bashrc或.bash_profile来进行配置。此外，还有一些常见的配置文件如下：

- git - ~/.gitconfig
- vim - ~/.vimrc 和 ~/.vim 目录
- ssh - ~/.ssh/config
- tmux - ~/.tmux.conf

非常建议在点文件所在的文件夹下使用版本控制系统进行管理，然后通过脚本将其符号链接到需要的地方。这么做有如下好处：

- 安装简单: 如果您登录了一台新的设备，在这台设备上应用您的配置只需要几分钟的时间；
- 可移植性: 您的工具在任何地方都以相同的配置工作
- 同步: 在一处更新配置文件，可以同步到其他所有地方
- 变更追踪: 您可能要在整个程序员生涯中持续维护这些配置文件，而对于长期项目而言，版本历史是非常重要的

我们可以参考这个[网站](https://dotfiles.github.io/)了解如何个性化的配置自己的点文件，也可以参考[他人的配置文档](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)进行修改。

## Other

### Vim

### CI

## Reference

- [MIT-The Missing Semester of Your CS Education](https://missing.csail.mit.edu/2020/command-line/)

**Shell**：

- [NJU-OS笔记 13 Shell](https://zhytou.top/post/2023-6-19/nju-os/)

**SSH**：

- [阮一峰 SSH原理与运用](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
