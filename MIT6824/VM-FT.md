# VMware-FT

## ���ƺ��ݴ�

�����ܽ�������⣺

* **fail-stop ����**���������Ϊ��Դ�����ȵȹ���ֹͣ���У�
* һЩ��ת��Ϊ **fail-stop ����**��Ӳ�������� bug�����������������˷����������������ϵ����ݳ�����������

���Ʋ��ܽ�������⣺

* **����bug**���� master �ڵ�����������˴������������������ͬ�Ĵ�������

## ״̬ת�ƺ͸���״̬��

* **״̬ת��**��primary ������ÿ��һ��ʱ����ڴ���շ��͸� back-up �ڵ㣬back-up �ڵ㽫�ڴ����д���ڴ����ȫ�����ƣ�
* **����״̬��**��primary �ڵ�� back-up �ڵ㱣����ͬ�ĳ�ʼ״̬����ͬʱ����������ڻ�����ִ����ͬ�Ĳ�����������ͬ��״̬��
  * ��Ҫ��������ͬ��Ƶ�ʣ�
  * �������������������ӽڵ�ִ������ʧ�ܣ����ܵ�������״̬��һ�£���

## VMware-FT����ԭ��

VM-FT�ṹͼ��

![alt text](/MIT6824/VM-FT/Configuration.png){:height="60%" width="60%"}

�ṹ���ܣ�

* primary �� backup ���������ڲ�ͬ�ķ�������VMM��;
* primary, backup �Լ� client ��������ͬһ��������;
* VMware FT��Ķั������û��ʹ�ñ����̣�����ʹ����һЩDisk Server��Զ���̣�;
* primary �� backup ֮��ͨ�ŵ�ͨ����ΪLog Channel.

VM-FT��������:

![alt text](/MIT6824/VM-FT/workprocess.png)

����ִ�����̣�

1. client ���Ͱ�����������ݰ���
2. ���ݰ��������ϱ��յ�, ����һ���жϱ� primary �� VMM ����
3. primary��VMM�����жϺ�, �����������������
   1. ���Լ����⻯�� OS ��, ģ���������ݰ�������жϣ�
   2. ���������ݰ�����һ�ݣ���ͨ�� `Log Channel` �͸� Backup ������ڵ� VMM��
4. Backup ������ڵ� VMM �յ����ݰ�, ͬ�����Լ����⻯�� OS ��, ģ���������ݰ�������жϣ�
5. primary �� backup ִ�����ݰ��е����
6. primary �����ظ�����, ͨ�� VMM ���ظ��ͻ��ˣ�
7. Backup �� VMM �������Ļظ�������

## ��ȷ�����¼�

��ȷ�����¼���Ϊ���¼��ࣺ

* **�ͻ�������**��һ����Դ�ڿͻ��˵��������ݰ���������ʱ������ֳ�����Ԥ���ԡ����������ݰ��ʹ�ʱ��ͨ�������� DMA��Direct Memory Access���Ὣ�������ݰ������ݿ������ڴ棬֮�󴥷�һ���жϡ�����ϵͳ���ڴ���ָ��Ĺ�������������жϡ�**����Primary��Backup�����Ҫ����ͬ��ʱ�䣬��ͬ��λ�ô���������ִ�й��̾��ǲ�һ���ģ������ᵼ�����ǵ�״̬����ƫ�**
  * ���ӣ����û��򲩿���������һƪ���£���ͳ�Ƶ�ǰ���͵�����������д����˼���У����뱣֤���ӽڵ�����������������ִ��˳�򱣳�һ�£�������Ϊ���ݰ�����Ĳ���Ԥ���Զ��������ӽڵ��ڲ�ͬ��״̬��ִ�������״̬ƫ�
* **����ָ��**
  * �������������
  * ��ȡ��ǰʱ���ָ�
  * ��ȡ�������ΨһID��
  * ......
* **��CPU�Ĳ���**������̨�� back-up �������ֱ��ɲ�ͬ���߳�ִ�����������µĽ�����ܲ�ͬ�����Զ����һ���޴�ķ�ȷ�����¼���Դ��

Primary �������� back-up ������֮��ͨ�� Log Channel ��ͬ������־Ӧ�ð��������������ݣ�

1. **�¼�����ʱ��ָ�����**��ͨ��ָ����Ų�ͬ����������ͬ��ִ�������ʱ����
2. **��־��Ŀ������**�������������������룬Ҳ�����ǹ���ָ�
3. **����**�������һ���������ݰ�����ô���ݾ����������ݰ������ݡ������һ������ָ����ݽ�������Щ����ָ����Primary��ִ�еĽ��������Backup����Ϳ���α��ָ����ṩ��Primary��ͬ�Ľ����

