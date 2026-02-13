# 核心组件

## ISystemConfig 接口

所有配置类必须实现的接口，提供配置验证功能。

```csharp
public interface ISystemConfig
{
    /// <summary>
    /// 验证配置的有效性。
    /// </summary>
    void Validate();
}
```

### 使用示例

```csharp
public class DatabaseConfig : ISystemConfig
{
    [ConfigPath("database.name")]
    public string DatabaseName { get; set; } = string.Empty;

    public void Validate()
    {
        if (string.IsNullOrEmpty(DatabaseName))
            throw new InvalidOperationException("数据库名称是必填项");
    }
}
```

---

## ConfigPathAttribute 特性

用于标记配置属性与 YAML 路径的映射关系。

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `Path` | `string` | YAML 配置路径，使用点号分隔 |
| `DefaultValue` | `object?` | 默认值，当路径不存在时使用 |
| `HasDefaultValue` | `bool` | 指示是否已设置默认值 |

### 使用示例

```csharp
public class DatabaseConfig : ISystemConfig
{
    // 必填字段，无默认值
    [ConfigPath("database.name")]
    public string DatabaseName { get; set; } = string.Empty;

    // 可选字段，有默认值
    [ConfigPath("database.host", DefaultValue = "localhost")]
    public string Host { get; set; } = string.Empty;

    [ConfigPath("database.port", DefaultValue = 3306)]
    public int Port { get; set; }

    // 枚举类型
    [ConfigPath("database.provider", DefaultValue = DatabaseProvider.MySql)]
    public DatabaseProvider Provider { get; set; }
}
```

---

## YamlConfigLoader 类

提供静态方法用于加载 YAML 配置。

### 方法

| 方法 | 说明 |
|------|------|
| `Load<T>(string yamlContent)` | 从 YAML 字符串加载配置 |
| `LoadFromFile<T>(string filePath)` | 从 YAML 文件加载配置 |
| `BindConfig<T>(T config, Dictionary<object, object> yamlData)` | 将 YAML 数据绑定到配置对象 |

### 使用示例

```csharp
// 从文件加载
var config = YamlConfigLoader.LoadFromFile<AppConfig>("config.yaml");

// 从字符串加载
const string yamlContent = """
    app:
      name: MyApp
    """;
var config = YamlConfigLoader.Load<AppConfig>(yamlContent);
```

---

## YamlConfigurationExtensions 类

提供配置构建器的扩展方法。

### 方法

| 方法 | 说明 |
|------|------|
| `AddYamlFile(path, optional, reloadOnChange)` | 添加 YAML 配置文件 |
| `AddYamlFile(configureSource)` | 使用委托配置添加 YAML 配置源 |

### 使用示例

```csharp
var configuration = new ConfigurationBuilder()
    .AddYamlFile("config.yaml", optional: true, reloadOnChange: true)
    .Build();
```

---

## ServiceCollectionExtensions 类

提供依赖注入扩展方法。

### 方法

| 方法 | 说明 |
|------|------|
| `AddSystemConfig<TConfig>(yamlFilePath, optional, reloadOnChange)` | 从 YAML 文件注册配置 |
| `AddSystemConfig<TConfig>(IConfiguration)` | 从 IConfiguration 注册配置 |
| `AddSystemConfig<TConfig>(Action<TConfig>)` | 使用委托注册配置 |

### 使用示例

```csharp
// 从 YAML 文件注册
services.AddSystemConfig<AppConfig>("config.yaml");

// 从 IConfiguration 注册
services.AddSystemConfig<AppConfig>(configuration);

// 使用委托配置
services.AddSystemConfig<AppConfig>(config =>
{
    config.Name = "MyApp";
    config.Version = "1.0.0";
});
```

---

## YamlConfigurationSource 类

YAML 配置源定义，继承自 `FileConfigurationSource`。

### 属性

继承自 `FileConfigurationSource` 的主要属性：

| 属性 | 类型 | 说明 |
|------|------|------|
| `Path` | `string?` | 配置文件路径 |
| `Optional` | `bool` | 是否为可选文件 |
| `ReloadOnChange` | `bool` | 是否在文件更改时重新加载 |

---

## YamlConfigurationProvider 类

YAML 配置提供程序，负责解析 YAML 文件。

### 工作原理

1. 读取 YAML 文件内容
2. 使用 YamlDotNet 反序列化
3. 将嵌套结构展平为配置键值对
4. 使用冒号作为键分隔符

### 展平示例

```yaml
database:
  connection:
    host: localhost
    port: 3306
```

展平后：

| 键 | 值 |
|----|-----|
| `database:connection:host` | `localhost` |
| `database:connection:port` | `3306` |
