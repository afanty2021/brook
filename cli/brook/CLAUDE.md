[根目录](../../../CLAUDE.md) > [cli](../) > **brook**

# Brook CLI 模块

## 模块职责

Brook CLI 是项目的主要命令行入口，提供了完整的网络代理工具功能。该模块负责：
- 解析和处理命令行参数
- 初始化和配置各种服务组件
- 集成插件系统
- 提供用户交互界面

## 入口与启动

- **入口文件**：`main.go`
- **包名**：`main`
- **启动函数**：`main()`
- **CLI 框架**：`github.com/urfave/cli/v2`

### 主要命令分类
1. **服务器命令**：`server`, `wsserver`, `wssserver`, `quicserver`
2. **客户端命令**：`client`, `wsclient`, `wssclient`, `quicclient`, `connect`
3. **工具命令**：`link`, `relay`, `dnsserver`, `socks5`, `pac`
4. **测试命令**：`testsocks5`, `testbrook`, `echoserver`, `echoclient`
5. **实用工具**：`dnsclient`, `dohclient`, `dhcpserver`, `ipcountry`

## 对外接口

### 全局参数
```go
// 常用全局标志
--pprof              // 性能分析监听地址
--log               // 日志输出（文件路径或 'console'）
--tag               // 标签，用于进程标识
--dialWithDNS       // 自定义 DNS 服务器
--dialWithIP4/6     // 指定出口 IP
--prometheus        // Prometheus 监控地址
--blockDomainList   // 域名屏蔽列表
```

### 核心服务接口
- `brook server`：标准 TCP/UDP 代理服务器
- `brook client`：标准代理客户端
- `brook wsserver`：WebSocket 代理服务器
- `brook quicserver`：QUIC 代理服务器

## 关键依赖与配置

### 主要依赖
```go
// 核心依赖包
github.com/txthinking/brook      // Brook 核心库
github.com/urfave/cli/v2         // CLI 框架
github.com/txthinking/runnergroup // 进程组管理
github.com/txthinking/socks5      // SOCKS5 实现
```

### 插件集成
- `plugins/block`：访问控制和屏蔽
- `plugins/logger`：日志记录
- `plugins/pprof`：性能分析
- `plugins/prometheus`：监控指标
- `plugins/thedns`：自定义 DNS

## 配置文件支持

Brook 支持通过配置文件启动，配置文件格式为 JSON：
```bash
# 使用配置文件启动
brook /path/to/config.json
```

## 测试与质量

### 测试覆盖
- **命令行解析**：通过 CLI 框架内置测试
- **功能验证**：使用 `test` 命令进行端到端测试
- **集成测试**：通过各种场景的命令组合测试

### 质量保证
- 参数验证和错误处理
- 优雅的进程管理（信号处理）
- 资源清理和错误恢复

## 常见问题 (FAQ)

### Q: 如何启用调试模式？
A: 设置环境变量 `SOCKS5_DEBUG=true` 并使用 `--log console`

### Q: 如何查看所有可用命令？
A: 运行 `brook --help` 或 `brook completion` 生成自动补全

### Q: 配置文件必须使用绝对路径吗？
A: 是的，配置文件路径必须是绝对路径

### Q: 如何同时运行多个服务？
A: Brook 支持通过配置文件或使用进程管理工具

## 相关文件清单

- `main.go` (2962 行) - 主程序入口和所有命令定义
- 与根目录的 `go.mod` 配置依赖关系

## 变更记录 (Changelog)

### 2025-01-22
- 创建模块文档
- 分析主要命令结构和功能
- 记录关键依赖和配置项