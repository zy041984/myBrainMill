#编译
# 21.12版本编译
## 源代码及版本
 21.12
 https://github.com/protocolbuffers/protobuf/tree/v21.12 


## 第三方库及版本
 googleTest1.12.1
 https://github.com/google/googletest/releases/tag/release-1.12.1

## 用CMake编译
 https://github.com/protocolbuffers/protobuf/blob/v21.12/cmake/README.md

 推荐使用static library，不要选择CMake里面的BUILD_SHARED_LIBS
# 21.12版本使用
使用powershell进入protoc.exe所在路径
```
.\protoc.exe --proto_path=D:\practice\CommunicationProtocol\Dll1\Dll1 --cpp_out=D:\practice\CommunicationProtocol\Dll1\Dll1  myClassesProto.proto
```
 将生成myClassesProto.pb.h和myClassesProto.pb.cc

c++的dll工程
有一个设置 语言的符合模式，要从yes(/permissive)改为默认
要使用MTd模式才能编译通过



# 28.2版本
https://github.com/protocolbuffers/protobuf/releases
依赖
https://github.com/abseil/abseil-cpp/releases
## c++编译
先编译absl，选中ABSL_PROPAGATE_CXX_STD，会编译为static_lib，安装时自行生成debug和release文件夹，再把生成的debug版复制到debug中
再编译protobuf，由于absl自行设置了MD，所以protobuf要取消选择MSVC_STATIC_RUNTIME
或者把abseil放在protobuf中编译，在编译osgEarth时发现还是让protobuf编译abseil比较好
debug版本的几个exe项目中，链接到absl的debug版本的lib，要自行设置位置和文件名，链接到utf8的debug库也要在文件名自行加后缀d，exe项目的生成文件也要自行加后缀d，再修改cmake_install以正确安装
没有设置conformanceMode为Yes，后来用testProtobuf项目自己测试发现设置为yes或no都可以，估计是新的代码没有那些不符合新规范的部分。

## tutorial快速开始
### 1 写proto文件
- package像namespace，避免项目间名字冲突
- 每个数据结构写成一个message，有点像struct
    - 一个message里面有各个成员变量，即field
    - message可以互相嵌套，可以在message里面定义枚举
- field像message里面的成员变量
    - 每个field的数据类型可以是整型，浮点型，字符串，布尔
    - 每个field可以有独一无二的field number，在二进制编码时使用，所以把0-15尽量付给常用的，重复的field。
    - 每个field必须有修饰符，required，optional，repeated。
        - required指的是必须有初始化值，optional指的是可以不提供初始值，
        - repeated指的是这个field可以形成一个动态array
        - proto3不支持required，proto2强烈不推荐使用required
不支持类的继承
如果升级proto文件后想保持兼容旧的文件，则不要修改旧的required。可以删除旧的repeated和optional及添加新的repeated和optional，但field的number要用新数字。
应该指的是升级proto文件后不用重新制作.pb.cc和.pb.h
### 2 编译proto文件
使用protoc编译proto文件，得到.pb.cc和.pb.h，对数据进行序列化和反序列化。
`protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto`
例如
`.\protoc.exe -I="E:\work\testProtoBuf\testProtoBuf" --cpp_out="E:\work\testProtoBuf\testProtoBuf" "E:\work\testProtoBuf\testProtoBuf/address.proto"`
### 3 自己写程序读写
使用field的set方法设置非数组的成员变量
使用数组成员变量的add方法来在动态数组中添加一个新元素。
读写直接使用c++的fstream即可，写为binary形式
提示多个message写入一个fstream时，自行记住每个message的offset
最后析构
项目要使用absl和protobuf的头文件，使用它们的debug或release的库，使用utf8的debug或release的库。使用debug或release的libprotobuf.lib
提供了反射机制。


## version解释
https://protobuf.dev/getting-started/
版本号，以前的版本机制是Maj.Min.Sub，从3.20开始换成统一一个大版本号，然后每个语言内部使用Maj.Min.Sub
c++为例
protoc版本21.x对应的c++版本是3.21.x
protoc版本22.x对应的c++版本是4.22.x
protoc版本26.x对应的c++版本是5.26.x
protoc版本30.x对应的c++版本是6.30.x

