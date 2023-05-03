# 简介 - 用于序列化数据的协议

跨语言和平台

**序列化数据结构的方式**

更小、更快、更便捷

自带编译器（protoc），用 protoc 进行编译

.proto 文件定义，再使用特别生成的源代码(使用protobuf提供的生成工具)轻松的使用不同的数据流完成对结构数据的读写操作

可以更新.proto文件中对数据结构的定义而不会破坏依赖旧格式编译出来的程序



## 优缺点

优点：

A、性能好，效率高

B、有代码生成机制

C、支持向后和向前兼容

​	当客户端和服务器同时使用一块协议的时候， 当客户端在协议中增加一个字节，并不会影响客户端的使用

D、支持多种编程语言



缺点：

A、二进制格式导致可读性差

B、缺乏自描述



## 编译器使用 - protoc编译器

定义好的 .proto => 生成对应语言代码



## Protobuf 语法

message 即 结构化数据

// 注释

package 防止不同消息类型有命名冲突

字段类型

![](..\images\Protobuf字段类型.webp)



标识符：

​	字段唯一的数字标识符，一旦使用不能再改变

字段修饰符：

​	singular、repeated

保留标识符：

​	如果通过删除或者注释所有字段，以后的用户在更新消息类型的时候可能重用标识符。

​	注意：如果使用旧版本代码加载相同的.proto文件会导致严重的问题，包括数据损坏、隐私错误等等。

​	拒绝向前兼容 = 为字段tag指定reserved标识符

​	Protobuf编译器会警告未来尝试使用相应字段标识符的用户。

默认值：

​	对于string，默认是一个空string

​	对于bytes，默认是一个空的bytes	

​	对于bool，默认是false

​	对于数值类型，默认是0

​	对于枚举，默认是第一个定义的枚举值，必须为0

​	对于消息类型(message)，字段没有被设置，确切的消息是根据语言确定的，通常情况下是对应语言中空列表

​	对于标量消息字段，一旦消息被解析，就无法判断字段是被设置为默认值还是根本没有被设置，应该在定义消息类型时注意。

枚举：预定义序列，必须将其第一个类型映射为0，不推荐在enum中使用负数（可变编码方式的，对负数不够高效）

​	allow_alias选项为true，将不同的枚举常量指定为相同的值

引用其他消息类型：

​	将其它消息类型用作字段类型

Any 类型：

​	允许在没有指定.proto定义的情况下使用消息作为一个嵌套类型

Oneof：

​	代表在实现的时候，该组属性中有且只能有一个被定义，不能出现多个

Map：

​	关联映射

定义服务：

​	用在RPC(远程方法调用)系统中，可以在.proto文件中定义一个RPC服务接口

JSON映射：

​	Proto3支持JSON的编码规范，便于在不同系统之间共享数据

选项：

​	options 影响特定环境下处理方式



**更新消息类型：需要在消息中添加一个额外的字段，但同时旧版本写的代码仍然可用**

A、不要更改任何已有字段的标识符。

B、如果增加新的字段，使用旧格式的字段仍然可以被新产生的代码所解析。应该记住元素的默认值，新代码就可以以适当的方式和旧代码产生的数据交互。通过新代码产生的消息也可以被旧代码解析，但新增加的字段会被忽视掉。未被识别的字段会在反序列化的过程中丢弃掉，如果消息再被传递给新的代码，新的字段依然是不可用的。

C、非required的字段可以移除。只要标识符在新的消息类型中不再使用(推荐重命名字段，例如在字段前添加“OBSOLETE_”前缀)。

D、int32, uint32, int64, uint64,和bool是全部兼容的，可以相互转换，而不会破坏向前、 向后的兼容性。

E、sint32和sint64是互相兼容的，但与其它整数类型不兼容。

F、string和bytes是兼容的——只要bytes是有效的UTF-8编码。

G、嵌套消息与bytes是兼容的——只要bytes包含该消息的一个编码过的版本。

H、fixed32与sfixed32是兼容的，fixed64与sfixed64是兼容的。