### ��δ����������ݰ�����

��ͨ������£����������ݰ�����ʱ��������ͨ�� DMA �ķ�ʽֱ�ӽ����ݰ���������������ڴ��С����� VM-FT �У������������������Ὣ�������ݰ������� VMM ���ڴ棬֮�������жϻ��͸� VMM ����ʱ��**VMM ����ͣ Primary �������ס��ǰ��ָ�����**���������������ݰ������� Primary ������ڴ棬֮��ģ��һ�������жϷ��͸�primary�����primary ���������ݰ�������Backup�����**֮������ͬ��ָ�����λ��ģ��һ�������жϷ��͸� Backup ���**������������н��ܵ�Bounce Buffer���ơ�

## �������

ͨ��ȷ�����طŲ��������� VM �ܹ��� logging chanel �ж�ȡ�� VM �ĸ��ֲ�������ʵʱ�ڸ��� VM ��ִ�����Ʋ�����Ϊ��ʵ����־����ݴ����������ڶ������ݴ�Э�飬��Ҫ�� **Output Requirement** �� **Output Rule**��

**Output Requirement**��**����� VM �������Ϻ󸱱� VM �ӹܺ󣬸��� VM ����ӹ�ԭ���� VM �Ѿ���ɻ򲿷���ɵĹ��������������Ҫ����һ��**�� �� failover �ڼ䣬���������Կͻ�����͸���ģ��ͻ��˲��ᱻ�жϷ�������ǲ������һ�¡�

���������һ����������û��Դ�������ݽ����޸ģ��� primary �����������ݽ����޸ĺ���ͻ����޸ĳɹ���primary ����������������û�гɹ����޸����ݵ�ָ��ݸ� back-up ����������ʱ�� back-up �����������滻 primary ���������û��ٴβ�ѯ����ʱ��õ�ԭ���ľ����ݡ�

**Output Rule**��**����� VM ���������ͻ��˵������������ӳ��������������ֱ���� VM �յ������ڸ��� VM �Ĺ��ڸ������ȷ����־��**��

![alt text](/MIT6824/VM-FT/delayOuput.png)

## �ظ����

�ظ������Ѿ��� VMM �����ͻ����ˣ����Կͻ����յ��˻ظ���������ʱ Primary ��������ˡ�**���� Backup �࣬�ͻ������󻹶ѻ��� Backup ��Ӧ�� VMM �� Log �ȴ���������Ҳ����˵�ͻ�������û���������͵� Backup �����**��

�� Primary ����֮��Backup �ӹܷ���Backup ������Ҫ���������ڵȴ��������е� Log���Ա����� Primay ����ͬ��״̬������ Backup �������� Primary ��ͬ��״̬�ӹܷ��񡣶��� Backup �����������Ŀͻ�������ʱ��������ظ���������ͻ��ˣ���**�ͻ��˿���ͨ�� TCP Э����ͬ���к��ж����ݰ��Ƿ��ظ��Ӷ������Ƿ������ݰ�**����������ڴ�������ˡ�

�����κ��������л��ĸ���ϵͳ�������ϲ����ܽ�ϵͳ��Ƴɲ������ظ������Ϊ�˱����ظ��������һ��ѡ���������߶������������������һ���ǳ�����������**��Ϊ���ڿͻ�����˵����һ��ʧ�ܵ�����**���������������л�ʱ���л������߶��п��������ظ������������ζ�����и���ϵͳ�Ŀͻ�����Ҫһ���ظ������ơ�**����ʹ�õ���TCP������ظ���⣬���û��TCP���ǻ�������ҪӦ�ó��򼶱�����к�**��

## Test and Set

**test and set ��Ҫ���ڽ���������⣬������� VM ֹͣ�յ����������߷�������������� VM �͸��� VM ����Ϊ�Է��������ϣ���ʱ �� VM �͸��� VM ������Ϊ�ͻ����ṩ���������������**��

Ϊ��������⣬����Ҫȷ���������Ϲ���ʱ���� VM �͸��� VM ������һ�������ṩ���񣬱�����ʹ����������̵Ĺ����洢��������⣬�ڹ����洢��ִ�� atomic test-and-set ������

* ����ɹ������ VM ���߷���
* ���ʧ�ܣ���ʾ�Ѿ��� VM ���߷����ˣ���˵�ǰ VM �������ߣ�
* �����ǰ������ VM �޷����ʹ����洢������һֱ���ԡ�

��������洢�޷����ʣ������� VM �͸��� VM �������ô洢������ζ�Ŵ�ʱ VM Ҳ�޷���������ˣ�ʹ�ù����洢����������ⲻ���������Ĳ����á