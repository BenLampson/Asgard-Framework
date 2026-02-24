# 缓存模块

缓存模块是 Asgard 框架的核心组件之一，提供了多级缓存功能，结合本地内存缓存（一级）和分布式缓存（二级），兼顾性能与一致性。

## 功能特性

- **多级缓存**：结合内存缓存和分布式缓存
- **高性能**：本地缓存减少网络延迟，适合高频读取
- **一致性**：分布式缓存保证多实例数据一致
- **易用性**：统一的 API 接口，支持异步操作
- **可扩展性**：基于 Microsoft.Extensions.Caching 生态系统
- **配置驱动**：支持 YAML 配置文件，与 Asgard 配置体系集成

## 快速开始

### 1. 安装依赖

在项目中添加对 Asgard.Core 的引用。

### 2. 配置缓存

创建 `caching.yaml` 配置文件：

```yaml
caching:
  memory:
    enabled: true
    defaultExpirationMinutes: 5
    sizeLimit: 104857600  # 100MB
    compactOnMemoryPressure: 0.9
    expirationScanFrequencyMinutes: 1
  redis:
    enabled: true
    connectionString: "localhost:6379"
    instanceName: "Asgard:"
    defaultExpirationMinutes: 30
    connectTimeout: 5000
    syncTimeout: 5000
    asyncTimeout: 5000
    allowAdmin: false
    ssl: false
    password: ""
    database: 0
    retryCount: 3
    retryIntervalMilliseconds: 1000
    fallbackToMemoryCache: true
```

### 3. 注册服务

在 `Program.cs` 中注册多级缓存服务：

```csharp
// 方式1：从配置文件加载
builder.Services.AddSystemConfig<CacheConfig>("caching.yaml");
builder.Services.AddMultiLevelCache();

// 方式2：直接传入配置
var config = new CacheConfig
{
    Memory = { Enabled = true, DefaultExpirationMinutes = 5 },
    Redis = { Enabled = true, ConnectionString = "localhost:6379" }
};
builder.Services.AddMultiLevelCache(config);

// 方式3：使用配置委托
builder.Services.AddMultiLevelCache(c =>
{
    c.Memory.Enabled = true;
    c.Memory.DefaultExpirationMinutes = 5;
    c.Redis.ConnectionString = "localhost:6379";
    c.Redis.InstanceName = "MyApp:";
});
```

### 4. 使用缓存

通过依赖注入使用 `IMultiLevelCache` 接口：

```csharp
public class ProductService
{
    private readonly IMultiLevelCache _cache;

    public ProductService(IMultiLevelCache cache)
    {
        _cache = cache;
    }

    public async Task<Product> GetProductAsync(int id)
    {
        // 读取缓存
        var cacheKey = $"product:{id}";
        var product = await _cache.GetAsync<Product>(cacheKey);

        if (product == null)
        {
            // 缓存未命中，从数据源获取
            product = await FetchProductFromDatabaseAsync(id);

            if (product != null)
            {
                // 设置缓存，指定过期时间
                await _cache.SetAsync(cacheKey, product, TimeSpan.FromMinutes(30));
            }
        }

        return product;
    }

    public async Task UpdateProductAsync(Product product)
    {
        // 更新数据库
        await UpdateProductInDatabaseAsync(product);

        // 删除缓存
        await _cache.RemoveAsync($"product:{product.Id}");
    }

    private async Task<Product> FetchProductFromDatabaseAsync(int id)
    {
        // 模拟从数据库获取数据
        await Task.Delay(100);
        return new Product { Id = id, Name = $"Product {id}" };
    }

    private async Task UpdateProductInDatabaseAsync(Product product)
    {
        // 模拟更新数据库
        await Task.Delay(100);
    }
}
```

## API 参考

### IMultiLevelCache 接口

| 方法 | 说明 |
|------|------|
| `GetAsync<T>(string key, CancellationToken cancellationToken)` | 读取缓存（先查一级，再查二级） |
| `SetAsync<T>(string key, T value, TimeSpan expiration, CancellationToken cancellationToken)` | 设置缓存（指定过期时间） |
| `SetAsync<T>(string key, T value, CancellationToken cancellationToken)` | 设置缓存（使用默认过期时间） |
| `RemoveAsync(string key, CancellationToken cancellationToken)` | 删除缓存 |
| `ExistsAsync(string key, CancellationToken cancellationToken)` | 检查缓存是否存在 |
| `RefreshAsync<T>(string key, CancellationToken cancellationToken)` | 刷新缓存（从二级缓存重新加载） |
| `RemoveRangeAsync(IEnumerable<string> keys, CancellationToken cancellationToken)` | 批量删除缓存 |
| `ClearAsync(CancellationToken cancellationToken)` | 清空所有缓存 |

### CacheConfig 配置类

| 属性 | 说明 |
|------|------|
| `Memory` | 内存缓存配置 |
| `Redis` | Redis 缓存配置 |

### MemoryCacheOptions 配置类

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `Enabled` | bool | true | 是否启用内存缓存 |
| `DefaultExpirationMinutes` | int | 5 | 默认过期时间（分钟） |
| `SizeLimit` | long? | null | 缓存大小限制（字节） |
| `CompactOnMemoryPressure` | double | 0.9 | 压缩比例阈值 |
| `ExpirationScanFrequencyMinutes` | int | 1 | 过期扫描频率（分钟） |