I、枚举类型与int32，uint32，int64和uint64相兼容(注意如果值不相兼容则会被截断)，然而在客户端反序列化后可能会有不同的处理方式，例如，未识别的proto3枚举类型会被保留在消息中，但表示方式会依照语言而定。int类型的字段总会保留他们的

J、可以添加新的optional或repeated的字段, 但必须使用新的标识符(消息中从未使用过的标识符，不能使用已经被删除过的标识符)。



## 文件编码规范

Proto文件编码规范如下：

A、描述文件以.proto做为文件后缀。

B、除结构定义外的语句以分号结尾，结构定义包括：message、service、enum；rpc方法定义结尾的分号可有可无。

C、Message命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式。

D、Enums类型名采用驼峰命名方式，字段命名采用大写字母加下划线分隔方式。

E、Service与rpc方法名统一采用驼峰式命名。



# 序列化原理解析



## 大端和小端 - 字节的排列方式

一个多位的整数，按照存储地址从低到高排序的字节中

在网络应用中，因为不同机器类型可能采用不同标准的字节序，所以均按照网络标准转化

![](..\images\大端模式 和 小端模式.png)

0x1234abcd 从右往左 = 数据位数 从低往高 开始存放



12（最高位数据） ... ... cd（最低位数据）

低地址（左）... ...  高地址（右）



小端：x86 结构

​	cd 在 低地址 - 低地址（左）中存放的是 低位

​	ab 在 高地址 - 高地址（右）中存放的是 高位



大端：

​	cd 在 高地址 - 高地址（右）中存放的是 低位

​	ab 在 低地址 - 低地址（左）中存放的是 高位





## 序列化原理简介



### 序列化

序列化：将数据结构或对象转换成二进制字节流的过程

高效紧凑数据压缩：不同的字段类型 采用 不同的编码方式和数据存储方式



Protobuf 序列化过程如下：

**(有值 - 根据标识号和数据类型按照不同编码方式编码 - 字段类型不同的数据存储方式封装成二进制数据流)**

（1）判断每个字段是否有设置值，有值才进行编码。

（2）根据字段标识号与数据类型将字段值通过不同的编码方式进行编码。

（3）将编码后的数据块按照字段类型采用不同的数据存储方式封装成二进制数据流。



### 反序列化

二进制字节流转换成数据结构或者对象

Protobuf 反序列化过程如下：

**（parseFrom 解析 - 数据塞入结构）**

（1）调用消息类的 **parseFrom(input)** 解析从输入流读入的二进制字节数据流。

（2）将解析出来的数据按照指定的格式读取到 Java、C++、Phyton 对应的结构类型中。





## 补码 - 计算机 按照补码表示整数

**原码 取反 + 1 = 补码**



1、通常把一个数的最高为定义为符号位，用“0”表示正，用“1”表示负。

因为【－1】为负，所以【－1】的原码＝10000001

2、反码：对于负数，数符位为1，数符位不变，将数值位诸位取反为反码。

【－1】的反码＝11111110

3、补码：对于负数，数符位为1，数符位不变，将反码＋1＝补码。

【－1】的补码＝11111111

用不同二进制编码方式表示有符号数时，所得到的机器数可能不一样，但是真值是相同的。



## 编码方式

1个字节（Byte）等于8位（b）二进制



Protobuf 定义 sint32/sint64 表示负数：**Zigzag 编码 + Varint 编码**

先采用 Zigzag 编码：将有符号数转换成无符号数

再采用 Varint 编码，减少编码后的字节数



### Varint



#### 原理

用字节表示数字，值越小的数字使用越少的字节数表示

减少表示数字的字节数进行数据压缩



int32：一般4个字节 - Varint 编码 => 很小的 int32 ：1个字节表示

​														  => 很大的 int32：5个字节表示

大多数情况下消息不会有很大数字，Varint 总是可以用很少的字节数表示数字



#### 负数的表示

负数表示 会 被当成很大的整数：

因为计算机定义负数的符号位为数字的最高位，所以 计算机内 负数 被表示为 很大的整数

如果使用 Varint 编码方式表示一个负数，就一定需要 5个字节

负数的最高位是1，会被当做很大的整数处理



#### 最高有效位 - MSB

1：后续 byte 是该数字的一部分

