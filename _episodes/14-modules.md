---
title: "通过Module访问软件"
teaching: 30
exercises: 15
questions:
- "我们如何加载和卸载软件包？"
objectives:
- "了解如何加载和使用软件包。"
keypoints:
- "使用 `module load softwareName` 加载软件。"
- "使用“模块清除”卸载软件"
- "模块系统自动为您处理软件版本控制和包冲突。"
---

在高性能计算系统上，很少有我们想要使用的软件在登录时可用。它已安装，但我们需要“加载”它才能运行。

然而，在我们开始使用单个软件包之前，我们应该了解这种方法背后的原因。最大的三个因素是：

- 软件不兼容
- 版本控制
- 依赖关系

软件不兼容是程序员最头疼的问题。有时，软件包的存在（或不存在）会破坏依赖它的其他软件包。两个最著名的例子是Python 2和3以及C编译器版本。例如，Python 3提供了一个著名的“python”命令，该命令与Python 2提供的命令冲突。基于较新版本的C库编译的软件然后在这些库不存在时使用将导致令人讨厌的“未找到GLIBCXX_3.4.20”错误，

软件版本控制是另一个常见问题。一个团队可能依赖于他们的研究项目的某个包版本——如果软件版本发生变化（例如，如果一个包被更新），它可能会影响他们的结果。访问多个软件版本允许一组研究人员防止软件版本问题影响他们的结果。

依赖关系是特定软件包（甚至特定版本）依赖于访问另一个软件包（甚至是另一个软件包的特定版本）的地方。例如，VASP材料科学软件可能依赖于特定版本的FFTW（西方最快傅立叶变换）软件库可供其作业。

## 环境模块

环境模块是这些问题的解决方案。*模块*是软件包的自包含描述——它包含运行软件包所需的设置，并且通常编码对其他软件包所需的依赖项。

HPC系统上常用的环境模块实现有很多：最常见的两个是*TCL 模块*和*Lmod*。这两个都使用相似的语法并且概念是相同的，因此学习使用一个将允许您使用您正在使用的系统上安装的任何一个。在这两种实现中，`module` 命令都用于与环境模块进行交互。通常会在命令中添加一个额外的子命令来指定您想要执行的操作。对于子命令列表，您可以使用 `module -h` 或 `module help`。 对于所有命令，您可以使用`man module`访问*man*页面上的完整帮助。

登录时，您可以从加载的默认模块集开始，也可以从空环境开始；这取决于您使用的系统的设置。

### 列出可用模块

要查看可用的软件模块，请使用“module avail”：

```
{{ site.remote.prompt }} module avail
```
{: .language-bash}

{% include {{ site.snippets }}/modules/available-modules.snip %}

### 列出当前加载的模块

您可以使用`module list`命令查看当前在您的环境中加载了哪些模块。如果您没有加载任何模块，您将看到一条消息告诉您

```
{{ site.remote.prompt }} module list
```
{: .language-bash}

```
No Modulefiles Currently Loaded.
```
{: .output}

## 加载和卸载软件

要加载软件模块，请使用“模块加载”。在本例中，我们将使用Python 3。

最初，没有加载Python 3。我们可以使用`which` 命令对此进行测试。`which`查找程序的方式与Bash相同，因此我们可以使用它来告诉我们特定软件的存储位置。

```
{{ site.remote.prompt }} which python3
```
{: .language-bash}

{% include {{ site.snippets }}/modules/missing-python.snip %}

我们可以使用`module load`加载`python3`命令：

{% include {{ site.snippets }}/modules/module-load-python.snip %}

{% include {{ site.snippets }}/modules/python-executable-dir.snip %}

那么，刚刚发生了什么？

要了解输出，首先我们需要了解`$PATH`环境变量的性质。`$PATH` 是一个特殊的环境变量，它控制Linux系统查找软件的位置。具体来说，`$PATH`是一个目录列表（由 `:` 分隔），操作系统在放弃并告诉我们找不到命令之前会搜索该目录。与所有环境变量一样，我们可以使用`echo`将其打印出来。

```
{{ site.remote.prompt }} echo $PATH
```
{: .language-bash}

{% include {{ site.snippets }}/modules/python-module-path.snip %}

您会注意到与“which”命令的输出相似。在这种情况下，只有一个区别：开头的目录不同。当我们运行 `module load` 命令时，它会在`$PATH`的开头添加一个目录。让我们检查一下那里有什么：

{% include {{ site.snippets }}/modules/python-ls-dir-command.snip %}

{% include {{ site.snippets }}/modules/python-ls-dir-output.snip %}

综上所述，“module load”会将软件添加到您的“$PATH”中。它“加载”软件。对此特别注意 - 根据您站点上安装的“module”程序的版本，"module load"还将加载所需的软件依赖项。

{% include {{ site.snippets }}/modules/software-dependencies.snip %}

## 软件版本控制

到目前为止，我们已经学习了如何加载和卸载软件包。这非常有用。但是，我们还没有解决软件版本控制的问题。在某些时候，您会遇到某些软件只有一个特定版本适用的问题。也许一个关键的错误修复只发生在某个版本中，或者版本X破坏了与您使用的文件格式的兼容性。在这些示例情况下，非常具体地了解加载的软件是有帮助的。

让我们更仔细地检查`module avail`的输出。

```
{{ site.remote.prompt }} module avail
```
{: .language-bash}

{% include {{ site.snippets }}/modules/available-modules.snip %}

{% include {{ site.snippets }}/modules/wrong-gcc-version.snip %}

> ## 在脚本中使用软件模块
>
> 创建一个能够运行`python3 --version`的作业。请记住，默认情况下不加载任何软件！运行作业就像登录系统一样（您不应假设登录节点上加载的模块已加载到计算节点上）。
>
> > ## 解决方案
> >
> > ```
> > {{ site.remote.prompt }} nano python-module.sh
> > {{ site.remote.prompt }} cat python-module.sh
> > ```
> > {: .language-bash}
> >
> > ```
> > {{ site.remote.bash_shebang }}
> >
> > module load {{ site.remote.module_python3 }}
> >
> > python3 --version
> > ```
> > {: .output}
> >
> > ```
> > {{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}python-module.sh
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

{% include links.md %}
