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

创建 `config.yaml` 文件：

```yaml
app:
  name: Asgard
  version: 1.0.0
  debug: false
```

### 3. 加载配置

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

#### 方式三：依赖注入

```csharp
// 在服务配置中注册
services.AddSystemConfig<AppConfig>("config.yaml");

// 在服务中使用
public class MyService
{
    private readonly AppConfig _config;

    public MyService(IOptions<AppConfig> options)
    {
        _config = options.Value;
    }
}
```

## 配置路径语法

配置路径使用点号 `.` 分隔嵌套层级：

| YAML 结构 | 配置路径 |
|-----------|----------|
| `app.name` | `app.name` |
| `database.connection.host` | `database.connection.host` |
| `cache.ttl` | `cache.ttl` |

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
