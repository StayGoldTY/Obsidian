好的,我来给你简单介绍下Autofac的用法:

**简介**

Autofac是一个强大的依赖注入(DI)容器框架,可以用来构建应用中的依赖关系。

**安装**

```
// 通过Nuget安装Autofac
Install-Package Autofac
```

**入门**

1. 定义服务接口和实现类

```csharp
public interface IService1{}

public class Service1 : IService1{}
```

2. 创建容器生成器

```csharp 
var builder = new ContainerBuilder();
```

3. 注册服务

```csharp
builder.RegisterType<Service1>().As<IService1>();
```

4. 构建容器

```csharp
IContainer container = builder.Build();
```

5. 解析服务

```csharp
var service = container.Resolve<IService1>();
```

这样就完成了一个基础的依赖注入配置。

**高级用法**

1. 支持生命周期管理

```csharp
builder.RegisterType<Service1>()
       .As<IService1>()
       .SingleInstance(); // 单例 
       
builder.RegisterType<Service2>()
       .InstancePerLifetimeScope(); // 每生命周期一个实例
```

2. 支持属性注入

```csharp
public class UserService 
{
  public UserService(IDbContext ctx) 
  {
    ...
  }
}

builder.RegisterType<UserService>()
       .WithParameter(new TypedParameter(typeof(IDbContext), ctx));
```

3. 支持自动注入

```csharp
builder.RegisterType<UserService>().As<IUserService>()
       .PropertiesAutowired();  
```

4. 支持方法注入

```csharp
public void SetCache(ICache cache)
{
  _cache = cache;
}

builder.RegisterType<MyService>()
       .OnActivated(e => e.Instance.SetCache(e.Context.Resolve<ICache>()));
```
      
5. 更多高级用法

好的,来继续补充一些Autofac更高级的用法:

**模块化注册**

Autofac支持将注册逻辑封装到Module类中,然后统一加载Module进行注册,实现模块化管理。

```csharp
public class ServiceModule : Module 
{
  protected override void Load(ContainerBuilder builder)
  {
    builder.RegisterType<Service1>().As<IService1>(); 
    builder.RegisterType<Service2>();
  }
}

builder.RegisterModule<ServiceModule>();
```

**自动批量注册**

可以自动注册某个程序集下的所有实现:

```csharp
// 自动注册IService接口的实现
builder.RegisterAssemblyTypes(asm)
       .Where(t => t.IsAssignableTo<IService>())
       .AsImplementedInterfaces();
```

**拦截器**

可以通过拦截器实现AOP功能:

```csharp
public class LogInterceptor : IInterceptor
{
  public void Intercept(IInvocation invocation)
  {
    // 调用前后添加日志
  }
}

builder.RegisterType<Service>()
       .EnableInterfaceInterceptors()
       .InterceptedBy(typeof(LogInterceptor));
```

**范围生命周期**

可以自定义生命周期范围,例如实现一个request范围:

```csharp 
builder.RegisterType<RequestCache>()
       .WithParameter(new TypedParameter(typeof(IDictionary), requestItems));
       
builder.RegisterType<UserService>()
       .InstancePerMatchingLifetimeScope("request");
```

**组件事件**

利用事件机制在组件各生命周期触发自定义事件:

```csharp
public class MyComponent 
{
  public bool IsInitialized {get;set;}
  
  public void Init()
  {
    IsInitialized = true;
  }
}

builder.RegisterType<MyComponent>()
       .OnActivating(e => e.Instance.Init())
       .OnActivated(e => Console.WriteLine(e.Instance.IsInitialized))
```

以上是一些更高级的用法,可以让Autofac的DI功能更加强大和灵活。这些概念掌握后可以应用到实际系统中,构建出松耦合、可扩展的应用程序结构。

***

#原理概述

ContainerBuilder类是Autofac中的一个核心类,它的主要作用是用来注册服务及组件,并最终构建出IoC容器。

ContainerBuilder的内部原理主要包含以下几点:

1. 它内部维护了一个ServiceRegistry实例,这个实例是组件注册表,用于存储所有注册的组件。

2. 在ContainerBuilder上调用各种Register/RegisterAssembly/RegisterModule等注册方法,会向这个ServiceRegistry中添加对应的注册信息。

3. ServiceRegistry以树形结构存储组件的元数据,包括组件类型、生命周期、依赖关系等。

4. ContainerBuilder上的Build()方法会使用ServiceRegistry生成 container 实例。

5. Build方法会先对ServiceRegistry中的数据进行编译和验证,看注册是否存在问题。

6. 然后通过一系列优化,生成用于实例化组件的委托和缓存。

7. 最后构建出 IoC 容器 Container 实例,并内置了服务解析、依赖查找、生命周期管理等功能。

