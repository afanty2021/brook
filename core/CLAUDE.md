[根目录](../../CLAUDE.md) > **核心模块**

# Brook 核心模块

## 模块职责

Brook 核心模块是整个系统的基础，实现了高性能网络代理的核心功能，包括多种协议支持、加密通信、连接管理和数据转发等关键特性。

## 核心协议实现

### 1. StreamServer - 加密流协议

**文件位置**：`/Users/berton/Github/brook/streamserver.go`

**核心特性**：
- **加密算法**：AES-256-GCM 认证加密
- **密钥派生**：HKDF-SHA256 安全密钥派生
- **双工通信**：客户端和服务器双向加密通信
- **时间戳验证**：60秒请求过期机制
- **协议支持**：TCP 和 UDP 双协议支持

**安全机制**：
```go
// 客户端 nonce: 12字节随机数
cn := x.BP12.Get().([]byte)

// 服务器 nonce: 12字节随机数
sn := x.BP12.Get().([]byte)

// 32字节密钥通过 HKDF 派生
ck := hkdf.New(sha256.New, password, cn, ClientHKDFInfo)
sk := hkdf.New(sha256.New, password, sn, ServerHKDFInfo)
```

**性能优化**：
- 缓存池管理（BP12, BP32, BP2048, BP65507）
- 内存复用减少 GC 压力
- 分片读写优化 I/O 性能

### 2. PacketServer - UDP 包转发协议

**文件位置**：`/Users/berton/Github/brook/packetserver.go`

**核心特性**：
- **无状态设计**：每个数据包独立处理
- **动态密钥**：每包使用新的加密密钥
- **最大包长**：支持 65507 字节 UDP 包
- **高性能缓冲**：优化的大包处理能力

**数据包格式**：
```
[12字节随机数][加密的数据包]
```

### 3. WebSocket - 隧道传输协议

**文件位置**：`/Users/berton/Github/brook/websocket.go`

**核心特性**：
- **标准协议**：完整 WebSocket RFC 6455 实现
- **TLS 支持**：安全的 wss:// 连接
- **指纹伪装**：uTLS Chrome 指纹模拟
- **分片传输**：TLS 分片防检测技术

**TLS 分片实现**：
```go
type TLSFragmentConn struct {
    MinLength int64  // 最小分片长度
    MaxLength int64  // 最大分片长度
    MinDelay  int64  // 最小延迟(微秒)
    MaxDelay  int64  // 最大延迟(微秒)
}
```

**握手过程**：
1. 生成随机 WebSocket Key
2. 发送标准 HTTP Upgrade 请求
3. 验证服务器 101 响应
4. 检查 Sec-WebSocket-Accept 签名

### 4. QUIC - HTTP/3 代理协议

**文件位置**：`/Users/berton/Github/brook/quic.go`

**核心特性**：
- **HTTP/3 支持**：基于 QUIC 协议
- **双模式**：Stream 模式和 Datagram 模式
- **连接复用**：0-RTT 连接优化
- **自适应**：自动 TCP/UDP 地址转换

**连接模式**：
```go
// TCP over QUIC (Stream 模式)
func QUICDialTCP() -> QUICConn{Stream: quic.Stream}

// UDP over QUIC (Datagram 模式)
func QUICDialUDP() -> QUICConn{Stream: nil}
```

### 5. BrookLink - 统一链接格式

**文件位置**：`/Users/berton/Github/brook/brooklink.go`

**支持的链接类型**：
- `brook://server` - 基础 TCP/UDP 服务器
- `brook://socks5` - SOCKS5 代理
- `brook://wsserver` - WebSocket 代理
- `brook://wssserver` - 安全 WebSocket 代理
- `brook://quicserver` - QUIC 代理

**链接参数**：
```go
type BrookLink struct {
    Kind              string  // 链接类型
    Address           string  // 服务器地址
    Host              string  // HTTP Host 头
    Path              string  // WebSocket 路径
    Password          []byte  // 连接密码
    Tc                *tls.Config  // TLS 配置
    TLSFingerprint    utls.ClientHelloID  // TLS 指纹
}
```

## 核心组件架构

### Server - 代理服务器

**文件位置**：`/Users/berton/Github/brook/server.go`