0：后续 byte 直到第一个 1 之前都不是该数字的一部分



#### 编码举例

比如：值 300 的 Varint 编码

二进制 100101100 = 256 + 32 + 8 + 4 = 300

300 二进制为 0001 0010 1100

先按每7位一组分好 00010 0101100

再逆序(小端排序) 0101100 00010

然后再将每组加上最高有效位并填充到8位（1个字节），就变成了 10101100 00000010



**最高有效位的特殊含义：**

1 = 后续的字节也是数字的一部分

0 = 本字节是最后一个字节，其他7位都用来表示数字



解码是 编码的逆过程

这个字节读取到最高位 = 0：

​	这个字节是一个值 经过Varint编码后得到的字节流的最后一个字节



#### 总结：编码与解码

Varint 编码

- a.将整数的二进制按 7bit 分组，从低位到高位依次排列

- b.最后一组 MSB 设置 0，其他组的 MSB 设置为 1

  

Varint 编码

- a.去掉 MSB 重新组合 7bit 字符列表
- b.逆序存储 7bit 字符列表



#### 图解

变长编码的最大挑战是要找到每个字段边界

解决：

​	用连续字节的 msb = 1，表示后续的字节仍然是这个数字。当首 msb = 0，表示结束。

如：

源： 0x8888 1000 1000 1000 1000

编： 0x029188 <0 结束>000 0010 <1 连续>001 0001  <1 连续>000 1000



源：0xE8E8E8 1110 1000 1110 1000 1110 1000

编：0x07A3D1E8 <0 结束>000 0111 <1 连续>010 0011 <1 连续>101 0001 <1 连续>110 1000



```
message  S3
{
    optional int32 s3_2 = 2;      //设置为0x8888
    optional uint32 s3_3 = 3;     //设置为0xE8E8E8
}
```

![](..\images\Protobuf 图解 Varint.png)

![](..\images\Protobuf 图解 Varint_2.png)





### Zigzag



#### 原理

有符号 => 无符号

使用无符号数来表示有符号数字

使得绝对值小的数字可以使用较少字节来表示

对负数表示的数据能够更好的进行数据压缩



Zigzag编码 对 Varint编码 在表示负数时不足的补充



**算术左移低位补0；**

**算术右移，若符号位为0，高位补0；若符号位为1，高位补1；**



#### zigzag 编码



负数：如 -1

11111111 11111111 11111111 11111111

1、数据位整体左移1位 补0

n<<1    a = 11111111 11111111 11111111 1111111<0>

2、符号位移到最低位

右移31位 - 负数的算数右移高位补1

n>>31   b = 11111111 11111111 11111111 1111111<1>

3、

c = a ^ b

a 异或 b

c = 00000000 00000000 0000000 00000001



因为a中数据位 冗余的前导1 刚好与b中数据位 冗余的前导1 相对应，那么进行异或操作时，就能将这些 冗余的前导1 消除掉，数据位完成了 取反 动作，这样便能压缩数据了



正数：如 1

00000000 00000000 00000000 00000001

1、数据位整体左移1位

n<<1    a = 00000000 00000000 00000000 000000<1>0

2、符号位移到最低位

n>>31   b = 00000000 00000000 00000000 00000000

右移31位 - 非负数的算数右移高位补0

3、

c = a ^ b

a 异或 b

00000000 00000000 00000000 00000010

1“编码”成了2

对于正整数，异或操作没有影响



3、如 0

0^0还是0



#### zigzag 解码



对于zigzag编码的值2，(00000000 00000000 00000000 00000010)补

1、

n >> 1， a = (00000000 00000000 00000000 00000001)补，整体右移，还原数据位

2、

-(n&1)，b = -(2&1)10 = -(0)10 = (00000000 00000000 00000000 00000000)补，最低位按位与，取负号，还原符号位

3、

c=a^b，c = (00000000 00000000 00000000 00000001)补，将zigzag编码值2 解码还原成了1



对于zigzag编码的值3，(00000000 00000000 00000000 00000011)补

1、

n >> 1， a = (00000000 00000000 00000000 00000001)补，整体右移，还原数据位

2、

