---
title: "使用调度器"
teaching: 45
exercises: 30
questions:
- "什么是调度器，为什么要使用它们？"
- "如何启动一个程序以在集群中的任何一个节点上运行？"
- "如何捕获在集群中的节点上运行的程序的输出？"
objectives:
- "在集群上运行一个简单的Hello World风格的程序。"
- "向集群提交一个简单的Hello World样式脚本。"
- "使用批处理系统命令行工具来监视作业的执行。"
- "检查作业的输出和错误文件。"
keypoints:
- "调度器处理用户之间如何共享计算资源。"
- "您所做的一切都应该通过调度器运行。"
- "作业只是一个shell脚本。"
- "如果有疑问，请请求比您需要的更多的资源。"
---

## 作业调度器

一个HPC系统可能有数千个节点和数千个用户。我们如何决定谁在什么时候得到什么？我们如何确保任务使用所需的资源运行？这项作业由称为调度器的特殊软件处理。在HPC系统上，调度器管理哪些作业在何时何地运行。

下图将作业调度器的这些任务与餐厅的服务员进行了比较。如果你能想到一个你不得不排队等待一段时间才能进入一家受欢迎的餐厅的例子，那么你现在可能会理解为什么有时你的作业不像在笔记本电脑中那样立即开始。

{% include figure.html max-width="75%" caption=""
   file="/hpc1/fig/restaurant_queue_manager.svg"
   alt="Compare a job scheduler to a waiter in a restaurant" %}

> ## 作业调度角色扮演（可选）
>
> 我们可以将扮演HPC系统中的不同角色，将您分成在集群中扮演不同角色的小组（用户、计算节点和调度器）。在下面的引导下您完成此练习时，请按照指示进行操作。您将模拟作业调度器在集群上的作业方式。
>
> > ## 指导
> >
> > 要进行此练习，您将需要大约50-100张纸或便签。
> >
> > 1. 将房间分成具有特定角色的组。
> >    * 挑三四个人做“调度员”。
> >    * 选择房间的三分之一的人是“用户”，给定几张纸条（或便利贴）和笔。
> >    * 让房间的剩余三分之二成为“计算节点”。
> >    * 让“用户”走到房间的前面（或后面，只要有他们可以站立的地方），“调度员”站在用户和“计算节点”（应该留在他们的座位上）之间。
> >
> > 1. 将纸/便签分给“用户”，让他们用简单的数学问题和他们的名字填写所有便签。告诉大家，这些都是需要完成的作业，对应他们的计算研究问题。
> >
> > 1. 我们现在有作业，我们有可以解决这些问题的“计算节点”（仍然坐着的人）。作业将如何到达节点？ 答案是调度器，它将从用户那里获取作业并将它们交付给开放的计算节点。
> >
> > 1. 让所有“计算节点”举手。让用户通过将作业交给调度器来“提交”他们的作业。然后调度器应将它们交付给“开放”（举手）计算节点并收集完成的问题并将它们返回给适当的用户。
> >
> > 1. 等到大部分问题都解决了，然后重新安排每个人的座位。
> >
> > > ## 讨论
> > >
> > > 由于各种原因，“节点”可能无法解决指定的问题。
> > >
> > > * 没时间了。
> > > * 没内存了。
> > > * 存储空间不足，或无法加载输入文件或数据集。
> > > * 不知道从哪里开始：没有人“教”它，即无法加载程序。
> > > * 卡在一个困难的部分：程序有错误的算法，或者从未被告知加载包含正确算法的库。
> > > * 忙着别的事，还没解决问题。
> > {: .discussion}
> {: .challenge}
{: .callout}

本课中使用的调度器是{{ site.sched.name }}。 尽管{{ site.sched.name }}并非在任何地方都使用，但无论使用什么软件，运行作业都非常相似。确切的语法可能会改变，但概念保持不变。

## 运行批处理作业

调度器最基本的用途是以非交互方式运行命令。您想在集群上运行的任何命令（或一系列命令）都称为*job*，使用调度器运行作业的过程称为*batch job submit*。

在这种情况下，我们要运行的作业是一个shell脚本——本质上是一个文本文件，其中包含要按顺序执行的Linux命令列表。 我们的shell脚本将包含三个部分：