**核心功能**：
- 并发连接处理
- TCP/UDP 双协议监听
- StreamServer 集成
- 超时管理和错误处理

**监听流程**：
```go
// TCP 监听
l, err := net.ListenTCP("tcp", addr)
for {
    c, err := l.AcceptTCP()
    ss, err := NewStreamServer(password, src, c, timeout, udptimeout)
    go ss.Handle()
}

// UDP 监听
l1, err := net.ListenUDP("udp", addr1)
for {
    // UDP 包处理逻辑
}
```

### Client - 代理客户端

**文件位置**：`/Users/berton/Github/brook/client.go`

**核心功能**：
- SOCKS5 服务器实现
- 连接转发和代理
- PacketConnFactory 连接复用
- 多协议支持

**SOCKS5 处理**：
```go
func TCPHandle() {
    // CONNECT 命令处理
    if r.Cmd == socks5.CmdConnect {
        sc, err := NewStreamClient(...)
        return sc.Exchange(c)
    }

    // UDP 命令处理
    if r.Cmd == socks5.CmdUDP {
        return r.UDP(c, x.Server.ServerAddr)
    }
}
```

## 加密和安全机制

### AES-GCM 加密
- **密钥长度**：256 位 (32 字节)
- **Nonce 长度**：96 位 (12 字节)
- **认证标签**：128 位 (16 字节)
- **性能**：硬件加速 AES-NI 支持

### HKDF 密钥派生
- **哈希函数**：SHA-256
- **输入材料**：密码 + Nonce + 上下文信息
- **输出长度**：32 字节
- **安全性**：RFC 5869 标准

### 时间戳验证
- **有效期**：60 秒
- **防重放**：随机 Nonce + 时间戳
- **时钟容忍**：允许一定的时钟偏差

## 性能优化技术

### 内存管理
- **对象池**：sync.Pool 缓存复用
- **零拷贝**：减少不必要的内存分配
- **缓冲区**：预分配固定大小缓冲区

### 并发处理
- **Goroutine 池**：控制并发数量
- **Channel 通信**：无锁数据传递
- **超时控制**：防止连接泄漏

### 网络优化
- **系统调用优化**：减少系统调用次数
- **TCP 调优**：SO_REUSEADDR, TCP_NODELAY
- **UDP 优化**：大包处理，MTU 发现

## 错误处理和日志

### 错误分类
- **网络错误**：连接超时、网络不可达
- **协议错误**：格式错误、版本不兼容
- **认证错误**：密码错误、证书验证失败
- **系统错误**：资源不足、权限问题

### 日志机制
- **结构化日志**：Error map 格式
- **分级记录**：Info、Warning、Error
- **上下文信息**：包含网络、地址、时间等

## 测试和验证

### 协议测试
- **SOCKS5 兼容性**：标准协议测试
- **加密通信**：端到端加密验证
- **性能测试**：吞吐量和延迟测试
- **并发测试**：大量连接处理能力

### 安全测试
- **密钥安全**：密钥强度和随机性测试
- **认证机制**：防重放和防篡改测试
- **漏洞扫描**：常见网络漏洞检查

## 配置和部署

### 系统要求
- **操作系统**：Windows、Linux、macOS、OpenWrt
- **Go 版本**：1.22+ (支持最新语言特性)
- **网络权限**：需要绑定端口和网络访问

### 性能调优
- **文件描述符**：提高系统限制
- **内存限制**：合理的内存使用
- **CPU 亲和性**：多核优化
- **网络参数**：TCP/UDP 参数调优

## 相关文件清单

- `streamserver.go` - 加密流协议实现
- `packetserver.go` - UDP 包转发协议
- `websocket.go` - WebSocket 隧道传输
- `quic.go` - QUIC/HTTP3 代理支持
- `brooklink.go` - 统一链接格式解析
- `server.go` - 代理服务器实现
- `client.go` - 代理客户端实现
- `exchange*.go` - 数据交换接口
- `dns*.go` - DNS 相关功能

## 变更记录 (Changelog)

### 2025-01-22
- 创建核心模块文档
- 深度分析所有核心协议实现
- 整理安全机制和性能优化技术
- 完成架构和设计模式文档

### 技术债务
- 考虑引入更多网络协议支持
- 优化内存使用和垃圾回收
- 增强错误恢复能力
- 改进监控和调试功能