## programming guide
有个调试工具protoscope可以把二进制的内容转化为文本
### proto2
- 分配field号码的时候不要重用已有的，包括已经删除的field。
- optional的field如果没有默认值，是不会被序列化为wire形式的。
- field的描述符可以是optional，repeated，map，required。
    - 提示1不要使用required
    - 提示2如果是简单类型的动态array，不要用repeated，用`[option=true]`来获得更好的性能。
    - 真的删除某个field时，把field的名字和number放在一个reserved区域里，这样后面接手的人就不会误用这些。
    ```
    message Foo{
    reserved 2,15,9 to 11;
    reserved "foo","bar";
    }
    ```
- string类型需要utf8编码的字符串
- enum可以定义在message的外面，enum里面已经删除的值可以标记为reserved，想要删除的可以添加`[deprecated=true]`
    ```
    enum Foo{
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3 [deprecated=true];
    reserved 4,5;
    }
    ```
- 可以加入一个import声明来引入别的proto文件已经定义好的message，就像include一个头文件
    `import "myproj/other.proto"`
- message可以在一个message里面定义，像class的内部class。也可以两个message分别独立定义，一个message里面有另一个的成员变量
- group被废弃了，不要用
- 升级一个message，旧版pb.cc和新版pb.cc都能对它进行序列化和反序列化，但有很多注意事项
- extensions，可以把message的某个field定义在另一个proto文件中。有点类似FileA.h文件定义了structA。然后FileC.h文件定义了struct C，C里面有成员变量A。但是FileC.h文件不include FileA.h文件，而是FileA.h里include FileC.h
     只要在FileC.proto中添加
    ```
    package FileC;
    message CDef{
      extensions 100 to 199[
      declaration={
        number:126,
        full_name:"FileA.StructAInstance",
        type:"FileA.StructA",
        repeated:true
      },
      verification=DECLARATION];
    }
    ```
    即可，意思是C的编号100到199的field在另一些proto文件中定义。第126号field在FileA中定义，类型是StructA
    在FileA.proto中添加
    ```
    import "FileC.proto"
    package FileA;
    message StructA
    {
      optional int32 page_number = 2;
    }
    extend FileC.CDef{
      repeated StructA StructAInstance=126;
    }
    ```
    即可，即将本proto的StructA定义为FileC的CDef类的编号为126的extension成员变量
    看起来有点绕，就是为了解决不想在FileC中`import "FileA"`
- Any类型，把任意的数据序列化为byte
    例如
    ```
    import "google/protobuf/any.proto";
    message ErrorStatus{
      string message=1;
      repeated google.protobuf.Any detail=2; 
    }
    ```
- Oneof类型，有点类似union，几个变量共享一块内存。但不同的是这块内存只能用一个field来访问。一旦set了一个field，在Oneof中其余的field就变成unset了。
    ```
    message a{
      oneof test{
        string name = 4;
        SubMessage sub_mes=9;
      }
    }
    ```
    oneof不能用于repeated
    既然其余的field变成unset，使用其余的field时要小心内存问题
- map类型，类似c++的map，不能用于repeated，不能有重复的key
    `map<key_type,value_type> map_field=N;`
    key_type只能是整型或string，不能是float或enum或其他message
    valye_tye不能是map，可以是其余所有类型 
- options，分为文件级别的，message级别的，field级别的，亦可以自定义option
    - 文件级别的cc_enable_arenas，开启arena模式
    - 文件级别的optimize_for，使用speed，lite_runtime，code_size来区分pb.cc的运行速度
    - field级别的deprecated=true表示某个field已经不用了。
### proto3
大体与proto2差不多
第一行必须是`syntax = "proto3";`，表明使用3的语法
取消了extensions，只能在custom options时使用extensions
取消了group
### enum
不同语言的enum表现不一样，
enum对于未知值有两种相反的处理机制，开放和封闭的。
如果给一个enum类型的field赋的值超出了enum的定义，则开放机制会把这个值原封不动地读写，封闭机制则会存储到unknow field中，这个field会被赋enum的默认值，且被置于unset状态。
封闭机制下，如果field是repeated的，由于非法值已经被放到unknown field里了，剩下的合法值会占据它的位置，即动态数组的顺序不会被保证。
所以proto2的语法按照封闭式机制处理enum，而proto3会按照开放式处理emum
所以proto2和proto3的proto文件互相迁移时，要特别注意这一点
### encoding
如何把message变为wire或file，即二进制模式
形式上
    message被视为一系列的key-value，key就是每个field的number和类型和长度，value就是每个field的数值。
    嵌套的message，repeated都按照这个key-value形式拓展
