---
title: "搭建自己的生产环境"
date: 2023-07-03T10:01:07+08:00
draft: true
---

这篇博客记录了我对WSL、Docker、Shell、SSH、TMux和Vim等工具的学习和使用笔记，的一些环境配置。

## Virtual Machine

### WSL

### Docker

## Shell

Shell是Shell Language的解析器

补充：你可以阅读这篇[实验指南](https://jyywiki.cn/OS/2022/labs/M4)来了解如何实现一个类似Python Shell的交互式Shell。

### Simple Usage

**Hard Link**：

### Prettifying

**Oh-My-Zsh**：

**Oh-My-Posh**:

### Remote Shell(SSH)

SSH(Secure Shell)提供安全的远程登录和命令执行功能。

## Terminal & Job Control

终端（Terminal）早期是指一种允许工程师向计算机发送指令并观察其运行结果的硬件设备，但现在我们一般所说的终端其实都是终端模拟器（Terminal Emulator），即使用软件模拟文本输入输出的界面。

### Use Windows Terminal

**Shortcuts**:

![windows_terminal](./windows_terminal.png)

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

对于 bash来说，在大多数系统下，您可以通过编辑 .bashrc 或 .bash_profile 来进行配置。此外，还有一些常见的配置文件如下：

- git - ~/.gitconfig
- vim - ~/.vimrc 和 ~/.vim 目录
- ssh - ~/.ssh/config
- tmux - ~/.tmux.conf

我们在它们所在的文件夹下使用版本控制系统进行管理，然后通过脚本将其符号链接到需要的地方。这么做有如下好处：

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
- [NJU-OS笔记 13 Shell](https://zhytou.top/post/2023-6-19/nju-os/)
