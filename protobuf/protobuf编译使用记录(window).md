# protobuf 编译安装使用

## 生成静态库

### protobuf下载

Releases[下载地址](https://github.com/protocolbuffers/protobuf/releases)，选择cpp版本下载。

![alt text](/tutorial/下载.png)

### CMake下载

CMake [下载地址](https://cmake.org/download/)，选择window x64 install版本下载。

![alt text](/tutorial/cmake下载.png){:height="75%" width="75%"}

### CMake编译

选择protobuf目录下的cmake文件夹，因为这里生成64位的静态库，所以新建一个buildx64文件夹。

取消勾选 BUILD_TEST，依次点击 Configure，Generate，Open Project。

![alt text](/tutorial/Cmake编译.png){:height="75%" width="75%"}

进入Visual Studio后，使用 x64 Debug 生成，等待一段时间~

![alt text](/tutorial/Debug.png){:height="75%" width="75%"}

生成完毕后进入工程下 Debug 文件夹，主要用到以下三个文件：

![alt text](/tutorial/VS生成.png){:height="75%" width="75%"}

## protobuf使用

使用 Viual Studio 创建新的项目，新建 Person.proto 文件

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

使用 protoc.exe 生成 Person.pb.h 和 Person.pb.cc 文件，语法如下：

```(shell)
protoc -I [path] [.proto文件] --cpp_out=[输出路径]
```

编写main函数

```(c++)
#include "Person.pb.h"
#include <string>
#include <iostream>
int main()
{
    Person p;
    p.set_id(10);
    // 在 window 下对于数组
    // 要先使用 add_name 申请空间
    // 再使用 set_name 赋值
    p.add_name();
    p.set_name(0, "emiya");
    p.add_name();
    p.set_name(1, "lin");
    p.set_sex("man");
    p.set_age(32);
    p.mutable_home()->set_num(1001);
    p.mutable_home()->set_address("HaiNan");

    // 序列化
    std::string output;
    p.SerializePartialToString(&output);
    //std::cout << output << std::endl;

    // 反序列化
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

### 链接器配置

包含 protobuf 头文件，在附加包含目录中配置 protobuf 下载目录下的src文件夹。

![alt text](/tutorial/include.png){:height="75%" width="75%"}

附加库目录包含 Visual Studio 生成的Debug文件夹。

![alt text](/tutorial/lib目录.png){:height="75%" width="75%"}

引用Debug目录下 **libprotobufd.lib**, **libprotocd.lib** 静态库文件。

![alt text](/tutorial/lib文件.png){:height="75%" width="75%"}

因为之前生成解决方案时使用的是 **Debug** 生成方式而非 **Release** 生成方式，故这里的静态库文件后缀带 'd'，相应的在**代码生成**的**运行库**选项中选择 **/MTD**，意为使用**静态调试库**，若使用 Release 生成方式则选择 **/MT** 选项。

![alt text](/tutorial/运行库选项.png){:height="75%" width="75%"}

### 运行

之后再使用 Visual Studio 生成运行后可得到相应输出。

![alt text](/tutorial/输出.png){:height="75%" width="75%"}
