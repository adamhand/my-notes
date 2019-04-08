# PacificA：微软设计的分布式存储框架
---

## 分布式存储

## PacificA特点，框架

系统框架主要有两部分组成： **存储集群** 和 **配置管理集群** 。

- 存储集群：负责系统数据的读取和更新，通过使用多副本方式保证数据可靠性和可用性；
- 配置管理集群：维护副本信息，包括副本的参与节点，主副本节点，当前副本的版本等。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/pacifica-a.jpg">
</center>


- 主从复制。
分片的多个副本中存在一个主副本Primary和多个从副本Secondary。所有的数据更新和读取都进入主副本，当主副本出现故障无法访问时系统会从其他从副本中选择合适的节点作为新的主。
- paxos算法管理集群配置。
paxos集群维护存储集群的节点配置信息。有三种情况：
    - 从节点离线。如果主节点在一定时间内（lease period）未收到从节点对心跳的回应，那主节点认为从节点异常，于是向配置管理服务汇报更新复制集拓扑，将该异常从节点从复制集中移除，同时，它也将自己降级不再作为主；
    - 主节点离线。如果从节点在一定时间内（grace period）未收到主节点的心跳信息，那么其认为主节点异常，于是向配置管理服务汇报回信复制集拓扑，将主节点从复制集中移除，同时将自己提升为新的主。
    - 增加新节点，可能是因为原来离线的节点又重新上线了，此时主节点向配置管理服务汇报最新的拓扑，拓扑中加上该新增节点。

## kafka
kafka的整体结构实际上是PacificA架构的一种实现。kafka的整体结构如下所示。

- brocker。即存储节点，对应PacificA中的Node。
- topic。可以看成消息的集合，对应上面PacificA中的data。不过为了防止topic太大，kafka将其进行切片处理，即分成了不同的part。
- zookeeper。zookeeper的作用类似于上面的paxos集群。zookeeper中的一致性算法为ZAB(zookeeper atomic broadcast)，和paxos有相似之处。

所以，整个kafka的结构其实就是PacificA架构的一个实现。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/kafka.jpg">
</center>

---

[PacificA: Replication in Log-Based Distributed Storage Systems](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D66814)
[PacificA：微软设计的分布式存储框架](https://www.colabug.com/225369.html)

---