* 在第一行，添加 `{{ site.remote.bash_shebang }}`。`#!`（读作“hash-bang”或“shebang”）告诉计算机用什么程序来处理这个文件的内容。在这种情况下，我们告诉它后面的命令是为命令行shell编写的（到目前为止我们所做的一切）。 如果我们希望我们的脚本与其他东西一起运行——例如Python——我们可以将这一行更改为 `#!/usr/bin/python3`。
* 在第一行下方的任何地方，我们将添加一个带有友好问候语的`echo`命令。 运行时，shell脚本将打印终端中`echo` 之后的任何内容。
* `echo -n`将打印后面的所有内容，*而不*通过打印换行符结束行。
* 在最后一行，我们将调用`hostname` 命令，该命令将打印运行脚本的机器的名称。

```
{{ site.remote.prompt }} nano example-job.sh
```
{: .language-bash}
```
{{ site.remote.bash_shebang }}

echo -n "This script is running on "
hostname
```
{: .output}

让我们尝试运行它：

```
$ example-job.sh 
```
{: .language-bash}

```
bash: example-job.sh: command not found...
```
{: .error}

奇怪的是，Bash找不到我们的脚本。事实证明，Bash只会在某些目录中查找要运行的脚本。要运行其他任何东西，我们需要告诉 Bash确切的位置。要运行我们自己编写的脚本，我们需要明确告诉Bash它在哪里，这样它就不会进行搜索。我们可以使用以下两种方法之一：使用绝对路径 `{{ site.workshop_host_homedir }}/yourUserName/example-job.sh`（等效于 `~/example-job.sh`），或者使用相对路径 `./example-job.sh`（其中“.”代表“当前作业目录”）。

```
$ ./example-job.sh
```
{: .language-bash}

```
bash: ./example-job.sh: Permission denied
```
{: .error}

我们需要做的最后一件事。在文件可以运行之前，它需要“权限”才能执行。让我们使用 `ls` 的“long”格式选项查看此目录中每个文件的权限：

```
$ ls -l
```
{: .language-bash}

```
...
-rw-rw-r-- 1 yourUsername yourGroup  40 Jan 16 19:41 example-job.sh
...
```
{: .output}

让我们看看我们是否可以理解**给定行的每个字段代表的内容**，从左到右。

1. **Permissions:**在最左侧，有10个单字符列，其中包含以下部分或全部符号：
   * `d` 表示某个东西是否是一个目录
   * `r` 表示**r**读取文件或目录的权限
   * `w` 表示对文件或目录的 **w**rite 权限
   * `x` 表示权限 e**x**ecute 文件或目录
   * `-` 用于未授予权限的情况。

   在`d`的位置之后有 `rwx` 权限的三个字段：
  
   * 第一组`rwx`是拥有文件的`u`ser 拥有的权限（在这种情况下，所有者是`yourUsername`）。
   * 第二组`rwx`是所有者的组的其他成员共享的权限（在这种情况下，组名为`yourGroup`）； 短名是`g`。
   * 第三组`rwx`是所有其他用户都可以对文件执行的权限。
   * 尽管通常创建文件时每个人都具有读取权限，但通常您的主目录上的权限首先会阻止其他人访问该文件。
2. **References:** 这会计算对文件、文件夹、符号链接或“快捷方式”的引用（[硬链接]（https://en.wikipedia.org/wiki/Hard_link））的数量。
3. **Owner:** 这是拥有该文件的用户的用户名。他们的权限显示在第一个权限字段中。
4. **Group:** 这是拥有该文件的用户的用户组。此用户组的成员具有第二个权限字段中指示的权限。
5. **Size:** 这是文件中的字节数，或文件夹内容占用的[文件系统块]（https://en.wikipedia.org/wiki/Block_(data_storage)）的数量。（我们可以在这里使用 `-h` 选项来获得人类可读的文件大小，以兆字节、千兆字节等为单位）
6. **Time last modified:** 这是最后一次修改文件的时间。
7. **Filename:** 这是文件名。

更改权限是使用`chmod`完成的。要为我们自己的**username添加可执行权限，我们将输入：

```
$ chmod u+x example-job.sh
$ ls -l example-job.sh
```
{: .language-bash}
```
...
-rwxrw-r-- 1 yourUsername yourGroup  40 Jan 16 19:41 example-job.sh
```
{: .output}


