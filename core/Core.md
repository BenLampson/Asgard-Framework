# Core 核心模块

## 概述

Core 模块是 Asgard 项目的核心基础设施层，提供了系统配置、通用工具和基础抽象等功能。

## 模块列表

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| SystemConfig | 系统配置模块，提供 YAML 配置文件加载和管理功能 | [查看文档](./systemconfig/README.md) |

## 快速开始

### 安装

在项目中引用 Asgard.Core：

```xml
<ProjectReference Include="..\Common\Asgard.Core\Asgard.Core.csproj" />
```

### 基本使用

```csharp
using Asgard.Core.SystemConfig;

// 定义配置类
public class AppConfig : ISystemConfig
{
    [ConfigPath("app.name", DefaultValue = "DefaultApp")]
    public string Name { get; set; } = string.Empty;

    public void Validate()
    {
        if (string.IsNullOrEmpty(Name))
            throw new InvalidOperationException("应用程序名称不能为空");
    }
}

// 加载配置
var config = YamlConfigLoader.LoadFromFile<AppConfig>("config.yaml");
```

## 依赖项

- Microsoft.Extensions.Configuration
- Microsoft.Extensions.DependencyInjection
- Microsoft.Extensions.Options
- YamlDotNet
