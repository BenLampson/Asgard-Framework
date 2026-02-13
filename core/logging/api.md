# Logging API 参考

## SerilogConfig

日志主配置类，实现 `ISystemConfig` 接口。

```csharp
public class SerilogConfig : ISystemConfig
```

### 属性

#### MinimumLevel
```csharp
public LogLevel MinimumLevel { get; set; }
```
全局最小日志级别。只有大于或等于此级别的日志才会被记录。

**默认值**: `LogLevel.Information`

**可选值**:
- `LogLevel.Verbose`
- `LogLevel.Debug`
- `LogLevel.Information`
- `LogLevel.Warning`
- `LogLevel.Error`
- `LogLevel.Fatal`

---

#### Console
```csharp
public ConsoleSinkOptions Console { get; set; }
```
Console 输出配置。

**默认值**: 新实例（Enabled = true）

---

#### File
```csharp
public FileSinkOptions File { get; set; }
```
文件输出配置。

**默认值**: 新实例（Enabled = true）

---

### 方法

#### Validate
```csharp
public void Validate()
```
验证配置的有效性。

**异常**:
- `InvalidOperationException` - 配置无效时抛出

**验证规则**:
1. Console 配置对象不能为 null
2. File 配置对象不能为 null
3. 启用文件日志时，路径不能为空
4. 保留文件数必须大于等于 0
5. 日志级别必须有效

---

## ConsoleSinkOptions

Console 日志输出配置选项。

```csharp
public class ConsoleSinkOptions
```

### 属性

#### Enabled
```csharp
public bool Enabled { get; set; }
```
是否启用 Console 日志输出。

**默认值**: `true`

---

#### OutputTemplate
```csharp
public string OutputTemplate { get; set; }
```
日志输出模板。

**默认值**: `"[{Timestamp:yyyy-MM-dd HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"`

**可用占位符**:
- `{Timestamp}` - 日志时间戳
- `{Level}` - 日志级别
- `{Message}` - 日志消息
- `{Exception}` - 异常信息
- `{NewLine}` - 换行符

---

#### UseColors
```csharp
public bool UseColors { get; set; }
```
是否使用彩色输出。

**默认值**: `true`

**颜色映射**:
- Verbose - 灰色
- Debug - 灰色
- Information - 白色
- Warning - 黄色
- Error - 红色
- Fatal - 洋红色

---

## FileSinkOptions

文件日志输出配置选项。

```csharp
public class FileSinkOptions
```

### 属性

#### Enabled
```csharp
public bool Enabled { get; set; }
```
是否启用文件日志输出。

**默认值**: `true`

---

#### Path
```csharp
public string Path { get; set; }
```
日志文件路径。

**默认值**: `"logs/log-.txt"`

**说明**: 路径中的 `-` 用于日期分隔，例如 `logs/log-20250213.txt`

---

#### RollingInterval
```csharp
public RollingInterval RollingInterval { get; set; }
```
日志滚动间隔。

**默认值**: `RollingInterval.Day`

**可选值**:
- `RollingInterval.Infinite` - 不滚动
- `RollingInterval.Year` - 每年
- `RollingInterval.Month` - 每月
- `RollingInterval.Day` - 每天
- `RollingInterval.Hour` - 每小时
- `RollingInterval.Minute` - 每分钟

---

#### RetainedFileCountLimit
```csharp
public int? RetainedFileCountLimit { get; set; }
```
保留的日志文件数量限制。

**默认值**: `7`

**说明**: 设置为 `null` 表示不限制文件数量

---

#### OutputTemplate
```csharp
public string OutputTemplate { get; set; }
```
日志输出模板。

**默认值**: `"[{Timestamp:yyyy-MM-dd HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"`

---

## SerilogConfigurator

Serilog 配置器，用于从配置构建 LoggerConfiguration。

```csharp
public static class SerilogConfigurator
```

### 方法

#### FromConfig
```csharp
public static LoggerConfiguration FromConfig(SerilogConfig config)
```

从 `SerilogConfig` 创建 `LoggerConfiguration`。

**参数**:
- `config` - Serilog 配置对象，不能为空

**返回**: 配置好的 `LoggerConfiguration` 实例

**异常**:
- `ArgumentNullException` - config 为 null 时抛出

**示例**:
```csharp
var config = new SerilogConfig { MinimumLevel = LogLevel.Debug };
var loggerConfig = SerilogConfigurator.FromConfig(config);
Log.Logger = loggerConfig.CreateLogger();
```

---

## LoggingServiceCollectionExtensions

日志服务集合扩展方法。

```csharp
public static class LoggingServiceCollectionExtensions
```

### 方法

#### AddAsgardSerilog (无参数)
```csharp
public static IServiceCollection AddAsgardSerilog(this IServiceCollection services)
```

从服务容器中获取 `SerilogConfig` 并注册 Serilog 日志服务。

**参数**:
- `services` - 服务集合实例

**返回**: 服务集合实例，支持链式调用

**异常**:
- `ArgumentNullException` - services 为 null 时抛出
- `InvalidOperationException` - 服务容器中找不到 SerilogConfig 时抛出

**示例**:
```csharp
services.AddSystemConfig<SerilogConfig>("logging.yaml");
services.AddAsgardSerilog();
```

---

#### AddAsgardSerilog (传入配置)
```csharp
public static IServiceCollection AddAsgardSerilog(
    this IServiceCollection services, 
    SerilogConfig config)
```

使用指定的 `SerilogConfig` 注册 Serilog 日志服务。

**参数**:
- `services` - 服务集合实例
- `config` - Serilog 配置对象

**返回**: 服务集合实例，支持链式调用

**异常**:
- `ArgumentNullException` - services 或 config 为 null 时抛出

**示例**:
```csharp
var config = new SerilogConfig { MinimumLevel = LogLevel.Information };
services.AddAsgardSerilog(config);
```

---

#### AddAsgardSerilog (配置委托)
```csharp
public static IServiceCollection AddAsgardSerilog(
    this IServiceCollection services, 
    Action<SerilogConfig> configure)
```

使用配置委托注册 Serilog 日志服务。

**参数**:
- `services` - 服务集合实例
- `configure` - 配置委托

**返回**: 服务集合实例，支持链式调用

**异常**:
- `ArgumentNullException` - services 或 configure 为 null 时抛出

**示例**:
```csharp
services.AddAsgardSerilog(c =>
{
    c.MinimumLevel = LogLevel.Debug;
    c.Console.Enabled = true;
    c.File.Enabled = true;
    c.File.Path = "logs/app-.log";
});
```

---

## LogLevel 枚举

日志级别枚举，定义日志的严重程度。

```csharp
public enum LogLevel
```

**值**:

| 值 | 说明 |
|----|------|
| `Verbose` | 详细级别，用于记录最详细的信息 |
| `Debug` | 调试级别，用于记录调试信息 |
| `Information` | 信息级别，用于记录一般信息（默认） |
| `Warning` | 警告级别，用于记录可能的问题 |
| `Error` | 错误级别，用于记录错误信息 |
| `Fatal` | 致命级别，用于记录严重错误 |

---

## RollingInterval 枚举

日志文件滚动间隔枚举。

```csharp
public enum RollingInterval
```

**值**:

| 值 | 说明 |
|----|------|
| `Infinite` | 不滚动，所有日志写入单个文件 |
| `Year` | 每年滚动一次 |
| `Month` | 每月滚动一次 |
| `Day` | 每天滚动一次（默认） |
| `Hour` | 每小时滚动一次 |
| `Minute` | 每分钟滚动一次 |
