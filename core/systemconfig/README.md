# SystemConfig 系统配置模块

## 概述

SystemConfig 模块提供了基于 YAML 格式的配置文件加载和管理功能。该模块支持将 YAML 配置文件中的配置项映射到强类型的配置类，并提供配置验证机制。

## 功能特性

- **YAML 配置加载**：支持从 YAML 文件或字符串内容加载配置
- **路径映射**：通过特性标记实现 YAML 路径到属性的映射
- **类型转换**：自动支持字符串、整数、布尔、枚举等类型的转换
- **默认值支持**：支持为配置属性设置默认值
- **配置验证**：提供配置验证接口，确保配置的有效性
- **依赖注入集成**：与 Microsoft.Extensions.DependencyInjection 无缝集成

## 文档目录

| 文档 | 说明 |
|------|------|
| [快速开始](./quickstart.md) | 快速上手指南 |
| [核心组件](./components.md) | 核心组件详细说明 |
| [API 参考](./api.md) | API 详细文档 |

## 快速示例

```csharp
using Asgard.Core.SystemConfig;

// 定义配置类
public class AppConfig : ISystemConfig
{
    [ConfigPath("app.name", DefaultValue = "DefaultApp")]
    public string Name { get; set; } = string.Empty;

    [ConfigPath("app.version", DefaultValue = "1.0.0")]
    public string Version { get; set; } = string.Empty;

    public void Validate()
    {
        if (string.IsNullOrEmpty(Name))
            throw new InvalidOperationException("应用程序名称不能为空");
    }
}

// 加载配置
var config = YamlConfigLoader.LoadFromFile<AppConfig>("config.yaml");
```
