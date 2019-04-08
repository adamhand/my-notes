# Windows下安装RabbitMQ
---

# 环境
- Windows10
- Erlang 19.3
- Rabbitmq-server-3.6.5

# 安装Erlang
下载地址：http://www.erlang.org/downloads

选择下载win64版19.3版本（**注意版本，后面会说到一个问题，是由于版本引起的**），下载后**以管理员身份**双击exe进行安装，**安装路径不要有中文或空格**。

配置环境变量：

- `ERLANG_HOME` = erlang的安装目录
- 在path中增加`“%ERLANG_HOME%\bin”`
- 在命令行中输入“erl”,如果显示版本号就证明安装成功。

# 安装RabbitMQ
下载地址：http://www.rabbitmq.com/

选择下载windows版本3.5.6，下载后双击exe进行安装。**不要安装在带空格和汉字的目录下**

# 安装界面管理插件
打开命令行，进行入RabbitMQ安装目录下的sbin目录。

输入：`rabbitmq-plugins enable rabbitmq_management`命令安装插件。如果出现以下界面，证明安装成功：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/rabbitmqinstallsucc.PNG">
</center>

这个插件是为了有一个界面，更好地对RabbitMQ集群进行管理。有了这个插件，就可以访问`localhost:15672`，下面会说到。

# 安装服务
如果想以服务的方式启动RabbitMQ，需要使用`rabbitmq-service install`命令安装服务。然后使用`rabbitmq-service start`命令启动服务。

其他服务相关命令如下：

- rabbitmq-service stop 停止
- rabbitmq-service remove 移除

如果不想以服务的形式启动，进入RabbitMQ安装目录的sbin目录，启动`rabbitmq-server.bat`即可。

启动之后，应该可以看到如下界面：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/rabbitmqinstall.PNG">
</center>

这时，打开`localhost:15672`，应该可以看到如下界面。用户名和密码默认是`guest`，可以登录进去查看当前RabbitMQ管理界面。如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/rabbitmqinstall1.PNG">
</center>

# 添加用户并设置权限
首先打开RabbitMQ server，然后添加新用户，用户名和密码均为root。

添加新用户的命令为：
```
rabbitmqctl.bat add_user root root
```
为用户设置所有权限：
```
rabbitmqctl.bat set_permissions -p / root ".*" ".*" ".*"
```
为新用户设置身份为管理员：
```
rabbitmqctl.bat set_user_tags root administrator
```

然后使用`rabbitmqctl.bat list_users`命令查看用户是否被添加成功，如果成功会有如下显示：
```
Listing users ...
guest   [administrator]
root    [administrator]
```

# 安装时遇到的问题
在第一次安装完成之后，遇到了以下问题：
```
BOOT FAILED
===========

Error description:
   noproc

Log files (may contain more information):
   C:/Users/adaih/AppData/Roaming/RabbitMQ/log/RABBIT~1.LOG
   C:/Users/adaih/AppData/Roaming/RabbitMQ/log/RABBIT~2.LOG

Stack trace:
   [{gen,do_for_proc,2,[{file,"gen.erl"},{line,228}]},
    {gen_event,rpc,2,[{file,"gen_event.erl"},{line,239}]},
    {rabbit,ensure_working_log_handlers,0,
            [{file,"src/rabbit.erl"},{line,684}]},
    {rabbit,'-boot/0-fun-0-',0,[{file,"src/rabbit.erl"},{line,268}]},
    {rabbit,start_it,1,[{file,"src/rabbit.erl"},{line,403}]},
    {init,start_em,1,[{file,"init.erl"},{line,1111}]},
    {init,do_boot,3,[{file,"init.erl"},{line,819}]}]

=INFO REPORT==== 1-Jan-2019::19:05:50.526000 ===
Error description:
   noproc

Log files (may contain more information):
   C:/Users/adaih/AppData/Roaming/RabbitMQ/log/RABBIT~1.LOG
   C:/Users/adaih/AppData/Roaming/RabbitMQ/log/RABBIT~2.LOG

Stack trace:
   [{gen,do_for_proc,2,[{file,"gen.erl"},{line,228}]},
    {gen_event,rpc,2,[{file,"gen_event.erl"},{line,239}]},
    {rabbit,ensure_working_log_handlers,0,
            [{file,"src/rabbit.erl"},{line,684}]},
    {rabbit,'-boot/0-fun-0-',0,[{file,"src/rabbit.erl"},{line,268}]},
    {rabbit,start_it,1,[{file,"src/rabbit.erl"},{line,403}]},
    {init,start_em,1,[{file,"init.erl"},{line,1111}]},
    {init,do_boot,3,[{file,"init.erl"},{line,819}]}]


{"init terminating in do_boot",noproc}
init terminating in do_boot (noproc)

Crash dump is being written to: erl_crash.dump...done
```
按照提示，在`C:/Users/adaih/AppData/Roaming/RabbitMQ/log/RABBIT~1.LOG`里也没找到任何错误日志。查了百度，可能的原因有如下几个：

- RabbitMQ安装路径中有空格。这个有点坑，因为默认的安装路径中就有个空格。
- RabbitMQ和Erlang版本不对应。这是安装的Erlang版本是20*，RabbitMQ版本为3.6.5。
- 安装Erlang时必须使用管理员权限。

于是按照这几个解决方法试一试，首先重新安装了RabbitMQ，在安装时将安装路径中的空格去掉了，未果。

然后将Erlang卸载掉，并删除注册表`HKEY_LOCAL_MOCHINE/SOFTWARE/Ericsson/Erlang/ErlSrv`，重新下载了19.3版本的，安装时选择管理员权限，这样才成功了。

# 参考
[RabbitMQ:windows10下安装](https://blog.csdn.net/nnsword/article/details/79544349)
[windows安装RabbitMQ时遇到问题](https://segmentfault.com/q/1010000010411645/a-1020000010584921)
[解决RabbitMQ service is already present - only updating service parameters](https://yq.aliyun.com/ziliao/494064)
[Windows环境下安装RabbitMQ，以及添加用户，设置权限](https://blog.csdn.net/coderK2014/article/details/80991890)