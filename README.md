# Asgard 框架

Asgard 是一个现代化的 .NET 框架，提供了一系列核心功能和工具，帮助开发者快速构建可靠、高性能的应用程序。

## 框架简介

Asgard 框架设计理念注重模块化、可扩展性和易用性，为开发者提供了一套完整的解决方案，涵盖了应用程序开发中的常见需求。

## 核心模块

### 1. 缓存模块 (Caching)
- 多级缓存支持
- 内存缓存和 Redis 缓存集成
- 灵活的缓存配置选项

### 2. 日志模块 (Logging)
- 基于 Serilog 的日志实现
- 多 sink 支持（控制台、文件等）
- 可配置的日志级别和格式

### 3. 系统配置模块 (SystemConfig)
- YAML 配置文件支持
- 强类型配置对象
- 配置热加载能力

### 4. 消息队列模块 (Messaging)
- 支持 Kafka 和 RabbitMQ
- 延迟消息处理
- 死信队列和重试机制
- 消息追踪功能

### 5. 任务调度模块 (Job)
- 基于 Quartz.NET 的任务调度
- 灵活的触发器配置
- 任务执行监控和管理

### 6. 插件系统 (Plugin)
- 动态插件加载
- 插件依赖管理
- 插件生命周期控制

## 项目结构

```
Asgard/
├── Common/
│   ├── Asgard.Abstract/     # 抽象接口定义
│   ├── Asgard.Core/         # 核心实现
│   └── Asgard.Tools/        # 工具类
├── Test/                    # 测试代码
├── doc/                     # 文档
└── example/                 # 示例代码
```

## 快速开始

### 1. 安装依赖

通过 NuGet 安装 Asgard 相关包：

```bash
dotnet add package Asgard.Core
dotnet add package Asgard.Abstract
dotnet add package Asgard.Tools
```

### 2. 配置和使用

#### 基本配置

在 `appsettings.json` 或 `config.yaml` 中配置框架选项：

```yaml
# config.yaml
Asgard:
  Logging:
    Level: Information
  Caching:
    MemoryCache:
      Expiration: 3600
    RedisCache:
      ConnectionString: "localhost:6379"
  Messaging:
    Provider: RabbitMQ
    RabbitMQ:
      Host: "localhost"
      Port: 5672
      Username: "guest"
      Password: "guest"
```

#### 服务注册

在 `Program.cs` 中注册 Asgard 服务：

```csharp
var builder = WebApplication.CreateBuilder(args);

// 注册 Asgard 服务
builder.Services.AddAsgardCore();
builder.Services.AddAsgardLogging();
builder.Services.AddAsgardCaching();
builder.Services.AddAsgardMessaging();
builder.Services.AddAsgardJobs();
builder.Services.AddAsgardPlugins();

var app = builder.Build();

// 使用 Asgard 中间件
app.UseAsgard();

app.Run();
```

## 文档结构

- **core/**: 核心模块文档
  - **caching/**: 缓存模块文档
  - **logging/**: 日志模块文档
  - **systemconfig/**: 系统配置模块文档
  - **Core.md**: 核心模块总览

## 示例

查看 `example/Asgard.Example` 目录，了解如何在实际应用中使用 Asgard 框架。

## 贡献

欢迎提交 Issue 和 Pull Request 来改进 Asgard 框架。

## 许可证

Asgard 框架采用 MIT 许可证。