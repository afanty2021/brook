[根目录](../../CLAUDE.md) > **programmable**

# Brook 可编程模块

## 模块职责

Brook 可编程模块提供了强大的脚本功能，允许用户通过编写脚本来自定义网络行为、路由规则和数据处理逻辑。该模块使用 Tengo 脚本语言，支持安全、高效的脚本执行。

## 可编程架构

### 脚本类型
- **dnsserver**：DNS 服务器脚本，用于自定义 DNS 解析逻辑
- **server**：服务器脚本，用于处理客户端连接和数据转发
- **client**：客户端脚本，用于自定义代理行为
- **module**：通用模块，供 Brook GUI 客户端使用的功能模块

### 脚本执行环境
- **语言**：Tengo (https://github.com/d5/tengo)
- **安全沙箱**：脚本在受限环境中执行
- **性能优化**：JIT 编译提供高性能执行

## 目录结构

```
programmable/
├── dnsserver/          # DNS 服务器脚本
│   ├── example.tengo   # 示例脚本
│   └── check_syntax.js # 语法检查工具
├── server/             # 服务器脚本
│   ├── example.tengo
│   └── check_syntax.js
├── client/             # 客户端脚本
│   ├── example.tengo
│   └── check_syntax.js
├── modules/            # 通用模块库
│   ├── *.tengo        # 各种功能模块
│   ├── _header.tengo  # 模块头文件
│   ├── _footer.tengo  # 模块尾文件
│   └── check_syntax.js
├── gallery.json        # 脚本画廊配置
└── readme.md          # 使用说明
```

## 脚本画廊 (Script Gallery)

### 提交脚本
用户可以通过修改 `gallery.json` 文件来提交自己的脚本：

```json
{
  "name": "My Awesome Script",
  "url": "https://example.com/script.tengo",
  "kind": "module",
  "ca": false,
  "author": "Your Name",
  "author_url": "https://yourwebsite.com"
}
```

### 脚本分类
- **dnsserver**：用于 DNS 相关功能
- **server**：用于服务器端处理逻辑
- **module**：用于客户端 GUI 的功能模块
- **client**：用于客户端代理行为

## 核心模块库

### 常用功能模块

#### 网络控制模块
- `allow_app.tengo` - 应用允许列表
- `block_app.tengo` - 应用阻止列表
- `bypass_app.tengo` - 应用绕过规则

#### 域名处理模块
- `hosts.tengo` - 自定义 hosts 文件
- `block_ad_domain.tengo` - 广告域名屏蔽
- `bypass_china_domain_a.tengo` - 中国域名绕过
- `redirect_google_cn.tengo` - Google 中国重定向

#### 地理位置模块
- `bypass_geo.tengo` - 地理位置绕过
- `bypass_apple.tengo` - Apple 服务绕过

#### 应用特定模块
- `instagram_system_dns.tengo` - Instagram DNS 优化
- `chatgpt_advanced_voice.tengo` - ChatGPT 语音支持
- `xbox.tengo` - Xbox 网络优化
- `sanguosha.tengo` - 三国杀游戏优化
- `douban.tengo` - 豆瓣服务优化
- `xiaohongshu.tengo` - 小红书优化

#### 安全模块
- `block_google_secure_dns.tengo` - Google 安全 DNS 屏蔽
- `ios_app_downgrade.tengo` - iOS 应用降级防护

#### 工具模块
- `packet_capture.tengo` - 数据包捕获
- `mitmproxy_client.tengo` - 中间人代理客户端
- `brooklinks.tengo` - Brook 链接处理
- `response_sample.tengo` - 响应样本处理

## 脚本开发指南

### Tengo 语言特性
- 简单易学的语法
- 类型安全的变量系统
- 丰富的标准库
- 高性能的 JIT 编译

### 脚本模板

#### DNS 服务器脚本模板
```tengo
// dnsserver example
export func handle_request(msg) {
    // 处理 DNS 请求
    return msg
}

export func handle_response(msg) {
    // 处理 DNS 响应
    return msg
}
```

#### 服务器脚本模板
```tengo
// server example
export func handle_connection(conn) {
    // 处理客户端连接
    return true // 允许连接
}

export func handle_data(data) {
    // 处理数据
    return data
}
```

#### 客户端脚本模板
```tengo
// client example
export func should_proxy(domain) {
    // 判断是否需要代理
    return true
}

export func select_server(servers) {
    // 选择服务器
    return servers[0]
}
```

### 语法检查
每个目录都提供了 `check_syntax.js` 工具用于脚本语法检查：
```bash
node check_syntax.js script.tengo
```

## 配置与使用

### 脚本加载
```bash
# 加载 DNS 服务器脚本
brook dnsserver -l :53 --script /path/to/dns.tengo

# 加载服务器脚本
brook server -l :9999 -p password --script /path/to/server.tengo
```

### 模块引用
脚本可以引用其他模块：
```tengo
import "./modules/bypass_geo.tengo"
import "./modules/hosts.tengo"
```

## 测试与质量

### 脚本测试
- **语法检查**：使用提供的检查工具
- **功能测试**：在各种网络环境下测试
- **性能测试**：确保脚本不影响代理性能

### 质量保证
- 脚本安全审计
- 内存使用监控
- 错误处理验证

## 相关文件清单

- `dnsserver/example.tengo` - DNS 服务器示例脚本
- `server/example.tengo` - 服务器示例脚本
- `client/example.tengo` - 客户端示例脚本
- `modules/*.tengo` - 各种功能模块
- `gallery.json` - 脚本画廊配置
- `check_syntax.js` - 语法检查工具

## 变更记录 (Changelog)

### 2025-01-22
- 创建可编程模块文档
- 分析现有脚本库和模块
- 整理开发指南和使用说明