---
layout: mypost
title: zookeeper
categories: [java]
---
#### zookeeper命令
```
zk提供了一个多层级的节点znode命名空间，不管是叶节点还是非叶节点都可以关联数据。
create /znode1 firstZnode  # 创建持久节点znode1，数据为firstZnode
create -e /znode1/znode2 secondZnode # 创建临时节点znode2，数据为secondZnode
create -s /znode1/znodeSeq thridZnode # 创建持久顺序节点znodeSeq，节点名自动变成znodeSeq0000000001,数据为thridZnode
create -e -s /znode1/znodeSeq fourZnode # 创建持久顺序节点znodeSeq，节点名自动变成znodeSeq0000000002,数据为fourZnode
```
#### ZooKeeper三大特性
```
1、数据节点(znode，叶子节点和非叶子节点都可以存储数据)
2、Watcher机制(watcher，监控本节点和下一级子节点的变化)
3、ACL权限控制。
```

#### zookeeper的四种类型的节点
```
1. 持久节点。
2. 临时节点（不能拥有子节点，与客户端回话绑定，一旦会话失效就自动删除）。
3. 持久顺序节点。
4. 临时顺序节点。
```
#### zk从以下几点保证了以下分布式一致性
```
1. 顺序一致性：从同一个客户端发起的事务请求最终将会严格的按照顺序被应用到zk中去。
2. 原子性：所有事务请求的处理结果在整个zk集群所有机器的应用结果是一致的。
3. 单一视图：无论客户端连到哪一个zk服务器上，其看到的数据都是一致的。
4. 持久性：一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。
客户端的读请求可以被集群中的任何一个节点处理，
但对于写请求，会转发给leader，leader向所有节点发出写入命令，一半以上节点返回成功才会返回成功。
因此节点越多，读的吞吐量会提高，写的吞吐量会下降。
有序性是zk一个重要的特性，所有的更新都是全局有序的并带有zxid的时间戳，读请求返回的结果会带上最新的zxid。
```
#### ZAB协议【Zookeeper Atomic Broadcast （Zookeeper原子广播）】
```
1. 保证强一致性。
2.
```
#### ZAB协议（客户端、Leader、Follower）的两种模式
```
1. 广播模式
   1.1 客户端发起一个写操作请求。
   1.2 Leader服务器将客户端请求转化为事务Proposal提案，同时为每个Proposal分配一个全局id，即zxid。
   1.3 Leader服务器为每个Follower服务器分配一个单独的队列，然后将需要广播的Proposal以此放到队列中去，并且根据FIFO策略进行消息发送。
   1.4 Follower接收到Proposal后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向Leader反馈一个ACK响应消息。
   1.5 Leader接收到半数以上ACK响应后，即认为消息发送成功，可以发送commit消息。
   1.6 Leader向所有Follower广播commit消息，同时自身也会完成事务提交。Follower接收到commit消息后会将上一个事务提交。
2. 恢复模式
   2.1 选举：当Leader崩溃后，集群进入选举阶段，一般选举出zxid最大的为潜在的新Leader
   2.2 发现：Follower与潜在Leader沟通，如果发现超过半数服务器同意，则潜在的新Leader的epoch加一，新Leader诞生
   2.3 同步：集群间数据同步，保证各个节点事务一致
   2.4 广播：集群恢复到广播模式，开始接收客户端写请求
```
#### Zookeeper选举
```
1. myid：每个zk服务器都需要在数据文件夹下创建一个myid的文件，该文件此服务器在整个zk集群中唯一ID（1或2或3）
2. zxid：事务ID。为保证顺序，此zxid单调递增，高32位是Leader的epoch，每次选举出新Leader，epoch加一。低32位为序号，递增，epoch发生变化后序号重置。
3. 服务器状态：Looking、Following、Leading、Observing
4. 选票数据结构：logicClock：表示第N轮投票
               state：当前服务器的状态
               self_id：当前服务器的myid
               self_zxid：当前服务器所保存的最大zxid
               vote_id：被推举服务器的myid
               vote_zxid：被推举服务器上最大zxid
5. 选票PK（选epoch最大的，若epoch相等，选zxid最大的，若epoch和zxid相等，选择myid最大的)
   5.1 外部投票logicClock大于自己的logicClock，则将自己的logicClock及自己的选票logicClock变更为收到的logicClock
   5.2 若logicClock相等，则比较vote_zxid，如果外部投票的vote_zxid比较大，则将自己票中的vote_id和vote_zxid修改为外部票的vote_id和vote_zxid并广播出去。
   5.3 若logicClock相等，vote_zxid相等，则比较vote_id，若外部投票的vote_id较大，则将自己票的vote_id修改为外部票的vote_id并广播出去。
6. 统计选票、更新状态
   有过半服务器选举了同一个服务器为Leader，则终止投票，更新此服务器为Leader，剩余服务器为Follower
7. 集群启动选举过程
   7.0 初始投票都投自己(logicClock, self_id, self_zxid, vote_id, vote_zxid)
   7.1 第1台服务器的广播(1,1,0,1,0), 第2台服务器的广播(1,2,0,2,0), 第3台服务器的广播(1,3,0,3,0)
   7.2 此时第1台服务器的投票箱(self_id,vote_id)里为(1,1),(2,2),(3,3),
       vote_id=3最大，所以更新自己的选票为(1,3)，并更新广播出去的选票(1,1)为(1,3)
       第2台服务器的投票箱(self_id,vote_id)里为(1,1),(2,2),(3,3),
       vote_id=3最大，所以更新自己的选票为(2,3)，，并更新广播出去的选票(2,2)为(2,3)
       第3台服务器的投票箱(self_id,vote_id)里为(1,1),(2,2),(3,3)
       所以三台服务器的选票都更新成了(1,3),(2,3),(3,3)
   7.3 服务器3成了Leader，服务器1和2成了Follower
8. Follower1重启进入Looking状态要求重新选举，Leader收到选举请求后将自己的状态Leading和选票发送给Follower1，Follower2将自己的状态Following和选票发送给Follower1，Follower1确定自己的状态为Following
9. Leader宕机，Follower发起新的一轮投票投自己，此时先比较zxid，zxid大(数据新)的被选为Leader，然后广播最新数据到Follower，若zxid相同则myid大的被选为Leader

```
#### zookeeper的典型的应用场景
```
1. 数据发布/订阅(注册中心)。
    数据存储：Provider将配置信息存储在znode上
    数据获取：Consumer应用在启动是读取znode上的数据，并在该znode上注册一个数据变更watcher【getData()，getChildren()，exists()】。
    数据变更：当znode上的数据发生变化时，zk将数据变更通知发送给客户端，客户端重新读取变更后的数据。
2. 分布式锁（第一种独占锁：我们把zk上的一个znode看做一把锁，通过createznode的方式来实现。所有线程都去创建/distribute_lock节点，最终成功创建的那个线程也即获得了这把锁，用完删除即可。第二种锁控制时序：/distribute_lock已经预先存在，所有线程都在它下面创建临时顺序节点，编号最小的获得锁，每个节点watch上一个节点）。
3. 命名服务（dubbo Provider启动时将自己的URL写入/dubbo/com.x.service/providers/下）。
4. 集群管理。
   4.1.机器的加入和退出：所有节点在某个父节点下创建临时节点，并监控父节点下所有子节点的变化，如果有机器加入或退出都会收到通知。
   4.2.选举master：所有节点在某个父节点下创建临时顺序节点，编号最小的为master。
5. 分布式队列(生产者创建顺序节点，消费者消费节点并watch节点的变化)。
6. 配置管理（程序部署在不同的机器上，将配置文件放在znode的数据上，当znode的数据发生变化时，利用watcher通知各个机器）。
7. 负载均衡。
8. 分布式协调/通知。
```
#### zk权限ACL
```
1. create : 创建子节点的权限
2. read ：获取节点数据和子节点的权限。
3. write: 更新节点数据的权限。
4. delete:删除子节点的权限。
5. admin:设置节点ACL的权限。
```
#### zk的server的工作状态
```
1. Looking
2. Leading
3. Followering
4. Observing
```
#### 分布式一致性问题
分布式事务是指会涉及到操作多个数据库的事务。
```
1. 两阶段提交协议2PC：（引入一个协调者统一掌控所有节点）
   1. 投票阶段（协调者询问各参与者事务是否可以正常执行）
   1.1 参与者执行事务，但不提交事务。
   1.2 所有参与者返回成功给协调者。
   2. 提交阶段（协调者向所有参与者发起事务提交通知）
   2.1 参与者提交事务，并释放资源。
   2.2 参与者反馈事务提交结果。
2. 三阶段提交协议3PC：
   1. canCommit阶段。协调者向参与者发送can-commit请求，询问是否可以执行事务。
   1.1 各参与者返回确定信息。
   2. preCommit阶段。协调者向参与者发送pre-commit请求
   2.1 参与者执行事务，但不提交。
   2.2 参与者返回事务执行结果。
   3. doCommit阶段。协调者向参与者发送do-commit请求
   3.1 参与者收到请求，提交事务并释放资源。
   3.2 参与者返回结果给协调者。
3. Paxos算法
4. Zab协议（zookeeper）
5. Raft
```
