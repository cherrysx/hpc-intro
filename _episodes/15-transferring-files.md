---
title: "与远程计算机传输文件"
teaching: 15
exercises: 15
questions:
- "如何将文件传输到（和从）集群？"
objectives:
- "能够将文件传入和传出计算集群。"
keypoints:
- "`wget` 和 `curl -O` 从Internet下载文件。"
- "`scp`将文件传输到您的计算机或从您的计算机传输文件。"
- "您可以使用 FileZilla 等SFTP客户端通过GUI传输文件。"
---

如果我们无法将文件传入或传出集群，则使用远程计算机进行计算的用途非常有限。在计算资源之间传输数据有多种选择，从命令行选项到GUI程序，我们将在这里介绍。

## 从Internet下载文件

下载文件最直接的方法之一是使用“curl”或“wget”，其中一种通常安装在大多数Linux shell、Mac OS 终端和Git Bash中。任何可以通过直接链接在Web浏览器中下载的文件都可以使用`curl -O`或`wget`下载。这是下载数据集或源代码的快速方法。

这些命令的语法是：`curl -O https://some/link/to/a/file` 和 `wget https://some/link/to/a/file`。通过从本地计算机上的终端下载，我们稍后将使用的一些材料来尝试一下。

```
{{ site.local.prompt }} curl -O {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
```
{: .language-bash}
or
```
{{ site.local.prompt }} wget {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
```
{: .language-bash}


> ## `tar.gz`?
>
> 这是一种存档文件格式，就像`.zip`，在Linux上常用并默认支持，Linux是大多数HPC集群机器运行的操作系统。您可能还会看到扩展名 `.tgz`，它完全相同。稍后，我们将更多地讨论“tar”，因为“tar-dot-g-z”是拗口的，
{: .discussion}

## 使用`scp`传输单个文件和文件夹

要将单个文件复制到集群或从集群下载，我们可以使用`scp`（“安全复制”）。对于新用户来说，语法可能有点复杂，但我们将对其进行分解。

要*上传到*另一台计算机：

```
{{ site.local.prompt }} scp path/to/local/file.txt {{ site.remote.user }}@{{ site.remote.login }}:/path/on/{{ site.remote.name }}
```
{: .language-bash}

要*从*另一台计算机下载：

```
{{ site.local.prompt }} scp {{ site.remote.user }}@{{ site.remote.login }}:/path/on/{{ site.remote.name }}/file.txt path/to/local/
```
{: .language-bash}

请注意，`:` 之后的所有内容都相对于远程计算机上的主目录。如果我们不关心文件的去向，我们可以将其保留。

```
{{ site.local.prompt }} scp local-file.txt {{ site.remote.user }}@{{ site.remote.login }}:
```
{: .language-bash}

