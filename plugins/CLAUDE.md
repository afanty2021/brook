[根目录](../../CLAUDE.md) > **plugins**

# Brook 插件系统

## 模块职责

Brook 插件系统提供了可扩展的功能模块，允许用户根据需要启用各种网络增强功能。插件系统采用函数覆盖的方式实现，具有简单、高效、易于开发的特点。

## 插件架构

### 插件注册机制
所有插件通过覆盖 Brook 核心包中的公共函数变量来实现功能扩展：
```go
// 示例：插件注册
func TouchBrook() {
    brook.Log = MyCustomLog
    brook.Dial = MyCustomDial
}
```

## 插件模块列表

| 插件名 | 功能描述 | 文件路径 | 配置方式 |
|--------|----------|----------|----------|
| **block** | 访问控制和域名屏蔽 | `block/block.go` | `--blockDomainList` |
| **logger** | 日志记录和管理 | `logger/logger.go` | `--log` |
| **pprof** | 性能分析服务 | `pprof/pprof.go` | `--pprof` |
| **prometheus** | 监控指标收集 | `prometheus/prometheus.go` | `--prometheus` |
| **thedns** | 自定义 DNS 处理 | `thedns/thedns.go | DNS 相关命令 |
| **dialwithdns** | 指定 DNS 解析 | `dialwithdns/dialwithdns.go` | `--dialWithDNS` |
| **dialwithip** | 指定出口 IP | `dialwithip/dialwithip.go` | `--dialWithIP4/6` |
| **dialwithnic** | 指定网络接口 | `dialwithnic/dialwithnic.go` | `--dialWithNIC` |
| **socks5dial** | SOCKS5 代理链 | `socks5dial/dial.go` | `--dialWithSocks5` |

## 核心插件详解

### 1. Logger 插件
- **职责**：统一的日志记录和管理
- **特性**：支持文件和控制台输出
- **标签系统**：支持自定义标签过滤
- **信号处理**：支持 SIGUSR1 重置日志文件

### 2. Prometheus 插件
- **职责**：暴露监控指标供 Prometheus 收集
- **指标类型**：连接数、流量、错误率等
- **安全特性**：支持自定义路径避免扫描

### 3. Block 插件
- **职责**：访问控制和内容过滤
- **支持列表**：域名、CIDR、地理位置
- **动态更新**：支持定时更新屏蔽列表
- **数据源**：HTTP/HTTPS URL 或本地文件

### 4. Pprof 插件
- **职责**：Go 运行时性能分析
- **功能**：CPU、内存、goroutine 分析
- **用途**：性能调优和问题诊断

## 插件开发指南

### 创建新插件
1. 在 `plugins/` 目录下创建新文件夹
2. 实现 `TouchBrook()` 函数
3. 在 `main.go` 中导入插件
4. 添加相应的命令行参数

### 插件模板
```go
package myplugin

import "github.com/txthinking/brook"

func TouchBrook() {
    // 覆盖核心函数
    brook.Log = myLog
    // 其他初始化代码
}
```

## 配置与使用

### 全局插件参数
所有插件都支持以下全局参数：
- `--tag`：为插件添加标签
- `--log`：启用日志记录
- `--pprof`：启用性能分析

### 插件组合使用
```bash
# 示例：同时启用多个插件
brook server -l :9999 -p password \
    --blockDomainList https://example.com/blocklist.txt \
    --prometheus :7070 --prometheusPath /metrics \
    --log /var/log/brook.log \
    --tag "env:prod,region:us-west"
```

## 测试与质量

### 测试策略
- **单元测试**：每个插件独立测试
- **集成测试**：与其他插件组合测试
- **性能测试**：确保插件不影响核心性能

### 质量保证
- 内存泄漏检测
- 并发安全性验证
- 错误处理完整性

## 相关文件清单

- `plugins/block/` - 访问控制插件
- `plugins/logger/` - 日志插件
- `plugins/pprof/` - 性能分析插件
- `plugins/prometheus/` - 监控插件
- `plugins/thedns/` - DNS 处理插件
- `plugins/dialwithdns/` - DNS 解析插件
- `plugins/dialwithip/` - IP 指定插件
- `plugins/dialwithnic/` - 网卡指定插件
- `plugins/socks5dial/` - SOCKS5 代理插件

## 变更记录 (Changelog)

### 2025-01-22
- 创建插件系统文档
- 分析现有插件架构和功能
- 整理插件开发指南