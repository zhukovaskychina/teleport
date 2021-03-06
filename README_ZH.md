# Teleport [![GitHub release](https://img.shields.io/github/release/henrylee2cn/teleport.svg?style=flat-square)](https://github.com/henrylee2cn/teleport/releases) [![report card](https://goreportcard.com/badge/github.com/henrylee2cn/teleport?style=flat-square)](http://goreportcard.com/report/henrylee2cn/teleport) [![github issues](https://img.shields.io/github/issues/henrylee2cn/teleport.svg?style=flat-square)](https://github.com/henrylee2cn/teleport/issues?q=is%3Aopen+is%3Aissue) [![github closed issues](https://img.shields.io/github/issues-closed-raw/henrylee2cn/teleport.svg?style=flat-square)](https://github.com/henrylee2cn/teleport/issues?q=is%3Aissue+is%3Aclosed) [![GoDoc](https://img.shields.io/badge/godoc-reference-blue.svg?style=flat-square)](http://godoc.org/github.com/henrylee2cn/teleport) [![view examples](https://img.shields.io/badge/learn%20by-examples-00BCD4.svg?style=flat-square)](https://github.com/henrylee2cn/teleport/tree/master/samples) [![view Go网络编程群](https://img.shields.io/badge/官方QQ群-Go网络编程(42730308)-27a5ea.svg?style=flat-square)](http://jq.qq.com/?_wv=1027&k=fzi4p1)


Teleport是一个通用、高效、灵活的Socket框架。

可用于Peer-Peer对等通信、RPC、长连接网关、微服务、推送服务，游戏服务等领域。