> ## 创建我们的测试作业
>
> 运行脚本。它是在集群上执行还是仅在我们的登录节点上执行？
>
> > ## 解决方案
> >
> > ```
> > {{ site.remote.prompt }} ./example-job.sh
> > ```
> > {: .language-bash}
> > ```
> > This script is running on {{ site.remote.host }}
> > ```
> > {: .output}
> >
> > 此作业在登录节点上运行。
> {: .solution}
{: .challenge}

如果您成功完成了上一个挑战，您可能会意识到通过调度器运行作业与仅“运行”它之间存在区别。要将此作业提交给调度器，我们使用`{{ site.sched.submit.name }}`命令。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/basic-job-script.snip %}

这就是我们提交作业所需要做的一切。我们的作业已经完成——现在调度器接管并尝试为我们运行作业。当作业等待运行时，它会进入一个名为*queue*的作业列表。为了检查我们的作业状态，我们使用命令`{{ site.sched.status }} {{ site.sched.flag.user }}`检查队列。

```
{{ site.remote.prompt }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/basic-job-status.snip %}

检查我们的作业状态的最好方法是使用`{{ site.sched.status }}`。当然，反复运行`{{ site.sched.status }}`来检查事情可能会有点烦人。要查看我们作业的实时视图，我们可以使用`watch`命令。`watch`以2秒的间隔重新运行给定的命令。这太频繁了，可能会让您的系统管理员感到不安。您可以使用`-n 15`参数将时间间隔更改为更合理的值，例如15秒。让我们尝试使用它来监控另一个作业。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
{{ site.remote.prompt }} watch -n 15 {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

您应该会看到作业状态的自动更新显示。完成后，它将从队列中消失。当你想停止`watch`命令时按`Ctrl-c`。

> ## 输出在哪里？
>
> 在登录节点上，这个脚本将输出打印到终端——但是当我们退出`watch`时，什么也没有。它去哪儿了？
>
> 集群作业输出通常重定向到您启动它的目录中的文件。使用`ls`查找和读取文件。
{: .discussion}

## 自定义作业

我们刚刚运行的作业使用了调度器的所有默认选项。在现实世界的场景中，这可能不是我们想要的。默认选项代表一个合理的最小值。很有可能，我们将需要更多的内核、更多的内存、更多的时间，以及其他特殊考虑。要访问这些资源，我们必须自定义我们的作业脚本。

Linux shell脚本中的注释（用`#`表示）通常会被忽略，但也有例外。例如，脚本开头的特殊`#!`注释指定了应该使用哪个程序来运行它（您通常会看到 `{{ site.local.bash_shebang }}`）。像{{ site.sched.name }}这样的调度器也有一个特殊的注释，用于表示特殊的调度器特定选项。尽管这些注释因调度器而异，但{{ site.sched.name }}的特殊注释是`{{ site.sched.comment }}`。`{{ site.sched.comment }}`注释后面的任何内容都被解释为对调度器的指令。

让我们通过例子来说明这一点。默认情况下，作业的名称是脚本的名称，但`{{ site.sched.flag.name }}`选项可用于更改作业的名称。向脚本添加一个选项：

```
{{ site.remote.prompt }} cat example-job.sh
```
{: .language-bash}

```
{{ site.remote.bash_shebang }}
{{ site.sched.comment }} {{ site.sched.flag.name }} new_name

echo -n "This script is running on "
hostname
echo "This script has finished successfully."
```
{: .output}

提交作业并监控其状态：

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
{{ site.remote.prompt }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/job-with-name-status.snip %}

太棒了，我们已经成功地更改了我们的作业名称！

