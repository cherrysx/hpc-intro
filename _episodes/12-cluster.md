---
title: "在远程HPC系统上作业"
teaching: 25
exercises: 10
questions:
- "什么是HPC系统？"
- "HPC 系统如何作业？"
- "如何登录远程HPC系统？"
objectives:
- "连接到远程HPC系统。"
- "了解一般HPC系统架构。"
keypoints:
- "HPC系统是一组联网的机器。"
- "HPC系统通常提供登录节点和一组作业节点。"
- "在独立（作业）节点上找到的资源在数量和类型（RAM 量、处理器架构、网络挂载文件系统的可用性等）上可能有所不同。"
- "保存在一个节点上的文件在所有节点上都可用。"
---

## 什么是HPC系统？

“云”、“集群”和“高性能计算”或“HPC”在不同的上下文中被大量使用，并具有各种相关的含义。那么它们是什么意思呢？更重要的是，我们如何在作业中使用它们？

*cloud*是一个通用术语，通常用于指代以下计算资源：a) *按需或按需提供给用户，b) 代表可能位于地球上任何地方的真实或*虚拟*资源。例如，在巴西、津巴布韦和日本拥有计算资源的大公司可以将这些资源作为自己的*内部*云来管理，同一家公司也可以使用阿里、亚马逊或微软提供的商业云资源。云资源可能是指执行相对简单任务的机器，例如服务网站、提供共享存储、提供网络服务（例如电子邮件或社交媒体平台），以及更传统的计算密集型任务，例如运行计算仿真模拟。

