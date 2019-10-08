# 《linux就该这么学》笔记(一)
---
# RPM(Redhat Package Manager)红帽软件包管理器

RPM 为了解决包的安装、升级、依赖操作的难度过大而设计的，常见命令有：

|命令|作用|
|-|-|
|rpm -ivh filename.rpm | 安装软件的命令格式|
|rpm -Uvh filename.rpm | 升级软件的命令格式|
|rpm -e   filename.rpm | 卸载软件的命令格式|
|rpm -qpi filename.rpm | 查询软件描述信息的命令格式|
|rpm -qpl filename.rpm | 列出软件文件信息的命令格式|
|rpm -qf  filename.rpm | 查询文件属于哪个RPM的命令格式|

# yum软件仓库

yum软件仓库则是为了进一步简化RPM管理软件难度而设计的，能够根据用户的需求分析出所需软件包及其相关依赖关系，自动从服务器下载软件包并安装到系统。

常用命令有：
|命令|作用|
|-|-|
|yum repolist all        |列出所有仓库|
|yum list all             |列出仓库中所有软件包|
|yum info 软件包名称       |查看软件包信息|
|yum install 软件包名     |安装软件包|
|yum reinstall 软件包名   |重新安装软件包|
|yum update 软件包名      |升级软件包|
|yum remove 软件包名      |移除软件包|
|yum clean all            |清除所有仓库缓存|
|yum check-update         |检查可更新的软件包|
|yum grouplist            |查看系统中已经安装的软件包组|
|yum grouplist 软件包组   |安装指定的软件包组|
|yum groupremove 软件包组 |移除指定的软件包组|
|yum groupinfo 软件包组   |查询指定的软件包信息|

# system初始化过程

在RHEL7时，弃用了之前的init 初始化进程，更新为systemctl接管

常见命令有：

|命令|作用|
|-|-|
|systemctl restart 服务名称  | 重启服务|
|systemctl start   服务名称  | 启动服务|
|systemctl stop    服务名称  | 停止服务|
|systemctl enable  服务名称  | 加入到开机启动项|
|systemctl disable 服务名称  | 取消加入开机启动项|
|systemctl status  服务名称  | 查看服务的状态|

# shell和bash
shell的意思是“壳”，充当的是人和内核之间的翻译官，提供了一组接口，使人可以对Linux内核进行操作。bash全称为Bourne Again Shell，是一种shell，可以说是linux系统默认的shell。例如mac系统使用的shell为zsh。

# 常用热键
## [Tab]按键
该按键有两个作用：

- 命令补全：[Tab]接在一串命令的第一个命令的后面
- 文件补全：[Tab]接在一串命令的第二个以及第二个命令以后

例如：
```bash
[root@localhost ~]$ ca[]tab[tab]
cacertdir_rehash     cache_metadata_size  cache_writeback      ca-legacy            captoinfo            catchsegv            
cache_check          cache_repair         cairo-sphinx         caller               case                 catman               
cache_dump           cache_restore        cal                  capsh                cat 
```
```bash
[zhangxh@localhost ~]$ ls -al ~/.bash[tab][tab]
.bash_history  .bash_logout   .bash_profile  .bashrc  
```

# 常用命令
## man
帮助命令，常用的操作按键有：

|命令|作用|
|-|-|
|空格键        |  向下翻一页|
|[Page Down]  |   向下翻一页|
|[Page Up]     |  向上翻一页|
|[Home]        |  直接前往首页|
|[End]         |  直接前往尾页|
|/ 关键词      |  从上之下搜过某个关键词|
|? 关键词      |  从下至上搜索某个关键词|
|n             |  定位到下一个搜索的关键词|
|N             |  定位到下一个搜索的关键词|
|q             |  退出帮助文档|

除了man命令，帮助命令还有help命令和info命令。

info命令件说明文件分成多个节点(node)，每个节点都有定位与链接，用“*”标识。在进入info页面后敲击“?”可以基本命令，常用命令如下：

