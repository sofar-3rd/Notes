# Raft 共识算法

**Raft 是一个管理 replicated log （复制日志）的算法**。

Raft 实现共识的机制是：

1. 共同选举出一个 leader；
2. 给予这个 leader 管理 replicated log 的完全职责，
3. Leader 接受来自客户端的 log entries，然后复制给其他节点， 并在安全（不会导致冲突）时，告诉这些节点将这些 entries 应用到它们各自的状态机。

**只有一个 leader 的设计简化了 replicated log 的管理**：

1. Leader 能决定将新的 entry 放到 log 中的什么位置，而不用咨询其他机器。
2. 数据流也是从 leader 到其他节点的简单单向方式。
3. 在 Leader 挂掉或者失联的情况下，网络会新选举一个 Leader。

基于以上机制，Raft 将共识问题分解为三个相对独立的子问题：

1. **Leader election**：当现有领袖故障时，必须选出新的领袖。
2. **Log replication**：领袖从客户端得到日志条目并在服务器集群中同步日志，使日志保持一致。
3. **Safety**：如果任何服务器将特定的日志条目应用于其状态机，则任何其他服务器都不能为相同的日志索引应用不同的命令。

## Ratf 核心知识点

* **Election Safety** (**选举安全**): 在给定任期内最多只能选举一位领导者。
* **Leader Append-Only** (**领导者只追加新条目**): 从不覆盖或删除其日志中的条目；领导者日志是只读的。
* **Log Matching** (**日志匹配**)：如果两个日志包含具有相同索引和术语的条目，那么这两份日志在该条目索引之前的所有条目都是相同的。
* **Leader Completeness** (**领导者完整性**): 如果一个日志条目在给定术语中被提交，那么在所有更高序号的术语中，领导者的日志中都会包含该条目。
* **State Machine Safety** (**状态机安全**): 如果一个服务器在给定索引上应用了一个日志条目到其状态机中，那么其他服务器将永远不会对同一索引应用不同的日志条目。

### 状态

![alt text](/MIT6824/Raft/state.png){:height="60%" width="60%"}

**所有节点的持久状态**（==处理客户端请求时，先更新这些持久状态（储存到磁盘），在响应请求==）

* `currentTerm`：**该节点已知的当前任期**（每选举一位新的领袖会使任期加一），节点启动时初始化为 0，然后单调递增。
* `votefor`：**当前任期内被投票的候选节点**，如果没有就是 none。
* `log[]`：**日志条目**，每个条目包含状态机的命令，以及该条目被领导者接收的时间（使用 term 和 index 标识，第一个 index 为 1）。

**所有节点上的易失状态**：

follower 节点在接收到 leader 的 replicate entry（日志复制） 消息后，会在将 logEntry 写入 log[]（本地日志），再向 leader 返回 commit（提交） 消息，当 leader 收到过半数 follower 的 commit 消息后，会向 follower 回复 apply（应用）消息，follower 接收到消息后，则执行日志条目中的命令。（**注意这里是先写入日志等到 leader commit 后才执行命令**）

* `commitIndex`：**最后提交的 entry 的 index**，初始化为 0，然后单调递增。
* `lastAppliedIndex`：**最后应用(apply)到状态机的命令索引(index)**，初始化为 0，单调递增。

**Leader 节点上的易失消息**（==在选举之后重新初始化==）：

* `nextIndex[]`：**为每个节点分别维护，下次 replicate entry 时使用的编号**，初始化为 leader_last_log_index + 1。
* `matchIndex[]`：**为每个节点分别维护，表示已知的、复制成功的最大 index**，初始化为 0，单调递增。

### AppendEntries RPC

![alt text](/MIT6824/Raft/appendEntry.png){:height="60%" width="60%"}

* **用途**：由 leader 发起，**用于 replicate log entries ，也用作心跳**。
* **参数**：
  * `term`：**leader 的任期编号**；
  * `leaderId`：**follower 重定向客户端时会用到**；
  * `prevLogIndex`：**上一个 log entry 的 index**；
  * `prevLogTerm`：**prevLogIndex entry 的 term**；
  * `entries[]`：需要追加到 log 的新 entry（如果是 heartbeat，那数组为空；否则出于效率考虑可能会有多个）；
  * `leaderCommit`：**leader 的 commitIndex**；
* 返回结果
  * `term`：currentTerm，leader 用来更新它自己；
  * `success`：如果 follower 包含了匹配 prevLogIndex and prevLogTerm 的 entry，则返回 true。

