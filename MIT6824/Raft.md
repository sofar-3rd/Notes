# Raft ��ʶ�㷨

**Raft ��һ������ replicated log ��������־�����㷨**��

Raft ʵ�ֹ�ʶ�Ļ����ǣ�

1. ��ͬѡ�ٳ�һ�� leader��
2. ������� leader ���� replicated log ����ȫְ��
3. Leader �������Կͻ��˵� log entries��Ȼ���Ƹ������ڵ㣬 ���ڰ�ȫ�����ᵼ�³�ͻ��ʱ��������Щ�ڵ㽫��Щ entries Ӧ�õ����Ǹ��Ե�״̬����

**ֻ��һ�� leader ����Ƽ��� replicated log �Ĺ���**��

1. Leader �ܾ������µ� entry �ŵ� log �е�ʲôλ�ã���������ѯ����������
2. ������Ҳ�Ǵ� leader �������ڵ�ļ򵥵���ʽ��
3. �� Leader �ҵ�����ʧ��������£��������ѡ��һ�� Leader��

�������ϻ��ƣ�Raft ����ʶ����ֽ�Ϊ������Զ����������⣺

1. **Leader election**���������������ʱ������ѡ���µ����䡣
2. **Log replication**������ӿͻ��˵õ���־��Ŀ���ڷ�������Ⱥ��ͬ����־��ʹ��־����һ�¡�
3. **Safety**������κη��������ض�����־��ĿӦ������״̬�������κ�����������������Ϊ��ͬ����־����Ӧ�ò�ͬ�����

## Ratf ����֪ʶ��

* **Election Safety** (**ѡ�ٰ�ȫ**): �ڸ������������ֻ��ѡ��һλ�쵼�ߡ�
* **Leader Append-Only** (**�쵼��ֻ׷������Ŀ**): �Ӳ����ǻ�ɾ������־�е���Ŀ���쵼����־��ֻ���ġ�
* **Log Matching** (**��־ƥ��**)�����������־����������ͬ�������������Ŀ����ô��������־�ڸ���Ŀ����֮ǰ��������Ŀ������ͬ�ġ�
* **Leader Completeness** (**�쵼��������**): ���һ����־��Ŀ�ڸ��������б��ύ����ô�����и�����ŵ������У��쵼�ߵ���־�ж����������Ŀ��
* **State Machine Safety** (**״̬����ȫ**): ���һ���������ڸ���������Ӧ����һ����־��Ŀ����״̬���У���ô��������������Զ�����ͬһ����Ӧ�ò�ͬ����־��Ŀ��

### ״̬

![alt text](/MIT6824/Raft/state.png){:height="60%" width="60%"}

**���нڵ�ĳ־�״̬**��==����ͻ�������ʱ���ȸ�����Щ�־�״̬�����浽���̣�������Ӧ����==��

* `currentTerm`��**�ýڵ���֪�ĵ�ǰ����**��ÿѡ��һλ�µ������ʹ���ڼ�һ�����ڵ�����ʱ��ʼ��Ϊ 0��Ȼ�󵥵�������
* `votefor`��**��ǰ�����ڱ�ͶƱ�ĺ�ѡ�ڵ�**�����û�о��� none��
* `log[]`��**��־��Ŀ**��ÿ����Ŀ����״̬��������Լ�����Ŀ���쵼�߽��յ�ʱ�䣨ʹ�� term �� index ��ʶ����һ�� index Ϊ 1����

**���нڵ��ϵ���ʧ״̬**��

follower �ڵ��ڽ��յ� leader �� replicate entry����־���ƣ� ��Ϣ�󣬻��ڽ� logEntry д�� log[]��������־�������� leader ���� commit���ύ�� ��Ϣ���� leader �յ������� follower �� commit ��Ϣ�󣬻��� follower �ظ� apply��Ӧ�ã���Ϣ��follower ���յ���Ϣ����ִ����־��Ŀ�е������**ע����������д����־�ȵ� leader commit ���ִ������**��

