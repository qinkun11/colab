###复制的常用拓扑结构

**复制的体系结构有以下一些基本原则：**

(1) 每个slave只能有一个master；
(2) 每个slave只能有一个唯一的服务器ID；
(3) 每个master可以有很多slave；
(4) 如果你设置log_slave_updates，slave可以是其它slave的master，从而扩散master的更新。



MySQL不支持多主服务器复制(Multimaster Replication)——即一个slave可以有多个master。但是，通过一些简单的组合，我们却可以建立灵活而强大的复制体系结构。



####单一master和多slave

由一个master和一个slave组成复制系统是最简单的情况。Slave之间并不相互通信，只能与master进行通信。如下：



![img](https://mmbiz.qpic.cn/mmbiz/NVvB3l3e9aEvbfOficKKBWQafhXAUNG1QJpo9iboiaRDNIlA1EiaQpgnK7iajErg0zG9vSTb0UAx17OOh5tjUV7StHg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



如果写操作较少，而读操作很时，可以采取这种结构。你可以将读操作分布到其它的slave，从而减小master的压力。但是，当slave增加到一定数量时，slave对master的负载以及网络带宽都会成为一个严重的问题。



这种结构虽然简单，但是，它却非常灵活，足够满足大多数应用需求。一些建议：



(1) 不同的slave扮演不同的作用(例如使用不同的索引，或者不同的存储引擎)；
(2) 用一个slave作为备用master，只进行复制；
(3) 用一个远程的slave，用于灾难恢复；



####主动模式的Master-Master(Master-Master in Active-Active Mode)



Master-Master复制的两台服务器，既是master，又是另一台服务器的slave。如图：



![img](https://mmbiz.qpic.cn/mmbiz/NVvB3l3e9aEvbfOficKKBWQafhXAUNG1QVgYwFX5lzuaeSCibrmoI6C255VkeLg3NDCwedE7K32uWrUicibh8NQm0A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



主动的Master-Master复制有一些特殊的用处。例如，地理上分布的两个部分都需要自己的可写的数据副本。这种结构最大的问题就是更新冲突。假设一个表只有一行(一列)的数据，其值为1，如果两个服务器分别同时执行如下语句：
在第一个服务器上执行：



- 

```
mysql> UPDATE tbl SET col=col + 1;
```



**在第二个服务器上执行：**

- 

```
mysql> UPDATE tbl SET col=col * 2;
```





那么结果是多少呢？一台服务器是4，另一个服务器是3，但是，这并不会产生错误。



实际上，MySQL并不支持其它一些DBMS支持的多主服务器复制(Multimaster Replication)，这是MySQL的复制功能很大的一个限制(多主服务器的难点在于解决更新冲突)，但是，如果你实在有这种需求，你可以采用MySQL Cluster，以及将Cluster和Replication结合起来，可以建立强大的高性能的数据库平台。但是，可以通过其它一些方式来模拟这种多主服务器的复制。



####主动-被动模式的Master-Master(Master-Master in Active-Passive Mode)


这是master-master结构变化而来的，它避免了M-M的缺点，实际上，这是一种具有容错和高可用性的系统。它的不同点在于其中一个服务只能进行只读操作。如图：





![img](https://mmbiz.qpic.cn/mmbiz/NVvB3l3e9aEvbfOficKKBWQafhXAUNG1QraQPty1LfnrrfI4890vsqYMxsia4l3bTHcWrL44kJssibItWk9VyOeog/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



####带从服务器的Master-Master结构(Master-Master with Slaves)

这种结构的优点就是提供了冗余。在地理上分布的复制结构，它不存在单一节点故障问题，而且还可以将读密集型的请求放到slave上。

![img](https://mmbiz.qpic.cn/mmbiz/NVvB3l3e9aEvbfOficKKBWQafhXAUNG1QxdDXeWTIaB22Sv2daibRO1XkjdWSAfAqvSf4OpNjYPrBf7qIXiaAblVQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)





1