细节上
    每个field实际被序列化的是 fieldnumber+类型+长度+数值
    序列化之后的二进制形式的基本单位是变长字符varint，每个字节只有低7bit为实际数据，最高一bit表示后续的byte是新的一个数据还是前一个数据
    类型是protobuffer自定义的，0代表变长字符varint，1代表8字节浮点，5代表4字节浮点数，2代表容器类型。
    长度部分，整数或浮点数的字节数在decode时候会自解释，无需再写出长度数值。string或byte或packed repeated field类型才有这一部分，因为需要写清楚一个字符串里有几个utf8。
    数值部分，整数用变长字符varint来存储，浮点数用8byte或4byte，
    所有4个部分的存储单位都是变长字符varint
更细节
    signed integer的encode是zigzag，不是传统的2补码形式
    float被encode为4字节little endian，可以memcpy
    double被encode为8字节little endian，可以memcpy
小提示
    一个message的field的顺序不是固定的，即使是同一版本的protoc产生的pb.cc也不能保证对一个message的每次序列化产生相同的二进制内容。所以计算出来的hash，crc，fingerprint也不保证一致。

### style guide
不同版本的style guide不一样，所以尊重已有文件的style，保持一致性
文件名蛇形规则，lower_snake_case.proto
文件内容的顺序安排
    - license
    - overview
    - syntax
    - package，名字最好小写
    - imports
    - option
    - 其他message等
package的名字最好小写
message的名字PascalCase，首字母大写，驼峰式，缩写部分的只大写首字，如`GetDnsRequest`不要写成`GetDNSRequest`
field的名字lower_snake_case，如果名字里有数字，数字不要单独占一个部分，数字与前面的字母写成一个部分
如果field是repeated的，名字最好变为复数形式
enum的每个value写成UPPER_SNAKE_CASE;不要用逗号
enum的名字写成PascalCase
如果enum里还没有value，加前缀UNSPECIFIED
### Technique
- 文件名后缀，文本形式的写成.txtpb，wire格式的写成.binpb，json格式的写成.json
- 如果把多个message写入一个文件，必须自己记录每个message的字节偏移量。按字节偏移量来读取每个message，参考类`CodedInputStream`
- 如果一个包大于megabyte，最好不要用protobuf。但是可以考虑把大包拆解为小包，每个小包单独用protobuf
### field presence
field presence的意思为一个field有没有值，表现形式有两种，no presence(不写出该field有没有值)，explicit presence(明确写出该field有没有值)。
proto2要求明确写出该field有没有值，proto3则要求不写出该field有没有值。
在proto3里，如果一个field写为optional，则按照explict presence形式执行
例子
```
syntax="proto3"
package example;
message myMessage{
  int32 not_tracked=1;//no presence
  optional int32 tracked=2;//explice presence
}
```
在代码中判断有没有值，使用的代码不一样
```
Msg m=getProto();
if(m.foo()!=0)//no presence的方式
  m.set_foo(0);
if(m.has_foo2())//explicit presence的方式
  m.set_foo2(0);
```
### best practice
不要重复使用删除的field number和名字，不要重复使用已删除的enum的value和number
不要改变field的数据类型，不要修改其默认值
Enum里一定要有一个Unspecified的value，且是第一个
一个文件一个message，不要搞得太大，便于后期更改
1-1-1 Rule
一个proto文件里只有一个顶级的message或enum或extension，一个proto文件里只有一个proto_library规则
### API best practice
不要一个proto文件里写很多message，遵守1-1-1原则
为传输和存储使用不同的message
顶级的proto最好少用基本数据类型，把基本数据类型改成一个message类，以为了后续扩充方便。
同理，用bool的地方多想想是否以后是否需要负数
ID最好不用整数，而是用string
不要把数据写为string，选择一个正确的数据类型
## Editions解释
使用edition来代替传统的syntax=proto2或syntax=proto3，最新的edition为2023
一个edition代表feature的集合，每个feature其实就是一个关于file或message或field的option，每个feature在当前edition里会有默认值。在下一个edition里，某个feature可能变化了，如果不符合你的要求，你可以override某个feature。
syntax=proto2或syntax=proto3其实就是这些option使用了默认值，但是使用edition可以允许更加灵活地使用option，有些option可以在file级别设置，某些message可以单独设置这个option来覆盖文件里的设置。
即使某个feature被标记为deprecated了，也有充足的时间让用户删除这些旧的影响
有一个工具prototiller来把旧的文件迁移到edition
https://protobuf.dev/editions/features/ 介绍了当前启用的feature。
## arena技术
这是c++版本才有的特性，用于优化message创建过程中的内存使用，核心思想是预分配迟回收
## UPB
把protobuf的核心用c实现了一遍，来实现被其他语言来包装，