* `commitIndex`��**����ύ�� entry �� index**����ʼ��Ϊ 0��Ȼ�󵥵�������
* `lastAppliedIndex`��**���Ӧ��(apply)��״̬������������(index)**����ʼ��Ϊ 0������������

**Leader �ڵ��ϵ���ʧ��Ϣ**��==��ѡ��֮�����³�ʼ��==����

* `nextIndex[]`��**Ϊÿ���ڵ�ֱ�ά�����´� replicate entry ʱʹ�õı��**����ʼ��Ϊ leader_last_log_index + 1��
* `matchIndex[]`��**Ϊÿ���ڵ�ֱ�ά������ʾ��֪�ġ����Ƴɹ������ index**����ʼ��Ϊ 0������������

### AppendEntries RPC

![alt text](/MIT6824/Raft/appendEntry.png){:height="60%" width="60%"}

* **��;**���� leader ����**���� replicate log entries ��Ҳ��������**��
* **����**��
  * `term`��**leader �����ڱ��**��
  * `leaderId`��**follower �ض���ͻ���ʱ���õ�**��
  * `prevLogIndex`��**��һ�� log entry �� index**��
  * `prevLogTerm`��**prevLogIndex entry �� term**��
  * `entries[]`����Ҫ׷�ӵ� log ���� entry������� heartbeat��������Ϊ�գ��������Ч�ʿ��ǿ��ܻ��ж������
  * `leaderCommit`��**leader �� commitIndex**��
* ���ؽ��
  * `term`��currentTerm��leader �����������Լ���
  * `success`����� follower ������ƥ�� prevLogIndex and prevLogTerm �� entry���򷵻� true��

* **ʵ��**��
  1. ��� leader �� term С�ڵ�ǰ currentTerm������ false��
  2. �����־�в�����һ������Ϊ prevLogIndex ����Ŀ����������prevLogTermƥ�䣬�򷵻�false��
  3. ���������Ŀ������Ŀ��**index ��ͬ�� term ��ͬ**��������ͻ����ɾ��������Ŀ�������к�����Ŀ��
  4. ����κ���δ��¼����־�е�����Ŀ��
  5. ��� `leaderCommit` ���� `commitIndex`���� `commitIndex` ����Ϊ`leaderCommit` �� `index of last new entry` �е���Сֵ��

### RequestVote RPC

![alt text](/MIT6824/Raft/requestVote.png){:height="60%" width="60%"}

* **��;**���� candidate ���������ռ�ѡƱ��
* **����**��
  * `term`����ѡ�ߵ����ڣ�
  * `candidateId`����ѡ�� ID��
  * `lastLogIndex`����ѡ�����һ����־��Ŀ��������
  * `lastLogTerm`����ѡ�����һ����־��Ŀ�����ڣ�
* **���ؽ��**��
  * `term`����ǰ���ڣ����ں�ѡ�߸����Լ����ڣ�
  * `voteGranted`����ѡ���Ƿ���յ�ͶƱ��
* **ʵ��**
  * ����ѡ�˵� `term` < `currentTerm`����Ͷ�ݷ���Ʊ
  * �����ѡ�˵���־���»����뱾����־һ���£���Ͷ���޳�Ʊ��

* **ͶƱ����**
  * **��ѡ�����һ��Log��Ŀ�����ںŴ��ڱ������һ��Log��Ŀ�����ں�**��
  * **��ѡ�����һ��Log��Ŀ�����ںŵ��ڱ������һ��Log��Ŀ�����ںţ��Һ�ѡ�˵�Log��¼���ȴ��ڵ��ڱ���Log��¼�ĳ���**

### InstallSnapshot RPC

![alt text](/MIT6824/Raft/InstallSnapshotRPC.png){:height="60%" width="60%"}