![Teleport-Architecture](https://github.com/henrylee2cn/teleport/raw/master/doc/teleport_architecture.png)


## 性能测试

**测试用例**

- 一个服务端与一个客户端进程，在同一台机器上运行
- CPU:    Intel Xeon E312xx (Sandy Bridge) 16 cores 2.53GHz
- Memory: 16G
- OS:     Linux 2.6.32-696.16.1.el6.centos.plus.x86_64, CentOS 6.4
- Go:     1.9.2
- 信息大小: 581 bytes
- 信息编码：protobuf
- 发送 1000000 条信息

**测试结果**

- teleport

| 并发client | 平均值(ms) | 中位数(ms) | 最大值(ms) | 最小值(ms) | 吞吐率(TPS) |
| -------- | ------- | ------- | ------- | ------- | -------- |
| 100      | 1       | 0       | 16      | 0       | 75505    |
| 500      | 9       | 11      | 97      | 0       | 52192    |
| 1000     | 19      | 24      | 187     | 0       | 50040    |
| 2000     | 39      | 54      | 409     | 0       | 42551    |
| 5000     | 96      | 128     | 1148    | 0       | 46367    |

- teleport/socket

| 并发client | 平均值(ms) | 中位数(ms) | 最大值(ms) | 最小值(ms) | 吞吐率(TPS) |
| -------- | ------- | ------- | ------- | ------- | -------- |
| 100      | 0       | 0       | 14      | 0       | 225682   |
| 500      | 2       | 1       | 24      | 0       | 212630   |
| 1000     | 4       | 3       | 51      | 0       | 180733   |
| 2000     | 8       | 6       | 64      | 0       | 183351   |
| 5000     | 21      | 18      | 651     | 0       | 133886   |

**[test code](https://github.com/henrylee2cn/rpc-benchmark/tree/master/teleport)**

- CPU火焰图 teleport/socket

![tp_socket_cpu_torch](https://github.com/henrylee2cn/teleport/raw/master/doc/tp_socket_torch.png)

**[svg file](https://github.com/henrylee2cn/teleport/raw/master/doc/tp_socket_torch.svg)**

## 1. 版本

| 版本   | 状态      | 分支                                       |
| ---- | ------- | ---------------------------------------- |
| v3   | release | [v3](https://github.com/henrylee2cn/teleport/tree/master) |
| v2   | release | [v2](https://github.com/henrylee2cn/teleport/tree/v2) |
| v1   | release | [v1](https://github.com/henrylee2cn/teleport/tree/v1) |

## 2. 安装

```sh
go get -u github.com/henrylee2cn/teleport
```

## 3. 特性

- 服务器和客户端之间对等通信，两者API方法基本一致
- 支持定制通信协议
- 可设置底层套接字读写缓冲区的大小
- 底层通信数据包包含`Header`和`Body`两部分
- 支持单独定制`Header`和`Body`编码类型，例如`JSON` `Protobuf` `string`
- 数据包`Header`包含与HTTP header相同格式的元信息
- 支持推、拉、回复等通信方法
- 支持插件机制，可以自定义认证、心跳、微服务注册中心、统计信息插件等
- 无论服务器或客户端，均支持优雅重启、优雅关闭
- 支持实现反向代理功能
- 日志信息详尽，支持打印输入、输出消息的详细信息（状态码、消息头、消息体）
- 支持设置慢操作报警阈值
- 端点间通信使用I/O多路复用技术
- 支持设置读取包的大小限制（如果超出则断开连接）
- 提供Hander的上下文
- 客户端的Session支持断线后自动重连
- 支持的网络类型：`tcp`、`tcp4`、`tcp6`、`unix`、`unixpacket`等

## 4. 架构

### 4.1 名称解释

- **Peer：** 通信端点，可以是服务端或客户端
- **Socket：** 对net.Conn的封装，增加自定义包协议、传输管道等功能
- **Packet：** 数据包内容元素对应的结构体
- **Proto：** 数据包封包／解包的协议接口
- **Codec：** 用于`Packet.Body`的序列化工具
- **XferPipe：** 数据包字节流的编码处理管道，如压缩、加密、校验等
- **XferFilter：** 一个在数据包传输前，对数据进行加工的接口
- **Plugin：** 贯穿于通信各个环节的插件
- **Session：** 基于Socket封装的连接会话，提供的推、拉、回复、关闭等会话操作
- **Context：** 连接会话中一次通信（如PULL-REPLY, PUSH）的上下文对象
- **Pull-Launch：** 从对端Peer拉数据
- **Pull-Handle：** 处理和回复对端Peer的拉请求
- **Push-Launch：** 将数据推送到对端Peer
- **Push-Handle：** 处理同伴的推送
- **Router：** 通过请求信息（如URI）索引响应函数（Handler）的路由器


### 4.2 数据包内容

每个数据包的内容如下:

```go
// in socket package
type (
    // Packet a socket data packet.
    Packet struct {
        // packet sequence
        seq uint64
        // packet type, such as PULL, PUSH, REPLY
        ptype byte
        // URL string
        uri string
        // URL object
        url *url.URL
        // metadata
        meta *utils.Args
        // body codec type
        bodyCodec byte
        // body object
        body interface{}
        // newBodyFunc creates a new body by packet type and URI.
        // Note:
        //  only for writing packet;
        //  should be nil when reading packet.
        newBodyFunc NewBodyFunc
        // XferPipe transfer filter pipe, handlers from outer-most to inner-most.
        // Note: the length can not be bigger than 255!
        xferPipe *xfer.XferPipe
        // packet size
        size uint32
        // ctx is the packet handling context,
        // carries a deadline, a cancelation signal,
        // and other values across API boundaries.
        ctx context.Context
        // stack
        next *Packet
    }

    // NewBodyFunc creates a new body by header info.
    NewBodyFunc func(seq uint64, ptype byte, uri string) interface{}
)

// in xfer package
type (
    // XferPipe transfer filter pipe, handlers from outer-most to inner-most.
    // Note: the length can not be bigger than 255!
    XferPipe struct {
        filters []XferFilter
    }
    // XferFilter handles byte stream of packet when transfer.
    XferFilter interface {
        Id() byte
        OnPack([]byte) ([]byte, error)
        OnUnpack([]byte) ([]byte, error)
    }
)
```

### 4.3 通信协议

支持通过接口定制自己的通信协议：

```go
type (
    // Proto pack/unpack protocol scheme of socket packet.
    Proto interface {
        // Version returns the protocol's id and name.
        Version() (byte, string)
        // Pack writes the Packet into the connection.
        // Note: Make sure to write only once or there will be package contamination!
        Pack(*Packet) error
        // Unpack reads bytes from the connection to the Packet.
        // Note: Concurrent unsafe!
        Unpack(*Packet) error
    }
    ProtoFunc func(io.ReadWriter) Proto
)
```


接着，你可以使用以下任意方式指定自己的通信协议：

```go
func SetDefaultProtoFunc(socket.ProtoFunc)
type Peer interface {
    ...
    ServeConn(conn net.Conn, protoFunc ...socket.ProtoFunc) Session
    DialContext(ctx context.Context, addr string, protoFunc ...socket.ProtoFunc) (Session, *Rerror)
    Dial(addr string, protoFunc ...socket.ProtoFunc) (Session, *Rerror)
    Listen(protoFunc ...socket.ProtoFunc) error
    ...
}
```

## 5. 用法

### 5.1 Peer端点（服务端或客户端）示例

```go
// Start a server
var peer1 = tp.NewPeer(tp.PeerConfig{
    ListenAddress: "0.0.0.0:9090", // for server role
})
peer1.Listen()

...

// Start a client
var peer2 = tp.NewPeer(tp.PeerConfig{})
var sess, err = peer2.Dial("127.0.0.1:8080")
```


### 5.2 PullController模板示例

```go
type XxxPullController struct {
    tp.PullCtx
}
// XxZz register the route: /aaa/xx_zz
func (x *XxxPullController) XxZz(args *<T>) (<T>, *tp.Rerror) {
    ...
    return r, nil
}
// YyZz register the route: /aaa/yy_zz
func (x *XxxPullController) YyZz(args *<T>) (<T>, *tp.Rerror) {
    ...
    return r, nil
}
```

### 5.3 PushController模板示例

```go
type XxxPushController struct {
    tp.PushCtx
}
// XxZz register the route: /bbb/yy_zz
func (b *XxxPushController) XxZz(args *<T>) *tp.Rerror {
    ...
    return r, nil
}
// YyZz register the route: /bbb/yy_zz
func (b *XxxPushController) YyZz(args *<T>) *tp.Rerror {
    ...
    return r, nil
}
```

### 5.4 UnknownPullHandler模板示例

```go
func XxxUnknownPullHandler (ctx tp.UnknownPullCtx) (interface{}, *tp.Rerror) {
    ...
    return r, nil
}
```

### 5.5 UnknownPushHandler模板示例

```go
func XxxUnknownPushHandler(ctx tp.UnknownPushCtx) *tp.Rerror {
    ...
    return nil
}
```

### 5.6 插件示例

```go
// NewIgnoreCase Returns a ignoreCase plugin.
func NewIgnoreCase() *ignoreCase {
    return &ignoreCase{}
}

type ignoreCase struct{}

var (
    _ tp.PostReadPullHeaderPlugin = new(ignoreCase)
    _ tp.PostReadPushHeaderPlugin = new(ignoreCase)
)

func (i *ignoreCase) Name() string {
    return "ignoreCase"
}

func (i *ignoreCase) PostReadPullHeader(ctx tp.ReadCtx) *tp.Rerror {
    // Dynamic transformation path is lowercase
    ctx.Url().Path = strings.ToLower(ctx.Url().Path)
    return nil
}

func (i *ignoreCase) PostReadPushHeader(ctx tp.ReadCtx) *tp.Rerror {
    // Dynamic transformation path is lowercase
    ctx.Url().Path = strings.ToLower(ctx.Url().Path)
    return nil
}
```

### 5.7 注册以上操作和插件示例到路由

```go
// add router group
group := peer.SubRoute("test")
// register to test group
group.RoutePull(new(XxxPullController), NewIgnoreCase())
group.RoutePush(new(XxxPushController))
peer.SetUnknownPull(XxxUnknownPullHandler)
peer.SetUnknownPush(XxxUnknownPushHandler)
```

### 5.8 配置信息

```go
type PeerConfig struct {
    Network            string        `yaml:"network"              ini:"network"              comment:"Network; tcp, tcp4, tcp6, unix or unixpacket"`
    ListenAddress      string        `yaml:"listen_address"       ini:"listen_address"       comment:"Listen address; for server role"`
    DefaultDialTimeout time.Duration `yaml:"default_dial_timeout" ini:"default_dial_timeout" comment:"Default maximum duration for dialing; for client role; ns,µs,ms,s,m,h"`
    RedialTimes        int32         `yaml:"redial_times"         ini:"redial_times"         comment:"The maximum times of attempts to redial, after the connection has been unexpectedly broken; for client role"`
    DefaultBodyCodec   string        `yaml:"default_body_codec"   ini:"default_body_codec"   comment:"Default body codec type id"`
    DefaultSessionAge  time.Duration `yaml:"default_session_age"  ini:"default_session_age"  comment:"Default session max age, if less than or equal to 0, no time limit; ns,µs,ms,s,m,h"`
    DefaultContextAge  time.Duration `yaml:"default_context_age"  ini:"default_context_age"  comment:"Default PULL or PUSH context max age, if less than or equal to 0, no time limit; ns,µs,ms,s,m,h"`
    SlowCometDuration  time.Duration `yaml:"slow_comet_duration"  ini:"slow_comet_duration"  comment:"Slow operation alarm threshold; ns,µs,ms,s ..."`
    PrintBody          bool          `yaml:"print_body"           ini:"print_body"           comment:"Is print body or not"`
    CountTime          bool          `yaml:"count_time"           ini:"count_time"           comment:"Is count cost time or not"`
}
```

### 5.9 通信优化

- SetPacketSizeLimit 设置包大小的上限，
  如果 maxSize<=0，上限默认为最大 uint32

    ```go
    func SetPacketSizeLimit(maxPacketSize uint32)
    ```

- SetSocketKeepAlive 是否允许操作系统的发送TCP的keepalive探测包

    ```go
    func SetSocketKeepAlive(keepalive bool)
    ```


- SetSocketKeepAlivePeriod 设置操作系统的TCP发送keepalive探测包的频度

    ```go
    func SetSocketKeepAlivePeriod(d time.Duration)
    ```

- SetSocketNoDelay 是否禁用Nagle算法，禁用后将不在合并较小数据包进行批量发送，默认为禁用

    ```go
    func SetSocketNoDelay(_noDelay bool)
    ```

- SetSocketReadBuffer 设置操作系统的TCP读缓存区的大小

    ```go
    func SetSocketReadBuffer(bytes int)
    ```

- SetSocketWriteBuffer 设置操作系统的TCP写缓存区的大小

    ```go
    func SetSocketWriteBuffer(bytes int)
    ```


## 6. 完整示例

### server.go

```go
package main

import (
    "fmt"
    "time"

    tp "github.com/henrylee2cn/teleport"
)

func main() {
    svr := tp.NewPeer(tp.PeerConfig{
        CountTime:     true,
        ListenAddress: ":9090",
    })
    svr.RoutePull(new(math))
    svr.Listen()
}

type math struct {
    tp.PullCtx
}

func (m *math) Add(args *[]int) (int, *tp.Rerror) {
    if m.Query().Get("push_status") == "yes" {
        m.Session().Push(
            "/push/status",
            fmt.Sprintf("%d numbers are being added...", len(*args)),
        )
        time.Sleep(time.Millisecond * 10)
    }
    var r int
    for _, a := range *args {
        r += a
    }
    return r, nil
}
```

### client.go

```go
package main

import (
    tp "github.com/henrylee2cn/teleport"
)

func main() {
    tp.SetLoggerLevel("ERROR")
    cli := tp.NewPeer(tp.PeerConfig{})
    defer cli.Close()
    cli.RoutePush(new(push))
    sess, err := cli.Dial(":9090")
    if err != nil {
        tp.Fatalf("%v", err)
    }

    var reply int
    rerr := sess.Pull("/math/add?push_status=yes",
        []int{1, 2, 3, 4, 5},
        &reply,
    ).Rerror()

    if rerr != nil {
        tp.Fatalf("%v", rerr)
    }
    tp.Printf("reply: %d", reply)
}

type push struct {
    tp.PushCtx
}

func (p *push) Status(args *string) *tp.Rerror {
    tp.Printf("server status: %s", *args)
    return nil
}
```

[More](https://github.com/henrylee2cn/teleport/blob/master/samples)


## 7. 扩展包

### 编解码器

| package                                  | import                                   | description                  |
| ---------------------------------------- | ---------------------------------------- | ---------------------------- |
| [json](https://github.com/henrylee2cn/teleport/blob/master/codec/json_codec.go) | `import "github.com/henrylee2cn/teleport/codec"` | JSON codec(teleport own)     |
| [protobuf](https://github.com/henrylee2cn/teleport/blob/master/codec/protobuf_codec.go) | `import "github.com/henrylee2cn/teleport/codec"` | Protobuf codec(teleport own) |
| [string](https://github.com/henrylee2cn/teleport/blob/master/codec/string_codec.go) | `import "github.com/henrylee2cn/teleport/codec"` | String codec(teleport own)   |

### 插件

| package                                  | import                                   | description                              |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [auth](https://github.com/henrylee2cn/teleport/blob/master/plugin/auth.go) | `import "github.com/henrylee2cn/teleport/plugin"` | A auth plugin for verifying peer at the first time |
| [binder](https://github.com/henrylee2cn/tp-ext/blob/master/plugin-binder) | `import binder "github.com/henrylee2cn/tp-ext/plugin-binder"` | Parameter Binding Verification for Struct Handler |
| [heartbeat](https://github.com/henrylee2cn/tp-ext/blob/master/plugin-heartbeat) | `import heartbeat "github.com/henrylee2cn/tp-ext/plugin-heartbeat"` | A generic timing heartbeat plugin        |
| [proxy](https://github.com/henrylee2cn/teleport/blob/master/plugin/proxy.go) | `import "github.com/henrylee2cn/teleport/plugin"` | A proxy plugin for handling unknown pulling or pushing |

### 协议

| package                                  | import                                   | description                              |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [fastproto](https://github.com/henrylee2cn/teleport/blob/master/socket/protocol.go#L70) | `import "github.com/henrylee2cn/teleport/socket` | A fast socket communication protocol(teleport default protocol) |
| [jsonproto](https://github.com/henrylee2cn/tp-ext/blob/master/proto-jsonproto) | `import jsonproto "github.com/henrylee2cn/tp-ext/proto-jsonproto"` | A JSON socket communication protocol     |

### 传输过滤器

| package                                  | import                                   | description                              |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [gzip](https://github.com/henrylee2cn/teleport/blob/master/xfer/gzip.go) | `import "github.com/henrylee2cn/teleport/xfer"` | Gzip(teleport own)                       |
| [md5Hash](https://github.com/henrylee2cn/tp-ext/blob/master/xfer-md5Hash) | `import md5Hash "github.com/henrylee2cn/tp-ext/xfer-md5Hash"` | Provides a integrity check transfer filter |

### 其他模块

| package                                  | import                                   | description                              |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [cliSession](https://github.com/henrylee2cn/tp-ext/blob/master/mod-cliSession) | `import cliSession "github.com/henrylee2cn/tp-ext/mod-cliSession"` | Client session which has connection pool |
| [websocket](https://github.com/henrylee2cn/tp-ext/blob/master/mod-websocket) | `import websocket "github.com/henrylee2cn/tp-ext/mod-websocket"` | Makes the Teleport framework compatible with websocket protocol as specified in RFC 6455 |


[扩展库](https://github.com/henrylee2cn/tp-ext)

## 8. 基于Teleport的项目

| project                                  | description                              |
| ---------------------------------------- | ---------------------------------------- |
| [ant](https://github.com/henrylee2cn/ant) | Ant 是一套简单、灵活的基于 Teleport 的微服务框架          |
| [ants](https://github.com/xiaoenai/ants) | Ants 是一套基于 Ant 和 Teleport 的高可用的微服务架构解决方案 |
| [pholcus](https://github.com/henrylee2cn/pholcus) | Pholcus（幽灵蛛）是一款纯Go语言编写的支持分布式的高并发、重量级爬虫软件，定位于互联网数据采集，为具备一定Go或JS编程基础的人提供一个只需关注规则定制的功能强大的爬虫工具 |

## 9. 企业用户

[![深圳市梦之舵信息技术有限公司](https://statics.xiaoenai.com/v4/img/logo_zh.png)](http://www.xiaoenai.com)
&nbsp;&nbsp;
[![北京风行在线技术有限公司](http://static.funshion.com/open/static/img/logo.gif)](http://www.fun.tv)
&nbsp;&nbsp;
[![北京可即时代网络公司](http://simg.ktvms.com/picture/logo.png)](http://www.kejishidai.cn)

## 10. 开源协议

Teleport 项目采用商业应用友好的 [Apache2.0](https://github.com/henrylee2cn/teleport/raw/master/LICENSE) 协议发布
