---
title: "运行并行作业"
teaching: 30
exercises: 30
questions:
- "我们如何并行执行任务？"
objectives:
- "了解如何在集群上运行并行作业。"
keypoints:
- "并行性是HPC集群的一个重要特性。"
- "MPI 并行性是一种常见的情况。"
- "队列系统有助于执行并行任务。"
---

我们现在拥有运行多处理器作业所需的工具 这是HPC系统的一个非常重要的方面，因为并行性是我们提高计算任务性能的主要工具之一。

我们的示例实现了一个随机算法来估计&#960;的值，即圆周与直径的比值。程序在以 (&frac12;,&frac12;) 为中心的 1×1 正方形上生成大量随机点，并检查这些点中有多少落在单位圆内。
平均而言，随机选择的点中有&#960;/4应该落在圆圈中，所 &#960;可以从4*f*估计，其中*f*是观察到的落在圆中的点的分数。因为每个样本都是独立的，所以这个算法很容易并行实现。

{% include figure.html url="" caption="" max-width="40%"
   file="/hpc1/fig/pi.png"
   alt="Algorithm for computing pi through random sampling" %}

## 问题的串行解决方案

从Python脚本开始。我们希望允许用户指定应该使用多少随机点来计算&#960; 通过命令行参数。该脚本在整个运行过程中只使用一个CPU，因此它被归类为串行进程。

让我们编写一个Pytho 程序`pi.py`来估计&#960;。首先导入用于计算结果的`numpy`模块，以及用于处理命令行参数的`sys`模块：

```
import numpy as np
import sys
```
{: .language-python}

我们定义了一个 Python 函数“inside_circle”，它接受一个用于计算 &#960; 的随机点数的参数。
它在半开区间[0, 1)上随机采样带有*x*和*y*的点。
然后它计算它们与原点的距离（即半径），并返回这些距离中有多少小于或等于 1.0。
所有这些都是使用双精度（64 位）浮点值的*vectors*完成的。

```
def inside_circle(total_count):
    x = np.random.uniform(size=total_count)
    y = np.random.uniform(size=total_count)
    radii = np.sqrt(x*x + y*y)
    count = len(radii[np.where(radii<=1.0)])
    return count
```
{: .language-python}

接下来，我们创建一个main函数来调用`inside_circle`函数并计算&#960; 从其返回的结果。

```
def main():
    n_samples = int(sys.argv[1])
    counts = inside_circle(n_samples)
    my_pi = 4.0 * counts / n_samples
    print(my_pi)

if __name__ == '__main__':
    main()
```
{: .language-python}

如果我们使用命令行参数在本地运行Python脚本，例如在 `python pi-serial.py 1024` 中，我们应该会看到脚本打印出它对 &#960; 的估计值：

```
{{ site.local.prompt }} python pi-serial.py 1024
3.10546875
```
{: .language-bash}

> ## 随机数生成
>
> 在前面的代码中，使用NumPy的内置功能可以方便地生成随机数。一般来说，随机数生成很难做好，很容易在生成的序列中意外引入相关性。
>
> * 讨论为什么生成高质量的随机数可能很困难。
> * 在这个实现中，生成的随机数的质量是否足以估计&#960;？
>
> > ## 解决方案
> >
> > * 计算机是确定性的，并使用算法产生伪随机数。算法及其参数的选择决定了生成数字的随机程度。伪随机数生成算法通常会产生一个序列号，将前一个输出作为输入来生成下一个数字。在某些时候，伪随机数序列会重复，因此需要注意确保重复周期较长，并且生成的数字具有与真随机数相似的统计特性。
> > * 是的。
> {: .solution }
{: .discussion }

## 测量串行解决方案的性能

用于估计&#960;的随机方法随着随机点数的增加，应该收敛于真实值。但是随着点数的增加，创建变量 `x`、`y` 和 `radii` 需要更多的时间和更多的内存。
最终，所需的内存可能会超过我们本地笔记本电脑或台式机的可用内存，或者所需的时间可能太长而无法满足最后期限。所以我们想对脚本需要多少内存和时间进行一些测量，然后在创建脚本的并行版本后进行相同的测量，以了解并行化所需计算的好处。

