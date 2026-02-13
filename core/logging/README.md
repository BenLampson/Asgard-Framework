# Logging 日志模块

## 概述

Logging 模块基于 **Serilog** 构建，提供了高性能、结构化的日志记录能力。通过与 Asgard 的 SystemConfig 体系集成，支持从 YAML 配置文件加载日志配置。

## 特性

- **高性能**：基于 Serilog 的异步写入机制
- **多输出目标**：支持 Console 和 File 输出
- **日志级别控制**：Verbose/Debug/Information/Warning/Error/Fatal
- **文件滚动**：支持按时间自动滚动日志文件
- **彩色输出**：Console 支持不同级别的彩色显示
- **配置驱动**：通过 YAML 配置文件管理日志行为

## 快速开始

### 1. 安装

Logging 模块已包含在 Asgard.Core 包中，无需额外安装。

### 2. 配置

创建 `logging.yaml` 配置文件：

```yaml
MinimumLevel: Information
Console:
  Enabled: true
  OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
  UseColors: true
File:
  Enabled: true
  Path: "logs/app-.log"
  RollingInterval: Day
  RetainedFileCountLimit: 7
```

### 3. 注册服务

```csharp
// 在 Program.cs 中
services.AddSystemConfig<SerilogConfig>("logging.yaml");
services.AddAsgardSerilog();
```

### 4. 使用日志

```csharp
public class MyService(ILogger<MyService> logger)
{
    public void DoWork()
    {
        logger.LogInformation("开始执行工作");
        
        try
        {
            // 业务逻辑
            logger.LogDebug("处理中...");
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "执行失败");
        }
    }
}
```

## 配置说明

### SerilogConfig

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| MinimumLevel | LogLevel | Information | 全局最小日志级别 |
| Console | ConsoleSinkOptions | - | Console 输出配置 |
| File | FileSinkOptions | - | 文件输出配置 |

### ConsoleSinkOptions

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| Enabled | bool | true | 是否启用 |
| OutputTemplate | string | 标准模板 | 输出格式模板 |
| UseColors | bool | true | 是否使用彩色输出 |

### FileSinkOptions

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| Enabled | bool | true | 是否启用 |
| Path | string | logs/log-.txt | 日志文件路径 |
| RollingInterval | RollingInterval | Day | 滚动间隔 |
| RetainedFileCountLimit | int? | 7 | 保留文件数 |

## 日志级别

按严重程度从低到高：

1. **Verbose** - 最详细的信息，仅调试使用
2. **Debug** - 调试信息
3. **Information** - 一般信息（默认）
4. **Warning** - 警告信息
5. **Error** - 错误信息
6. **Fatal** - 致命错误

## 更多文档

- [API 参考](api.md) - 完整的 API 文档
- [配置示例](examples.md) - 更多配置示例