### RedisCacheOptions 配置类

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `Enabled` | bool | true | 是否启用 Redis 缓存 |
| `ConnectionString` | string | "localhost:6379" | Redis 连接字符串 |
| `InstanceName` | string | "Asgard:" | 实例名称前缀 |
| `DefaultExpirationMinutes` | int | 30 | 默认过期时间（分钟） |
| `ConnectTimeout` | int | 5000 | 连接超时（毫秒） |
| `SyncTimeout` | int | 5000 | 同步操作超时（毫秒） |
| `AsyncTimeout` | int | 5000 | 异步操作超时（毫秒） |
| `AllowAdmin` | bool | false | 是否允许管理操作 |
| `Ssl` | bool | false | 是否使用 SSL |
| `Password` | string? | null | 密码 |
| `Database` | int | 0 | 数据库索引 |
| `RetryCount` | int | 3 | 重试次数 |
| `RetryIntervalMilliseconds` | int | 1000 | 重试间隔（毫秒） |
| `FallbackToMemoryCache` | bool | true | 连接失败时是否回退到内存缓存 |

## 实现原理

### 缓存读取流程

1. 首先查询本地内存缓存（一级缓存）
2. 如果本地缓存命中，直接返回
3. 如果本地缓存未命中，查询分布式缓存（二级缓存）
4. 如果分布式缓存命中，将数据写入本地缓存并返回
5. 如果分布式缓存也未命中，返回默认值

### 缓存写入流程

1. 将数据序列化为 JSON
2. 写入分布式缓存（二级缓存）
3. 写入本地内存缓存（一级缓存）

### 缓存删除流程

1. 从本地内存缓存中删除数据
2. 从分布式缓存中删除数据

## 最佳实践

### 1. 缓存键设计

使用有意义的前缀和唯一标识：

```csharp
// 推荐
var key = $"user:{userId}:profile";
var key = $"product:{productId}:detail";
var key = $"order:{orderId}:items";

// 不推荐
var key = userId.ToString();
var key = "cache_" + productId;
```

### 2. 过期时间设置

根据数据更新频率设置合理的过期时间：

```csharp
// 热点数据：较短过期时间
await _cache.SetAsync(key, data, TimeSpan.FromMinutes(5));

// 配置数据：较长过期时间
await _cache.SetAsync(key, config, TimeSpan.FromHours(1));

// 静态数据：更长过期时间
await _cache.SetAsync(key, staticData, TimeSpan.FromDays(1));
```

### 3. 缓存预热

系统启动时预先加载热点数据：

```csharp
public class CacheWarmupService : IHostedService
{
    private readonly IMultiLevelCache _cache;

    public async Task StartAsync(CancellationToken cancellationToken)
    {
        // 预加载热点数据
        var hotProducts = await GetHotProductsAsync();
        foreach (var product in hotProducts)
        {
            await _cache.SetAsync($"product:{product.Id}", product, TimeSpan.FromMinutes(30));
        }
    }
}
```

### 4. 缓存穿透防护

使用缓存空值防止缓存穿透：

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    var key = $"product:{id}";
    var product = await _cache.GetAsync<Product>(key);

    if (product == null)
    {
        product = await _dbContext.Products.FindAsync(id);

        if (product != null)
        {
            await _cache.SetAsync(key, product, TimeSpan.FromMinutes(30));
        }
        else
        {
            // 缓存空值，防止缓存穿透
            await _cache.SetAsync(key, new Product(), TimeSpan.FromMinutes(5));
        }
    }

    return product?.Id > 0 ? product : null;
}
```

### 5. 容错处理

处理 Redis 连接失败的情况：

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    try
    {
        var key = $"product:{id}";
        return await _cache.GetAsync<Product>(key);
    }
    catch (RedisConnectionException)
    {
        // Redis 连接失败，直接从数据库获取
        return await _dbContext.Products.FindAsync(id);
    }
}
```

## 常见问题

### Q: 为什么有时候读取到的是旧数据？

A: 本地缓存有一定的过期时间，可能会短暂存在数据不一致的情况。可以通过以下方式缓解：
- 缩短本地缓存的过期时间
- 在数据更新时主动删除缓存
- 使用缓存失效通知机制

### Q: 如何处理复杂对象的缓存？

A: 多级缓存会自动处理对象的序列化和反序列化，默认使用 JSON 序列化。确保对象是可序列化的。

### Q: Redis 不可用时会怎样？

A: 如果配置了 `FallbackToMemoryCache = true`，Redis 连接失败时会自动回退到仅使用内存缓存模式。建议添加异常处理和重试机制。

### Q: 如何监控缓存命中率？

A: 可以通过扩展 `IMultiLevelCache` 接口，添加监控指标：

```csharp
public class MonitoredMultiLevelCache : IMultiLevelCache
{
    private readonly IMultiLevelCache _innerCache;
    private readonly IMetricsCollector _metrics;

    public async Task<T?> GetAsync<T>(string key, CancellationToken cancellationToken = default)
    {
        var result = await _innerCache.GetAsync<T>(key, cancellationToken);
        _metrics.RecordCacheHit(result != null);
        return result;
    }
}
```

### Q: 如何在分布式环境下保证缓存一致性？

A: 可以使用 Redis 的发布/订阅功能实现缓存失效通知：

```csharp
// 发布缓存失效消息
await _redis.PublishAsync("cache:invalidate", key);

// 订阅缓存失效消息
_redis.Subscribe("cache:invalidate", (channel, message) =>
{
    _memoryCache.Remove(message);
});
```