另一方面，*HPC系统*一词描述了一组用于计算密集型作业负载的独立资源。它们通常由大量CPU和存储组成，旨在处理大量数据和/或大量浮点运算（[FLOPS](https://en.wikipedia.org/wiki/FLOPS)) 且具有尽可能高的性能。例如，[Top-500](https://www.top500.org) 列表中的所有机器都是HPC系统。为了支持这些限制条件，HPC资源必须存在于特定的固定位置：网络电缆只能延伸这么远，而电和光信号的传输速度也只能这么快。

“集群”一词通常用于中小型HPC资源，其影响力不如[Top-500] (https://www.top500.org)。计算中心通常维护和支持多个此类的集群系统，所有系统都共享公共网络和存储以支持常见的计算密集型任务。

## 安全连接

使用集群的第一步是建立从我们的笔记本电脑到集群的连接。当我们坐在电脑前（或站着，或将其握在手中或手腕上）时，我们已经开始期待带有图标、小部件，也许还有一些窗口或应用程序的视觉显示：图形用户界面或GUI。由于计算机集群是我们连接到的远程资源，通常通过缓慢或滞后的接口（尤其是 WiFi和VPN），使用命令行接口或CLI更为实用，其中命令和结果仅通过文本传输。除文本（例如图像）之外的任何内容都必须写入磁盘并使用单独的程序打开。

如果您曾经打开过 Windows命令提示符或macOS终端，那么您就会看到一个CLI。如果您已经学习了[Linux Shell](https://www.yuanmadesign.com/shell1/)或版本控制课程，那么您已经在本地机器上熟练使用了CLI。这里唯一的不同是在*远程*机器上打开 CLI，同时采取一些预防措施，使网络上的其他人无法看到（或更改）您正在运行的命令或远程机器的结果。我们将使用Secure Shell协议（或SSH）在两台机器之间打开一个加密的网络连接，让您可以发送和接收文本和数据，而不必担心被窥探。

{% include figure.html url="" max-width="50%"
   file="/hpc1/fig/connect-to-remote.svg"
   alt="Connect to cluster" caption="" %}

SSH客户端通常是命令行工具，您可以在其中提供远程机器地址作为唯一必需的参数。 如果您在远程系统上的用户名与您在本地使用的用户名不同，您也必须提供该用户名。如果您的SSH客户端具有图形前端，例如PuTTY或MobaXterm，您将在单击“连接”之前设置这些参数。在终端中，您将编写类似 `ssh userName@hostname` 的内容，其中参数就像一个电子邮件地址：“@”符号用于将个人ID与共享资源的地址分开。

登录笔记本电脑、平板电脑或其他个人设备时，通常需要用户名、密码或图案来防止未经授权的访问。在这些情况下，其他人拦截您的密码的可能性很低，因为记录您的键盘输入需要恶意利用或物理访问。对于像 {{ site.remote.host }}这样运行SSH服务器的系统，网络上的任何人都可以登录或尝试登录。由于用户名通常是公开的或易于猜测，因此您的密码通常是安全链中最薄弱的环节。因此，许多集群禁止基于密码的登录，而是要求您生成并配置具有更强密码的公私钥对。即使您的集群不需要它，下一节将指导您使用SSH密钥和SSH代理来增强您的安全性*并*使登录远程系统更加方便。

### 使用SSH密钥提高安全性

[课程设置]({{ page.root }}/setup) 提供了使用SSH安装shell应用程序的说明。如果您还没有这样做，请使用类似于Linux的命令行界面打开该shell应用程序到您的系统。

SSH密钥是获得远程计算系统访问权限的另一种身份验证方法。它们还可用于传输文件时的身份验证或访问版本控制系统。在本节中，您将创建一对SSH密钥：

* 您保存在自己计算机上的私钥，以及
* 可以放置在您将访问的任何远程系统上的公钥。

> ## 私钥是您的安全数字护照
>
> 任何人都可以看到的私钥，那么您的私钥应该被视为已泄露，必须销毁。泄露的原因包括对存储它（或副本）的目录拥有不适当的权限，遍历任何不安全（加密）的网络，未加密电子邮件的附件，甚至在终端窗口上显示密钥。
>
> 保护这把钥匙，否则私有数据将可能泄露给恶意的人。
{: .caution}

#### Linux、Mac、MobaXterm 和适用于 Linux 的 Windows 子系统上的 SSH 密钥

打开终端后，请检查现有的SSH密钥和文件名，因为现有的SSH密钥已被覆盖。

```
{{ site.local.prompt }} ls ~/.ssh/
```
{: .language-bash}

如果`~/.ssh/id_rsa`已经存在，您需要为新的密钥对指定一个不同的名称。

使用以下命令生成一个新的公钥-私钥对，这将通过调用下面这些参数生成比`ssh-keygen`默认值更强的密钥：

* `-a` (默认值是16): 密码推导的轮数；增加以减缓蛮力攻击。
* `-t` (默认值是rsa): 指定“类型”或加密算法。`ed25519` 使用256位密钥指定EdDSA；它比RSA更安全更快。
* `-f`（默认值是/home/user/.ssh/id_algorithm）：用于存储您的私钥的文件名。公钥文件名和私钥相同，并添加了“.pub”扩展名。

```
{{ site.local.prompt }} ssh-keygen -a 100 -f ~/.ssh/id_ed25519 -t ed25519
```
{: .language-bash}

出现提示时，输入强密码并记住。有两种常见的方法：

1. 使用一些标点符号和字母数字替换（32个字符或更长）创建一个令人难忘的密码短语。街道地址是好的选择；小心社会工程或公共记录攻击。
2. 使用密码管理器及其内置密码生成器，包含所有字符类别（25个字符或更长）。KeePass和BitWarden是两个不错的选择。

请注意，在您输入密码时，终端似乎不会改变，密码没有显示出来：这是为了您的安全考虑的。系统会提示您再次输入，所以不要太担心拼写错误。

查看`~/.ssh`（使用`ls ~/.ssh`）。您应该看到两个新文件：

* 你的私钥（`~/.ssh/id_ed25519`）：*不要与任何人分享！*
* 可共享的公钥（`~/.ssh/id_ed25519.pub`）：如果系统管理员要求提供密钥，发送公钥。上传到GitHub等网站也是安全的。

> ## 禁止空密码
>
> 没有什么比没有密码的私钥更*不安全*了。如果您不小心跳过了密码输入，请返回并生成带有强密码的新密钥对。
{: .warning}

#### 将RSA用于旧系统

如果由于ed25519不可用而导致密钥生成失败，请尝试使用较旧的（但仍然强大且值得信赖）的RSA密码系统。同样，首先检查现有密钥：

```
{{ site.local.prompt }} ls ~/.ssh/
```
{: .language-bash}

如果 `~/.ssh/id_rsa` 已经存在，您需要为新的密钥对指定一个不同的名称。如上所述生成它，并带有以下额外参数：

* `-b` 设置密钥中的位数。默认值为2048。EdDSA使用固定的密钥长度，因此该标志无效。
* `-o`（无默认值）：使用OpenSSH密钥格式，而不是PEM。

```
{{ site.local.prompt }} ssh-keygen -a 100 -b 4096 -f ~/.ssh/id_rsa -o -t rsa
```
{: .language-bash}

出现提示时，输入您能记住的强密码。有两种常见的方法：

1. 使用一些标点符号和字母数字替换（32个字符或更长）创建一个令人难忘的密码短语。街道地址是好的选择；小心社会工程或公共记录攻击。
2. 使用密码管理器及其内置密码生成器，包含所有字符类别（25个字符或更长）。KeePass和BitWarden是两个不错的选择。

查看`~/.ssh`（使用`ls ~/.ssh`）。您应该看到两个新文件：

* 你的私钥（`~/.ssh/id_rsa`）：*不要与任何人分享！*
* 可共享的公钥（`~/.ssh/id_rsa.pub`）：如果系统管理员要求提供密钥，这就是要发送的密钥。上传到 GitHub 等网站也是安全的。

#### PuTTY上的SSH密钥

如果您在Windows上使用 PuTTY，请下载并使用“puttygen”生成密钥对。有关详细信息，请参阅 [PuTTY 文档][putty-gen]。

* 选择“EdDSA”作为密钥类型。
* 选择“255”作为密钥大小或强度。
* 点击“生成”按钮。
* 您无需输入评论。
* 出现提示时，输入您会记住的强密码。 有两种常见的方法：

1. 使用一些标点符号和字母数字替换（32个字符或更长）创建一个令人难忘的密码短语。街道地址是好的选择；小心社会工程或公共记录攻击。
2. 使用密码管理器及其内置密码生成器，包含所有字符类别（25个字符或更长）。KeePass和BitWarden是两个不错的选择。

* 将密钥保存在系统的其他用户无法读取的文件夹中。

查看您指定的文件夹。 您应该看到两个新文件：

* 你的私钥（`~/.ssh/id_ed25519`）：*不要与任何人分享！*
* 可共享的公钥（`~/.ssh/id_ed25519.pub`）：如果系统管理员要求提供密钥，发送公钥。上传到GitHub等网站也是安全的。

### 用于更轻松地处理密钥的SSH代理

SSH密钥的强度仅与用于解锁它的密码一样强，但在另一方面，每次连接到机器时都要输入复杂的密码很乏味而且很快更改。这是[SSH 代理][ssh-agent]
为何出现的原因。

使用 SSH 代理，您可以输入您的私钥密码一次，然后让代理记住几个小时或直到您注销。除非一些邪恶的人可以物理访问您的机器，否则这可以保持
密码安全，免去多次输入密码的繁琐。

只需记住您的密码，因为一旦在代理中过期，您必须再次输入。

#### Linux、macOS 和 Windows 上的 SSH 代理

打开您的终端应用程序并检查代理是否正在运行：

```
{{ site.local.prompt }} ssh-add -l
```
{: .language-bash}

* 如果你遇到这样的错误:

  ```
  Error connecting to agent: No such file or directory
  ```
  {: .error}

  ... 那么您需要启动代理*作为后台进程*。

  ```
  {{ site.local.prompt }} eval $(ssh-agent)
  ```
  {: .language-bash}

* 否则，您的代理已经在运行：不要乱用它。

将您的密钥添加到代理，会话在 8 小时后到期：

```
{{ site.local.prompt }} ssh-add -t 8h ~/.ssh/id_ed25519
```
{: .language-bash}
```
Enter passphrase for .ssh/id_ed25519: 
Identity added: .ssh/id_ed25519
Lifetime set to 86400 seconds
```
{: .output}

在持续时间（8 小时）内，无论您何时使用该密钥，SSH代理都会代表您提供该密钥，而无需您敲击一次按键。

#### PuTTY上的SSH代理

如果您在Windows上使用PuTTY，请下载并使用`pageant`作为SSH代理。请参阅 [PuTTY 文档][putty-agent]。

### 传输您的公钥

{% if site.remote.portal %}
访问{{ site.remote.portal }}上传您的SSH公钥。
{% else %}
使用 **s**ecure **c**o**p**y 工具将您的公钥发送到集群。

```
{{ site.local.prompt }} scp ~/.ssh/id_ed25519.pub {{ site.remote.user }}@{{ site.remote.login }}:~/
```
{: .language-bash}
{% endif %}

## 登录集群

继续并打开您的终端或图形SSH客户端，然后登录到集群。将`{{ site.remote.user }}`替换为您的用户名。

```
{{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
```
{: .language-bash}

可能会要求您输入密码。注意：您在密码提示后键入的字符不会显示在屏幕上。一旦您按下“Enter”，将恢复正常输出。

您可能已经注意到，当您使用终端登录到远程系统时，提示符发生了变化（如果您使用PuTTY登录，这将不适用，因为它不提供本地终端）。此更改很重要，因为它可以帮助您区分当您将键入的命令传递到终端时将在哪个系统上运行它们。这种变化也是一个小问题，我们需要在整个学习中进行目录切换。当终端连接到本地系统和远程系统时，在终端中的“$”之前对于每个用户来说通常是不同的。我们仍然需要指出我们在哪个系统上输入命令，所以我们将采用以下约定：

* 当连接到本地计算机的终端上输入命令时使用`{{ site.local.prompt }}`
* 当要连接到远程系统的终端上输入命令时使用`{{ site.remote.prompt }}`
* 当终端连接到哪个系统并不重要时使用`$`

## 远程家目录

很多时候，许多用户倾向于将高性能计算系统视为一台巨大的、神奇的机器。有时，人们会假设他们登录的计算机是整个计算集群。那么到底发生了什么？我们登录了哪台电脑？我们登录的当前计算机的名称可以使用 `hostname` 命令检查。（您可能还注意到当前主机名也是我们提示的一部分！）

```
{{ site.remote.prompt }} hostname
```
{: .language-bash}

```
{{ site.remote.host }}
```
{: .output}

所以，我们肯定在远程机器上。接下来，让我们通过运行 `pwd` 来 **p**rint **w**orking **d**iectory找出我们在哪里。

```
{{ site.remote.prompt }} pwd
```
{: .language-bash}

```
{{ site.remote.homedir }}/{{ site.remote.user }}
```
{: .output}

太好了，我们知道我们在哪里！让我们看看我们当前目录中有什么：

```
{{ site.remote.prompt }} ls
```
{: .language-bash}

系统管理员可能已经为您的主目录配置了一些有用的文件、文件夹和链接（快捷方式），这些链接（快捷方式）指向其他文件系统上为您保留的空间。如果没有，您的主目录可能会显示为空。要仔细检查，请在目录列表中包含隐藏文件：

```
{{ site.remote.prompt }} ls -a
```
{: .language-bash}
```
  .            .bashrc           id_ed25519.pub
  ..           .ssh
```
{: .output}

在第一列中，`.`是对当前目录的引用，`..`是对其父目录（`{{ site.remote.homedir }}`）的引用。您可能会或可能不会看到其他文件或类似的文件：`.bashrc`是一个bash shell配置文件，您可以根据自己的喜好对其进行编辑；`.ssh`是一个存储SSH密钥和授权连接记录的目录。

### 安装你的 SSH 密钥

如果您使用`scp`传输了SSH公钥，您应该会在主目录中看到`id_ed25519.pub`。要“安装”此密钥，它必须列在`.ssh`文件夹下名为`authorized_keys`的文件中。

如果上面没有列出`.ssh`文件夹，那么它还不存在：创建它。

```
{{ site.remote.prompt }} mkdir ~/.ssh
```
{: .language-bash}

现在，使用`cat`打印您的公钥，但重定向输出，将其附加到`authorized_keys`文件中：

```
{{ site.remote.prompt }} cat ~/id_ed25519.pub >> ~/.ssh/authorized_keys
```
{: .language-bash}

就这样！断开连接，然后尝试重新登录远程：如果您的密钥和代理已正确配置，则不应提示您输入密码。

```
{{ site.remote.prompt }} logout
```
{: .language-bash}

```
{{ site.local.prompt }}  ssh {{ site.remote.user }}@{{ site.remote.login }}
```
{: .language-bash}


```
{{ site.remote.prompt }} ls
```
{: .language-bash}

> ## 你的机器和远程机器有什么不同？
>
> 在本地计算机上打开第二个终端窗口并运行`ls`命令（无需登录到 {{ site.remote.name }}）。你看到什么不同？
>
> > ## 解决方案
> >
> > 您可能会看到更多类似这样的内容：
> >
> > ```
> > {{ site.local.prompt }} ls
> > ```
> > {: .language-bash}
> > ```
> > Applications Documents    Library      Music        Public
> > Desktop      Downloads    Movies       Pictures
> > ```
> > {: .output}
> >
> > 远程计算机的主目录与本地计算机几乎没有任何共同之处：它们是完全独立的系统！
> {: .solution}
{: .discussion}

## 系统的其余部分

大多数高性能计算系统运行Linux操作系统，该操作系统是围绕Linux[文件系统层次结构标准][fshs]构建的。所有文件和设备都挂载到“根”目录，即`/`，而不是为每个硬盘驱动器或存储介质设置一个单独的根目录：

```
{{ site.remote.prompt }} ls /
```
{: .language-bash}
```
bin   etc   lib64  proc  sbin     sys  var
boot  {{ site.remote.homedir | replace: "/", "" }}  mnt    root  scratch  tmp  working
dev   lib   opt    run   srv      usr
```
{: .output}

"{{ site.remote.homedir | replace: "/", "" }}"目录是我们通常希望保留所有文件的目录。Linux操作系统上的其他文件夹包含系统文件，并在您安装新软件或升级操作系统时进行修改和更新。

> ## 使用HPC文件系统
>
> 在HPC系统上，您可以在许多地方存储文件。它们在分配的空间量和是否备份上都不同。
>
> * **Home** -- 通常是*网络文件系统*，此处存储的数据在整个HPC系统中都可用，并且通常会定期备份。存储在此处的文件通常访问速度较慢，数据实际上存储在另一台计算机上，并且正在通过网络传输和提供！
> * **Scratch** -- 通常比网络主目录快，但通常不备份，不应用于长期存储。
> * **Work** -- 有时作为临时空间的替代品，Work是一种通过网络访问的快速文件系统。通常，这将比您的主目录具有更高的性能，但低于Scratch的性能； 它可能没有备份。它与暂存空间的不同之处在于，作业文件系统中的文件不会自动为您删除：您必须自己管理空间。
{: .callout}

## 节点

组成集群的单个计算机通常称为*nodes*（尽管您也会听到人们称它们为*servers*、*computers* 和 *machines*）。在集群上，不同类型的任务有不同类型的节点。您现在所在的节点称为 *head node*、*login node*、*landing pad* 或 *submit node*。登录节点用作集群的访问点。

作为网关，它非常适合上传和下载文件、设置软件和运行快速测试。一般来说，登录节点不应该用于耗时或资源密集型的任务。您应该对此保持警惕，并与您网站的运营商或文档联系，了解允许和不允许的详细信息。在这些实验中，我们将避免在头节点上运行作业。

> ## 专用传输节点
>
> 如果您想将大量数据传入或传出集群，某些系统会提供仅用于数据传输的专用节点。这样做的动机在于较大的数据传输不应妨碍其他任何人的登录节点的操作。如果此类传输节点可用，请咨询您的集群文档或其支持团队。根据经验，将大于500MB到1GB的数据的所有传输都视为大。但是这些数字会发生变化，例如，取决于您自己和集群的网络连接或其他因素。
{: .callout}

集群上的实际作业由*worker*（或 *compute*）*nodes*完成。作业节点有多种形状和大小，但通常专用于需要大量计算资源的长任务或艰巨任务。

与作业节点的所有交互都由称为调度器的专用软件处理（本课中使用的调度器称为 {{ site.sched.name }}）。我们将详细了解如何使用调度器接下来提交作业，它还可以告诉我们有关作业节点的更多信息。

例如，我们可以通过运行命令“{{ site.sched.info }}”查看所有作业节点。

```
{{ site.remote.prompt }} {{ site.sched.info }}
```
{: .language-bash}

{% include {{ site.snippets }}/cluster/queue-info.snip %}

还有用于管理磁盘存储、用户身份验证和其他与基础设施相关的任务的专用机器。尽管我们通常不会直接登录到这些机器或与这些机器进行交互，但它们启用了许多关键功能，例如确保我们的用户帐户和文件在整个HPC系统中可用。

## 节点中有什么？

HPC系统中的所有节点都具有与您自己的笔记本电脑或台式机相同的组件：*CPU*（有时也称为*处理器*或*内核*）、*内存*（或*RAM*）和*磁盘*空间。CPU是用于实际运行程序和计算的计算机工具。有关当前任务的信息存储在计算机的内存中。磁盘是指可以像文件系统一样访问的所有存储。这通常是可以永久保存数据的存储，即即使计算机重新启动，数据仍然存在。虽然此存储可以是本地的（安装在其中的硬盘驱动器），但更常见的是节点连接到共享的远程文件服务器或服务器集群。

{% include figure.html url="" max-width="40%"
   file="/hpc1/fig/node_anatomy.png"
   alt="Node anatomy" caption="" %}

> ## 探索您的计算机
>
> 尝试找出您个人计算机上可用的CPU数量和内存量。
>
> 请注意，如果您已登录到远程计算机集群，则需要先注销。为此，请键入 `Ctrl+d` 或 `exit`：
>
> ```
> {{ site.remote.prompt }} exit
> {{ site.local.prompt }}
> ```
> {: .language-bash}
>
> > ## 解决方案
> >
> > 有几种方法可以做到这一点。大多数操作系统都有图形系统监视器，如Windows任务管理器。更详细的信息可以在命令行中找到：
> >
> > * 运行系统实用程序
> >   ```
> >   {{ site.local.prompt }} nproc --all
> >   {{ site.local.prompt }} free -m
> >   ```
> >   {: .language-bash}
> >
> > * 从`/proc`读取
> >   ```
> >   {{ site.local.prompt }} cat /proc/cpuinfo
> >   {{ site.local.prompt }} cat /proc/meminfo
> >   ```
> >   {: .language-bash}
> >
> > * 运行系统监视器
> >   ```
> >   {{ site.local.prompt }} htop
> >   ```
> >   {: .language-bash}
> {: .solution}
{: .challenge}

> ## 探索头节点
>
> 现在将您计算机的资源与头节点的资源进行比较。
>
> > ## 解决方案
> >
> > ```
> > {{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
> > {{ site.remote.prompt }} nproc --all
> > {{ site.remote.prompt }} free -m
> > ```
> > {: .language-bash}
> >
> > 您可以使用`lscpu`获得有关处理器的更多信息，并通过阅读文件`/proc/meminfo`获得有关内存的大量详细信息：
> >
> > ```
> > {{ site.remote.prompt }} less /proc/meminfo
> > ```
> > {: .language-bash}
> >
> > 您还可以使用`df`探索可用的文件系统以显示**d**isk **f**ree可用空间。 `-h` 标志以人类友好的格式呈现大小，即GB而不是B。**t**ype标志 `-T` 显示每个资源的文件系统类型。
> >
> > ```
> > {{ site.remote.prompt }} df -Th
> > ```
> > {: .language-bash}
> >
> > > ## `df` 的不同结果
> > >
> > > * 本地文件系统（ext、tmp、xfs、zfs）将取决于您是否在同一个登录节点（或计算节点，稍后）。
> > > * 网络文件系统（beegfs、lustre、cifs、gpfs、nfs、pvfs）将是相似的——但可能包括 {{ site.remote.user }}，这取决于它是如何[挂载]（https://en.wikipedia.org/ wiki/Mount_（计算））。
> > {: .discussion}
> >
> > > ## 共享文件系统
> > >
> > > 记住这一点很重要：保存在一个节点（计算机）上的文件通常在集群的任何地方都可用！
> > {: .callout}
> {: .solution}
{: .challenge}

{% include {{ site.snippets }}/cluster/specific-node-info.snip %}

> ## 比较您的计算机、头节点和作业节点
>
> 将您的笔记本电脑的处理器和内存数量与您在集群头节点和作业节点上看到的数字进行比较 您认为这些差异可能会对在不同系统和节点上运行您的研究作业产生什么影响？
>
> > ## 解决方案
> >
> > 计算节点通常使用比头节点或个人计算机具有*更高核心数*的处理器构建，以支持高度并行的任务。计算节点通常还安装了比个人计算机更多的内存（RAM）*。更多的内核往往有助于依赖于一些易于在*并行*中执行的作业的作业，而更多、更快的内存是大型或*复杂数字任务*的关键。
> {: .solution}
{: .discussion}

> ## 节点之间的差异
>
> 许多HPC集群具有针对特定作业负载优化的各种节点。某些节点可能具有更大的内存量或专用资源，例如图形处理单元（GPU）。
{: .callout}

考虑到所有这些，我们现在将介绍如何与集群的调度器通信，并使用它来开始运行我们的脚本和程序！

{% include links.md %}

[fshs]: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard
[putty-gen]: https://tartarus.org/~simon/putty-prerel-snapshots/htmldoc/Chapter8.html#pubkey-puttygen
[putty-agent]: https://tartarus.org/~simon/putty-prerel-snapshots/htmldoc/Chapter9.html#pageant
[ssh-agent]: https://www.ssh.com/academy/ssh/agent
[ssh-flags]: https://stribika.github.io/2015/01/04/secure-secure-shell.html
[wiki-rsa]: https://en.wikipedia.org/wiki/RSA_(cryptosystem)
[wiki-dsa]: https://en.wikipedia.org/wiki/EdDSA