> ## 设置电子邮件通知
>
> HPC 系统上的作业可能会运行数天甚至数周。与使用`{{ site.sched.status }}`不断检查作业状态相比，我们可能有更好的事情要做。查看 `{{ site.sched.submit.name }}`的手册页，您能否设置我们的测试作业以在完成时向您发送电子邮件？
>
> > ## 暗示
> >
> > 您可以使用{{ site.sched.name }}实用程序的*手册页* 来查找有关其功能的更多信息。在命令行上，这些可以通过`man`实用程序访问：运行`man <program-name>`。您可以通过搜索 > "man <program-name>"在线找到相同的信息。
> >
> > ```
> > {{ site.remote.prompt }} man {{ site.sched.submit.name }}
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

### 资源请求

但是更重要的变化呢，比如我们作业的核心数量和内存？在HPC系统上作业时绝对关键的一件事是指定运行作业所需的资源。这允许调度器找到合适的时间和地点来安排我们的作业。如果您没有指定要求（例如您需要的时间量），您可能会被网站的默认资源卡住，这可能不是您想要的。

以下是几个关键资源请求：

{% include {{ site.snippets }}/scheduler/option-flags-list.snip %}

请注意，仅*请求*这些资源并不能使您的作业运行得更快，也不一定意味着您将消耗所有这些资源。这仅意味着这些内容可供您使用。您的作业最终可能会使用比您请求的更少的内存、更少的时间或更少的任务或节点，并且它仍然会运行。

最好是您的请求准确地反映了您的作业要求。我们将在本课的后面部分详细讨论如何确保您有效地使用资源。

> ## 提交资源请求
>
> 修改我们的“主机名”脚本，让它运行一分钟，然后在集群上为它提交一个作业。
>
> > ## 解决方案
> >
> > ```
> > {{ site.remote.prompt }} cat example-job.sh
> > ```
> > {: .language-bash}
> >
> > ```
> > {{ site.remote.bash_shebang }}
> > {{ site.sched.comment }} {{ site.sched.flag.time }} 00:01:15
> >
> > echo -n "This script is running on "
> > sleep 60 # time in seconds
> > hostname
> > echo "This script has finished successfully."
> > ```
> > {: .output}
> >
> > ```
> > {{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
> > ```
> > {: .language-bash}
> >
> > 为什么{{ site.sched.name }}运行时和`sleep`时间不相同？
> {: .solution}
{: .challenge}

{% include {{ site.snippets }}/scheduler/print-sched-variables.snip %}

资源请求通常是绑定的。如果你超过了他们，你的作业就会被扼杀。让我们以walltime为例。我们将请求30秒的walltime，并尝试运行作业两分钟。

```
{{ site.remote.prompt }} cat example-job.sh
```
{: .language-bash}

```
{{ site.remote.bash_shebang }}
{{ site.sched.comment }} {{ site.sched.flag.name }} long_job
{{ site.sched.comment }} {{ site.sched.flag.time }} 00:00:30

echo "This script is running on ... "
sleep 120 # time in seconds
hostname
echo "This script has finished successfully."
```
{: .output}

提交作业并等待它完成。完成后，检查日志文件。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
{{ site.remote.prompt }} watch -n 15 {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/runtime-exceeded-job.snip %}

{% include {{ site.snippets }}/scheduler/runtime-exceeded-output.snip %}

我们的作业因超出其请求的资源量而被终止。尽管这看起来很苛刻，但这实际上是一个功能。严格遵守资源请求允许调度器为您的作业找到最佳位置。更重要的是，它确保另一个用户不能使用比他们所获得的资源更多的资源。如果另一个用户搞砸了并且不小心尝试使用节点上的所有内核或内存，{{ site.sched.name }} 将限制他们的作业到请求的资源或直接终止作业。节点上的其他作业将不受影响。这意味着一个用户不能破坏其他用户的体验，唯一受调度错误影响的作业将是他们自己的。

## 取消作业

有时我们会犯错，需要取消作业。这可以通过`{{ site.sched.del }}`命令来完成。让我们提交一个作业，然后使用它的作业编号取消它（记住更改walltime以便它运行足够长的时间让您在它被杀死之前取消它！）。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}example-job.sh
{{ site.remote.prompt }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/terminate-job-begin.snip %}

现在取消带有作业编号的作业（打印在您的终端中）。命令提示符的干净返回表明取消作业的请求成功。

```
{{ site.remote.prompt }} {{site.sched.del }} 38759
# It might take a minute for the job to disappear from the queue...
{{ site.remote.prompt }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/terminate-job-cancel.snip %}

{% include {{ site.snippets }}/scheduler/terminate-multiple-jobs.snip %}

## 其他类型的作业

到目前为止，我们一直专注于以批处理模式运行作业。{{ site.sched.name }}还提供了启动交互式会话的能力。

有非常频繁的作业需要以交互方式完成。创建一个完整的作业脚本可能是多余的，但所需的资源量对于登录节点来说太多了。一个很好的例子可能是建立一个基因组索引，以便与[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml) 等工具对齐。幸运的是，我们可以使用`{{ site.sched.interactive }}`一次性运行这些类型的任务。

{% include {{ site.snippets }}/scheduler/using-nodes-interactively.snip %}

{% include links.md %}
