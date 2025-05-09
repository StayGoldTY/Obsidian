

动态代理的核心原理有两种主要实现方式：

1. 接口代理（Interface Proxy）
```csharp
public interface IUserService {
    User GetUser(int id);
}

// 动态代理实现
public class UserServiceProxy : IUserService {
    private readonly IUserService _realService;
    private readonly IInterceptor _interceptor;

    public UserServiceProxy(IUserService realService, IInterceptor interceptor) {
        _realService = realService;
        _interceptor = interceptor;
    }

    public User GetUser(int id) {
        // 拦截方法调用
        var context = new InvocationContext {
            Method = GetType().GetMethod("GetUser"),
            Arguments = new object[] { id }
        };

        // 执行拦截器逻辑
        _interceptor.Intercept(context);

        // 可以决定是否调用真实方法
        return _realService.GetUser(id);
    }
}
```

2. 动态类型代理（Dynamic Type Proxy）
```csharp
// 使用DispatchProxy实现动态代理
public class CustomProxy<T> : DispatchProxy where T : class 
{
    private T _realInstance;
    private IInterceptor _interceptor;

    public static T Create(T realInstance, IInterceptor interceptor) 
    {
        object proxy = Create<T, CustomProxy<T>>();
        var customProxy = (CustomProxy<T>)proxy;
        
        customProxy._realInstance = realInstance;
        customProxy._interceptor = interceptor;

        return (T)proxy;
    }

    // 拦截方法调用的核心方法
    protected override object Invoke(MethodInfo targetMethod, object[] args) 
    {
        // 创建调用上下文
        var invocation = new InvocationContext {
            Method = targetMethod,
            Arguments = args,
            TargetInstance = _realInstance
        };

        // 执行拦截器逻辑
        _interceptor.Intercept(invocation);

        // 可以决定是否调用真实方法
        return targetMethod.Invoke(_realInstance, args);
    }
}

// 拦截器接口
public interface IInterceptor 
{
    void Intercept(InvocationContext context);
}

// 调用上下文
public class InvocationContext 
{
    public MethodInfo Method { get; set; }
    public object[] Arguments { get; set; }
    public object TargetInstance { get; set; }
    public object ReturnValue { get; set; }
}
```

实际使用示例：
```csharp
// 使用动态代理
public class LoggingInterceptor : IInterceptor 
{
    public void Intercept(InvocationContext context) 
    {
        // 前置拦截
        Console.WriteLine($"调用方法: {context.Method.Name}");
        
        // 可以修改参数
        // 可以阻止方法执行
        
        // 执行原始方法
        context.ReturnValue = context.Method.Invoke(
            context.TargetInstance, 
            context.Arguments
        );
        
        // 后置拦截
        Console.WriteLine($"方法执行完成: {context.Method.Name}");
    }
}

// 创建代理
IUserService userService = new UserService();
IUserService proxyService = CustomProxy<IUserService>.Create(
    userService, 
    new LoggingInterceptor()
);
```

3. 现代Mock框架(如Moq)的实现
Moq等框架通过更复杂的技术实现，主要步骤：

```csharp
public class MoqProxyGenerator 
{
    public static T CreateMock<T>() where T : class 
    {
        return new ProxyGenerator().CreateInterfaceProxyWithoutTarget<T>(
            new StandardInterceptor()
        );
    }
}
```

关键技术点：
- 反射
- 动态代理
- 运行时类型生成
- 方法拦截
- 调用上下文管理

要点解释：
1. 不是简单地返回 `new MockInterceptor()`
2. 是创建一个动态代理对象
3. 使用拦截器控制方法调用行为
4. 可以完全自定义方法执行逻辑

核心原理图：
```
原始对象 
    ↓
动态代理对象 
    ↓
拦截器(IInterceptor)
    ↓
控制方法调用
    ↓
决定是否执行原始方法
    ↓
返回结果
```

这就是动态代理的基本原理，也是Mock框架的核心技术实现方式。