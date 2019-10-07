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

