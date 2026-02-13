# API 参考

## ISystemConfig 接口

```csharp
namespace Asgard.Core.SystemConfig;

public interface ISystemConfig
{
    void Validate();
}
```

### Validate 方法

验证配置的有效性。

**异常：**

| 异常类型 | 说明 |
|----------|------|
| `InvalidOperationException` | 配置无效时抛出 |

---

## ConfigPathAttribute 类

```csharp
namespace Asgard.Core.SystemConfig;

[AttributeUsage(AttributeTargets.Property, AllowMultiple = false)]
public sealed class ConfigPathAttribute : Attribute
{
    public string Path { get; }
    public object? DefaultValue { get; set; }
    public bool HasDefaultValue { get; }

    public ConfigPathAttribute(string path);
}
```

### 构造函数

| 构造函数 | 说明 |
|----------|------|
| `ConfigPathAttribute(string path)` | 使用指定的配置路径初始化 |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `path` | `string` | 配置路径，使用点号分隔 |

**异常：**

| 异常类型 | 说明 |
|----------|------|
| `ArgumentException` | 路径为 null 或空时抛出 |

### 属性

| 属性 | 类型 | 访问 | 说明 |
|------|------|------|------|
| `Path` | `string` | 只读 | 配置路径 |
| `DefaultValue` | `object?` | 读写 | 默认值 |
| `HasDefaultValue` | `bool` | 只读 | 是否已设置默认值 |

---

## YamlConfigLoader 类

```csharp
namespace Asgard.Core.SystemConfig;

public static class YamlConfigLoader
{
    public static T Load<T>(string yamlContent) where T : class, ISystemConfig, new();
    public static T LoadFromFile<T>(string filePath) where T : class, ISystemConfig, new();
    public static void BindConfig<T>(T config, Dictionary<object, object> yamlData) where T : class, ISystemConfig;
}
```

### Load 方法

从 YAML 字符串内容加载配置对象。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `T` | `class, ISystemConfig, new()` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `yamlContent` | `string` | YAML 格式的字符串内容 |

**返回值：**

填充了配置数据的配置对象实例。

### LoadFromFile 方法

从 YAML 文件加载配置对象。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `T` | `class, ISystemConfig, new()` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `filePath` | `string` | YAML 配置文件的完整路径 |

**返回值：**

填充了配置数据的配置对象实例。

**异常：**

| 异常类型 | 说明 |
|----------|------|
| `FileNotFoundException` | 文件不存在时抛出 |

### BindConfig 方法

将 YAML 数据绑定到配置对象的属性上。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `T` | `class, ISystemConfig` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `config` | `T` | 要绑定数据的配置对象实例 |
| `yamlData` | `Dictionary<object, object>` | 解析后的 YAML 数据字典 |

---

## YamlConfigurationExtensions 类

```csharp
namespace Asgard.Core.SystemConfig;

public static class YamlConfigurationExtensions
{
    public static IConfigurationBuilder AddYamlFile(
        this IConfigurationBuilder builder,
        string path,
        bool optional = false,
        bool reloadOnChange = false);

    public static IConfigurationBuilder AddYamlFile(
        this IConfigurationBuilder builder,
        Action<YamlConfigurationSource> configureSource);
}
```

### AddYamlFile 方法（参数形式）

向配置构建器添加 YAML 配置文件。

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `builder` | `IConfigurationBuilder` | 配置构建器实例 |
| `path` | `string` | YAML 文件的路径 |
| `optional` | `bool` | 是否为可选文件，默认 false |
| `reloadOnChange` | `bool` | 是否在文件更改时重新加载，默认 false |

**返回值：**

配置构建器实例，支持链式调用。

### AddYamlFile 方法（委托形式）

向配置构建器添加 YAML 配置源，使用委托进行配置。

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `builder` | `IConfigurationBuilder` | 配置构建器实例 |
| `configureSource` | `Action<YamlConfigurationSource>` | 配置源的配置委托 |

**返回值：**

配置构建器实例，支持链式调用。

---

## ServiceCollectionExtensions 类

```csharp
namespace Asgard.Core.SystemConfig;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddSystemConfig<TConfig>(
        this IServiceCollection services,
        string yamlFilePath,
        bool optional = false,
        bool reloadOnChange = false) where TConfig : class, ISystemConfig, new();

    public static IServiceCollection AddSystemConfig<TConfig>(
        this IServiceCollection services,
        IConfiguration configuration) where TConfig : class, ISystemConfig, new();

    public static IServiceCollection AddSystemConfig<TConfig>(
        this IServiceCollection services,
        Action<TConfig> configure) where TConfig : class, ISystemConfig, new();
}
```

### AddSystemConfig 方法（YAML 文件）

从 YAML 文件注册系统配置。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `TConfig` | `class, ISystemConfig, new()` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `services` | `IServiceCollection` | 服务集合实例 |
| `yamlFilePath` | `string` | YAML 配置文件的路径 |
| `optional` | `bool` | 是否为可选文件，默认 false |
| `reloadOnChange` | `bool` | 是否在文件更改时重新加载，默认 false |

**返回值：**

服务集合实例，支持链式调用。

### AddSystemConfig 方法（IConfiguration）

从 IConfiguration 注册系统配置。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `TConfig` | `class, ISystemConfig, new()` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `services` | `IServiceCollection` | 服务集合实例 |
| `configuration` | `IConfiguration` | 已存在的 IConfiguration 实例 |

**返回值：**

服务集合实例，支持链式调用。

### AddSystemConfig 方法（委托）

使用配置委托注册系统配置。

**类型参数：**

| 参数 | 约束 |
|------|------|
| `TConfig` | `class, ISystemConfig, new()` |

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `services` | `IServiceCollection` | 服务集合实例 |
| `configure` | `Action<TConfig>` | 配置对象的配置委托 |

**返回值：**

服务集合实例，支持链式调用。