* **实现**：
  1. 如果 leader 的 term 小于当前 currentTerm，返回 false；
  2. 如果日志中不包含一个索引为 prevLogIndex 的条目，其条款与prevLogTerm匹配，则返回false；
  3. 如果现有条目与新条目（**index 相同但 term 不同**）发生冲突，则删除现有条目及其所有后续条目；
  4. 添加任何尚未记录在日志中的新条目；
  5. 如果 `leaderCommit` 大于 `commitIndex`，则将 `commitIndex` 设置为`leaderCommit` 和 `index of last new entry` 中的最小值；

### RequestVote RPC

![alt text](/MIT6824/Raft/requestVote.png){:height="60%" width="60%"}

* **用途**：由 candidate 发起，用于收集选票。
* **参数**：
  * `term`：候选者的任期；
  * `candidateId`：候选者 ID；
  * `lastLogIndex`：候选者最后一条日志条目的索引；
  * `lastLogTerm`：候选者最后一条日志条目的任期；
* **返回结果**：
  * `term`：当前任期，用于候选者更新自己任期；
  * `voteGranted`：候选者是否接收到投票；
* **实现**
  * 若候选人的 `term` < `currentTerm`，则投递反对票
  * 如果候选人的日志更新或者与本地日志一样新，则投递赞成票；

* **投票条件**
  * **候选人最后一条Log条目的任期号大于本地最后一条Log条目的任期号**；
  * **候选人最后一条Log条目的任期号等于本地最后一条Log条目的任期号，且候选人的Log记录长度大于等于本地Log记录的长度**

### InstallSnapshot RPC

![alt text](/MIT6824/Raft/InstallSnapshotRPC.png){:height="60%" width="60%"}

* **用途**：由 leader 调用，将快照发送给 follower，leader 总是按照顺序发送块。
* **参数**
  * `Term`：领导者的任期
  * `leaderID`：领导者的编号
  * `lastIncludedIndex`：快照包含的最后一条日志的索引
  * `lastIncludedTerm`：快照包含的最后一条日志的任期
  * `offset`：数据块于快照文件中的字节偏移量
  * `done`：是否为最后一个数据块
* **返回值**
  * `term`：追随者的当前任期
* **实现**
  1. 如果 term < currentTerm 立即返回；
  2. 如果第一个块（偏移量为0）创建一个新的快照文件；
  3. 如果 done 为 false，则回复并等待更多数据块；
  4. 保存快照文件，丢弃索引较小的任何现有或部分快照；
  5. 如果现有日志条目与快照的 `lastIncludedIndex` 具有相同的索引和术语，则保留该日志条目之后的条目并回复。
  6. 丢弃整个日志条目
  7. 使用快照内容重置状态机（并加载快照的集群配置）

### Rules for Servers

![alt text](/MIT6824/Raft/rulesforserver.png){:height="60%" width="60%"}

#### All Servers

* 如果 `commitIndex` > `lastApplied`，则自增 `lastApplied`，并执行 `log[]` 中序号为 `lastApplied` 的命令；
* 如果 RPC 的请求或回应的 `term` T > `currentTerm`，则将 `currentTerm` 设置为 T，并转化为追随者（**用于应对旧 leader 从故障中苏醒并转换为 follower**）；

#### Followers

* 响应来自 `candidates` 和 `leaders` 的 RPCs；
* 如果在 `election timeout` 时间内没有收到 `AppendEntries`，则发起新的一轮选举来获取新 leader，发起选举的 follower 自动转变为 candidate；

#### candidates

* 转换为 candidate 开始选举：
  * 自增 `currentTerm`；
  * 给自身投票；
  * 重置 `election timeout` 时间；
  * 发送 `RequestVote` 给其他候选人；

* 如果收到超过半数服务器的投票，则转换为 `leader`；
* 如果收到来自新 `leader` 的 `AppendEntries` 调用，则转化为 `follower`；
* 若 `election timeout` 超时，则再次发次新的选举；

#### leaders

* 当选后：向每个服务器发送初始空的 AppendEntries RPCs（心跳）以防止选举超时；在空闲期间重复此操作。
* 如果从客户端接收到的命令是：**将条目添加到本地日志中，在将条目应用于状态机之后做出响应**。

