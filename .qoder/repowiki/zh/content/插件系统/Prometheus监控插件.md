# Prometheus监控插件

<cite>
**本文档引用的文件**  
- [prometheus.go](file://plugins/prometheus/prometheus.go#L1-L92)
- [main.go](file://cli/brook/main.go#L121-L128)
- [exchanger.go](file://exchanger.go#L21-L27)
</cite>

## 目录
1. [简介](#简介)
2. [启用监控服务](#启用监控服务)
3. [暴露的监控指标](#暴露的监控指标)
4. [HTTP端点访问方式](#http端点访问方式)
5. [安全建议](#安全建议)
6. [与Nico等工具结合使用](#与nico等工具结合使用)

## 简介
Prometheus监控插件用于收集Brook服务的运行时指标，并通过HTTP端点暴露给Prometheus服务器进行采集。该插件实现了基本的连接统计功能，可用于监控网络流量和连接行为。

**Section sources**
- [prometheus.go](file://plugins/prometheus/prometheus.go#L1-L92)

## 启用监控服务
要启用Prometheus监控服务，需要在启动Brook时使用`--prometheus`和`--prometheusPath`参数：

```bash
brook server --listen :9999 --password hello \
    --prometheus :7070 --prometheusPath /metrics \
    --tag "env:prod,region:us-west"
```

其中：
- `--prometheus`：指定Prometheus HTTP服务监听地址，如`:7070`
- `--prometheusPath`：指定指标暴露的HTTP路径，如`/metrics`

如果启用了`--prometheus`但未指定`--prometheusPath`，程序将返回错误提示"You forgot the --prometheusPath"。

**Section sources**
- [main.go](file://cli/brook/main.go#L240-L265)
- [main.go](file://cli/brook/main.go#L121-L128)

## 暴露的监控指标
当前插件仅暴露一种监控指标：

### dst_counter
- **名称**: `dst_counter`
- **类型**: 计数器（Counter）
- **描述**: 目标地址连接总数
- **标签（Labels）**:
  - `network`: 网络类型（tcp/udp）
  - `from`: 源地址（客户端IP）
  - `dst`: 目标地址（目标主机和端口）
  - 自定义标签：通过`--tag`参数指定的标签

该指标通过拦截`brook.ServerGate`和`brook.ClientGate`函数来统计所有经过的连接请求，每次有新的交换器（Exchanger）创建时，计数器就会递增。

**Section sources**
- [prometheus.go](file://plugins/prometheus/prometheus.go#L56-L92)
- [exchanger.go](file://exchanger.go#L21-L27)

## HTTP端点访问方式
Prometheus监控服务通过标准HTTP协议暴露指标数据：

- **访问URL**: `http://<prometheus地址><prometheusPath>`
- **示例**: `http://localhost:7070/metrics`
- **响应格式**: Prometheus文本格式（text/plain; version=0.0.4）

返回的数据格式如下：
```
# HELP dst_counter Number of dst in total
# TYPE dst_counter counter
dst_counter{from="192.168.1.100",network="tcp",dst="google.com:443",env="prod"} 15
dst_counter{from="192.168.1.101",network="udp",dst="8.8.8.8:53",env="prod"} 8
```

Prometheus服务器可以通过配置job定期抓取此端点获取监控数据。

**Section sources**
- [prometheus.go](file://plugins/prometheus/prometheus.go#L32-L38)

## 安全建议
由于监控端点可能暴露敏感的连接信息，建议采取以下安全措施：

1. **使用难以猜测的路径值**：`--prometheusPath`应使用复杂、随机的值，避免使用常见的`/metrics`或`/prometheus`等路径，以防止被扫描发现。
2. **限制访问来源**：通过防火墙规则限制只有Prometheus服务器能够访问监控端口。
3. **结合反向代理**：使用Nginx等反向代理添加身份验证。
4. **避免公网暴露**：不要将监控服务直接暴露在公网上。

**Section sources**
- [main.go](file://cli/brook/main.go#L126-L128)

## 与Nico等工具结合使用
文档建议在公网传输时结合Nico等工具使用。Nico是一个网络混淆工具，可以对流量进行加密和混淆，防止被识别和干扰。

结合使用的典型场景：
1. Prometheus插件在本地暴露监控端点
2. 使用Nico对监控端点的流量进行加密
3. Prometheus服务器通过安全通道采集指标

这样既能实现监控功能，又能保证监控数据传输的安全性，特别适用于需要在公网环境中进行监控的场景。

**Section sources**
- [main.go](file://cli/brook/main.go#L123-L124)