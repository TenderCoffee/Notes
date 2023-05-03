# RPC 定义

远程过程调用

调用远端机器的函数或方法，且不需要关心底层的调用细节，如网络协议和传输协议等，对于调用者来说，和调用本地方法没有什么区别



必须实现以下几个能力：

## 服务发现 - 接口注册用于定位

定位到调用的服务器地址以及调用的具体方法的进程

通过 注册中心 或者 哈希查找，将被调的接口注册到某个地方

比如：

​	使用CallerId和函数名作为映射，调用者通过CallerId进行调用，服务端收到调用请求后，通过CallerId查找到对应函数，再执行后续处理流程



## 序列化和反序列化 - 二进制+协议编码解码

网络传输的是二进制数据

基本类型 => 二进制

二进制 => 基本类型

![](..\images\RPC序列化与反序列化.jpeg)

技术选型：

1、安全性：优先考虑被入侵的风险

2、通用和兼容：序列化协议在版本升级后的兼容性

3、空间开销：序列化后的字节数据体积越小，网络传输的数据量就越小，传输数据的速度也就越快

![](..\images\RPC序列化反序列化的选型标准.jpeg)



筛选后：

Hessian：使用更方便，在对象的兼容性上更好

Protobuf：更高效更通用



### 序列化方式1 - JDK - ObjectOutputStream+ObjectInputStream

```java
import java.io.*;

public class Student implements Serializable {
    //学号
    private int no;
    //姓名
    private String name;

    public int getNo() {
        return no;
    }

    public void setNo(int no) {
        this.no = no;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "no=" + no +
                ", name='" + name + '\'' +
                '}';
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        String home = System.getProperty("user.home");
        String basePath = home + "/Desktop";
        FileOutputStream fos = new FileOutputStream(basePath + "student.dat");
        Student student = new Student();
        student.setNo(100);
        student.setName("TEST_STUDENT");
        
        //序列化具体由 ObjectOutputStream 完成
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(student);
        oos.flush();
        oos.close();

        //反序列化的具体实现是由 ObjectInputStream 完成
        FileInputStream fis = new FileInputStream(basePath + "student.dat");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Student deStudent = (Student) ois.readObject();
        ois.close();

        System.out.println(deStudent);

    }
}
```

JDK 序列化过程：

写头部 + 写对象

**本质：不断加入一些特殊分隔符，反序列化时截断用**

![](..\images\JDK 序列化过程.jpeg)

1. 头部数据：序列化协议、序列化版本（兼容）

2. 对象数据：为了反序列化用的元数据 + 属性值

   为了反序列化用的元数据 ：类名、签名、（属性名、属性类型）、当然还有开头结尾等数据

   属性值：属于真正的对象值

3. 对象引用、继承：递归遍历“写对象”逻辑



将对象的类型、属性类型、属性值按固定格式写到二进制字节流中来完成序列化，再按固定格式读出对象的类型、属性类型、属性值，通过这些信息重建一个新的对象，完成反序列化。



缺点：JDK 序列化有安全漏洞，线上服务可能被入侵



### 序列化方式2 - JSON - 文本型序列化

1. 进行序列化的额外空间开销较大
2. 没有类型，但像Java这种强类型语言，需通过反射统一解决，性能不太好



传输的数据量要相对较小



### 序列化方式3 - Hessian

动态类型

二进制：生成的字节数更小

紧凑的：比JDK、JSON更加紧凑

性能高效：比JDK、JSON序列化高效

可跨语言



缺点：对Java里面一些常见对象的类型不支持



### 序列化方式4 - Protobuf

轻便、高效的结构化数据存储格式

结构化数据序列化

定义IDL + IDL编译器 => 生成 序列化工具类



1. 序列化后体积相比 JSON、Hessian小很多；
2. IDL能清晰地描述语义，所以足以帮助并保证应用程序之间的类型不会丢失，无需类似 XML 解析器；
3. 序列化反序列化速度很快，不需要通过反射获取类型；
4. 消息格式升级和兼容性不错，可以做到向后兼容。



缺点：对于具有反射和动态能力的语言来说，Protobuf 用起来很费劲



### 序列化的注意点

1、不要把对象构造太复杂相互嵌套，消耗性能CPU

2、List 或者 Map 太大会浪费性能耗费时间影响请求耗时

3、不要使用第三方集合类，使用原生、最常用的集合类 HashMap、ArrayList

4、不要有太复杂的继承关系



## 网络传输 - 一般TCP

一般使用TCP进行网络传输

HTTP、UDP 也可以



# 为什么使用 RPC - 拆分后服务间通信需要

在同一个机器上，将所有功能实现在一个服务中或者实现在多个服务中

服务功能迭代后，单体应用出现性能瓶颈

对服务根据业务功能进行拆分

将不同模块部署到不同集群上，模块进行通信完成功能



一个服务拆分为不同的模块，或者单体应用拆分为多个微服务后

RPC：不同模块以及不同服务之间通信需要



分布式系统架构 或者 微服务架构 必不可少的实现手段



# RPC原理 - 调用全过程（10步）的封装



![](..\images\RPC原理.jpeg)



当客户端发起一次远程调用时：

(1) 客户端（client）以本地调用方式（即以接口的方式）调用服务；

(2) 客户端存根（client stub）接收到调用后，负责将方法、参数等组装成能够进行网络传输的消息体（将消息体对象序列化为二进制）；

(3) 客户端通过sockets将消息发送到服务端；

(4) 服务端存根( server stub）收到消息后进行解码（将消息对象反序列化）；

(5) 服务端存根( server stub）根据解码结果调用本地的服务；

(6) 本地服务执行并将结果返回给服务端存根( server stub）；

(7) 服务端存根( server stub）将返回结果打包成消息（将结果消息对象序列化）；

(8) 服务端（server）通过sockets将消息发送到客户端；

(9) 客户端存根（client stub）接收到结果消息，并进行解码（将结果消息发序列化）；

(10) 客户端（client）得到最终结果。

RPC的目标是要把2、3、4、7、8、9这些步骤都封装起来。



RPC框架所要实现的便是将序列化、反序列化以及网络传输等流程进行**封装**，这些流程对于调用者来说是透明的，调用者并不需要了解这些细节以及调用流程。



# 常用 RPC 框架



## Thrift - 轻量级、跨语言

通过自身的IDL中间语言，并借助代码生成引擎生成各种主流语言的RPC服务端和客户端模板代码



## gRPC - 高性能、通用

面向移动应用开发 并 基于HTTP/2标准协议而设计，基于**ProtoBuf** 序列化协议开发

客户端充分利用高级流和链式功能，从而有助于节省带宽、降低TCP连接次数、节省CPU使用、电池寿命



## Dubbo - 分布式服务框架、SOA治理

高性能NIO通讯 及 多协议集成，服务动态寻址与路由，软负载均衡与容错，依赖分析与降级等



![](..\images\gRPC VS Thrift_1.png)

![](..\images\gRPC VS Thrift_2.png)



**什么时候应该选择gRPC而不是Thrift**

​	1、需要良好的文档、示例

​	2、喜欢、习惯HTTP/2、ProtoBuf

​	3、对网络传输带宽敏感



**什么时候应该选择Thrift而不是gRPC**

​	1、需要在非常多的语言间进行数据交换

​	2、对CPU敏感

​	3、协议层、传输层有多种控制要求

​	4、需要稳定的版本

​	5、不需要良好的文档和示例