* 如果 follower 的 `lastLogIndex` ≥ `nextIndex` ：发送从 `nextIndex` 开始的 AppendEntries RPC消息
  * 如果成功：更新 follower 的 `nextIndex` 和 `matchIndex`;
  * 如果由于日志不一致导致 `AppendEntries` 失败：则减少 `nextIndex` 并重试。
* 如果存在一个 N，使得 N > `commitIndex`，且 `matchIndex[i]` 的多数元素大于等于 N，并且 `log[N].term` == `currentTerm`，则将 `commitIndex` 设置为 N（**在设置 commitIndex 的同时应该也会执行相应命令**）。

#### question

    为什么 leader 要等一半以上的服务器将 log entry 写入日志才执行相应命令？

这里的设计十分巧妙，将 commit 的条件设置为一半以上的服务器确认有两点好处：

* **能更快地响应客户端**，若是等待所有的 follower 确认，则需要花费更长的时间；
* **保证选举 Leader 时，拥有最新日志的 follower 能够当选**，这点是如何做到的呢？==candidate 的当选的条件是超过一半的服务器给 candidate 投了赞成票，而投递赞成票的条件是 candidate 的日志条目不能落后于投票服务器的日志条目==。**如果一名拥有旧日志的 candidate 发起选举，它是不可能获取超过一半的选票，因为一半以上拥有新日志的服务器会给其投反对票**，这就保证了 leader 所有小于 commitIndex 的命令执行之后具有持久性，不会被后来当选的新 leader 影响。

---

    为什么持久化 currentTerm 和 votedFor 而不持久化 commitIndex 和 lastApplied ？

持久化 currentTerm 和 votedFor 的原因是保证一个任期内一个 Server 仅给一名 candidate 投票，防止一个任期内出现两名 leader。

因为重启后 leader 可以通过 AppendEntries 的结果来判断哪些内容被 commit 了。

#### tips

* **leader 只能 commit 当前任期（Term）内的日志，不能提交过去任期（Term）的日志**。为了上任后同步其他 follower，论文中的做法是 leader 可以先 AppendEntries 一条空的日志；本文在 lab 中的做法为 leader 上任后广播的心跳包进行同步；
* **心跳包不仅可以保持连接，也可以携带相关信息例如 leaderCommitIndex 从而更新 follower 的 commitIndex**；以及上任时第一个发送的心跳包，可以携带初始的 LastLogIndex（leader最后一条日志的 index + 1）来与 follower 进行快速同步。
* 如果领导者发出 `AppendEntriesRPC`，并且被拒绝，但不是因为日志不一致而是因为 follower 的 `Term` > `currentTerm`，**说明该 leader 可能是故障重启的服务器，此时集群中已经选出了一位新 leader，那么该 leader 转换为 follower**。
* leader在调用 `AppendEntriesRPC` 后收到回复之间没有发生变化。
  * 一个常见的做法是设置 `matchIndex` = `nextIndex` - 1 或 `matchIndex` = `len(log)`。**这可能带来安全隐患，因为这两个值可能在 leader 发送 RPC 后已被更新**。
  * 正确的做法为设置 `matchIndex` = `prevLogIndex` + `len(entries[])` （leader 最初在 RPC 中发送的参数）。

##### [**加速日志回溯（Fast Backup）**](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/lecture-07-raft2/7.3-hui-fu-jia-su-backup-acceleration)

通过在 `AppendEntries` 的返回结果中加入 `XTerm`，`XIndex` 帮助 leader 快速找到日志同步处。

* `XTerm`：记录冲突日志条目的 Term；
* `XIndex`：记录对应任期号为 XTerm 的第一条日志条目的 Index；

**follower**：

* 如果 follower 的日志没有 `prevLogIndex`，则应该返回 `XTerm` = -1 和 `XIndex` = len(logs)。
* 如果追随者日志索引为 `prevLogIndex` 的 `term` 不匹配，则应该返回 `XTerm` = log[prevLogIndex].Term，`XIndex` = `XTerm` 的第一条索引。

**leader**：

* 收到冲突响应后，领导者应首先在其日志中搜索 `XIndex` 索引的日志条目是否为 `XTerm`。如果匹配，则设置 `nextIndex[peer]` = `XIndex` + 1 和 `matchIndex[peer]` = `XIndex`。（**更好的做法是设置为 `XTerm` 中最后一条日志的 `index` + 1**）
* 如果不匹配，则证明 leader 不含有 `XTerm` 的日志条目，则设置 `nextIndex[peer]` = `XIndex`。

