# Patterns and Hints for Concurrency in Go

1. Use the race detector,for development and even production.
   1. �ڿ�������������ʹ�þ��������
2. Convert data state into code state when it makes programs clearer.
   1. ���ܼ򻯳�������ǰ���£�ͬһ������Ĵ���ʵ�֣����Կ���**������״̬ת��ɴ���״̬**
3. Convert mutexes into goroutines when it makes programs clearer.
   1. ���ܼ򻯳�������ǰ���£�����ʹ�� goroutine �����������
4. Use additional goroutines to hold additional code state.
   1. **����ͨ�������µ�goroutine���洢code state����״̬**
5. Use goroutines to let independent concerns run independently.
   1. ʹ�� goroutines ����������Ҫ�Ĳ��ַ��롣
6. Consider the effect of slow goroutines.
   1. �������л��� goroutines �Գ�����ɵ�Ӱ�졣
7. Know why and when each communication will proceed.
   1. �˽�����ܳ������е�����
8. Know why and when each goroutine will exit.
   1. �˽�ÿ�� goroutine �Ľ�������
9. Type Ctrl-\to kill a program and dump all its goroutine stacks.
10. Use the HTTP server's /debug/pprof/goroutine to inspect live goroutine stacks.
11. Use a buffered channel as a concurrent blocking queue.
    1. �����л����� channel ��Ϊһ�������������С�
12. Think carefully before introducing unbounded queuing.
13. Close a channel to signal that no more values will be sent.
    1. �ر�һ�� channel �������ٷ����������ݡ�
14. Stop timers you don't need.
    1. ��ʱ�ͷŲ���Ҫ�Ķ�ʱ����
15. Prefer defer for unlocking mutexes.
    1. ʹ�� defer ����֤���������ͷ�
16. Use a mutex if that is the clearest way to write the code.
17. Use a goto if that is the clearest way to write the code.
18. Use goroutines,channels,and mutexes together if that is the clearest way to write the code.

## ����/����ģʽ

�Է������ķ������Ĵ���ʵ�־������������ԭ���ķ���������صķ����У���Ҫ��mutex�����ٽ���map�Ķ�д������Ƶ���ļ������������̡�

![alt text](image/pubsub.png)

**hint**��think carefully before introducing unbounded queuing.

ԭ�ȵĴ���ͨ������ά�������б����Ӻ�ɾ��������ͨ�� select ѭ��ʵ�ִӶ�������
�ľ���������channel�������Ĺ��ܣ�channel���������ֱ����д����׼���ã��ҵ�����ͨ����ȡ����� channel �õ�����ֵ��

![alt text](image/pubsubChange1.png)

![alt text](image/pubsubChange2.png)

![alt text](image/pubsubChange3.png)

��������ͨ�� select ѭ���� channel ά���˶��Ķ��У����Ƕ��� publish �޷�����ϸ���ȵĿ��ơ������������Ϊÿ���������½� helper goroutine ������ loop ѭ���� publish ������**ͨ��ʹ�� goroutine ���Խ������й�ע����룬����ÿ�� goroutine ��ȥ����ؼ�����**

![alt text](image/helper.png)

![alt text](image/pubsubchange4.png)

## Work scheduler(����������)

�޸�bug���հ����ûᵼ��ÿһ�� call ����ʹ����ͬ��task
![alt text](image/Workscheduler.png)

�����ʱ�������������ڷ����������������п��з�����ʱ������һ��goroutine���������Ա��ⴴ����������� goroutine��ռ��ϵͳ��Դ��
![alt text](image/Workscheduler1.png)

Ϊÿ��server����һ�� goroutine ����Ϊÿ��task����goroutine��������̳߳ص�˼����񣬿��Լ��ٴ����������̵߳Ŀ�������
�� servers �����ݽṹ**�������Ϊchannel**�����ҿ���һ��goroutineѭ����ȡserver��ʹ�ÿ��Զ�̬������ server ������

![alt text](image/Workscheduler2.png)

���ƴ��룺���� task �����ظ� done ���� server ����

![alt text](image/Workscheduler3.png)

���ƴ��룺�������������� server ����ʱ�����������server�������� `done <- true`������ͨ��**�½�һ�� goroutine ����ɷ�������Ĺ����������� channel �������������**����hints��**�˽�ÿ�� goroutine ������״̬**��

![alt text](image/Workscheduler4.png)

���ƴ��룺**���������ʧ��ʱ���䷵��work����**��**�ر� channel ����ֹͣ��������**������������ÿ��ܻ�ʧ�ܵ��� task ���ض��У�����Ҫ�ȵ����� task ����ɺ��ٹر� work channel��

![alt text](image/Workscheduler5.png)

���ƴ��룺**ȷ��ÿ��goroutine������˳�**

![alt text](image/Workscheduler6.png)

## ���Ʒ���Ŀͻ���(Replicated service client)

�ͻ��˵��ýӿڣ��ýӿ�ӵ��һ��ɵ��õķ������Լ�һ��Զ�̵��÷�����**�û���ʹ�õ��÷���ʱ���ù��ĶԷ������ĵ���ϸ��**������ѡ���ĸ������������ó�ʱ���·�������...��

![alt text](image/Replicated.png)

Ϊ done channel �ṩ�㹻�Ļ���������֤������÷���ʱ������������Ϊ�û�ֻ���ĵ�һ�����صĽ��������ķ������ݽ��� GC ����

![alt text](image/Replicated2.png)

**��ʹ�� goto ����ʹ���������ʱ��Ҫ����ʹ����**��

![alt text](image/Replicated1.png)

## Э��ѡ����(protocol multiplexer)

һ�������� **epoll �� I/O ��·����**�ӿ�

![alt text](image/protocolmultiplexer1.png)

ʹ�� Readtag ����Ϣӳ���Ψһ�� tag����ʹ�� map �� tag �뷵�ؽ���� channel ��ӳ�䣬�ڷ��ؽ��ʱ������ tag ѡ����Ӧ�� channel ���ؽ����
����Ҫע����ǣ�**������Ϣ�ͷ��ؽ����readtag ���뽫��ӳ�䵽��ͬ��Ψһ�� tag ��**��

![alt text](image/protocolmultiplexer2.png)

![alt text](image/protocolmultiplexer3.png)
