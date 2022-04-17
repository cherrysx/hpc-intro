---
layout: reference
permalink: /reference/
title: 基础知识
---

### 作业调度系统参考

在线搜索最适合您的，但这里有一些供参考：

* [Slurm](https://slurm.schedmd.com/pdfs/summary.pdf)，开源软件
* [OpenPBS](https://www.openpbs.org/)，开源软件
* [Slurm和PBS区别](https://www.msi.umn.edu/slurm/pbs-conversion)
* [LSF](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=overview-lsf-introduction), 商业软件

### 单位和语言

计算机的内存和磁盘用*Bytes*作为单位来衡量（一个字节是8位）大小。由于今天的文件和内存已经变得很大，鉴于历史
标准，磁盘卷使用
[SI](https://en.wikipedia.org/wiki/International_System_of_Units) 前缀来标记。 所以1000 字节是千字节 (kB)，1000 千字节是兆字节 (MB)，1000兆字节是千兆字节 (GB) 等。

然而，历史上，人们使用不同的意思来标记这种符号。当人们说“千字节”时，他们的意思是 1024 字节。也就是说，一兆字节是1024千字节。

为了解决这种模糊性，[国际数量系统]（https://en.wikipedia.org/wiki/International_System_of_Quantities）制定标准：
*binary* 前缀（基数为2<sup>10</sup>=1024）由前缀 Kibi(ki)、Mebi (Mi)、Gibi (Gi) 等表示。详情请参阅[这里]（https://en.wikipedia.org/wiki/Binary_prefix）。

### “没有这样的文件或目录”或“符号 0096”错误

`scp` 和 `rsync` 可能会抛出一个令人费解的文件是否存在的错误。这些错误的一个来源是复制和粘贴来自Web浏览器命令行参数，其中双破折号字符串 `--` 呈现为 em-dash字符“&mdash;” （或短划线“&ndash;”，或单杠`―`）。 例如，以下命令没有实时显示传输速率，它神秘地失败了。

```
{{ site.local.prompt }} rsync —progress my_precious_data.txt {{ site.remote.user }}@{{ site.remote.login }}
rsync: link_stat "/home/{{ site.local.user }}/—progress" failed:
No such file or directory (2)
rsync error: some files/attrs were not transferred (see previous errors)
(code 23) at main.c(1207) [sender=3.1.3]
```
{: .language-bash}

正确的命令是progress前面有两个破折号：

```
{{ site.local.prompt }} rsync --progress my_precious_data.txt {{ site.remote.user }}@{{ site.remote.login }}
```
{: .language-bash}

我们已经尽力完善上述代码，如果遇到错误，请点击最下方`源码设计师`链接联系我。

### sftp交互式传输文件

`scp` 很有用，但是如果我们不知道目标位置或者我们不确定要传输哪些文件该怎么办？。`sftp` 是一种以交互式方式上传下载文件的工具，让我们
使用 `sftp` 连接到集群 - 你会注意到它的作业方式和SSH相同。

```
{{ site.local.prompt }} sftp yourUsername@remote.computer.address
```
{: .language-bash}

这将启动一个看起来像 bash shell 的东西（提示`sftp>`）。但是，我们只能使用有限数量的命令。我们可以使用 `help` 查看哪些命令可用：

```
sftp> help
```
{: .language-bash}
```
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp grp path                     Change group of file 'path' to 'grp'
chmod mode path                    Change permissions of file 'path' to 'mode'
chown own path                     Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afPpRr] remote [local]       Download file
reget [-fPpRr] remote [local]      Resume download file
reput [-fPpRr] [local] remote      Resume upload file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing

# omitted further output for clarity
```
{: .output}

请注意存在多个提及本地和远程的命令。我们实际上同时连接到本地和远程计算机（有两个作业目录！）。

显示远程作业目录:
```
sftp> pwd
```
{: .language-bash}
```
Remote working directory: /global/home/yourUsername
```
{: .output}

在pwd前面加l：lpwd，显示本地作业目录:

```
sftp> lpwd
```
{: .language-bash}
```
Local working directory: /home/jeff/Documents/teaching/hpc-intro
```
{: .output}

所有其他命令都遵循相同的模式：

* `ls` 显示我们的远程目录的内容，而 `lls` 显示我们的本地目录目录内容。
* `cd` 改变远程目录，`lcd` 改变本地目录。

要上传文件，我们输入`put some-file.txt`（tab文件名补全在这里可以用）。

```
sftp> put config.toml
```
{: .language-bash}
```
Uploading config.toml to /global/home/yourUsername/config.toml
config.toml                                  100%  713     2.4KB/s   00:00
```
{: .output}

要下载文件，我们输入`get some-file.txt`:

```
sftp> get config.toml
```
{: .language-bash}
```
Fetching /global/home/yourUsername/config.toml to config.toml
/global/home/yourUsername/config.toml        100%  713     9.3KB/s   00:00
```
{: .output}

我们可以通过添加 `-r` 递归地上传/下载文件。请注意，目录需要事先存在。

```
sftp> mkdir content
sftp> put -r content/
```
{: .language-bash}
```
Uploading content/ to /global/home/yourUsername/content
Entering content/
content/scheduler.md              100%   11KB  21.4KB/s   00:00
content/index.md                  100% 1051     7.2KB/s   00:00
content/transferring-files.md     100% 6117    36.6KB/s   00:00
content/.transferring-files.md.sw 100%   24KB  28.4KB/s   00:00
content/cluster.md                100% 5542    35.0KB/s   00:00
content/modules.md                100%   17KB 158.0KB/s   00:00
content/resources.md              100% 1115    29.9KB/s   00:00
```
{: .output}

退出请输入`exit`或者`bye`.

{% include links.md %}