|命令|作用|
|-|-|
|空格键        |  向下翻一页|
|[Page Down]  |   向下翻一页|
|[Page Up]     |  向上翻一页|
|[Tab] | 在节点之间移动|
|[Enter]|进入节点|
|B|移动光标到该info界面中的第一个节点处|
|E|移动光标到该info界面中的最后一个节点处|
|N|前往下一个节点|
|P|前往上一个节点|
|U|向上移动一层|
|S(/)|在info page中进行查询|
|H|显示求助菜单|
|?|命令一览表|
|Q|结束info page|

**注意**：上述命令中字母的大小写不影响命令。

## 关机相关命令
### sync
在关机之前需要将磁盘上的数据同步到硬盘上，使用sync命令。shutdown/reboot/halt命令在关机前也会调用sync命令。

### 关机重启命令
- shutdown：安全关机。可以自定义关机时间和关机消息，具体的可以使用man命令查看
- reboot：重启
- halt：硬件关机，不推荐
- poweroff：关机并关电源
- init 0：Linux系统七种执行等级的第0级，关机

## echo命令
用于在终端显示字符串或者变量，格式为: echo [字符串|变量]

例如：
```
echo Linux.com
echo $SHELL      //这里的$命令意思是将SHELL变量的值取出来
```
## date命令
|   参数  | 作用 |
|--------|---------|
|  %t      | [tab]键     |
|  %H      | 小时(00~23)        |
|  %I      |    小时(00~12)     |
|  %M      |分钟(00~59)         |
|  %S      |秒(00~59)         |
|  %j      |今年中第几天         |

[年-月-日 小时:分钟:秒] 的格式输出
```
date "+%Y-%m-%d %H:%M:%S"
```

## wget命令
该命令用于下载网络文件。格式为 `wget[参数] 下载地址`。

主要参数和作用有：

|参数|作用|
|-|-|
|-b|后台下载模式|
|-O|指定下载目录和下载完之后的文件名|
|-t|最大尝试次数|
|-c|断点续传|
|-p|下载页面内所有资源，包括图片、视频等|
|-r|递归下载|

例子：

```bash
wget -O ./temp/hehehe.png https://c1.hoopchina.com.cn/uploads/star/event/images/191005/a966007a44a62599e2ea5423413ba90b626a5a51.png
```

## uname命令
用于查看内核版本等信息。常用命令为`uname -a`，表示查看所有内核信息。

如需向查看系统详细版本，可以看redhat-release文件。

```bash
[root@localhost ~]$ cat /etc/redhat-release 
CentOS Linux release 7.7.1908 (Core)
```

## uptime
用于查看系统负载情况，格式为：`uptime`。也可以使用`watch -n 1 uptime`来每隔1秒刷新一下。如下：

```bash
[root@localhost ~]$ uptime
 10:45:43 up 38 days,  5:53,  1 user,  load average: 0.00, 0.01, 0.05
```
输出内容分别为**系统当前时间**、**系统已运行时间**、**当前在线用户**和**平均负载值**。平均负载有三个值，分别为最近1分钟、5分钟和15分钟的负载情况。负载值越低越好，小于1是正常。

## top命令
用于持续观察系统中的进程快照。命令为`top`，如下：

<div align="center">
<img src="top-command.PNG">
</div>
第一行的内容和uptime相同。进程详细信息的含义为：

|名称|含义|
|-|-|
|PID|进程ID|
|USER|进程所有者的用户名|
|PR|进程优先级，有两种格式，一种是数字（默认20），一种是RT字符串|
|NI|动态修正CPU调度。范围（-20~19）。越大，cpu调度越一般，越小，cpu调度越偏向它|
|VIRT  |  进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES|
|RES |   进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA|
|SHR  |  共享内存大小，单位kb|
|S  |  进程状态|
|%CPU|总体CPU百分比，按H可以显示所有线程|
|%MEM  |  进程使用的物理内存百分比|
|TIME+ |   进程使用的CPU时间总计，单位1/100秒|
|COMMAND |   命令名/命令行|

其中进程状态常见的有如下几种：

- D：不可中断的睡眠状态
- R：运行
- S：睡眠
- T：跟踪/停止
- Z：僵尸进程

## free
显示内存使用情况，主要单数为：

