# entry-task（用户管理系统)技术设计文档



## 一、背景及目的

  通过实现简单用户管理系统，帮忙团队新人快速熟悉团队的技术栈,希望能达到几个目的:

- 熟悉简单的Web API后台架构
- 熟悉使用Go实现HTTP API（JSON、文件）
- 熟悉使用Go实现基于TCP的RPC框架（设计和实现通信协议）
- 熟悉基于Auth Token的鉴权机制和流程
- 熟悉使用Go对MySQL、Redis进行基本操作
- 对任务进度和时间有所意识
- 对代码规范、测试、文档、性能调优需要有所意识


## 二、逻辑架构设计
![系统框架图](https://github.com/wxquare/enry_task/blob/master/doc/images/1.png)

### 主要模块功能:
- 用户通过浏览器向web server发送http请求，登录请求，查看用户信息、编辑用户信息
- Webserver 接收请求，解析参数，然后构造rpc请求tcpserver
- tcpserver 中系统的主要逻辑，负责查询db、cache鉴权，查询和写用户的信息
- mysql 存储用户的信息，redis 缓存用户信息以及token信息

### 主要接口：
- login：登录接口
- logout：退出登录
- getuserinfo：获取用户信息
- edituserinfo：编辑用户昵称
- uploadpic：上传用户图像


## 三、核心逻辑详细设计

### 1、用户登录时序图
![用户登录时序图](https://github.com/wxquare/enry_task/blob/master/doc/images/2.png)

### 2、RPC 框架的设计
![rpc框架设计](https://github.com/wxquare/enry_task/blob/master/doc/images/3.png)

#### RPC 通用协议
``` 
   // RPCdata 表示rpcclient和server之间的通用协议
  type RPCdata struct {
    Name string        // 函数名称
    Args []interface{} // 请求或者返回参数
    Err  string        // 执行过程中的错误信息
  }
```
#### RPC 数据编解码

```
  func Encode(data RPCdata) ([]byte, error) {
    // serialize
    var serial bytes.Buffer
    encoder := gob.NewEncoder(&serial)
    if err := encoder.Encode(data); err != nil {
      return nil, err
    }
    // encode
    encodeData := make([]byte, 4+len(serial.Bytes()))
    binary.BigEndian.PutUint32(encodeData[:4], uint32(len(serial.Bytes())))
    copy(encodeData[4:], serial.Bytes())
    return encodeData, nil
  }
```
### 3、池化设计
   为减少资源的消耗httpserver和tcpserver之间采用连接池来实现链接的复用，提高系统的性能

```
  type GPool struct {
    factory  Factory       // factory method to create connection
    conns    chan Conn     // connections' chan
    init     uint32        // init pool size
    capacity uint32        // max pool size
    maxIdle  time.Duration // how long an idle connection remains open
    rwl      sync.RWMutex  // read-write mutex
}
```
- 系统启动时，tcpserver需要先启动，使其处于监听状态
- httpserver初始化连接池，创建与tcpserver的连接conn
- 链接conn存储在channel中
- httpserver请求tcpserver时，从连接池获取有效链接conn，通过该conn与tcpserver数据传输
- 一次请求结束后，将该conn放回连接池conns
- 为每个链接设置空闲时间maxIdle
- 限制连接池的容量capacity


### 4、存储的淘汰和更新机制


## 四、接口设计
给出对外接口描述，包括名称、地址或URL、类型（GRPC/HTTPS等）、协议内容（PB/JSON等）、各参数（类型、描述、限制）等；
对外接口需要给出鉴权机制和其他安全性考虑；


## 五、存储设计
可包括：
1、数据库表定义、字段定义、索引、主/备库读写等；
2、缓存KV设计、加载/更新/失效逻辑等；



## 七、部署方案与环境要求
可包括：

1、配置初始化、更改、下发、推送等；
2、各种存储容量的预估、需要扩容的实例、备库、账号要求等；
3、接入层、逻辑层实例数；



## 八、遗留问题与风险预估
可包括：
1、本次方案受限于时间、人力、外部因素等原因，未充分设计或者实现，可能带来的影响，以及下阶段的改进计划。
2、受限于外部依赖限制、硬件资源、网络等不可控条件，存在的运行和运营风险。


## 十、附录
可包括一些附带的内容，例如引用的文档链接、提供的操作手册附件等。