### 估计内存需求

由于脚本中最大的变量是“x”、“y”和“radii”，每个变量都包含“n_samples”个点，我们将修改脚本以报告它们所需的总内存。`x`、`y`或`radii`中的每个点都存储为NumPy`float64`，我们可以使用NumPy的[`dtype` (https://numpy.org/doc/stable/reference/generated/numpy.dtype.html)函数来计算`float64`的大小。

将`print(my_pi)`行替换为以下内容：

```
size_of_float = np.dtype(np.float64).itemsize
memory_required = 3 * n_samples * size_of_float / (1024**3)
print("Pi: {}, memory: {} GiB".format(my_pi, memory_required))
```
{: .language-python}

第一行使用`dtype`函数计算单个`float64`值所需的内存字节数。第二行估计存储三个包含`n_samples` `float64`值的变量所需的内存总量，将值转换为单位[gibibytes](https://en.wikipedia.org/wiki/Byte#Multiple-byte_units)。第三行打印出&#960;的估计值。以及脚本使用的估计内存量。

更新后的Python脚本是：

```
import numpy as np
import sys

def inside_circle(total_count):
    x = np.random.uniform(size=total_count)
    y = np.random.uniform(size=total_count)
    radii = np.sqrt(x*x + y*y)
    count = len(radii[np.where(radii<=1.0)])
    return count

def main():
    n_samples = int(sys.argv[1])
    counts = inside_circle(n_samples)
    my_pi = 4.0 * counts / n_samples
    size_of_float = np.dtype(np.float64).itemsize
    memory_required = 3 * n_samples * size_of_float / (1024**3)
    print("Pi: {}, memory: {} GiB".format(my_pi, memory_required))

if __name__ == '__main__':
    main()
```
{: .language-python}

使用几个不同的样本数值再次运行脚本，并查看所需的内存如何变化：

```
{{ site.local.prompt }} python pi-serial.py 1000
Pi: 3.144, memory: 2.2351741790771484e-05 GiB
{{ site.local.prompt }} python pi-serial.py 2000
Pi: 3.18, memory: 4.470348358154297e-05 GiB
{{ site.local.prompt }} python pi-serial.py 1000000
Pi: 3.140944, memory: 0.022351741790771484 GiB
{{ site.local.prompt }} python pi-serial.py 100000000
Pi: 3.14182724, memory: 2.2351741790771484 GiB
```
{: .language-bash }

在这里我们可以看到估计所需的内存量与使用的样本数量成线性关系。实际上，脚本的其他部分需要一些内存，但`x`、`y`和`radii`变量是迄今为止对所需内存总量影响最大的变量。

### 估计计算时间

估计&#960;所需的大部分计算在`inside_circle`函数中：

1. 为`x`和`y`生成`n_samples`随机值。
1. 从`x`和`y`计算`n_samples`的`radii`值。
1. 计算`radii`中有多少个值小于1.0。

在主函数中，将“counts”值转换为&#960;的最终估计值还需要一次乘法运算和一次除法运算。

测量计算时间的一种简单方法是使用Python的`datetime`模块存储计算前后计算机的当前日期和时间，并计算这些时间之间的差异。

要将时间测量添加到脚本中，请在`import sys`行下方添加以下行：

```
import datetime
```
{: .language-python}

然后，在计算`counts`行的正上方添加以下行：

```
start_time = datetime.datetime.now()
```
{: .language-python}

在计算`counts`行的正下方添加以下两行：

```
end_time = datetime.datetime.now()
elapsed_time = (end_time - start_time).total_seconds()
```
{: .language-python}

最后，使用以下内容修改`print`语句：

```
print("Pi: {}, memory: {} GiB, time: {} s".format(my_pi, memory_required,
                                                  elapsed_time))
```
{: .language-python}

串行解决方案的最终Python脚本是：

```
import numpy as np
import sys
import datetime

def inside_circle(total_count):
    x = np.random.uniform(size=total_count)
    y = np.random.uniform(size=total_count)
    radii = np.sqrt(x*x + y*y)
    count = len(radii[np.where(radii<=1.0)])
    return count

def main():
    n_samples = int(sys.argv[1])
    start_time = datetime.datetime.now()
    counts = inside_circle(n_samples)
    my_pi = 4.0 * counts / n_samples
    end_time = datetime.datetime.now()
    elapsed_time = (end_time - start_time).total_seconds()
    size_of_float = np.dtype(np.float64).itemsize
    memory_required = 3 * n_samples * size_of_float / (1024**3)
    print("Pi: {}, memory: {} GiB, time: {} s".format(my_pi, memory_required,
                                                      elapsed_time))

if __name__ == '__main__':
    main()
```
{: .language-python}

使用几个不同的样本数值再次运行脚本，看看求解时间如何变化：

```
{{ site.local.prompt }} python pi-serial.py 1000000
Pi: 3.139612, memory: 0.022351741790771484 GiB, time: 0.034872 s
{{ site.local.prompt }} python pi-serial.py 10000000
Pi: 3.1425492, memory: 0.22351741790771484 GiB, time: 0.351212 s
{{ site.local.prompt }} python pi-serial.py 100000000
Pi: 3.14146608, memory: 2.2351741790771484 GiB, time: 3.735195 s
```
{: .language-bash }

在这里，我们可以看到所需的时间量与使用的样本数量大致呈线性关系。
由于运行的时间受同时在计算机上运行的其他程序的影响，因此具有相同数量的样本的脚本的运行时间可能会有一些变化。
但如果脚本是当时计算机上运行的进程中计算最密集，那么它的计算过程对运行时间的影响最大。

现在我们已经开发了估计&#960;的初始脚本，我们可以看到随着样本数量的增加：

1. &#960;的估计趋于变得更加准确。
1. 所需的内存量大致呈线性增长。
1. 计算的时间量大致呈线性变化。

一般来说，实现更好的估计&#960; 需要更多的点数。
仔细看看“inside_circle”：我们应该期望在单台机器上获得高精度吗？

可能不是。
该函数分配三个大小为*N*的数组，其大小等于属于该进程的点数。
使用64位浮点数，这些数组的内存占用会变得非常大。每100,000,000个采样点消耗2.24GiB的内存。 对400,000,000个点进行采样会消耗 8.94 GiB 的内存，如果您的机器的RAM小于此值，它将停止运行。如果您安装了16GiB，那么您将无法达到750,000,000点。

## 在计算节点上运行串行作业

创建提交文件，请求单个节点上的一个任务和足够的内存以防止作业内存不足：

```
{{ site.remote.prompt }} nano serial-pi.sh
{{ site.remote.prompt }} cat serial-pi.sh
```
{: .language-bash}

{% include {{ site.snippets }}/parallel/one-task-with-memory-jobscript.snip %}

然后提交你的作业。我们将使用批处理文件来设置选项，而不是命令行。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} serial-pi.sh
```
{: .language-bash}

和以前一样，使用状态命令检查作业何时运行。使用`ls`定位输出文件并检查它。是你所期望的吗？

* &#960;的值有多精确？
* 它需要多少内存？
* 作业需要多长时间才能完成？

修改作业脚本以增加样本数和请求的内存量（可能增加2倍，然后增加10倍），并每次重新提交作业。

* &#960;的值有多精确？
* 它需要多少内存？
* 作业需要多长时间才能完成？

即使有足够的内存来存储必要的变量，脚本也可能需要大量时间在单个CPU上进行计算。为了减少所需的时间，我们需要修改脚本以使用多个CPU进行计算。在最大的问题规模中，我们可以在多个计算节点中使用多个CPU，将内存需求分布在用于计算解决方案的所有节点上。

## 运行并行作业

我们将运行一个使用消息传递接口 (MPI) 进行并行处理的示例——这是HPC系统上的常用工具。

> ## 什么是MPI？
>
> 消息传递接口是一组允许多个并行任务相互通信的工具。
> 通常，单个可执行文件可能在不同的机器上运行多次，并且MPI工具用于通知可执行文件的每个实例有多少实例，它是哪个实例。MPI还提供了允许实例之间进行通信和协调的工具。
> MPI实例通常拥有自己的所有局部变量的副本。
{: .callout}

虽然MPI作业通常可以作为独立的可执行文件运行，但为了让它们并行运行，它们必须使用MPI*运行时系统*，这是MPI*标准*的特定实现。
为此，它们应该通过诸如`mpiexec`（或`mpirun`，或`srun`等，取决于您需要使用的MPI运行时）之类的命令启动，这将确保适当的运行 - 包括对并行性的时间支持。

> ## MPI运行时参数
>
> 就其自身而言，诸如`mpiexec`之类的命令可以使用许多参数来指定将有多少台机器参与执行，如果您想在笔记本电脑上运行MPI程序（例如），您可能需要这些参数。
> 然而，在排队系统（调度器）的上下文中，通常情况下我们不需要指定此信息，MPI运行时已配置为通过检查在作业启动时设置的环境变量从排队系统（调度器）获取它。
{: .callout}

> ## &#960;的MPI计算版本需要进行哪些更改？
>
> 首先，我们需要从Python模块`mpi4py`导入`MPI`对象，方法是在`import datetime`行的正下方添加`from mpi4py import MPI`行。
>
> 其次，我们需要修改“main”函数来执行所需的开销和统计作业：
>
> * 细分要采样的总点数，
> * *partition*可用的各种并行处理器之间的总作业量，
> * 让每个并行进程将其作业负载的结果报告回“等级 0”进程，该进行最终计算并打印出结果。
>
> 对串行脚本的修改展示了四个重要概念：
>
> * COMM_WORLD: 默认的MPI Communicator，为这个`mpiexec`中涉及的所有进程提供一个通道，以相互交换信息。
> * Scatter: 一种集体操作，其中一个MPI等级上的一组数据被划分，单独的部分被发送到合作伙伴rank。每个伙伴rank从主机阵列的匹配索引中接收数据。
> * Gather: 散射的倒数。一个等级填充一个本地数组，每个索引处的数组元素分配由相应伙伴rank提供的值 - 包括主机自己的值。
> * Conditional Output: 由于每个rank都在运行*相同的代码*，因此分区、最终计算和`print`语句都包含在条件中，因此只有一个rank执行这些操作。
{: .discussion}

我们添加以下行：

```
comm = MPI.COMM_WORLD
cpus = comm.Get_size()
rank = comm.Get_rank()
```
{: .language-python}

在`n_samples`行之前为每个进程设置MPI环境。

我们将`start_time`和`counts`行替换为以下行：

```
if rank == 0:
  start_time = datetime.datetime.now()
  partitions = [ int(n_samples / cpus) ] * cpus
  counts = [ int(0) ] * cpus
else:
  partitions = None
  counts = None
```
{: .language-python}

这确保只有rank 0进程测量时间并协调分配给所有rank的作业，而其他rank获取`partitions`和`counts`变量的占位符值。

在这些线的正下方，让我们

* 使用MPI `scatter`在队伍中分配作业，
* 调用`inside_circle`函数，以便每个rank都可以执行其作业份额，
* 使用MPI`gather`将每个排名的结果收集到rank 0的`counts`变量中。

通过添加以下三行：

```
partition_item = comm.scatter(partitions, root=0)
count_item = inside_circle(partition_item)
counts = comm.gather(count_item, root=0)
```
{: .language-python}

这些步骤的图示如下所示。

---

设置MPI环境并初始化局部变量——包括包含要在每个并行处理器上生成的点数的向量：

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/initialize.png"
   alt="MPI initialize" %}

将原始向量中的点数分配给所有并行处理器：

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/scatter.png"
   alt="MPI scatter" %}

并行执行计算：

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/compute.png"
   alt="MPI compute" %}

从所有并行进程中检索计数：

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/gather.png"
   alt="MPI gather" %}

打印报告：

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/finalize.png"
   alt="MPI finalize" %}

---

最后，我们将确保`my_pi`到`print`行仅在rank 0上运行。否则，每个并行处理器都会打印其本地值，并且报告将变得无可救药的乱码：

```
if rank == 0:
   my_pi = 4.0 * sum(counts) / sum(partitions)
   end_time = datetime.datetime.now()
   elapsed_time = (end_time - start_time).total_seconds()
   size_of_float = np.dtype(np.float64).itemsize
   memory_required = 3 * sum(partitions) * size_of_float / (1024**3)
   print("Pi: {}, memory: {} GiB, time: {} s".format(my_pi, memory_required,
                                                            elapsed_time))
```
{: .language-python}

[此处](/files/pi-mpi.py) 提供了最终 MPI 并行 python 代码的完整注释版本。

我们在这里的目的是练习集群的并行作业流程，而不是优化程序以最小化其内存占用。
与其把我们的本地机器推到断点（或者更糟的是登录节点），不如把它交给一个拥有更多资源的集群节点。

创建提交文件，在单个节点上请求多个任务：

```
{{ site.remote.prompt }} nano parallel-pi.sh
{{ site.remote.prompt }} cat parallel-pi.sh
```
{: .language-bash}

{% include {{ site.snippets }}/parallel/four-tasks-jobscript.snip %}

然后提交你的作业。我们将使用批处理文件来设置选项，而不是命令行。

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} parallel-pi.sh
```
{: .language-bash}

和以前一样，使用状态命令检查作业何时运行。使用`ls`定位输出文件并检查它。是你所期望的吗？

* &#960;的值有多精确？
* 它需要多少内存？
* 这次运行比100000000点的串行运行快多少？

修改作业脚本以增加样本数和请求的内存量（可能增加2倍，然后增加10倍），并每次重新提交作业。
您还可以增加CPU的数量。

* &#960;的值有多精确？
* 它需要多少内存？
* 作业需要多长时间才能完成？

## MPI能提高多少性能？

理论上，通过划分&#960; *n* MPI进程之间的计算，我们应该看到运行时间减少了*n*倍。
在实践中，启动额外的MPI进程需要一些时间，以便MPI进程进行通信和协调，并且某些类型的计算可能只能在单个CPU上有效运行。

此外，如果MPI进程在计算机中的不同物理CPU上运行，或者跨多个计算节点运行，则与在单个CPU上运行的所有进程相比，通信需要额外的时间。

阿姆达尔定律是预测**固定**并行作业负载执行时间改进的一种方法。如果一个作业负载需要20小时在单核上完成，而其中1小时用于无法并行化的任务，则只有剩余的 19小时可以并行化。
即使无限数量的内核用于作业负载的并行部分，总运行时间也不能少于一小时。

在实践中，通常通过以下方式评估MPI程序的并行性

* 在一系列CPU数量上运行程序，
* 记录每次运行的执行时间，
* 将每次执行时间与使用单个CPU时的时间进行比较。

加速因子*S*计算为单CPU执行时间除以多CPU执行时间。对于具有8个内核的笔记本电脑，加速因子与使用的内核数量的关系图显示，在使用2、4或8个内核时，改进相对一致，但使用额外内核显示的收益递减。

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/laptop-mpi_Speedup_factor.png"
   alt="MPI speedup factors on an 8-core laptop" %}

对于一组每个包含28个内核的HPC节点，加速因子与内核数量的关系图显示了三个节点和84个内核的持续改进，但在添加具有额外28个内核的第四个节点时性能变**差**。
这是由于MPI进程之间所需的通信和协调量需要比通过减少每个MPI进程必须完成的作业量获得的更多时间。这种通信开销不包括在阿姆达尔定律中。

{% include figure.html url="" caption="" max-width="50%"
   file="/hpc1/fig/hpc-mpi_Speedup_factor.png"
   alt="MPI speedup factors on an 8-core laptop" %}

在实践中，MPI加速因素受以下因素影响：

* CPU设计，
* 计算节点之间的通信网络，
* MPI库实现，以及
* MPI程序本身的细节。

在HPC环境中，我们试图减少所有类型作业的执行时间，而MPI是一种非常常见的方法，可以将数十个、数百个或数千个CPU组合成解决单个问题。

{% include links.md %}
