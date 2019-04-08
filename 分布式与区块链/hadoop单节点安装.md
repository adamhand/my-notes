# hadoop单节点安装
---

# 环境和版本
- ubuntu18.04
- jdk1.8.0
- hadoop2.9.2

# 安装jdk

略

# 安装hadoop
## 下载
选择在国内镜像下载，速度比较快。
```linux
wget http://mirror.bit.edu.cn/apache/hadoop/common/stable/hadoop-2.9.2.tar.gz
```

## 解压
```linux
tar -zxvf hadoop-2.9.2.tar.gz
```

进入`/etc/hadoop`目录
```linux
cd /etc/hadoop
```
## 修改`hadooop-env.sh`
```linux
vim hadoop-env.sh
```
```linux
export JAVA_HOME=/opt/installed/jdk1.8.0_161/  # 注意这里要写绝对路径，不能写${JAVA_HOME}
```

## 配置core-site.xml
fs.defaultFS ： 这个属性用来指定namenode的hdfs协议的文件系统通信地址，可以指定一个主机+端口，也可以指定为一个namenode服务（这个服务内部可以有多台namenode实现ha的namenode服务。

hadoop.tmp.dir : hadoop集群在工作的时候存储的一些临时文件的目录。
```linux
vim core-site.xml
```

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
 
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/data/hadoopdata</value>
    </property> 
</configuration>
```

## 配置hdfs-site.xml
dfs.namenode.name.dir：namenode数据的存放地点。也就是namenode元数据存放的地方，记录了hdfs系统中文件的元数据。

dfs.datanode.data.dir： datanode数据的存放地点。也就是block块存放的目录了。

dfs.replication：hdfs的副本数设置。也就是上传一个文件，其分割为block块后，每个block的冗余副本个数，默认配置是3。

dfs.secondary.http.address：secondarynamenode 运行节点的信息，和 namenode 不同节点
```linux
vim hdfs-silte.xml
```

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/hadoop/data/hadoopdata/name</value>
        <description>为了保证元数据的安全一般配置多个不同目录</description>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/hadoop/data/hadoopdata/data</value>
        <description>datanode 的数据存储目录</description>
    </property>

    <property>
        <name>dfs.replication</name>
        <value>2</value>
        <description>HDFS 的数据块的副本存储个数, 默认是3</description>
    </property>
    <!-- 
    <property>
        <name>dfs.secondary.http.address</name>
        <value>slave2:50090</value>
        <description>secondarynamenode 运行节点的信息，和 namenode 不同节点</description>
    </property>  -->
</configuration>
```

## 配置mapred-site.xml
mapreduce.framework.name：指定mr框架为yarn方式，Hadoop二代MP也基于资源管理系统Yarn来运行 。
```linux
vim mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>   
    </property>
</configuration>
```

## 配置yarn-site.xml
yarn.resourcemanager.hostname：yarn总管理器的IPC通讯地址

yarn.nodemanager.aux-services：YARN 集群为 MapReduce 程序提供的服务（常指定为 shuffle ）
```linux
vim
```

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
<!--    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>slave3</value>
    </property>
 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        <description>YARN 集群为 MapReduce 程序提供的 shuffle 服务</description>
    </property>
</configuration>
```

## 配置hadoop环境变量
```linux
vim /etc/profile
```

```linux
# User specific aliases and functions
 
export HADOOP_HOME=/home/hadoop/apps/hadoop-2.9.1
 
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:
```

```linux
source /etc/profile
```
验证环境变量是否生效，查看hadoop版本：
```linux
hadoop version
```

## 初始化hadoop namenode节点
```linux
hdfs namenode -format
```
成功后会有日志提示。

## 启动hdfs
```linux
start-dfs.sh
```
然后输入root的密码。

---
注意，如果启动的时候遇到`root@localhost's password:localhost:permission denied,please try again`问题，可能的原因有以下几个：

- root密码输入错误
可以通过该密码的方式：
```linux
sudo root
```
然后输入两次新密码。

- 防火墙的问题
需要将防火墙关闭：
```linux
sudo ufw enable
```
可以通过`sudo ufw status`查看防火墙状态。

- root没有开启允许ssh的远程连接
首先要确认ssh已经安装成功：
```linux
sudo apt-get install openssh-server  #open ssh
```
辑配置文件，允许以 root 用户通过 ssh 登录：sudo vi /etc/ssh/sshd_config
找到：PermitRootLogin prohibit-password禁用(如果没有找到这一行，直接添加PermitRootLogin yes即可)
添加：PermitRootLogin yes
然后重启ssh服务：
```linux
sudo service ssh restart
```

---

## 参考
[Hadoop学习（1）Hadoop2.9.1完全分布式环境搭建和测试](https://blog.csdn.net/u010366748/article/details/82843454)
[ubuntu16.04安装伪分布式Hadoop2.9.1](https://blog.csdn.net/y12345678904/article/details/80743333)
[问题root@localhost's password:localhost:permission denied,please try again](https://www.cnblogs.com/hmy-blog/p/6500909.html)