8. 容器实例中也会持有对 ServiceRegistry 的引用,用来查找组件的注册信息。

所以ContainerBuilder的核心作用是提供了一系列的注册API,这些API会操作内部的注册表,并在Build时用于生成IoC容器。

它屏蔽了组件元数据的存储、树形结构等底层细节,使得我们可以通过简单的API进行组件配置。


## <font color= #A35568>尝试对ContainerBuilder的内部原理做一个更详细的说明:</font>

1. ContainerBuilder类中有一个字段_serviceRegistry,它的类型是ServiceRegistry。这个ServiceRegistry是组件注册的存储仓库。

2. 当我们调用builder.RegisterType\<A\>()时,它会创建一个ComponentRegistration实例,包含了A的类型信息、生命周期等元数据。

3. 然后它会调用_serviceRegistry.AddService(componentRegistration)方法,将这个组件注册信息添加到ServiceRegistry中去。

4. ServiceRegistry内部会以树形结构存储每个组件的注册信息,一级是组件类型,二级是生命周期等修饰符。

5. 当调用builder.Build()时,它会首先调用ServiceRegistry.Validate()进行验证,确保注册是合法的。

6. 然后遍历ServiceRegistry的树形结构,将每个组件的Resolve方法生成委托,存入一个字典中。

7. 同时还会分析依赖关系,使用算法排序确定初始化顺序,也缓存到字典。

8. 最后使用这些字典构建出Container实例,包含了组件的解析和初始化逻辑。

9. 这个Container也持有对ServiceRegistry的引用,用来查找元数据。

所以简单来说,ContainerBuilder负责收集注册信息,ServiceRegistry存储注册数据,最后ContainerBuilder使用注册数据构建真正的容器。

Builder和Container分工明确,实现了职责隔离。希望这样能帮你对整个过程有一个更清晰的理解。如果你还有其他疑问,欢迎随时提出来。

***
对于被拦截服务
为了发挥Autofac拦截器的最大作用,通常有以下几点建议:

1. 被拦截的类最好是接口类,或者至少定义有接口。因为拦截器是作用于接口的,如果类没有接口的话,就只能拦截类的公共虚方法。
```
拦截器是作用于接口的,因为Autofac在生成代理类时,是为接口生成代理,然后实现接口方法的拦截逻辑。所以如果类没有接口,就无法为其生成代理类,只能直接拦截类的虚方法
```

2. 类需要注册为服务才能被拦截。Autofac需要在注册时启用拦截器,所以非服务类无法在注册时指定拦截器。
```
只有注册为服务的类才会经过容器的控制,在注入时启用拦截器。非服务类是直接 new 出来的,无法在创建时应用拦截器
```
***
```
进一步解释：
“注册为服务的类”有以下特点:

1. 被定义为一个接口或抽象类的实现。这样容器可以通过接口或抽象类类型来获取实例。
2. 通过代码的方式被注册到容器中。如在Autofac中,通过builder.RegisterType<MyClass>()的方式明确注册。
3. 可以有标记特性,被容器自动发现并注册。如 ASP.NET Core的[FromService]特性。
4. 注册时会指定生命周期范围。如单例,每请求一次等。
5. 允许容器在注入时自动解析其依赖。
6. 可以在注册时启用扩展功能,如拦截器,验证等。
7. 注册信息会被容器的内部服务表记录,以支持类型解析。
8. 外部代码需要通过容器解析实例,而不是直接 new 创建。

满足这些条件的类,我们就可以认为它是一个正式由容器管理的服务类了。

相对来说,非服务类指的是任何直接通过 new 创建的类,它们不会被容器控制和扩展
```
3. 类需要实例化才能被拦截。拦截器作用于具体的类实例,静态方法无法拦截。
```
1. 拦截器的拦截逻辑需要注入到具体的实例中,所以作用对象必须是实例,静态方法属于类不属于实例,无法注入拦截逻辑。
```

4. 类最好使用依赖注入而不是直接 new 创建。因为拦截器需要在依赖注入时通过容器启用。
```
同样原因,依赖注入时可以通过容器启用拦截器,直接new出来的对象无法启用拦截器
```

5. 如果要拦截虚方法,被拦截的类需要是public或internal可访问性。private方法无法拦截。
```
Autofac生成的代理类会继承被拦截类,所以只能拦截公开的虚方法,private方法对代理类不可见,无法拦截
```

6. 构造函数无法被拦截。因为在构造时实例还未完成注入。
```
构造函数在实例创建时就被调用了,这时拦截器还未注入,所以无法拦截构造函数
```

