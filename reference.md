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

计算机的内存和磁盘以称为 *Bytes* 的单位来衡量（一个字节是8位）。由于今天的文件和内存已经变得很大，鉴于历史
标准，磁盘卷使用
[SI](https://en.wikipedia.org/wiki/International_System_of_Units) 前缀来标记。 所以1000 字节是千字节 (kB)，1000 千字节是兆字节 (MB)，1000兆字节是千兆字节 (GB) 等。

然而，历史上，人们使用不同的意思来标记这种符号。当人们说“千字节”时，他们的意思是 1024 字节。也就是说，一兆字节是1024千字节。

为了解决这种模糊性，[国际数量系统]（https://en.wikipedia.org/wiki/International_System_of_Quantities）制定标准：
*binary* 前缀（基数为2<sup>10</sup>=1024）由前缀 Kibi(ki)、Mebi (Mi)、Gibi (Gi) 等。详情请参阅[这里]（https://en.wikipedia.org/wiki/Binary_prefix）。

### “没有这样的文件或目录”或“符号 0096”错误

`scp` 和 `rsync` 可能会抛出一个令人费解的文件是否存在的错误。这些错误的一个来源是复制和粘贴来自Web浏览器命令行参数，其中双破折号字符串 `--` 呈现为 em-dash字符“&mdash;” （或短划线“&ndash;”，或单杠`―`）。 例如，而不是实时显示传输速率，以下命令神秘地失败了。

```
{{ site.local.prompt }} rsync —progress my_precious_data.txt {{ site.remote.user }}@{{ site.remote.login }}
rsync: link_stat "/home/{{ site.local.user }}/—progress" failed:
No such file or directory (2)
rsync error: some files/attrs were not transferred (see previous errors)
(code 23) at main.c(1207) [sender=3.1.3]
```
{: .language-bash}

The correct command, different only by two characters, succeeds:

```
{{ site.local.prompt }} rsync --progress my_precious_data.txt {{ site.remote.user }}@{{ site.remote.login }}
```
{: .language-bash}

We have done our best to wrap all commands in code blocks, which prevents this
subtle conversion. If you encounter this error, please open an issue or pull
request on the lesson repository to help others avoid it.

### Transferring Files Interactively With `sftp`

`scp` is useful, but what if we don't know the exact location of what we want
to transfer? Or perhaps we're simply not sure which files we want to transfer
yet. `sftp` is an interactive way of downloading and uploading files. Let's
connect to a cluster, using `sftp` -- you'll notice it works the same way
as SSH:

```
{{ site.local.prompt }} sftp yourUsername@remote.computer.address
```
{: .language-bash}

This will start what appears to be a bash shell (though our prompt says
`sftp>`). However we only have access to a limited number of commands. We can
see which commands are available with `help`:

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

Notice the presence of multiple commands that make mention of local and remote.
We are actually connected to two computers at once (with two working
directories!).

To show our remote working directory:
```
sftp> pwd
```
{: .language-bash}
```
Remote working directory: /global/home/yourUsername
```
{: .output}

To show our local working directory, we add an `l` in front of the command:

```
sftp> lpwd
```
{: .language-bash}
```
Local working directory: /home/jeff/Documents/teaching/hpc-intro
```
{: .output}

The same pattern follows for all other commands:

* `ls` shows the contents of our remote directory, while `lls` shows our local
  directory contents.
* `cd` changes the remote directory, `lcd` changes the local one.

To upload a file, we type `put some-file.txt` (tab-completion works here).

```
sftp> put config.toml
```
{: .language-bash}
```
Uploading config.toml to /global/home/yourUsername/config.toml
config.toml                                  100%  713     2.4KB/s   00:00
```
{: .output}

To download a file we type `get some-file.txt`:

```
sftp> get config.toml
```
{: .language-bash}
```
Fetching /global/home/yourUsername/config.toml to config.toml
/global/home/yourUsername/config.toml        100%  713     9.3KB/s   00:00
```
{: .output}

And we can recursively put/get files by just adding `-r`. Note that the
directory needs to be present beforehand.

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

To quit, we type `exit` or `bye`.

{% include links.md %}