-(n&1)，b = -(3&1)10 = -(1)10 = (11111111 11111111 11111111 11111111)补，最低位按位与，取负号，还原符号位

3、

c=a^b，c = (11111111 11111111 11111111 11111110)补，将zigzag编码值3 解码还原成了-2

对于32位整数，(n>>1)^-(n&1)，即能实现zigzag解码。



#### 总结：编码与解码

zigzag编码：对于32位整数，**(n<<1)^(n>>31)**

zigzag解码：对于32位整数，**(n>>1)^-(n&1)**   或者  (n << 1) ^ (n >> 63)



#### 图解

编码：(n << 1) ^ (n >> 31)

解码：(n << 1) ^ (n >> 63)

| Signed Original | Encoded As |
| --------------- | ---------- |
| 0               | 0          |
| -1              | 1          |
| 1               | 2          |
| -2              | 3          |
| 2147483647      | 4294967294 |
| -2147483648     | 4294967295 |

```
message  S3
{
    optional sint32 s3_9 = 9;     //设置为0x8888
}
```

![](..\images\Protobuf 图解 zigzag.png)



### 4字节和8字节的固定长度编码

double，float, 这些都是IEEE规定好的格式。

fixed32，sfixed32，fixed64，fixed64，适合存放大数字数字。

![](..\images\Protobuf 4字节和8字节的固定长度编码.png)



repeated fixed32的编码：key重复出现

![](..\images\Protobuf repeated fixed32的编码.png)





### 变长LENGTH_DELIMITED

主要针对string类型、repeated类型和嵌套类型，对这些类型编码时需要存储他们的长度信息



#### string,bytes

string的编码还是key+value，只是value里面多了一个长度。

string的要求是UTF8的编码的。所以如果不是这个编码最好用bytes。

string的编码带入没有'\0'

```
message  S3
{
	//...
    optional string s3_19 = 19;   //设置为 "I love you,C++!"
}
```

![](..\images\Protobuf 图解 变长_1.png)



下图是repeated string：

```
message  S3
{
	//...
    repeated string s3_23 = 23;   //设置为"love","hate","C++"
}
```

![](..\images\Protobuf 图解 变长_2.png)



#### repeated Varint packed 

repeated 的 Varint 有带packed=true 时也是变长，带packed=true的描述会压缩更多，但和普通repeated模式不太一样。

下面的例子是带有packed的例子。

    message  S3
    {
    	//...
    	repeated int32 s3_21 = 21;    //设置为3, 270, and 86942, 用google文档的例子
    	repeated int32 s3_22 = 22 [packed = true]; //设置为3, 270, and 86942
    }

![](..\images\Protobuf 图解 变长_3.png)



下面是不带packe=true的例子。

![](..\images\Protobuf 图解 变长_4.png)



#### 内嵌类

```
message S2
{
    optional int32 s2_1 = 1;
    optional string s2_2 = 2 ;
}

message  S3
{
	//... ...
    optional S2 s3_24 = 24;       //设置为 0x1,"love"
    repeated S2 s3_25 = 25;       //设置为 0x16,"love"  and 0x16,"hate"
   	//... ...
}
```

假如：s3_24｛1，"love"｝

内嵌类里面的编码方式和外部一样，只是内嵌类的tag使用其自己的tag

![](..\images\Protobuf 变长 内嵌类.png)