> ## 上传一个文件
>
> 将您刚刚从Internet下载的文件复制到{{ site.remote.name }}上的主目录。
>
> > ## 解决方案
> >
> > ```
> > {{ site.local.prompt }} scp hpc-intro-data.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:~/
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

> ## 为什么不直接在{{ site.remote.name }}上下载？
>
> 一些计算机集群位于防火墙后面，设置为只允许从*外部*发起的传输。这意味着“curl”命令将失败，因为防火墙外部的地址无法从内部访问。要解决这个问题，请从本地计算机运行`curl`或`wget`命令下载互联网文件，然后使用`scp`命令（就在此处下方）将其上传到集群。
>
> > ## `curl -O` from {{ site.remote.login }}
> > or
> > ## `wget` from {{ site.remote.login }}
> > 
> > 尝试直接下载文件。请注意，它很可能会失败，这没关系！
> >
> > > ## 命令
> > >
> > > ```
> > > {{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
> > > {{ site.remote.prompt }} curl -O {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
> > > or
> > > {{ site.remote.prompt }} wget {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
> > > ```
> > > {: .language-bash}
> > {: .solution}
> >
> > 它奏效了吗？如果不是，终端输出告诉你发生了什么？
> {: .challenge}
{: .discussion}

要复制整个目录，我们添加 `-r` 命令行选项，用于“**r**ecursive”：复制指定的项目，以及它下面的每个项目，以及它们下面的每个项目......直到它到达底部 以您提供的文件夹名称为根的目录树。

```
{{ site.local.prompt }} scp -r some-local-folder {{ site.remote.user }}@{{ site.remote.login }}:target-directory/
```
{: .language-bash}

> ## 警告
>
> 对于一个大目录——无论是大小还是文件数量——使用 `-r` 复制可能需要很长时间才能完成。
{: .callout}

## `/` 中有什么？

使用`scp`时，您可能已经注意到`:` *always*跟在远程计算机名称后面； 有时后面是`/`，有时不是，有时是最后的`/`。在 Linux 计算机上，`/` 是 ***root*** 目录，是整个文件系统（以及附加到它的其他文件系统）的目录位置。 以 `/` 开头的路径称为*absolute*，因为在根`/`之上不能有任何东西。不以`/`开头的路径称为 *relative*，因为它没有根目录前缀。

如果您想将文件上传到您的主目录中的某个位置（通常是这种情况），那么您不需要`/`。在 `:` 之后，开始写入指向文件最终存储位置的文件夹序列，或者，如上所述，如果您的主目录*是*目标，则不提供任何内容。

目标目录的尾部斜杠是可选的，对`scp -r`没有影响，但在其他命令中很重要，例如`rsync`。

> ## 关于`rsync`的注意事项
>
> 随着您获得传输文件的经验，您可能会发现`scp`命令的限制。 [rsync](https://rsync.samba.org/) 实用程序为文件传输提供了高级功能，并且与 `scp`和`sftp`相比通常更快（见下文）。它对于传输大型和/或许多文件以及创建同步备份文件夹特别有用。
>
> 语法类似于`scp`。使用常用选项传输*到*另一台计算机：
>
> ```
> {{ site.local.prompt }} rsync -avzP path/to/local/file.txt {{ site.remote.user }}@{{ site.remote.login }}:directory/path/on/{{ site.remote.name }}/
> ```
> {: .language-bash}
>
> `a`（存档）选项保留文件时间戳和权限等；`v`（详细）选项提供详细输出以帮助监控传输；`z`（压缩）选项在传输过程中压缩文件以减小大小和传输时间； 并且`P`（部分/进度）选项在中断的情况下保留部分传输的文件，并显示传输进度。
>
> 要递归复制目录，我们可以使用相同的选项：
>
> ```
> {{ site.local.prompt }} rsync -avzP path/to/local/dir {{ site.remote.user }}@{{ site.remote.login }}:directory/path/on/{{ site.remote.name }}/
> ```
> {: .language-bash}
>
> 如所写，这会将本地目录及其内容放在远程系统上的指定目录下。如果目标上的尾部斜杠被省略，则不会创建与传输目录对应的新目录（示例中为“dir”），源目录的内容将直接复制到目标目录中。
>
> `a`（归档）选项意味着递归。
>
> 要下载文件，我们只需更改源和目标：
>
> ```
> {{ site.local.prompt }} rsync -avzP {{ site.remote.user }}@{{ site.remote.login }}:path/on/{{ site.remote.name }}/file.txt path/to/local/
> ```
> {: .language-bash}
{: .callout}

> ## 关于端口的说明
>
> 使用上述方法的所有文件传输都使用SSH对通过网络发送的数据进行加密。因此，如果您可以通过SSH连接，您将能够传输文件。默认情况下，SSH使用网络端口22。如果正在使用自定义SSH端口，则必须使用适当的命令行选项来指定它，通常是`-p`、`-P`或`--port`。如果不确定，请检查 `--help` 或 `man` 页面。
>
> > ## Rsync端口
> >
> > 假设我们必须通过端口768而不是22连接“rsync”。我们将如何修改此命令？
> >
> > ```
> > {{ site.local.prompt }} rsync test.txt {{ site.remote.user }}@{{ site.remote.login }}:
> > ```
> > {: .language-bash}
> >
> > > ## 解决方案
> > >
> > > ```
> > > {{ site.local.prompt }} rsync --help | grep port
> > >      --port=PORT             specify double-colon alternate port number
> > > See http://rsync.samba.org/ for updates, bug reports, and answers
> > > {{ site.local.prompt }} rsync --port=768 test.txt {{ site.remote.user }}@{{ site.remote.login }}:
> > > ```
> > > {: .language-bash}
> > {: .solution}
> {: .challenge}
{: .callout}

## 使用 FileZilla 交互式传输文件

FileZilla是一个跨平台客户端，用于从远程计算机下载和上传文件。它的功能和性能良好。它使用 `sftp` 协议。您可以在[此处]({{ site.baseurl }}{% link _extras/discuss.md %})中阅读有关使用`sftp`协议的更多信息。

从[https://filezilla-project.org](https://filezilla-project.org)下载并安装FileZilla客户端。安装并打开程序后，您应该会在屏幕左侧看到一个带有本地系统文件浏览器的窗口。当您连接到集群时，您的集群文件将出现在右侧。

要连接到集群，我们只需要在屏幕顶部输入我们的凭据：

* 机器名/IP: `sftp://{{ site.remote.login }}`
* 用户名: 您的集群用户名
* 密码：您的集群密码
* 端口：（留空以使用默认端口）

点击“快速连接”进行连接。您应该会看到您的远程文件出现在屏幕的右侧。您可以在屏幕的左侧（本地）和右侧（远程）之间拖放文件以传输文件。

最后，如果您需要将大文件（通常大于1GB）从一台远程计算机移动到另一台远程计算机，请通过SSH连接到托管文件的计算机并使用`scp`或`rsync`传输到另一台计算机。这将比使用从源文件复制到本地计算机，然后复制到目标计算机的FileZilla（或相关应用程序）更有效。

## 归档文件

在远程HPC系统之间传输数据时，我们经常面临的最大挑战之一是大量文件。传输每个单独的文件都会产生开销，当我们传输大量文件时，这些开销会在很大程度上减慢我们的传输速度。

这个问题的解决方案是在我们传输数据之前将多个文件*归档*成较小数量的较大文件，以提高我们的传输效率。有时我们会将归档与*压缩*结合起来，以减少我们必须传输的数据量，从而加快传输速度。

您将在(Linux)HPC集群上使用的最常见的归档命令是`tar`。`tar`可用于将文件组合成单个存档文件，并且可以选择压缩它。

让我们从从本课的站点下载的文件“hpc-lesson-data.tar.gz”开始。“gz”部分代表*gzip*，它是一个压缩库。读取这个文件名，似乎有人拿了一个名为“hpc-lesson-data”的文件夹，用“tar”将其所有内容打包在一个文件中，然后用“gzip”压缩该存档以节省空间。让我们检查使用带有`-t`命令行选项的 `tar`，它在远程计算机上打印“**t**able of contents”而不解包由`-f <filename>`指定的文件。请注意，您可以连接两个命令行选项，而不是单独编写 `-t -f`。

```
{{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
{{ site.remote.prompt }} tar -tf hpc-lesson-data.tar.gz
hpc-intro-data/
hpc-intro-data/north-pacific-gyre/
hpc-intro-data/north-pacific-gyre/NENE01971Z.txt
hpc-intro-data/north-pacific-gyre/goostats
hpc-intro-data/north-pacific-gyre/goodiff
hpc-intro-data/north-pacific-gyre/NENE02040B.txt
hpc-intro-data/north-pacific-gyre/NENE01978B.txt
hpc-intro-data/north-pacific-gyre/NENE02043B.txt
hpc-intro-data/north-pacific-gyre/NENE02018B.txt
hpc-intro-data/north-pacific-gyre/NENE01843A.txt
hpc-intro-data/north-pacific-gyre/NENE01978A.txt
hpc-intro-data/north-pacific-gyre/NENE01751B.txt
hpc-intro-data/north-pacific-gyre/NENE01736A.txt
hpc-intro-data/north-pacific-gyre/NENE01812A.txt
hpc-intro-data/north-pacific-gyre/NENE02043A.txt
hpc-intro-data/north-pacific-gyre/NENE01729B.txt
hpc-intro-data/north-pacific-gyre/NENE02040A.txt
hpc-intro-data/north-pacific-gyre/NENE01843B.txt
hpc-intro-data/north-pacific-gyre/NENE01751A.txt
hpc-intro-data/north-pacific-gyre/NENE01729A.txt
hpc-intro-data/north-pacific-gyre/NENE02040Z.txt
```
{: .language-bash}

这显示了一个包含另一个文件夹的文件夹，其中包含一堆文件。如果您最近学习过Linux Shell小白入门课，这些内容可能看起来很熟悉。让我们看看压缩情况，使用`du`表示“**d**isk **u**sage”。

```
{{ site.remote.prompt }} du -sh hpc-lesson-data.tar.gz
36K     hpc-intro-data.tar.gz
```
{: .language-bash}

> ## 文件占用至少一个“块”
>
> 如果文件系统块大小大于36KB，您会看到更大的数字：文件不能小于一个块。
{: .callout}

现在让我们解压缩存档。我们将使用一些常见的命令行选项运行`tar`：

* `-x` 到e**x**tract归档
* `-v` 用于**v**erbose输出
* `-z` 用于g**z**ip压缩
* `-f` 用于解压文件

完成后，使用`du`检查目录大小并进行比较。

> ## 提取档案
>
> 使用上面的四个命令行选项，使用`tar`解压课程数据。然后，使用`du`检查整个解压目录的大小。
>
> 提示：`tar`允许你连接这些命令行选项。
>
> > ## 命令
> >
> > ```
> > {{ site.remote.prompt }} tar -xvzf hpc-lesson-data.tar.gz
> > ```
> > {: .language-bash}
> >
> > ```
> > hpc-intro-data/
> > hpc-intro-data/north-pacific-gyre/
> > hpc-intro-data/north-pacific-gyre/NENE01971Z.txt
> > hpc-intro-data/north-pacific-gyre/goostats
> > hpc-intro-data/north-pacific-gyre/goodiff
> > hpc-intro-data/north-pacific-gyre/NENE02040B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01978B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02043B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02018B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01843A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01978A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01751B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01736A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01812A.txt
> > hpc-intro-data/north-pacific-gyre/NENE02043A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01729B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02040A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01843B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01751A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01729A.txt
> > hpc-intro-data/north-pacific-gyre/NENE02040Z.txt
> > ```
> > {: .output}
> >
> > 请注意，由于命令行选项连接，我们没有输入`-x -v -z -f`，尽管该命令的作业方式相同。
> >
> > ```
> > {{ site.remote.prompt }} du -sh hpc-lesson-data
> > 144K    hpc-intro-data
> > ```
> > {: .language-bash}
> {: .solution}
>
> > ## 数据是否被压缩？
> >
> > 文本文件压缩得很好：“tarball”是原始数据总大小的四分之一！
> {: .discussion}
{: .challenge}

如果你想反转这个过程——压缩原始数据而不是提取它——设置一个`c`命令行选项而不是`x`，设置存档文件名，然后提供一个目录来压缩：

```
{{ site.local.prompt }} tar -cvzf compressed_data.tar.gz hpc-intro-data
```
{: .language-bash}

> ## 使用Windows
>
> 当您将文本文件从Windows系统传输到Linux系统（Mac、Linux、BSD、Solaris 等）时，这可能会导致问题。Windows对其文件的编码与Linux略有不同，并在每一行添加一个额外的字符。
>
> 在Linux系统上，文件中的每一行都以 `\n`（换行符）结尾。在 Windows 上，文件中的每一行都以 `\r\n` 结尾（回车 + 换行）。这有时会导致问题。
>
> 尽管大多数现代编程语言和软件都能正确处理此问题，但在极少数情况下，您可能会遇到问题。解决方案是使用`dos2Linux`命令将文件从Windows转换为Linux编码。
>
> 您可以运行命令`cat -A 文件名`识别文件是否为Windows行结尾。带有Windows行尾的文件将在每行的末尾有`^M$`。具有Linux行尾的文件将在行尾有`$`。
>
> 要转换文件，只需运行`dos2Linux filename`。（相反，要转换回 Windows 格式，您可以运行 `Linux2dos filename`。）
{: .callout}

{% include links.md %}
