---
layout: page
title: 准备事项
root: .
---

在学习之前，您需要安装几款软件。

1. [终端应用程序或命令行界面](
   #where-to-type-commands-how-to-open-a-new-shell)
2. [ssh应用程序](#ssh-for-secure-connections)

> ## Bash和SSH
>
> 本课需要一个终端应用程序（`bash`、`zsh` 或其他）能够安全地连接到远程机器（`ssh`）。
{: .prereq}

## 在哪里键入命令：如何打开一个新的Shell

Shell是一个程序，它使我们能够向计算机发送命令并接收输出。它也被称为终端或命令行。

某些计算机包含默认的Linux Shell程序。如果您已经安装了一个Linux Shell程序，下面的步骤描述了一些识别和打开Linux Shell程序的方法。还有用于识别和下载Linux Shell程序、Linux仿真器或访问服务器上的Linux Shell的程序的选项。

### Windows上的Linux Shell

具有Windows 作系统的计算机不会自动安装Linux Shell程序。在本课中，我们鼓励您使用Git for Windows中包含的模拟器，它可以让您访问Bash shell命令和 Git。

安装后，您可以通过从Windows开始菜单运行Git Bash程序来打开终端。

#### 用于Window 的Shell程序

* [Git for Windows][git4win] -- *推荐*
* [Windows Linux子系统][ms-wsl] -- Windows 10

> ## Windows版Git的替代品
>
> 其他解决方案可用于在Windows上运行Bash命令。现在有一个适用于Windows 10的Bash shell命令行工具。此外，您可以从您的Windows计算机在已经具有 Linux Shell的远程计算机或服务器上运行Bash命令。这通常可以通过 (SSH) 客户端来完成。 适用于 Windows 计算机的免费客户端之一是PuTTY。有关安装和使用PuTTY、使用Windows 10命令行工具或安装和使用Linux模拟器的信息，请参阅下面的参考资料。
>
> 对于高级用户，您可以选择以下选项之一：
>
> * 安装 [Windows Linux子系统] [ms-wsl]
> * 使用 Windows[PowerShell][ms-shell]
> * 阅读[使用Linux模拟器][Linux 模拟器] (Cygwin) 或 Secure Shell (SSH) 客户端 (PuTTY)
>
> > ## 警告
> >
> > 适用于Linux的Windows子系统 (WSL)、PowerShell或Cygwin中的命令可能与课程中显示或研讨会中介绍的命令略有不同。请询问您是否遇到这种不匹配的情况。
> {: .challenge}
{: .discussion}

### macOS上的Linux Shell

在macOS上，可以通过在Finde 的“/Application/Utilities”文件夹中运行终端程序来访问默认的Linux Shell。

要打开终端，请尝试以下一项或两项操作：

* 在Finder中，选择前往菜单，然后选择实用程序。 在Utilities文件夹中找到终端并打开它。
* 使用Mac的“Spotlight”电脑搜索功能。 搜索`Terminal`并按 <kbd>Return</kbd>。

有关介绍，请参阅[如何在 Mac 上使用终端][mac-terminal]。

### Linux上的Shell

在大多数Linux版本上，可以通过运行 [(Gnome) Terminal](https://help.gnome.org/users/gnome-terminal/stable/) 或 [(KDE) Konsole](https ://konsole.kde.org/) 或 [xterm](https://en.wikipedia.org/wiki/Xterm)，可以通过应用程序菜单或搜索栏找到。

### 特别案例

如果以上选项都不能解决您的情况，请尝试在线搜索：`Linux shell [您的操作系统]`。

## 用于安全连接的SSH

所有学生都应安装SSH客户端。SSH是一种工具，它允许我们连接到远程计算机并将其用作我们自己的计算机。

### Windows SSH

Git for Windows 预装了SSH：您无需执行任何操作。

> ## 对 Windows 的 GUI 支持
>
> 如果您知道您将在集群上运行的软件需要图形用户界面（需要打开一个 GUI 窗口才能使应用程序正常运行），请安装 [MobaXterm](https://mobaxterm.mobatek.net) 主页版。
{: .discussion}

### 适用于 macOS 的 SSH

macOS 预装了 SSH：您无需执行任何操作。

> ## 对 macOS 的 GUI 支持
>
> 如果您知道您将运行的软件需要图形用户界面，请安装 [XQuartz](https://www.xquartz.org)。
{: .discussion}

### Linux SSH

Linux预装了SSH和X窗口支持：您无需执行任何操作。

<!-- links -->
[git4win]: https://gitforwindows.org/
[mac-terminal]: https://www.macworld.co.uk/feature/mac-software/how-use-terminal-on-mac-3608274/
[ms-wsl]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
[ms-shell]: https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-7
[mobax-gen]: https://mobaxterm.mobatek.net/documentation.html
[Linux-emulator]: https://faculty.smu.edu/reynolds/Linuxtut/windows.html
[wsl]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