|参数|含义|
|-|-|
|-b -k -m -g | 分别以字节(B)\KB\MB\GB为单位显示内存使用情况|
|-s seconds | 显示每隔多少秒数来显示一次内存使用情况|
|-t | 显示内存总和列|

## who命令
用来显示当前登录主机的用户情况。

```bash
[root@localhost ~]$ who
root     pts/0        2019-10-08 10:25 (192.168.10.144)
```

## history命令
用来显示历史执行过的命令，默认是1000条，可以在/etc/profile中的HISTSIZE修改。历史命令保存在用户家目录的.bash_history文件中。

## 目录相关的命令
### pwd命令
显示当前目录，主要有两个参数：

|参数|作用|
|-|-|
|-L|逻辑目录，如果有软链接就使用软链接的目录|
|-P|显示真实路径，非软链接的地址|

### cd命令
change directory，主要有以下参数：

|参数|作用|
|-|-|
|.|代表当前目录|
|..|代表当前目录的上一个目录|
|-|代表前一个工作目录|
|~|代表当前用户的“家”目录|
|~account|代表用户account的“家”目录|

### ls命令
用于查看目录中的文件，格式为 `ls[选项][文件]`。

常用的参数有：

|参数|作用|
|-|-|
|-a|查看全部文件，包括隐藏文件|
|-h|和-l连用，以易读的方式显示文件容量，如k,m,g|
|-l|显示文件的详细信息|

另外，`ll`命令等同于`ls -l`。

## 文本文件编辑命令
### cat命令
用于查看纯文本文件，格式为`cat [选项][文件]`。

常用的参数有：

|参数|作用|
|-|-|
|-n|显示行号|
|-b|显示行号，但不包括空行|
|-A|显示出“不可见”的符号，如空格，tab键等|

### more
查看文本文件，格式为`more [选项] 文件名`

常用参数有：

|参数|作用|
|-|-|
|- 数字|预先显示的行数，默认为一页|
|-d|显示命令提示语句，如"[Press space to continue, 'q' to quit.]|

### head命令
用于查看纯文本文件的前N行，格式为`head[选项][文件]`

主要参数如下：

|参数|作用|
|-|-|
|-n 10 |显示前10行，等同于 -10|
|-n -10|不显示最后10行|

### tail命令
用于查看纯文本文件的后N行，格式为`tail[选项][文件]`

主要参数如下：

|参数|作用|
|-|-|
|-n 10 |显示后10行，等同于 -10，-n -10|
|-f|持续刷新显示内容|

### od命令
将文件的内容以特定的格式输出，格式为`od[选项][文件]`

主要参数如下：

|参数|作用|
|-|-|
|-t a|以默认字符输出，等同于-a|
|-t c|以ASCII字符输出，等同于-c|
|-t o|以八进制输出，等同于-o|
|-t d|以十进制输出，等同于-d|
|-t x|以十六进制输出，等同于-x|
|-t f|以浮点数输出，等同于-f|

### tr命令
用于转换文本文件中的字符，格式为`tr [options] [set1][set2]`

|参数|作用|
|-|-|
|-d|删除文本文件中与set1中字符重复的字符，使用-d时只能有set1不能有set2|

例如，将test.java文件中的小写字母变为大写并输出的命令为：

```bash
cat test.java | tr [a-z][A-Z]
```

### wc命令
用于统计文本文件正的行数、单词数和字节数，格式为`wc [参数] 文本`

|参数|作用|
|-|-|
|-l|只显示行数|
|-w|只显示单词数|
|-c|只显示字节数|

### cut命令
用于通过列来提取文本字符，格式为`cut [参数] 文本`

|参数|作用|
|-|-|
|-d 分隔符|指定分隔符，默认为Tab|
|-f|执行显示的列数|

例如，下面的命令会将t.txt 中的文本以“:”分隔，然后显示第一列

```bash
cut -d : -f 1 t.txt
```

### diff命令
用于比较多个文本文件的差异，格式为`diff [参数] 文件1 文件2`

|参数|作用|
|-|-|
|-b|忽略空格引起的差异|
|-B|忽略空行引起的差异|
|-q或者--brief|仅报告是否存在差异|