假如：repeated S2 s3_25 设置为｛22，"love"｝，{22,"hate"｝

![](..\images\Protobuf 变长 repeated 内嵌类.png)



## 数据存储方式



### 总结

一个消息由多个字段组成



每个字段 = **wire_type** + field_number + Length（可选） + Value

**不同 字段数据类型（wire_type）决定 数据存储方式（T-L-V、T-V） 和 数据存储方式中的T、L、V 中的 不同的编码（Varint 或 其他编码）** 





### T-L-V 数据存储方式 （标识符 - 长度 - 字段值）

​	用这种存储方式将所有数据拼接成一个字节流

​	Length：可选存储 存储 Varint 编码数据 不需要存储 Length，此时为 T-V 存储方式

![](..\images\Protobuf T-L-V 数据存储方式.webp)

T-L-V 存储方式的优点：
A、不需要分隔符就能分隔开字段，减少了分隔符的使用。
B、各字段存储得非常紧凑，存储空间利用率非常高。
C、如果某个字段没有被设置字段值，那么该字段在序列化时的数据中是完全不存在的，即不需要编码，相应字段在解码时才会被设置为默认值。



### T-V 数据存储方式

标识号、数据类型、字段值 经过 采用 Varint 和 Zigzag编码后，以T-V（Tag-Value）方式进行数据存储

省略了T-L-V中的字节长度Length

![](..\images\Protobuf T-V 数据存储方式.webp)



Tag = 字段数据类型（wire_type） + 标识号（field_number）

​			3bit										+  4bit

​	占用一个字节的长度（如果标识符大于15，则占用多一个字节的位置）

​	最高位用于Varint编码保留

```
Tag = (field_number << 3) | wire_type
enum WireType { 
      WIRETYPE_VARINT = 0, 
      WIRETYPE_FIXED64 = 1, 
      WIRETYPE_LENGTH_DELIMITED = 2, 
      WIRETYPE_START_GROUP = 3, 
      WIRETYPE_END_GROUP = 4, 
      WIRETYPE_FIXED32 = 5

   };
```



### 解码时 - 根据 Tag 读取 数据存储

根据 Tag 将 Value 对应于消息中的字段

```
message person
{ 
   required int32     id = 1;  
   // wire type = 0，field_number =1 
   
   required string    name = 2;  
   // wire type = 2，field_number =2 
 
 }
```



对于Person消息的name字段的Tag编码如下：

```
nameTag = 2 << 3 | 2
nameTag = 0001 0010
```



根据Tag解码得到filed_number、wire_type：

```
nameTag = 0001 0010

field_number = nameTag >> 3
field_number = 0010

wire_type = nameTag & 3
wire_type = 010
```



### ProtoBuf 数据存储三大原则

（1）将消息中的每个 **字段** 进行编码后，利用T - L - V 存储方式进行数据的存储，最终得到一个二进制字节流。

（2）对于不同 **字段数据类型（wire_type）** 采用不同的序列化方式（数据编码方式与数据存储方式）

​		**决定 不同的编码和数据存储方式** 对消息字段进行序列化，以确保得到高效紧凑的数据压缩

![](..\images\ProtoBuf 数据存储三大原则.webp)

​	

​	Varint编码数据的存储：不需要存储字节长度Length，使用T-V存储方式进行存储；

​	其它编码方式（如 变长）的数据：使用T-L-V存储方式进行存储。

（3）对于 **数据字段值** 的独特编码方式与T-L-V数据存储方式，使得 序列化后数据量体积极小。





### Tag 中 字段数据类型（wire_type） 指定编码方式和数据存储方式

#### 1、WireType=0 的序列化

WireType=0 的类型包括 **int32，int64，uint32，unint64，bool，enum以及sint32和sint64**。

编码方式：

​	Varint编码（如果为负数，采用Zigzag辅助编码）

数据存储方式：

​	T-V方式存储二进制字节流。



#### 2、WireType=1 的序列化

WireType=1 的类型包括 **fixed64，sfixed64，double**。

编码方式：

​	64bit编码（编码后数据大小为64bit，高位在后，低位在前）

数据存储方式：

​	T-V方式存储二进制字节流。



#### 3、WireType=2 的序列化

WireType=2 的类型包括 **string，bytes，嵌套消息，packed repeated字段**。

对于编码方式：

​	标识符Tag：采用Varint编码，

​	字节长度Length：Varint编码

​	字段值：string类型字段值采用UTF-8编码，嵌套消息类型的字段值根据嵌套消息内部的字段数据类型进行选择

数据存储：

​	T-L-V方式存储二进制字节流。



#### 4、WireType=5 的序列化

WireType=5 的类型包括 **fixed32，sfixed32，float**。

编码方式：

​	32bit编码（编码后数据大小为32bit，高位在后，低位在前）

数据存储：

​	T-V方式存储二进制字节流。



### 字段值 的序列化

1、String类型

String类型字段的值使用UTF-8编码。消息数据流如下：

2、嵌套消息类型
嵌套消息类型采用T-L-V的存储方式，外部消息的V即为嵌套消息的字段 ，在T-L-V的V中嵌套了一系列的T-L-V。
编码方式：字段值（即V）根据字段的数据类型采用不同编码方式。

3、通过packed修饰的 repeat 字段

如果序列化时对多个 T - V对存储（不带packed=true），则会导致Tag的冗余，即相同的Tag存储多次。

为了解决Tag数据冗余，采用带packed=true的repeated字段存储方式，即将相同的Tag只存储一次、添加repeated字段下所有字段值的长度Length、连续存储repeated字段值，组成一个大的Tag - Length - Value -Value -Value对，即T - L - V - V - V对。

通过采用带packed=true 的 repeated字段存储方式，从而更好地压缩序列化后的数据长度。



# 图解 Protobuf

```
message S2
{
    optional int32 s2_1 = 1;
    optional string s2_2 = 2 ;
}

enum E1 
{
    E1_1 = 1;
    E1_3 = 3;
    E1_5 = 5;
}

message  S3
{
    optional int32 s3_1 = 1;      //设置为0x88
    optional int32 s3_2 = 2;      //设置为0x8888
    optional uint32 s3_3 = 3;     //设置为0xE8E8E8
    optional uint32 s3_4 = 4;     //设置为0xE8E8E8E8
    optional int64 s3_5 = 5;      //设置为0x8888
    optional int64 s3_6 = 6;      //设置为0xE8E8E8E8
    optional uint64 s3_7 = 7;     //设置为0xE8E8E8E8
    optional uint64 s3_8 = 8;     //设置为0xE8E8E8E8E8E8E8E8
    optional sint32 s3_9 = 9;     //设置为0x8888
    optional sint32 s3_10 = 10;   //设置为-0x8888
    optional sint64 s3_64 = 64;   //注意这个tag id  设置为0xE8E8E8E8
    optional sint64 s3_65 = 65;   //注意这个tag id  设置为-0xE8E8E8E8
    optional E1 s3_11 = 11;       //设置为E1_5
    optional bool s3_12 = 12;     //设置为true
    optional float s3_13 = 13;    //设置 float，设置为88.888
    optional fixed32 s3_14 = 14;  //设置为 0x8888
    optional sfixed32 s3_15 = 15; //设置为 -0x8888
    optional double s3_16 = 16;   //设置 double，设置为8888.8888
    optional fixed64 s3_17 = 17;  //设置为 0x8888888888
    optional sfixed64 s3_18 = 18; //设置为 -0x8888888888
    optional string s3_19 = 19;   //设置为 "I love you,C++!"
    optional bytes s3_20 = 20;    //设置为 "I hate you,C++!"
    repeated int32 s3_21 = 21;    //设置为3, 270, and 86942, 用google文档的例子
    repeated int32 s3_22 = 22 [packed = true]; //设置为3, 270, and 86942
    repeated string s3_23 = 23;   //设置为"love","hate","C++"
    optional S2 s3_24 = 24;       //设置为 0x1,"love"
    repeated S2 s3_25 = 25;       //设置为 0x16,"love"  and 0x16,"hate"
    repeated fixed32 s3_26 = 26;  //设置为1，2，3
    optional int32 s3_27 = 27;    //不设置
}
```



| **分类说明**                                                 | **定义**            | **TAG** | **WriteType**                        | **设置的值**                                                 | **编码后的16进制数据 KEY+(LENGTH)+VLAUE**                   | **函数**                                      |
| ------------------------------------------------------------ | ------------------- | ------- | ------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- | :-------------------------------------------- |
| **VALUE用VARINT表示 VARINT（0）**                            | **optional int32**  | **1**   | **0**                                | **0x88**                                                     | **08 88 01**                                                | WriteInt32ToArray                             |
| **optional int32**                                           | **2**               | **0**   | **0x8888**                           | **10 88 91 02**                                              | WriteInt32ToArray                                           |                                               |
| **optional uint32**                                          | **3**               | **0**   | **0xE8E8E8**                         | **18 e8 d1 a3 07**                                           | WriteUInt32ToArray                                          |                                               |
| **optional uint32**                                          | **4**               | **0**   | **0xE8E8E8E8**                       | **20 e8 d1 a3 c7 0e**                                        | WriteUInt32ToArray                                          |                                               |
| **optional int64**                                           | **5**               | **0**   | **0x8888**                           | **28 88 91 02**                                              | WriteInt64ToArray                                           |                                               |
| **optional int64**                                           | **6**               | **0**   | **0xE8E8E8E8**                       | **30 e8 d1 a3 c7 0e**                                        | WriteInt64ToArray                                           |                                               |
| **optional uint64**                                          | **7**               | **0**   | **0xE8E8E8E8**                       | **38 e8 d1 a3 c7 0e**                                        | WriteUInt64ToArray                                          |                                               |
| **optional uint64**                                          | **8**               | **0**   | **0xE8E8E8E8E8E8E8E8**               | **40 e8 d1 a3 c7 8e 9d ba f4 e8 01**                         | WriteUInt64ToArray                                          |                                               |
| **optional sint32**                                          | **9**               | **0**   | **0x8888**                           | **48 90 a2 04**                                              | WriteSInt32ToArray                                          |                                               |
| **optional sint32**                                          | **10**              | **0**   | **-0x8888**                          | **50 8f a2 04**                                              | WriteSInt32ToArray                                          |                                               |
| **optional E1（enum）**                                      | **11**              | **0**   | **E1_5**                             | **58 05**                                                    | WriteEnumToArray                                            |                                               |
| **optional bool**                                            | **12**              | **0**   | **true**                             | **60 01**                                                    | WriteBoolToArray                                            |                                               |
| **VALUE固定4个字节 FIXED32（5）**                            | **optional float**  | **13**  | **5**                                | **88.888**                                                   | **6d a8 c6 b1 42**                                          | WriteFloatToArray                             |
| **optional fixed32**                                         | **14**              | **5**   | **0x8888**                           | **75 88 88 00 00**                                           | WriteFixed32ToArray                                         |                                               |
| **optional sfixed32**                                        | **15**              | **5**   | **-0x8888**                          | **7d 78 77 ff ff**                                           | WriteSFixed32ToArray                                        |                                               |
| **VALUE固定8个字节 FIXED64（1）**                            | **optional double** | **16**  | **1**                                | **8888.8888**                                                | **81 01 58 ca 32 c4 71 5c c1 40**                           | WriteDoubleToArray                            |
| **optional fixed64**                                         | **17**              | **1**   | **0x8888888888**                     | **89 01 88 88 88 88 88 00 00 00**                            | WriteFixed64ToArray                                         |                                               |
| **optional sfixed64**                                        | **18**              | **1**   | **-0x8888888888**                    | **91 01 78 77 77 77 77 ff ff ff**                            | WriteSFixed64ToArray                                        |                                               |
| **repeated,message,string,btyes类的有长度的编码 LENGTH_DELIMITED（2）** | **optional string** | **19**  | **2**                                | **"I love you,C++!"**                                        | **9a 01 0f 49 20 6c 6f 76 65 20 79 6f 75 2c 43 2b 2b 21**   | VerifyUTF8StringNamedField WriteStringToArray |
| **optional bytes**                                           | **20**              | **2**   | **"I hate you,C++!"**                | **a2 01 0f 49 20 68 61 74 65 20 79 6f 75 2c 43 2b 2b 21**    | WriteBytesToArray                                           |                                               |
| **repeated int32 (对比)**                                    | **21**              | **0**   | **3,270,86942**                      | **a8 01 03 a8 01 8e 02 a8 01 9e a7 05**                      | WriteInt32ToArray                                           |                                               |
| **repeated int32 [packed=true]**                             | **22**              | **2**   | **3,270,86942**                      | **b2 01 06 03 8e 02 9e a7 05**                               | WriteTagToArray WriteVarint32ToArray WriteInt32NoTagToArray |                                               |
| **repeated string**                                          | **23**              | **2**   | **"love","hate","C++"**              | **ba 01 04 6c 6f 76 65 ba 01 04 68 61 74 65 ba 01 03 43 2b 2b** | VerifyUTF8StringNamedField WriteStringToArray               |                                               |
| **optional S2(message)**                                     | **24**              | **2**   | **{0x1,"love"}**                     | **c2 01 08 08 01 12 04 6c 6f 76 65**                         | WriteMessageNoVirtualToArray                                |                                               |
| **repeated S2**                                              | **25**              | **2**   | **S2{0x16,"love"} ,S2{0x16,"hate"}** | **ca 01 08 08 16 12 04 6c 6f 76 65 ca 01 08 08 16 12 04 68 61 74 65** | WriteMessageNoVirtualToArray                                |                                               |
| **repeated fixed32（对比）**                                 | **26**              | **5**   | **1,2,3**                            | **d5 01 01 00 00 00 d5 01 02 00 00 00 d5 01 03 00 00 00**    | WriteFixed32ToArray                                         |                                               |
| **可选没有设置**                                             | **optional int32**  | **27**  | **0**                                | **没有设置**                                                 | **没有数据**                                                |                                               |
| **数据是安装tag排序进行编码的**                              | **optional sint64** | **64**  | **0**                                | **0xE8E8E8E8**                                               | **80 04 90 a2 04**                                          | WriteSInt64ToArray                            |
| **optional sint64**                                          | **65**              | **0**   | **-0xE8E8E8E8**                      | **88 04 8f a2 04**                                           | WriteSInt64ToArray                                          |                                               |



## 整体编码 - key-value

每个字段按照 tag 顺序进行编码

字段都采用  **key-value** 的方式保存编码数据，保证了高低版本之间的兼容性

1、字段没有被设置 optional 或者 repeated，在编码的数据中完全不存在。相应的字段在解码的时候回设置为默认值

2、字段有 required 标识 但没有被设置，在 IsInitialized()检查会失败

编码的顺序和元数据.proto文件内fields的定义数据无关，而是 **根据tag的从小到大的顺序进行的编码**

整体数据都是 变长 的，而且有一定的自描述能力

**核心：能够识别出每一个Key，value，（Length）**

![](..\images\Protobuf Message 编码组成.png)



## 编码时对类型的再归类 - WireType



编码通过 WireType 进行归类：

| **Type，** | **枚举定义，WireType**    | **Meaning**      | **对应的protobuf类型**                                   | **编码长度**                         |
| ---------- | ------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------ |
| 0          | WIRETYPE_VARINT           | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum | 变长，1-10个字节，用VARINT编码且压缩 |
| 1          | WIRETYPE_FIXED64          | 64-bit           | fixed64, sfixed64, double                                | 固定8个字节                          |
| 2          | WIRETYPE_LENGTH_DELIMITED | Length-delimited | string, bytes, embedded messages, packed repeated fields | 变长，在key后会跟一个长度定义        |
| 3          | WIRETYPE_START_GROUP      | Start group      | groups (deprecated)                                      | 已经要废弃了，不看也罢               |
| 4          | WIRETYPE_END_GROUP        | End group        | groups (deprecated)                                      |                                      |
| 5          | WIRETYPE_FIXED32          | 32-bit           | fixed32, sfixed32, float                                 | 固定4个字节                          |



### Key - 编码使用 varint

Key = varint（fields_tag <<3 |WireType）

fields_tag：元数据描述.proto文件里面的tag。

WireType：字段类型对应的WireType的枚举值。



当类型 varint 整数数组 （比如repeated int32 ）：

​	如果不加packed=true修饰时，key=varint fields_tag<<3|WriteType ：0），视 WireType为 varint 

​	如果加上packed=true修饰时，仍然KEY = varint （fields_tag<<3|WireType:2）,视类型为 Length-delimited



用字段s3_17的key举例

fixed64 => WireType=1

```
message  S3
{
    optional fixed64 s3_17 = 17;  //设置为 0x8888888888
}
```

![](..\images\Protobuf 编码 Key.png)



### Length - 编码 使用 varint

Length = varint（fields_tag <<3 |WireType）

fields_tag：元数据描述.proto文件里面的tag。

WireType：字段类型对应的WireType的枚举值。





## 解码

proto的解码就是

1、找到key，根据key找到tag（代码里面叫fieldnumber），

2、根据tag进行解码，因为编码是KV的，编码本事有一定的防错性。