7. 尽量使用接口注入,不要使用具体类注入,这会绕过拦截器。
```
您提到的这句话的意思是:

如果我们直接将一个具体类注入到另一个类中,那么这个具体类的拦截器将不会生效。

举个例子:

```csharp
// A类中注入了具体类B 
public class A {
  public A(B b) {
    // ...
  } 
}

// B类启用了拦截器
builder.RegisterType<B>().As<IB>().EnableInterfaceInterceptors();

// 通过容器构造A,注入B
var a = container.Resolve<A>();
```

在这里,although类B启用了拦截器,但在A中注入的是具体类B,不是接口IB。

这时 although的方法调用不会触发B的拦截器。

因为拦截器只会生效在通过接口注入时。

如果改为:

```csharp 
public class A {
  public A(IB b) { // 注入接口 
    // ...
  }
}
```

那么当A通过IB接口注入B时,B的拦截器就会生效了。

所以接口注入可以触发拦截器,而具体类注入会绕过拦截器。

这是因为拦截器是在接口层面生成代理的。




8. 使用虚方法而不是override方法,后者也会绕过拦截器。

总之,被拦截的服务类最好作为接口的实现,注册为服务,使用依赖注入和虚方法,这样可以发挥拦截器最大的作用。但技术上任何公开类方法都可以拦截。

您提到的这句话是指,当使用override关键字重写了基类的虚方法时,该重写的方法会绕过拦截器。

这里的原因在于,当重写(override)一个基类虚方法时,该方法会直接调用基类的方法,而不会通过子类的代理。

让我举一个例子:

```csharp
public class Base {
  public virtual void Log() {
    //...
  }
}

public class Child : Base {

  public override void Log() { 
    // 重写了Log方法
  }

}
```

对Child类启用拦截器:

```csharp
builder.RegisterType<Child>().EnableClassInterceptors(); 
```

此时,当Child调用Log方法时,会直接调用Base的Log实现,不会通过Child的代理类。

也就相当于跳过了拦截器。

而如果是使用新方法(而不是重写):

```csharp
public class Child : Base {

  public new void Log() {
    // 新方法 
  }

}
```

这样当Child调用Log时,才会通过代理类,触发拦截器。

所以重写(override)基类方法会绕过拦截器,而新方法(new)可以被拦截。

这是因为 override 的工作机制决定了它会直接调用基类方法实现。

希望这个例子可以帮助您理解 override 与拦截器的关系。请让我知道如果还有不清楚的地方。


***
#问  ：builder.RegisterType\<MyClass\>() 这种方式注册后有什么用?
好的,关于将类注册为服务后可以通过接口或抽象类进行注入,我来详细解释一下:

1. 首先定义一个接口和实现类:

```csharp
public interface IService {}

public class MyService : IService {}
```

2. 在Autofac中注册MyService,并指定服务类型为IService接口:

```csharp
builder.RegisterType<MyService>().As<IService>();
```

3. 这样在其它类中,就可以通过构造函数注入IService接口来使用MyService类了:

```csharp
public class MyController
{
  public MyController(IService service) 
  {
    // 这里会注入MyService实例
  }
}
```

4. 注册时使用As\<T\>指定接口后,容器会知道用MyService类来满足IService类型的依赖。

5. 所以在任何需要IService的地方,都可以注入MyService实例,这是接口注入的好处。

6. 注册成接口类型,也方便后续切换不同实现,只需要注册另一个类也实现了IService即可。

7. 使用抽象类作为服务类型也是同理。

所以注册为接口服务的主要好处是,可以通过接口/抽象类解耦不同类的依赖关系,这样后续维护和扩展会更方便。

***
#问：我能这样理解吗？     cat 和 dog都注册指定服务类型为IAnimal，那么我们再大部分地方只需要使用IAnimal就行了，然后根据实际情况来使用cat或者dog?

您的理解完全正确!

使用接口注册服务的一个主要优点就是可以实现服务类之间的解耦。

举您这个例子:

```csharp
public interface IAnimal {}

public class Cat : IAnimal {} 

public class Dog : IAnimal {}

// 注册
builder.RegisterType<Cat>().As<IAnimal>();
builder.RegisterType<Dog>().As<IAnimal>();

// 使用
public class Zoo {
  public Zoo(IAnimal animal) {
    // 这里可以注入Cat或者Dog
  }
}
```

在大多数情况下,我们只需要依赖抽象的IAnimal接口,而不是具体的Cat或者Dog。

那么后续如果需要切换Cat为Dog,或者添加新的animal,我们可以简单地在注册时做修改,不需要改动太多使用Animal的代码。

这就是依赖注入和面向接口编程的好处 - 实现了耦合度的降低。

所以您的理解是非常准确的!接口注册能让我们写出更加松耦合、可扩展和灵活的代码。

