# 快速开始

## 安装

在项目中引用 Asgard.Core：

```xml
<ProjectReference Include="..\Common\Asgard.Core\Asgard.Core.csproj" />
```

## 基本使用

### 1. 定义配置类

创建一个实现 `ISystemConfig` 接口的配置类：

```csharp
using Asgard.Core.SystemConfig;

public class AppConfig : ISystemConfig
{
    [ConfigPath("app.name", DefaultValue = "DefaultApp")]
    public string Name { get; set; } = string.Empty;

    [ConfigPath("app.version", DefaultValue = "1.0.0")]
    public string Version { get; set; } = string.Empty;

    [ConfigPath("app.debug", DefaultValue = false)]
    public bool Debug { get; set; }

    public void Validate()
    {
        if (string.IsNullOrEmpty(Name))
            throw new InvalidOperationException("应用程序名称不能为空");
    }
}
```

### 2. 创建 YAML 配置文件

创建 `config.yaml` 文件，所有配置集中在一个文件中：

```yaml
# 应用程序配置
app:
  name: AsgardExample
  version: 1.0.0
  environment: Development
  maxConnections: 100
  enableCache: true

# 日志配置
logging:
  minimumLevel: Debug
  console:
    enabled: true
    outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
    useColors: true
  file:
    enabled: true
    path: "logs/app-.log"
    rollingInterval: Day
    retainedFileCountLimit: 7
```

### 3. 配置类定义

配置类的 `ConfigPath` 路径必须与 YAML 结构匹配：

```csharp
// 应用程序配置 - 路径以 app 开头
public class AppConfig : ISystemConfig
{
    [ConfigPath("app.name", DefaultValue = "DefaultApp")]
    public string Name { get; set; } = string.Empty;

    [ConfigPath("app.version", DefaultValue = "1.0.0")]
    public string Version { get; set; } = string.Empty;

    [ConfigPath("app.environment", DefaultValue = "Development")]
    public string Environment { get; set; } = string.Empty;

    public void Validate() { /* 验证逻辑 */ }
}

// 日志配置 - 路径以 logging 开头
public class LogConfig : ISystemConfig
{
    [ConfigPath("logging.minimumLevel", DefaultValue = LogLevel.Information)]
    public LogLevel MinimumLevel { get; set; }

    // 嵌套对象会自动绑定
    public ConsoleSinkOptions Console { get; set; } = new();
    public FileSinkOptions File { get; set; } = new();

    public void Validate() { /* 验证逻辑 */ }
}

// Console 配置 - 路径以 logging.console 开头
public class ConsoleSinkOptions
{
    [ConfigPath("logging.console.enabled", DefaultValue = true)]
    public bool Enabled { get; set; }

    [ConfigPath("logging.console.outputTemplate")]
    public string OutputTemplate { get; set; } = "...";

    [ConfigPath("logging.console.useColors", DefaultValue = true)]
    public bool UseColors { get; set; }
}
```

### 4. 加载配置

#### 方式一：直接加载

```csharp
var config = YamlConfigLoader.LoadFromFile<AppConfig>("config.yaml");
config.Validate();
```

#### 方式二：从字符串加载

```csharp
const string yamlContent = """
    app:
      name: MyApp
    """;

var config = YamlConfigLoader.Load<AppConfig>(yamlContent);
```

#### 方式三：依赖注入（推荐）

```csharp
var services = new ServiceCollection();
var configPath = "config.yaml";

// 从同一个配置文件加载多个配置类
services.AddSystemConfig<AppConfig>(configPath);    // 从 app 节读取
services.AddSystemConfig<LogConfig>(configPath);    // 从 logging 节读取

// 注册日志服务
services.AddAsgardSerilog();

var serviceProvider = services.BuildServiceProvider();

// 使用配置
public class MyService
{
    private readonly AppConfig _config;

    public MyService(AppConfig config)
    {
        _config = config;
    }
}
```

## 配置路径语法

配置路径使用点号 `.` 分隔嵌套层级，必须与 YAML 结构匹配：

| YAML 结构 | 配置路径 |
|-----------|----------|
| `app.name` | `app.name` |
| `logging.console.enabled` | `logging.console.enabled` |
| `database.connection.host` | `database.connection.host` |

## 默认值

通过 `DefaultValue` 参数设置默认值：

```csharp
[ConfigPath("app.name", DefaultValue = "DefaultApp")]
public string Name { get; set; } = string.Empty;

[ConfigPath("app.port", DefaultValue = 8080)]
public int Port { get; set; }
```

## 配置验证

在 `Validate` 方法中实现配置验证逻辑：

```csharp
public void Validate()
{
    if (string.IsNullOrEmpty(Name))
        throw new InvalidOperationException("应用程序名称不能为空");

    if (Port < 1 || Port > 65535)
        throw new InvalidOperationException("端口号必须在 1-65535 范围内");
}
```

## 嵌套配置对象

配置类可以包含嵌套的配置对象，系统会自动递归绑定。嵌套对象必须预先实例化：

```csharp
public class LogConfig : ISystemConfig
{
    [ConfigPath("logging.minimumLevel")]
    public LogLevel MinimumLevel { get; set; }

    // 嵌套对象 - 必须使用 new() 初始化
    public ConsoleSinkOptions Console { get; set; } = new();
    public FileSinkOptions File { get; set; } = new();

    public void Validate() { }
}

public class ConsoleSinkOptions
{
    [ConfigPath("logging.console.enabled", DefaultValue = true)]
    public bool Enabled { get; set; }

    [ConfigPath("logging.console.useColors", DefaultValue = true)]
    public bool UseColors { get; set; }
}
```
