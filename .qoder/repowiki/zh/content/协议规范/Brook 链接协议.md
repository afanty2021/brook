# Brook 链接协议

<cite>
**本文档引用的文件**   
- [link.go](file://link.go)
- [brooklink.go](file://brooklink.go)
- [protocol/brook-link-protocol.md](file://protocol/brook-link-protocol.md)
- [protocol/brook-server-protocol.md](file://protocol/brook-server-protocol.md)
- [protocol/brook-wsserver-protocol.md](file://protocol/brook-wsserver-protocol.md)
- [protocol/brook-quicserver-protocol.md](file://protocol/brook-quicserver-protocol.md)
- [cli/brook/main.go](file://cli/brook/main.go)
</cite>

## 目录
1. [简介](#简介)
2. [链接结构与生成规则](#链接结构与生成规则)
3. [链接解析流程](#链接解析流程)
4. [不同代理模式的链接格式差异](#不同代理模式的链接格式差异)
5. [安全处理机制](#安全处理机制)
6. [命令行工具使用](#命令行工具使用)
7. [应用场景](#应用场景)
8. [错误处理与格式验证](#错误处理与格式验证)

## 简介

Brook 链接协议是一种用于配置和共享代理服务器连接信息的标准化格式。它通过统一的 URL 结构封装了服务器地址、端口、密码、协议类型等关键参数，支持多种代理模式（如 server、wsserver、quicserver 等）。该协议的设计目标是简化配置过程，提高可移植性，并确保连接信息的安全性。

Brook 链接以 `brook://` 为前缀，遵循 RFC3986 标准进行 URL 编码，确保特殊字符的正确传输。链接的核心组成部分包括协议类型（KIND）和查询参数（QUERY），其中查询参数以键值对形式存储配置信息。

**Section sources**
- [protocol/brook-link-protocol.md](file://protocol/brook-link-protocol.md#L1-L17)

## 链接结构与生成规则

Brook 链接的基本结构为 `brook://KIND?QUERY`，其中 KIND 表示代理模式，QUERY 是经过 URL 编码的键值对参数集合。

### 链接生成函数

链接的生成由 `Link` 函数实现，该函数接受三个参数：`kind`（代理模式）、`server`（服务器地址）和 `v`（URL 参数值）。函数将 `kind` 作为主机名，`server` 作为对应 `kind` 的参数值，最终组合成完整的链接字符串。

```go
func Link(kind, server string, v url.Values) string {
    v.Set(kind, server)
    return fmt.Sprintf("brook://%s?%s", kind, v.Encode())
}
```

例如，生成一个标准服务器链接的调用方式如下：
```bash
brook link --server 1.2.3.4:9999 --password hello --name 'my server'
```

这将生成形如 `brook://server?password=hello&server=1.2.3.4%3A9999&name=my%20server` 的链接。

**Section sources**
- [link.go](file://link.go#L22-L25)
- [cli/brook/main.go](file://cli/brook/main.go#L1620-L1685)

## 链接解析流程

链接解析是将字符串形式的 Brook 链接转换为可操作的配置对象的过程。核心函数 `ParseLink` 负责此任务，它使用 Go 标准库中的 `url.Parse` 函数解析链接，并提取关键信息。

### 解析步骤

1. **URL 解析**：使用 `url.Parse` 将链接字符串分解为主机、路径、查询参数等部分。
2. **提取 KIND**：从 URL 的主机部分获取代理模式（如 server、wsserver）。
3. **提取服务器地址**：从查询参数中获取与 KIND 同名的值作为服务器地址。
4. **返回参数集合**：将所有查询参数以 `url.Values` 形式返回，供后续处理。

```go
func ParseLink(link string) (kind, server string, v url.Values, err error) {
    var u *url.URL
    u, err = url.Parse(link)
    if err != nil {
        return
    }
    kind = u.Host
    server = u.Query().Get(kind)
    v = u.Query()
    return
}
```

解析后的结果被用于创建 `BrookLink` 结构体实例，该实例包含了所有必要的连接配置和安全参数。

**Section sources**
- [link.go](file://link.go#L27-L37)
- [brooklink.go](file://brooklink.go#L54-L147)

## 不同代理模式的链接格式差异

Brook 支持多种代理模式，每种模式在链接格式上有特定的参数要求和行为差异。

### server 模式

最基础的 TCP/UDP 代理模式，链接仅需包含服务器地址和密码。

**示例**：
```
brook://server?password=hello&server=1.2.3.4%3A9999
```

### wsserver 模式

基于 WebSocket 的代理模式，链接包含 WebSocket 服务器地址、路径和可选的 TLS 配置。

**示例**：
```
brook://wsserver?password=hello&server=ws%3A%2F%2F1.2.3.4%3A80%2Fws
```

### wssserver 模式

基于 WebSocket Secure 的代理模式，支持 TLS 加密和 TLS 指纹伪装。

**示例**：
```
brook://wssserver?password=hello&server=wss%3A%2F%2Fexample.com%3A443%2Fws&insecure=true&tlsfingerprint=chrome
```

### quicserver 模式

基于 QUIC 协议的代理模式，使用 QUIC 数据流进行通信。

**示例**：
```
brook://quicserver?password=hello&server=quic%3A%2F%2Fexample.com%3A443&insecure=true
```

每种模式的链接格式差异主要体现在 `server` 参数的协议前缀（如 `ws://`、`wss://`、`quic://`）以及特定的扩展参数上。

**Section sources**
- [protocol/brook-link-protocol.md](file://protocol/brook-link-protocol.md#L7-L8)
- [cli/brook/main.go](file://cli/brook/main.go#L1627-L1638)

## 安全处理机制

Brook 链接协议通过多种机制确保连接的安全性，包括加密、身份验证和防重放攻击。

### 加密与密钥派生

Brook 使用 AES-256-GCM 进行数据加密，密钥通过 HKDF-SHA256 算法从用户密码派生。密钥派生过程中使用了非ces（nonce）和固定信息（info）作为输入，确保每次会话的密钥唯一性。

```go
KEY: HKDF_SHA256(Password, Nonce, Info)
```

其中 `Info` 默认为 `[0x62, 0x72, 0x6f, 0x6f, 0x6b]`（"brook" 的 ASCII 值），可通过 `clientHKDFInfo` 和 `serverHKDFInfo` 参数自定义。

### 防重放攻击

通过在数据包中包含时间戳来防止重放攻击。服务器会检查时间戳的有效性，拒绝过期的请求（通常超过 60 秒的请求被视为无效）。

### TLS 安全配置

对于 wssserver 和 quicserver 模式，支持以下安全选项：
- `insecure=true`：跳过证书验证（不推荐用于生产环境）
- `ca`：指定自定义 CA 证书用于验证服务器身份
- `tlsfingerprint=chrome`：使用 Chrome 浏览器的 TLS 指纹进行伪装，绕过深度包检测

这些安全机制共同确保了 Brook 链接在传输过程中的机密性、完整性和可用性。

**Section sources**
- [protocol/brook-server-protocol.md](file://protocol/brook-server-protocol.md#L24-L28)
- [brooklink.go](file://brooklink.go#L91-L104)

## 命令行工具使用

Brook 提供了丰富的命令行工具来生成和解析链接，简化了配置管理。

### 生成链接

使用 `brook link` 命令生成链接，支持多种参数：

```bash
brook link --server <address> --password <password> [options]
```

常用选项包括：
- `--name`：为链接指定名称
- `--udpovertcp`：启用 UDP over TCP 模式
- `--withoutBrookProtocol`：禁用 Brook 协议加密
- `--fragment`：设置 TLS 分片参数（用于 wssserver）

### 连接链接

使用 `brook connect` 命令通过链接建立代理连接：

```bash
brook connect --link 'brook://...' --socks5 127.0.0.1:1080
```

该命令会解析链接，创建 SOCKS5 代理服务器，并将流量转发到目标服务器。

**Section sources**
- [cli/brook/main.go](file://cli/brook/main.go#L1618-L1794)

## 应用场景

Brook 链接协议在自动化部署和配置共享中具有广泛的应用价值。

### 自动化部署

在 CI/CD 流程中，可以通过脚本自动生成 Brook 链接并部署到目标服务器。例如，在容器化环境中，可以将链接作为环境变量注入容器，实现动态配置。

### 配置共享

用户可以通过二维码或短链接的形式分享代理配置，接收方只需扫描或点击即可完成配置，极大地简化了跨设备配置同步的复杂性。

### 批量管理

通过脚本批量生成和管理多个 Brook 链接，适用于需要维护大量代理服务器的场景，如企业网络或数据中心。

这些应用场景充分利用了 Brook 链接的标准化、安全性和易用性特点。

**Section sources**
- [cli/brook/main.go](file://cli/brook/main.go#L1691-L1794)

## 错误处理与格式验证

Brook 在链接解析和使用过程中实现了严格的错误处理和格式验证机制。

### 格式验证

`ParseLink` 函数首先使用 `url.Parse` 验证链接的基本格式，确保其符合 URL 标准。如果解析失败，立即返回错误。

### 参数验证

`NewBrookLink` 函数在创建链接实例时会对关键参数进行验证：
- 检查服务器地址是否有效
- 验证 CA 证书的 PEM 格式
- 确保分片参数（fragment）的数值合理性

### 错误传播

所有错误都通过返回值传递，调用者可以根据错误类型进行相应处理。例如，`ErrorReply` 函数用于向 SOCKS5 客户端返回标准化的错误响应。

```go
func ErrorReply(r *socks5.Request, c *net.TCPConn, e error) error {
    // 发送拒绝连接的回复
}
```

这种分层的错误处理机制确保了系统的健壮性和可维护性。

**Section sources**
- [link.go](file://link.go#L27-L37)
- [brooklink.go](file://brooklink.go#L54-L147)
- [util.go](file://util.go#L28-L38)