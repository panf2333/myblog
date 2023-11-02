# article

poaxs 和 zab https://xie.infoq.cn/article/c8b1db19b9ec6dbb0b9764c3e

## 带你彻底认识 Paxos 算法、Zab 协议和 Raft 协议的原理和本质

* 2021-09-06
*   本文字数：9960 字

    阅读完需：约 33 分钟

#### Paxo 算法指南

**Paxos 算法的背景**

> 【Paxos 算法】是莱斯利·兰伯特(Leslie Lamport)**1990 年提出的一种基于消息传递的一致性算法，是目前公认的解决分布式一致性问题最有效的算法之一**，其解决的问题就是在分布式系统中如何就某个值（决议）达成一致。

**Paxos 算法的前提**

> **Paxos 算法的前提假设是不存在拜占庭将军问题，即：信道是安全的（信道可靠），发出的信号不会被篡改。**

**Paxos 算法的介绍**

**在 Paxos 算法中，有三种角色：**

\


![](https://static001.geekbang.org/infoq/ee/ee86794f3cd2f752c9dff62c77a37b2b.png)

\


* proposer：提案 Proposal 提出者。
* Acceptor：决策者，可以批准议案。
* Learner：最终决策的学习者。

\


![](https://static001.geekbang.org/infoq/3d/3d0c12fb673f4609cdaee96cc2edfe1c.png)

\


**Paxos 算法安全性前提如下：**

* 只有被提出的 value 才能被选定。
* 只有一个 value 被选定。
* 如果某个进程认为某个 value 被选定了，那么这个 value 必须是真的被选定的那个。

**Paxos 算法的过程描述：**

> **Paxos 算法类似于两阶段提提交，其算法执行过程分为两个阶段。具体如下**

prepare 阶段

* **proposer 提出一个编号为 N 的 proposal，发送给半数以上的 acceptor**。
* **acceptor 收到编号为 N 的 prepare 请求后:**如果小于它已经响应过的请求，则拒绝回复或者回复 error；如果 N 大于该 Acceptor 已经响应过的所有 Prepare 请求的编号，那么它就会将它已经接受过（已经经过第二阶段 accept 的提案）的编号最大的提案（如果有的话，如果还没有的 accept 提案的话返回{pok，null，null}）作为响应反馈给 Proposer，同时存储更新本地对应的提案编号，并且该 Acceptor 承诺不再接受任何编号小于 N 的提案。**分为两种情况**：**如果 acceptor 已经接受过提案，返回接受提案的最大 value；如果还没有接受过提案，就返回{pok，null，null})**

accept 阶段

* **如果 proposer 收到半数以上的 acceptor 回复的编号为 N 的提案的 prepare 响应，那么会发送一个针对\[N,V]提案的 Accept 请求给半数以上的 Acceptor。**

> **注意：V 就是收到的响应中编号最大的提案的 value。**

* **如果响应中不包含任何提案，那么 V 就由 Proposer 自己决定，可以是任意值。**
* **如果 Acceptor 收到一个针对编号为 N 的提案的 Accept 请求，只要该 Acceptor 没有对编号大于 N 的 Prepare 请求做出过响应，它就接受该提案。**
* **如果 N 小于 Acceptor 以及响应的 prepare 请求，则拒绝，不回应或回复 error（当 proposer 没有收到过半的回应，那么他会重新进入第一阶段，递增提案号，重新提出 prepare 请求）。**

> 具体如下图所示：\
>

![](https://static001.geekbang.org/infoq/81/81167916de02baecd8426166d9c6e03a.png)

> \
>

过半的 acceptor 都接受提案后，learner 会自动感知到，并开始学习提案。（同一个进程可以同时扮演多个角色）

**Learner 的学习过程**

**learner 学习过程包含两种场景：**

* Learner 所在节点参与了提案选举，Learner 需要知道其接受(accept)的提案值是否被选中(chosen)。
* Learner 所在节点已落后于其他节点，Learner 需要选择合适的策略快速完成追赶，并重新参与到提案选举当中。

选中通知

* 本节点 Proposer 的某个提案被选中(chosen)时，通过(MsgType\_PaxosLearner\_ProposerSendSuccess)消息通知到各个节点。
* 正常情况下，所有节点处于 online 状态，共同参与 paxos 选举。因此为了避免 instance id 冲突，paxos 建议只由主节点的 proposer 发起提案，这样保证接受提案和习得提案编号一致。
* 此时，Learn 习得的提案值实际上就是本节点 Accept 的数据，因此 learner 只更新内存状态即可，无需再次落盘(acceptor 已落盘)。
* **最后，如果存在 follower 节点，数据同步到 follower(follower 节点不参与 paxos 算法，相当于某个 paxos 节点的同步备)。**

提案值追赶

* **一旦节点处于落后状态，它无法再参与到 paxos 提案选举中来。这时需要由 learner 发起主动学习完成追赶。**
* **Paxos 启动时，启动 learner 定时器，定时发送 learn 请求到各个节点，发送请求携带本节点的 Instance ID、Node ID 信息。各节点收到该请求后，回复数据自主完成学习过程。**

\


![](https://static001.geekbang.org/infoq/2e/2e3bb512e8aea82383d75d9aea6971e4.png)

\


数据各节点保持数据一致性

\


![](https://static001.geekbang.org/infoq/ab/ab7a7811193f5e9a0f22932c99b76088.png)

\


**为什么要有两段提交**

* 一方面，第一次预提交后可能被告知已经有观点了，此时他不应该提出自己的观点，而应该尽快收敛，支持最新的观点。
* 另一方面，进行预加锁。

**怎么保证 Proposal 编号唯一**

* **假设有 K 台 Server 运行 paxos 算法，那么他们初始编号为 0…k-1。以后编号每次增加 k，从而保证全局唯一递增。**
* **正式提案被半数以上 Acceptor 接受后，就可以确定最终被接受的提案就是该观点。**
* **两个半数以上的集合的一定存在交集。**

**Paxos 算法有活锁问题存在。**

介绍了 Paxos 的算法逻辑，但在算法运行过程中，可能还会存在一种极端情况，当有两个 proposer 依次提出一系列编号递增的议案，那么会陷入死循环，无法完成第二阶段，也就是无法选定一个提案。如下图：\


![](https://static001.geekbang.org/infoq/73/73c1123169c6b8ca4f47af571dec6d2c.png)

\


**Paxos 算法的过半依据**

* 在 Paxos 算法中，采用了“过半”理念，也就是少数服从多数，这使 Paxos 算法具有很好的容错性。那么为什么采用过半就可以呢？
* Paxos 基于的过半数学原理： 我们称大多数（过半）进程组成的集合为法定集合， 两个法定（过半）集合必然存在非空交集，即至少有一个公共进程，称为法定集合性质。 例如 A,B,C,D,F 进程组成的全集，法定集合 Q1 包括进程 A,B,C，Q2 包括进程 B,C,D，那么 Q1 和 Q2 的交集必然不在空，C 就是 Q1，Q2 的公共进程。如果要说 Paxos 最根本的原理是什么，那么就是这个简单性质。也就是说：两个过半的集合必然存在交集，也就是肯定是相等的，也就是肯定达成了一致。
* Paxos 是基于消息传递的具有高度容错性的分布式一致性算法。Paxos 算法引入了过半的概念，解决了 2PC，3PC 的太过保守的缺点，且使算法具有了很好的容错性，另外 Paxos 算法支持分布式节点角色之间的轮换，这极大避免了分布式单点的出现，因此 Paxos 算法既解决了无限等待问题，也解决了脑裂问题，是目前来说最优秀的分布式一致性算法。其中，Zookeeper 的 ZAB 算法和 Raft 一致性算法都是基于 Paxos 的。在后边的文章中，我会逐步介绍优秀的分布式协调服务框架，也是极优秀的工业一致性算法的实现 Zookeeper 使用和实现。

***

#### ZAB 算法指南

> **Zookeeper 采用的 ZAB 协议也是基于 Paxos 算法实现的，不过 ZAB 对 Paxos 进行了很多改进与优化，两者的设计目标也存在差异——ZAB 协议主要用于构建一个高可用的分布式数据主备系统，而 Paxos 算法则是用于构建一个分布式的一致性状态机系统。**
>
> **ZAB 协议全称就是 ZooKeeper Atomic Broadcast protocol，是 ZooKeeper 用来实现一致性的算法，分成如下 3 个阶段**：

* fast leader election：快速选举阶段
* recovery：恢复阶段 Discovery；Synchrozation
* broadcasting：广播阶段

**先了解一些基本概念**

* electionEpoch：**每执行一次 leader 选举，electionEpoch 就会自增，用来标记 leader 选举的轮次**
* peerEpoch：**每次 leader 选举完成之后，都会选举出一个新的 peerEpoch，用来标记事务请求所属的轮次**
* zxid：**事务请求的唯一标记，由 leader 服务器负责进行分配高 32 位是上述的 peerEpoch，低 32 位是请求的计数，从 0 开始。**
* lastprocessZxid：最后一次 commit 的事务请求的 zxid
* LinkedList committedLog、long maxCommittedLog、long minCommittedLog：ZooKeeper 会保存最近一段时间内执行的事务请求议案，个数限制默认为 500 个议案。committedLog 就是用来保存议案的列表，maxCommittedLog 表示最大议案的 zxid，minCommittedLog 表示 committedLog 中最小议案的 zxid。
* ConcurrentMap\<Long, Proposal> outstandingProposals：Leader 拥有的属性，每当提出一个议案，都会将该议案存放至 outstandingProposals，一旦议案被过半认同了，就要提交该议案，则从 outstandingProposals 中删除该议案。
* ConcurrentLinkedQueue toBeApplied：Leader 拥有的属性，每当准备提交一个议案，就会将该议案存放至该列表中，一旦议案应用到 ZooKeeper 的内存树中了，然后就可以将该议案从 toBeApplied 中删除。
* state：当前服务器的状态
* recvQueue：消息接收队列，用于存放那些从其他服务器接收到的消息。
* queueSendMap：消息发送队列，用于保存那些待发送的消息，按照 SID 进行分组。
* senderWorkerMap：发送器集合，每个 SenderWorker 消息发送器，都对应一台远程 Zookeeper 服务器，负责消息的发送，也按照 SID 进行分组。
* lastMessageSent：最近发送过的消息，为每个 SID 保留最近发送过的一个消息。

**在 ZAB 协议中，服务的的状态 state 有四种：**

* LOOKING：进入 leader 选举状态
* FOLLOWING：leader 选举结束，进入 follower 状态
* LEADING：leader 选举结束，进入 leader 状态
* OBSERVING：处于观察者状态

**协议算法的具体描述**

**Broadcasting 过程**

1. leader 针对客户端的事务请求，创造出一个提案，zxid 由 leader 决定，并将该提案的 zxid，提案放到 outstandingProposals Map 中。
2. leader 向所有的 follower 发送该提案，如果过半的 follower 回复 OK 的话，则 leader 认为可以提交该议案，则将该议案从 outstandingProposals 中删除。
3. 然后存放到 toBeApplied 中 leader 对该议案进行提交，会向所有的 follower 发送提交该议案的命令，leader 自己也开始执行提交过程，会将该请求的内容应用到 ZooKeeper 的内存树中。
4. 然后更新 lastProcessedZxid 为该请求的 zxid，同时将该请求的议案存放到上述 committedLog，同时更新 maxCommittedLog 和 minCommittedLog。
5. leader 回复客户端，并将提案中 ToBeApplied 中删除

**fast leader election 过程**

> 选举过程关注两个要点：刚启动时进行 leader 选举和选举完 leader 后，刚启动的 server 怎么感知到 leader，投票过程有两个比较重要的数据：

* HashMap\<Long, Vote> recvset：用于收集 LOOKING、FOLLOWING、LEADING 状态下的 server 的投票
* HashMap\<Long, Vote> outofelection：用于收集 FOLLOWING、LEADING 状态下的 server 的投票（说明 leader 选举已经完成）

具体的过程有：

1. 服务器先自增 electionEpoch，给自己投票：

* 从快照日志和事务日志中加载数据，得到本机器的内存树数据，以及 lastProcessedZxid。投票内容为：proposedLeader：server 自身的 myid 值，初始为本机器的 idproposedZxid：最大事务 zxid，初始为本机器的 lastProcessedZxidproposedEpoch:peerEpoch 值，由上述的 lastProcessedZxid 的高 32 得到然后向所有的服务器发送投票。

1.  server 接收到投票通知后，进行 PK。

    如果收到的通知中的 electionEpoch 比自己的大，则更新自己的 electionEpoch 为 serverA 的 electionEpoch；

    如果、收到的通知中的 electionEpoch 比自己的小，则向 serverA 发送一个通知，将自己的投票以及 electionEpoch 发送给 serverA，serverA 收到后就会更新自己的 electionEpoch。

    如果 electionEpoch 相同，PK 的规则是 **proposedZxid**，然后再是 **myId**。
2.  根据 server 的状态来判定 leader

    **如果当前发来的投票的 server 的状态是 LOOKING 状态，则只需要判断本机器的投票是否在 recvset 中过半了，如果过半了则说明 leader 选举就算成功了，如果当前 server 的 id 等于上述过半投票的 proposedLeader,则说明自己将成为了 leader，否则自己将成为了 follower。**

    **如果当前发来的投票的 server 的状态是 FOLLOWING、LEADING 状态，则说明 leader 选举过程已经完成了，则发过来的投票就是 leader 的信息，这里就需要判断发过来的投票是否在 recvset 或者 outofelection 中过半了，同时还要检查 leader 是否给自己发送过投票信息，从投票信息中确认该 leader 是不是 LEADING 状态。**

**Recovery 过程**

> **一旦 leader 选举完成，就开始进入恢复阶段，就是 follower 要同步 leader 上的数据信息。**

1. 通信初始化

> **leader 会创建一个 ServerSocket，接收 follower 的连接，leader 会为每一个连接会用一个 LearnerHandler 线程来进行服务；**

1.  重新为 peerEpoch 选举出一个新的 peerEpoch

    follower 会向 leader 发送一个 Leader，FOLLOWERINFO 信息，包含自己的 peerEpoch 信息。

    leader 的 LearnerHandler 会获取到上述 peerEpoch 信息，从中选出一个最大的 peerEpoch，然后加 1 作为新的 peerEpoch。

    然后 leader 的所有 LearnerHandler 会向各自的 follower 发送一个 Leader.LEADERINFO 信息，包含上述新的 peerEpoch；

    follower 会使用上述 peerEpoch 来更新自己的 peerEpoch，同时将自己的 lastProcessedZxid 发给 leader，leader 的根据这个 lastProcessedZxid 和 leader 的 lastProcessedZxid 之间的差异进行同步。
2.  已经处理的事务议案的同步

    判断 LearnerHandler 中的 lastProcessedZxid 是否在 minCommittedLog 和 maxCommittedLog 之间

    LearnerHandler 中的 lastProcessedZxid 和 leader 的 lastProcessedZxid 一致，则说明已经保持同步了

    如果 lastProcessedZxid 在 minCommittedLog 和 maxCommittedLog 之间，从 lastProcessedZxid 开始到 maxCommittedLog 结束的这部分议案，重新发送给该 LearnerHandler 对应的 follower，同时发送对应议案的 commit 命令。

> 上述可能存在一个问题：即 lastProcessedZxid 虽然在他们之间，但是并没有找到 lastProcessedZxid 对应的议案，即这个 zxid 是 leader 所没有的，此时的策略就是完全按照 leader 来同步，删除该 follower 这一部分的事务日志，然后重新发送这一部分的议案，并提交这些议案。

* 如果 lastProcessedZxid 大于 maxCommittedLog，则删除该 follower 大于部分的事务日志
* 如果 lastProcessedZxid 小于 minCommittedLog，则直接采用快照的方式来恢复。

1.  未处理的事务议案的同步

    **LearnerHandler 还会从 leader 的 toBeApplied 数据中将大于该 LearnerHandler 中的 lastProcessedZxid 的议案进行发送和提交（toBeApplied 是已经被确认为提交的）**

    **LearnerHandler 还会从 leader 的 outstandingProposals 中大于该 LearnerHandler 中的 lastProcessedZxid 的议案进行发送，但是不提交（outstandingProposals 是还没被被确认为提交的）**
2. 将 LearnerHandler 加入到正式 follower 列表中
3.  LearnerHandler 发送 Leader.NEWLEADER 以及 Leader.UPTODATE 命令。

    leader 开始进入心跳检测过程，不断向 follower 发送心跳命令，不断检是否有过半机器进行了心跳回复，如果没有过半，则执行关闭操作，开始进入 leader 选举状态；

    LearnerHandler 向对应的 follower 发送 Leader.UPTODATE，follower 接收到之后，开始和 leader 进入 Broadcast 处理过程。

**事务持久化和恢复过程**

* 事务持久化分为：broadcasting 持久化和 leader shutdown 过程的持久化。leader 针对每次事务请求都会生成一个议案，然后向所有的 follower 发送该议案。follower 收到提案后，将该议案记录到事务日志中，每当记满 100000 个（默认），则事务日志执行 flush 操作，同时开启一个新的文件来记录事务日志同时会执行内存树的快照，snapshot.\[lastProcessedZxid]作为文件名创建一个新文件，快照内容保存到该文件中一旦 leader 过半的心跳检测失败，则执行 shutdown 方法，在该 shutdown 中会对事务日志进行 flush 操作

**事务的恢复分为快照恢复和日志恢复。**

* 事务快照的恢复：**会在事务快照文件目录下找到最近的 100 个快照文件，并排序，最新的在前；对上述快照文件依次进行恢复和验证，一旦验证成功则退出，否则利用下一个快照文件进行恢复。恢复完成更新最新的 lastProcessedZxid；**
* 事务日志的恢复：**从事务日志文件目录下找到 zxid 大于等于上述 lastProcessedZxid 的事务日志，然后对上述事务日志进行遍历，应用到 ZooKeeper 的内存树中，同时更新 lastProcessedZxid，同时将上述事务日志存储到 committedLog 中，并更新 maxCommittedLog、minCommittedLog**

***

#### Raft 算法指南

**Raft 背景**

> **在分布式系统中，一致性算法至关重要。在所有一致性算法中，Paxos 最负盛名，它由莱斯利·兰伯特（Leslie Lamport）于 1990 年提出，是一种基于消息传递的一致性算法，被认为是类似算法中最有效的。**
>
> **Paxos 算法虽然很有效，但复杂的原理使它实现起来非常困难，截止目前，实现 Paxos 算法的开源软件很少，比较出名的有 Chubby、LibPaxos。**

***

* 由于 Paxos 算法过于复杂、实现困难，极大地制约了其应用，而分布式系统领域又亟需一种高效而易于实现的分布式一致性算法，在此背景下，Raft 算法应运而生。
* Raft 是一个共识算法（consensus algorithm），所谓共识，就是多个节点对某个事情达成一致的看法，即使是在部分节点故障、网络延时、网络分割的情况下。
* 共识算法的实现一般是基于复制状态机（Replicated state machines），何为复制状态机：简单来说：相同的初识状态 + 相同的输入 = 相同的结束状态。

**Raft 角色**

> **一个 Raft 集群包含若干个节点，这些节点分为三种状态：Leader、 Follower、Candidate，每种状态负责的任务也是不一样的。正常情况下，集群中的节点只存在 Leader 与 Follower 两种状态。**

* leader：负责日志的同步管理，处理来自客户端的请求，与 Follower 保持 heartBeat 的联系；
* follower：响应 Leader 的日志同步请求，响应 Candidate 的邀票请求，以及把客户端请求到 Follower 的事务转发（重定向）给 Leader；
* candidate：负责选举投票，集群刚启动或者 Leader 宕机时，状态为 Follower 的节点将转为 Candidate 并发起选举，选举胜出（获得超过半数节点的投票）后，从 Candidate 转为 Leader 状态。

> 还有一个关键概念：term（任期）。以选举（election）开始，每一次选举 term 都会自增，充当了逻辑时钟的作用。

**Raft 的 3 个子问题**

为简化逻辑和实现，Raft 将一致性问题分解成了三个相对独立的子问题。

* 选举（Leader Election）：当 Leader 宕机或者集群初创时，一个新的 Leader 需要被选举出来；
* 日志复制（Log Replication）：Leader 接收来自客户端的请求并将其以日志条目的形式复制到集群中的其它节点，并且强制要求其它节点的日志和自己保持一致；
* 安全性（Safety）：如果有任何的服务器节点已经应用了一个确定的日志条目到它的状态机中，那么其它服务器节点不能在同一个日志索引位置应用一个不同的指令。

**选举过程**

> **如果 follower 在 election timeout 内没有收到来自 leader 的心跳，则会主动发起选举。**

**第一阶段：所有节点都是 Follower。**

> **Raft 集群在刚启动（或 Leader 宕机）时，所有节点的状态都是 Follower，初始 Term（任期）为 0。同时启动选举定时器，每个节点的选举定时器超时时间都在 100\~500 毫秒之间且并不一致。**

\


![](https://static001.geekbang.org/infoq/27/27954fc05d457ef3b7f527d3e3be4314.png)

\


**第二阶段：Follower 转为 Candidate 并发起投票。**

> **没有 leader 后，followers 状态自动转为 candidate，并向集群中所有节点发送投票请求并且重置选举定时器。**

投票过程有：

* 增加节点本地的 current term ，切换到 candidate 状态
* 投自己一票，并行给其他节点发送 RequestVote RPCs
* 等待其他节点的回复，可能出现三种结果：收到 majority 的投票（含自己的一票），则赢得选举，成为 leader 被告知别人已当选，那么自行切换到 follower 一段时间内没有收到 majority 投票，则保持 candidate 状态，重新发出选举

**第三阶段：投票策略**

投票的约束条件有：

* 在一个 term 内，一个节点只允许发出一次投票；
* 候选人知道的信息不能比自己的少（这一部分，后面介绍 log replication 和 safety 的时候会详细介绍）
* first-come-first-served 先来先得
* 如果参加选举的节点是偶数个，raft 通过 randomized election timeouts 来尽量避免平票情况，也要求节点的数目都是奇数个，尽量保证 majority 的出现。

**log Replication 原理**

* 当 leader 选举成功后，客户端所有的请求都交给了 leader，leader 调度请求的顺序性和 followers 的状态一致性。
* 在集群中，所有的节点都可能变为 leader，为了保证后续 leader 节点变化后依然能够使集群对外保持一致，需要通过 Log Replication 机制来解决如下两个问题：
* Follower 与 Leader 节点相同的顺序依次执行每个成功提案;
* 每个成功提交的提案必须有足够多的成功副本，来保证后续的访问一致

**第一阶段：客户端请求提交到 Leader。**

Leader 在收到 client 请求提案后，会将它作为日志条目（Entry）写入本地 log 中。需要注意的是，此时该 Entry 的状态是未提交（Uncommitted），Leader 并不会更新本地数据，因此它是不可读的。

**第二阶段：Leader 将 Entry 发送到其它 Follower**

* Leader 与 Floolwers 之间保持着心跳联系，随心跳 Leader 将追加的 Entry（AppendEntries）并行地发送给其它的 Follower，并让它们复制这条日志条目，这一过程称为复制（Replicate）。
* 为什么 Leader 向 Follower 发送的 Entry 是 AppendEntries，因为 Leader 与 Follower 的心跳是周期性的，而一个周期间 Leader 可能接收到多条客户端的请求，因此，随心跳向 Followers 发送的大概率是多个 Entry，即 AppendEntries。
* Leader 向 Followers 发送的不仅仅是追加的 Entry（AppendEntries）在发送追加日志条目的时候，Leader 会把新的日志条目紧接着之前条目的索引位置（prevLogIndex）， Leader 任期号（Term）也包含在其中。如果 Follower 在它的日志中找不到包含相同索引位置和任期号的条目，那么它就会拒绝接收新的日志条目，因为出现这种情况说明 Follower 和 Leader 不一致。
* 如何解决 Leader 与 Follower 不一致的问题，正常情况下，Leader 和 Follower 的日志保持一致。然而，Leader 和 Follower 一系列崩溃的情况会使它们的日志处于不一致状态。

有三种情况：

1. Follower 落后新的 leader，丢失一些在新的 Leader 中有的日志条目
2. Follower 领先新的 leader，有一些 Leader 没有的日志条目，
3. 或者两者都发生。丢失或者多出日志条目可能会持续多个任期。

> 要使 Follower 的日志与 Leader 恢复一致，Leader 必须找到最后两者达成一致的地方（就是回溯，找到两者最近的一致点），然后删除从那个点之后的所有日志条目，发送自己的日志给 Follower。Leader 为每一个 Follower 维护一个 nextIndex，它表示下一个需要发送给 Follower 的日志条目的索引地址。当一个 Leader 刚获得权力的时候，它初始化所有的 nextIndex 值，为自己的最后一条日志的 index 加 1。如果一个 Follower 的日志和 Leader 不一致，那么在下一次附加日志时一致性检查就会失败。在被 Follower 拒绝之后，Leader 就会减小该 Follower 对应的 nextIndex 值并进行重试。最终 nextIndex 会在某个位置使得 Leader 和 Follower 的日志达成一致。当这种情况发生，附加日志就会成功，这时就会把 Follower 冲突的日志条目全部删除并且加上 Leader 的日志。一旦附加日志成功，那么 Follower 的日志就会和 Leader 保持一致，并且在接下来的任期继续保持一致。

**第三阶段：Leader 等待 Followers 回应。**

Followers 接收到 Leader 发来的复制请求后，有两种可能的回应：

* 写入本地日志中，返回 Success；
* 一致性检查失败，拒绝写入，返回 False，原因和解决办法上面已做了详细说明。
* 当 Leader 收到大多数 Followers 的回应后，会将第一阶段写入的 Entry 标记为提交状态（Committed），并把这条日志条目应用到它的状态机中。

**第四阶段：Leader 回应客户端。**

完成前三个阶段后，Leader 会向客户端回应 OK，表示写操作成功。

**第五阶段，Leader 通知 Followers Entry 已提交**

Leader 回应客户端后，将随着下一个心跳通知 Followers，Followers 收到通知后也会将 Entry 标记为提交状态。至此，Raft 集群超过半数节点已经达到一致状态，可以确保强一致性。

**raft safety 保证**

1） election safety: 在一个 term 内，至多有一个 leader 被选举出来。raft 算法通过

一个节点某一任期内最多只能投一票；只有获得 majority 投票的节点才会成为 leader。2）log matching：说如果两个节点上的某个 log entry 的 log index 相同且 term 相同，那么在该 index 之前的所有 log entry 应该都是相同的。leader 在某一 term 的任一位置只会创建一个 log entry，且 log entry 是 append-only。

3）consistency check。leader 在 AppendEntries 中包含最新 log entry 之前的一个 log 的 term 和 index，如果 follower 在对应的 term index 找不到日志，那么就会告知 leader 不一致。当出现了 leader 与 follower 不一致的情况，leader 强制 follower 复制自己的 log。

3）leader completeness ：如果一个 log entry 在某个任期被提交（committed），那么这条日志一定会出现在所有更高 term 的 leader 的日志里面。

一个日志被复制到 majority 节点才算 committed 一个节点得到 majority 的投票才能成为 leader，而节点 A 给节点 B 投票的其中一个前提是，B 的日志不能比 A 的日志旧。4）stale leader: 落后的 leader，但在网络分割（network partition）的情况下，可能会出现两个 leader，但两个 leader 所处的任期是不同的。而在 raft 的一些实现或者 raft-like 协议中，leader 如果收不到 majority 节点的消息，那么可以自己 step down，自行转换到 follower 状态。

5）leader crash：新的节点成为 Leader，为了不让数据丢失，希望新 Leader 包含所有已经 Commit 的 Entry。为了避免数据从 Follower 到 Leader 的反向流动带来的复杂性，Raft 限制新 Leader 一定是当前 Log 最新的节点，即其拥有最多最大 term 的 Log Entry。

6）State Machine Safety

某个 leader 选举成功之后，不会直接提交前任 leader 时期的日志，而是通过提交当前任期的日志的时候“顺手”把之前的日志也提交了，具体的实现是：如果 leader 被选举后没有收到客户端的请求呢，论文中有提到，在任期开始的时候发立即尝试复制、提交一条空的 log。

总结：raft 将共识问题分解成两个相对独立的问题，leader election，log replication。流程是先选举出 leader，然后 leader 负责复制、提交 log（log 中包含 command）

**log replication 约束：**

一个 log 被复制到大多数节点，就是 committed，保证不会回滚 leader 一定包含最新的 committed log，因此 leader 只会追加日志，不会删除覆盖日志不同节点，某个位置上日志相同，那么这个位置之前的所有日志一定是相同的 Raft never commits log entries from previous terms by counting replicas.

\


> **本文作者：** [李浩宇/Alex](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.cnblogs.com%2Fliboware)\
>
>
> **本文链接：** [https://www.cnblogs.com/liboware/p/15229621.html](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.cnblogs.com%2Fliboware%2Fp%2F15229621.html)\
>

__划线__评论__复制发布于: 2021-09-06阅读数: 1702如对本文有异议，可点此反馈[Java](https://xie.infoq.cn/tag/3)[架构](https://xie.infoq.cn/tag/238)[面试](https://xie.infoq.cn/tag/283)[分布式](https://xie.infoq.cn/tag/289)[计算机](https://xie.infoq.cn/tag/1810)

### 评论

暂无评论
