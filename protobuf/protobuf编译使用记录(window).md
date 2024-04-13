# protobuf ���밲װʹ��

## ���ɾ�̬��

### protobuf����

Releases[���ص�ַ](https://github.com/protocolbuffers/protobuf/releases)��ѡ��cpp�汾���ء�

![alt text](/tutorial/����.png)

### CMake����

CMake [���ص�ַ](https://cmake.org/download/)��ѡ��window x64 install�汾���ء�

![alt text](/tutorial/cmake����.png){:height="75%" width="75%"}

### CMake����

ѡ��protobufĿ¼�µ�cmake�ļ��У���Ϊ��������64λ�ľ�̬�⣬�����½�һ��buildx64�ļ��С�

ȡ����ѡ BUILD_TEST�����ε�� Configure��Generate��Open Project��

![alt text](/tutorial/Cmake����.png){:height="75%" width="75%"}

����Visual Studio��ʹ�� x64 Debug ���ɣ��ȴ�һ��ʱ��~

![alt text](/tutorial/Debug.png){:height="75%" width="75%"}

������Ϻ���빤���� Debug �ļ��У���Ҫ�õ����������ļ���

![alt text](/tutorial/VS����.png){:height="75%" width="75%"}

## protobufʹ��

ʹ�� Viual Studio �����µ���Ŀ���½� Person.proto �ļ�

```(proto)
syntax = "proto3";

message addr
{
    int32 num = 1;
    bytes address = 2;
}

message Person
{
    int32 id = 1;
    repeated bytes name = 2;
    bytes sex = 3;
    int32 age = 4;
    addr home = 5;
}
```

ʹ�� protoc.exe ���� Person.pb.h �� Person.pb.cc �ļ����﷨���£�

```(shell)
protoc -I [path] [.proto�ļ�] --cpp_out=[���·��]
```

��дmain����

```(c++)
#include "Person.pb.h"
#include <string>
#include <iostream>
int main()
{
    Person p;
    p.set_id(10);
    // �� window �¶�������
    // Ҫ��ʹ�� add_name ����ռ�
    // ��ʹ�� set_name ��ֵ
    p.add_name();
    p.set_name(0, "emiya");
    p.add_name();
    p.set_name(1, "lin");
    p.set_sex("man");
    p.set_age(32);
    p.mutable_home()->set_num(1001);
    p.mutable_home()->set_address("HaiNan");

    // ���л�
    std::string output;
    p.SerializePartialToString(&output);
    //std::cout << output << std::endl;

    // �����л�
    Person pp;
    pp.ParseFromString(output);
    std::cout << pp.id() << ", " << pp.sex() << ", " << pp.age() << ", " << pp.home().num() << ", " << pp.home().address() << std::endl;
    int len = pp.name_size();
    for (int i = 0; i < len; i++) {
        std::cout << pp.name(i) << " ";
    }
    std::cout << std::endl;
}
```

### ����������

���� protobuf ͷ�ļ����ڸ��Ӱ���Ŀ¼������ protobuf ����Ŀ¼�µ�src�ļ��С�

![alt text](/tutorial/include.png){:height="75%" width="75%"}

���ӿ�Ŀ¼���� Visual Studio ���ɵ�Debug�ļ��С�

![alt text](/tutorial/libĿ¼.png){:height="75%" width="75%"}

����DebugĿ¼�� **libprotobufd.lib**, **libprotocd.lib** ��̬���ļ���

![alt text](/tutorial/lib�ļ�.png){:height="75%" width="75%"}

��Ϊ֮ǰ���ɽ������ʱʹ�õ��� **Debug** ���ɷ�ʽ���� **Release** ���ɷ�ʽ��������ľ�̬���ļ���׺�� 'd'����Ӧ����**��������**��**���п�**ѡ����ѡ�� **/MTD**����Ϊʹ��**��̬���Կ�**����ʹ�� Release ���ɷ�ʽ��ѡ�� **/MT** ѡ�

![alt text](/tutorial/���п�ѡ��.png){:height="75%" width="75%"}

### ����

֮����ʹ�� Visual Studio �������к�ɵõ���Ӧ�����

![alt text](/tutorial/���.png){:height="75%" width="75%"}
