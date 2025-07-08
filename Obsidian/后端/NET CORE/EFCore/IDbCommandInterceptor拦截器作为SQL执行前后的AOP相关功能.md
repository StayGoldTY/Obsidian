### 使用 `IDbCommandInterceptor`（推荐 ✅）

EF Core 从 5.0 开始提供了正式的拦截器机制（类似于 MyBatis 的 Interceptor 插件）！

#### 🧩 自定义拦截器统计 SQL 执行时间：


```
public class SqlTimingInterceptor : DbCommandInterceptor
{
    private readonly ILogger<SqlTimingInterceptor> _logger;

    public SqlTimingInterceptor(ILogger<SqlTimingInterceptor> logger)
    {
        _logger = logger;
    }

    public override InterceptionResult<DbDataReader> ReaderExecuting(
        DbCommand command, CommandEventData eventData, InterceptionResult<DbDataReader> result)
    {
        var stopwatch = Stopwatch.StartNew();
        var res = base.ReaderExecuting(command, eventData, result);
        stopwatch.Stop();

        _logger.LogInformation($"SQL 耗时：{stopwatch.ElapsedMilliseconds}ms\nSQL: {command.CommandText}");

        return res;
    }
}

```