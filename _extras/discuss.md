---
title: "更进一步"
teaching: 10
exercises: 60
questions:
- "如何开始使用 HPC？"
- "我可以从哪里获得有关开始使用 HPC 的帮助？"
objectives:
- "获取帮助以在 HPC 系统上启动和运行您的作业"
- "了解您将来可以从哪里获得帮助"
keypoints:
- "了解您使用 HPC 的后续步骤。"
- "了解如何获得帮助和支持以使用 HPC。"
---

现在您对HPC有了足够的了解，可以探索如何将它用于您的作业，并了解它的潜在好处是什么。在您的作业研究领域，如何使用和/或尝试 HPC，可能会碰到一些障碍和困难。

> ## 潜在的讨论
>
> 您可以与讲师和助手讨论的事情可能包括：
>
> - 您的计算作业流程以及高级计算可以提供帮助的地方
> - 如何获得帮助和支持以使用高级计算运行您的作业。 例如，软件开发、进一步培训、获得专业知识
{: .callout}

## 本次课程的选项

在本次课程期间，练习有许多不同的选择。挑战包括以下内容：探索自己的作业；一个可扩展的使用并行HPC应用程序例子；使用高吞吐量的计算多个序列分析扩展示例。课程的想法是帮助引导您使用高级计算，针对每个人，使用场景将有所不同。

> ## 使用HPC探索您的作业
>
> 如果你有一个你所在作业领域的实际例子，你愿意使用HPC系统上启动和运行你的例子，探索HPC系统的性能，太棒了！请随时对我们并提出问题（技术和非技术）和并一起讨论。
{: .challenge}

> ## 探索GROMACS的性能
>
> [GROMACS](http://www.gromacs.org) 是世界领先的生物分子模型，世界各地的HPC系统上大量使用该软件包。选择GROMACS计算的最佳资源并不重要，因为它取决于许多因素，包括：
>
> - 正在使用的HPC系统的底层硬件
> - 由GROMACS包建模的实际系统
> - 用于并行计算的进程与线程的平衡
>
> 在这个练习中，你应该为一个典型生物分子系统，尝试并决定一个好的资源选择和设置。这将涉及：
>
> - 为GROMACS下载输入文件：
>   [{{ site.url }}{{site.baseurl }}/files/ion-channel.tpr](
>   {{ site.url }}{{site.baseurl }}/files/ion-channel.tpr)
> - 使用系统文档，编写作业提交脚本以在高性能计算集群上运行GROMACS
> - 改变用于 GROMACS 作业的节点数量（从 1 到 32 个节点是一个很好的起点）和性能基准测试（以 ns/天为单位）
> - 使用研究的结果来为GROMACS计算提出一个良好的资源选择
>
> 如果您想进一步探索这个初始任务，那么有许多不同的有趣方法可以做到这一点。 例如：
>
> - 改变每个进程使用的线程数
> - 减少每个节点使用的核心数量
> - 如果启用，则允许计算使用对称多线程(SMT)
>
{: .challenge}

> ## 并行运行许多串行BLAST+分析
>
> [BLAST+](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)
> 查找生物序列之间的相似区域。该程序将核苷酸或蛋白质序列与序列数据库进行比较并计算统计学意义。
>
> 在本练习中，您应该使用到目前为止所学的知识来设置方法，并行运行多个串行BLAST+分析。有很多不同的可以单独使用或组合使用的方法。一些想法包括：
>
> - 使用{{ site.sched.name }}作业数组跨不同节点运行多个副本
> - 在节点中使用bash循环
> - 在节点内使用GNU并行
>
> 我们准备了一个示例数据集，其中包含100个要分析的序列（实际上这是10个重复10次的序列）。 此数据集基于[BLAST GNU并行示例](https://github.com/LangilleLab/microbiome_helper/wiki/Quick-Introduction-to-GNU-Parallel)
>
> 这个联系包含:
>
> - 将数据集下载并扩展至HPC系统：:
>   [{{ site.url }}{{site.baseurl }}/files/parallel_example.tar.gz](
>   {{ site.url }}{{site.baseurl }}/files/parallel_example.tar.gz)
> - 使用 `blast` 模块和以下命令编写作业提交脚本以运行单个分析：
>
>   ```
>   blastp -db pdb_blast_db_example/pdb_seqres.txt -query test_seq_0.fas
>   -evalue 0.0001 -word_size 7  -max_target_seqs 10 -num_threads 1 \
>   -out output_seq_0.blast -outfmt "6 std stitle staxids sscinames"
>   ```
>   {: .language-bash}
>
>   其中 `\` 字符告诉 `bash` 命令在下一行继续。 请注意，如果它正常作业，则此符号之后不会有输出）。
> - 选择一种方法来运行所有100个分析任务，多个分析副本以并行方式完成（并非所有100个必须同时运行）。
>
> 您可以通过研究并行化此问题的不同方法和/或组合多个并行策略来进一步探索。
>
> 您还可以调查在节点上运行多个副本时的性能变化。硬件在什么时候变得过载？
{: .challenge}