* **��;**���� leader ���ã������շ��͸� follower��leader ���ǰ���˳���Ϳ顣
* **����**
  * `Term`���쵼�ߵ�����
  * `leaderID`���쵼�ߵı��
  * `lastIncludedIndex`�����հ��������һ����־������
  * `lastIncludedTerm`�����հ��������һ����־������
  * `offset`�����ݿ��ڿ����ļ��е��ֽ�ƫ����
  * `done`���Ƿ�Ϊ���һ�����ݿ�
* **����ֵ**
  * `term`��׷���ߵĵ�ǰ����
* **ʵ��**
  1. ��� term < currentTerm �������أ�
  2. �����һ���飨ƫ����Ϊ0������һ���µĿ����ļ���
  3. ��� done Ϊ false����ظ����ȴ��������ݿ飻
  4. ��������ļ�������������С���κ����л򲿷ֿ��գ�
  5. ���������־��Ŀ����յ� `lastIncludedIndex` ������ͬ�������������������־��Ŀ֮�����Ŀ���ظ���
  6. ����������־��Ŀ
  7. ʹ�ÿ�����������״̬���������ؿ��յļ�Ⱥ���ã�

### Rules for Servers

![alt text](/MIT6824/Raft/rulesforserver.png){:height="60%" width="60%"}

#### All Servers

* ��� `commitIndex` > `lastApplied`�������� `lastApplied`����ִ�� `log[]` �����Ϊ `lastApplied` �����
* ��� RPC ��������Ӧ�� `term` T > `currentTerm`���� `currentTerm` ����Ϊ T����ת��Ϊ׷���ߣ�**����Ӧ�Ծ� leader �ӹ��������Ѳ�ת��Ϊ follower**����

#### Followers

* ��Ӧ���� `candidates` �� `leaders` �� RPCs��
* ����� `election timeout` ʱ����û���յ� `AppendEntries`�������µ�һ��ѡ������ȡ�� leader������ѡ�ٵ� follower �Զ�ת��Ϊ candidate��

#### candidates

* ת��Ϊ candidate ��ʼѡ�٣�
  * ���� `currentTerm`��
  * ������ͶƱ��
  * ���� `election timeout` ʱ�䣻
  * ���� `RequestVote` ��������ѡ�ˣ�

* ����յ�����������������ͶƱ����ת��Ϊ `leader`��
* ����յ������� `leader` �� `AppendEntries` ���ã���ת��Ϊ `follower`��
* �� `election timeout` ��ʱ�����ٴη����µ�ѡ�٣�

#### leaders

* ��ѡ����ÿ�����������ͳ�ʼ�յ� AppendEntries RPCs���������Է�ֹѡ�ٳ�ʱ���ڿ����ڼ��ظ��˲�����
* ����ӿͻ��˽��յ��������ǣ�**����Ŀ��ӵ�������־�У��ڽ���ĿӦ����״̬��֮��������Ӧ**��

* ��� follower �� `lastLogIndex` �� `nextIndex` �����ʹ� `nextIndex` ��ʼ�� AppendEntries RPC��Ϣ
  * ����ɹ������� follower �� `nextIndex` �� `matchIndex`;
  * ���������־��һ�µ��� `AppendEntries` ʧ�ܣ������ `nextIndex` �����ԡ�
* �������һ�� N��ʹ�� N > `commitIndex`���� `matchIndex[i]` �Ķ���Ԫ�ش��ڵ��� N������ `log[N].term` == `currentTerm`���� `commitIndex` ����Ϊ N��**������ commitIndex ��ͬʱӦ��Ҳ��ִ����Ӧ����**����

#### question

    Ϊʲô leader Ҫ��һ�����ϵķ������� log entry д����־��ִ����Ӧ���

��������ʮ������� commit ����������Ϊһ�����ϵķ�����ȷ��������ô���

* **�ܸ������Ӧ�ͻ���**�����ǵȴ����е� follower ȷ�ϣ�����Ҫ���Ѹ�����ʱ�䣻
* **��֤ѡ�� Leader ʱ��ӵ��������־�� follower �ܹ���ѡ**�����������������أ�==candidate �ĵ�ѡ�������ǳ���һ��ķ������� candidate Ͷ���޳�Ʊ����Ͷ���޳�Ʊ�������� candidate ����־��Ŀ���������ͶƱ����������־��Ŀ==��**���һ��ӵ�о���־�� candidate ����ѡ�٣����ǲ����ܻ�ȡ����һ���ѡƱ����Ϊһ������ӵ������־�ķ����������Ͷ����Ʊ**����ͱ�֤�� leader ����С�� commitIndex ������ִ��֮����г־��ԣ����ᱻ������ѡ���� leader Ӱ�졣

---

    Ϊʲô�־û� currentTerm �� votedFor �����־û� commitIndex �� lastApplied ��

�־û� currentTerm �� votedFor ��ԭ���Ǳ�֤һ��������һ�� Server ����һ�� candidate ͶƱ����ֹһ�������ڳ������� leader��

��Ϊ������ leader ����ͨ�� AppendEntries �Ľ�����ж���Щ���ݱ� commit �ˡ�

#### tips

* **leader ֻ�� commit ��ǰ���ڣ�Term���ڵ���־�������ύ��ȥ���ڣ�Term������־**��Ϊ�����κ�ͬ������ follower�������е������� leader ������ AppendEntries һ���յ���־�������� lab �е�����Ϊ leader ���κ�㲥������������ͬ����
* **�������������Ա������ӣ�Ҳ����Я�������Ϣ���� leaderCommitIndex �Ӷ����� follower �� commitIndex**���Լ�����ʱ��һ�����͵�������������Я����ʼ�� LastLogIndex��leader���һ����־�� index + 1������ follower ���п���ͬ����
* ����쵼�߷��� `AppendEntriesRPC`�����ұ��ܾ�����������Ϊ��־��һ�¶�����Ϊ follower �� `Term` > `currentTerm`��**˵���� leader �����ǹ��������ķ���������ʱ��Ⱥ���Ѿ�ѡ����һλ�� leader����ô�� leader ת��Ϊ follower**��
* leader�ڵ��� `AppendEntriesRPC` ���յ��ظ�֮��û�з����仯��
  * һ������������������ `matchIndex` = `nextIndex` - 1 �� `matchIndex` = `len(log)`��**����ܴ�����ȫ��������Ϊ������ֵ������ leader ���� RPC ���ѱ�����**��
  * ��ȷ������Ϊ���� `matchIndex` = `prevLogIndex` + `len(entries[])` ��leader ����� RPC �з��͵Ĳ�������

##### [**������־���ݣ�Fast Backup��**](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/lecture-07-raft2/7.3-hui-fu-jia-su-backup-acceleration)

ͨ���� `AppendEntries` �ķ��ؽ���м��� `XTerm`��`XIndex` ���� leader �����ҵ���־ͬ������

* `XTerm`����¼��ͻ��־��Ŀ�� Term��
* `XIndex`����¼��Ӧ���ں�Ϊ XTerm �ĵ�һ����־��Ŀ�� Index��

**follower**��

* ��� follower ����־û�� `prevLogIndex`����Ӧ�÷��� `XTerm` = -1 �� `XIndex` = len(logs)��
* ���׷������־����Ϊ `prevLogIndex` �� `term` ��ƥ�䣬��Ӧ�÷��� `XTerm` = log[prevLogIndex].Term��`XIndex` = `XTerm` �ĵ�һ��������

**leader**��

* �յ���ͻ��Ӧ���쵼��Ӧ����������־������ `XIndex` ��������־��Ŀ�Ƿ�Ϊ `XTerm`�����ƥ�䣬������ `nextIndex[peer]` = `XIndex` + 1 �� `matchIndex[peer]` = `XIndex`����**���õ�����������Ϊ `XTerm` �����һ����־�� `index` + 1**��
* �����ƥ�䣬��֤�� leader ������ `XTerm` ����־��Ŀ�������� `nextIndex[peer]` = `XIndex`��

