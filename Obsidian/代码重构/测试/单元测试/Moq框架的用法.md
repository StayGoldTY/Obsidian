
## 基础概念

1. **Mock 是什么？**
Mock 是在单元测试中创建的一个虚拟对象，用来模拟真实对象的行为。比如当你测试用户服务时，你可能不想真的连接数据库，这时就可以使用 Mock 对象来模拟数据库操作。

让我们通过一个简单的例子来理解基础概念：



```csharp
// 1. 首先我们有一个简单的接口
public interface ICalculator
{
    int Add(int a, int b);
    string GetVersion();
}

// 2. 基础概念示例
public class MoqBasicTests
{
    [Fact]
    public void BasicConceptsDemo()
    {
        // 创建 Mock 对象
        var mockCalculator = new Mock<ICalculator>();

        // Setup（设置）: 告诉 Mock 对象在特定情况下应该做什么
        // 下面这行代表：当调用 Add 方法，并且参数是 2 和 3 时，返回 5
        mockCalculator.Setup(calc => calc.Add(2, 3)).Returns(5);

        // Object（获取模拟对象）: 获取可以实际使用的模拟对象
        ICalculator calculator = mockCalculator.Object;

        // 使用模拟对象
        int result = calculator.Add(2, 3); // 将返回 5

        // Verify（验证）: 确认某个方法是否被调用
        mockCalculator.Verify(calc => calc.Add(2, 3), Times.Once());
    }

    // 3. Setup 的不同方式
    [Fact]
    public void SetupExamplesDemo()
    {
        var mockCalculator = new Mock<ICalculator>();

        // 3.1 固定值返回
        mockCalculator.Setup(calc => calc.GetVersion()).Returns("1.0");

        // 3.2 使用 It.IsAny<T>() 匹配任意参数
        mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()))
                     .Returns(10);

        // 3.3 根据条件返回不同的值
        mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>()))
                     .Returns(1);

        var calculator = mockCalculator.Object;

        // 测试这些设置
        string version = calculator.GetVersion();        // 返回 "1.0"
        int result1 = calculator.Add(5, 3);             // 返回 10
        int result2 = calculator.Add(1, 1);             // 返回 1
    }

    // 4. Verify 的基础用法
    [Fact]
    public void VerifyBasicsDemo()
    {
        var mockCalculator = new Mock<ICalculator>();

        var calculator = mockCalculator.Object;
        calculator.GetVersion();

        // 验证 GetVersion 被调用了一次
        mockCalculator.Verify(calc => calc.GetVersion(), Times.Once());

        // 验证 Add 从未被调用过
        mockCalculator.Verify(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()), Times.Never());
    }
}

```

让我详细解释这些基础概念：

1. **创建 Mock 对象**
```csharp
var mockCalculator = new Mock<ICalculator>();
```
- 这行代码创建了一个 ICalculator 接口的模拟对象
- Mock< T> 中的 T 是你要模拟的接口或类

2. **Setup（设置）**
```csharp
mockCalculator.Setup(calc => calc.Add(2, 3)).Returns(5);
```
- Setup 用来设置模拟对象的行为
- 它告诉模拟对象："当有人用这些参数调用这个方法时，你应该返回什么"
- `calc => calc.Add(2, 3)` 是一个 Lambda 表达式，指定了方法调用的条件
- `Returns(5)` 指定了返回值

3. **Object（获取模拟对象）**
```csharp
ICalculator calculator = mockCalculator.Object;
```
- `.Object` 用于获取实际可以使用的模拟对象
- 这个对象可以像真实对象一样使用

4. **Verify（验证）**
```csharp
mockCalculator.Verify(calc => calc.Add(2, 3), Times.Once());
```
- Verify 用来验证某个方法是否被调用过
- `Times.Once()` 表示方法应该被调用一次
- 也可以用 `Times.Never()`、`Times.Exactly(n)` 等

5. **参数匹配器**
```csharp
It.IsAny<int>()  // 匹配任何整数
It.Is<int>(x => x > 0)  // 匹配大于 0 的整数
```
- `It.IsAny<T>()` 匹配任何类型为 T 的参数
- `It.Is<T>(predicate)` 根据条件匹配参数

这些是最基础的概念。你更想深入了解哪个概念？或者需要我举一些更实际的例子吗？

***
***
***
问：里面的基础语法我还是不太情况，比如上面的IsAny或者is相关的属性用法我也不了解，能不能进一步从这些基础属性用法开始讲解

好的，让我详细讲解 Moq 中的这些基础属性和匹配器的用法。

```csharp
// 首先定义一个简单的接口用于演示
public interface IUserService
{
    User GetUser(int id);
    bool ValidateUser(string username, string password);
    void SaveUser(User user);
    bool UpdateUserAge(int userId, int age);
    List<User> FindUsers(string searchTerm);
}

public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public int Age { get; set; }
}

public class MatchersExampleTests
{
    private Mock<IUserService> _mockUserService;

    public MatchersExampleTests()
    {
        _mockUserService = new Mock<IUserService>();
    }

    // 1. It.IsAny<T>() 的用法 - 匹配任意值
    [Fact]
    public void IsAny_Examples()
    {
        // 匹配任意整数 ID
        _mockUserService.Setup(x => x.GetUser(It.IsAny<int>()))
            .Returns(new User { Id = 1, Username = "test" });

        // 匹配任意用户名和密码
        _mockUserService.Setup(x => x.ValidateUser(
            It.IsAny<string>(), 
            It.IsAny<string>()))
            .Returns(true);

        // 测试
        var user1 = _mockUserService.Object.GetUser(1);  // 返回测试用户
        var user2 = _mockUserService.Object.GetUser(100);  // 也返回测试用户
        var isValid = _mockUserService.Object.ValidateUser("任意用户名", "任意密码");  // 返回 true
    }

    // 2. It.Is<T>() 的用法 - 根据条件匹配
    [Fact]
    public void Is_Examples()
    {
        // 匹配大于 0 的 ID
        _mockUserService.Setup(x => x.GetUser(It.Is<int>(id => id > 0)))
            .Returns(new User { Id = 1, Username = "正常用户" });

        // 匹配小于等于 0 的 ID
        _mockUserService.Setup(x => x.GetUser(It.Is<int>(id => id <= 0)))
            .Returns((User)null);

        // 匹配特定条件的用户名和密码
        _mockUserService.Setup(x => x.ValidateUser(
            It.Is<string>(u => u.Length >= 3),  // 用户名长度大于等于3
            It.Is<string>(p => p.Length >= 6))) // 密码长度大于等于6
            .Returns(true);

        // 测试
        var validUser = _mockUserService.Object.GetUser(1);      // 返回正常用户
        var invalidUser = _mockUserService.Object.GetUser(-1);   // 返回 null
        var isValid = _mockUserService.Object.ValidateUser("abc", "123456"); // 返回 true
    }

    // 3. It.IsIn<T>() 和 It.IsNotIn<T>() 的用法 - 匹配集合中的值
    [Fact]
    public void IsIn_Examples()
    {
        // 匹配指定的几个 ID
        _mockUserService.Setup(x => x.GetUser(It.IsIn<int>(1, 2, 3)))
            .Returns(new User { Id = 1, Username = "特定用户" });

        // 匹配不在指定范围内的 ID
        _mockUserService.Setup(x => x.GetUser(It.IsNotIn<int>(1, 2, 3)))
            .Returns((User)null);

        // 测试
        var validUser = _mockUserService.Object.GetUser(2);    // 返回特定用户
        var invalidUser = _mockUserService.Object.GetUser(4);  // 返回 null
    }

    // 4. It.IsRegex() 的用法 - 使用正则表达式匹配
    [Fact]
    public void IsRegex_Examples()
    {
        // 匹配符合邮箱格式的搜索词
        _mockUserService.Setup(x => x.FindUsers(
            It.IsRegex(@"^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$")))
            .Returns(new List<User> { new User { Username = "email_user" } });

        // 测试
        var users1 = _mockUserService.Object.FindUsers("test@example.com"); // 返回用户列表
        var users2 = _mockUserService.Object.FindUsers("invalid-email");    // 返回 null
    }

    // 5. 组合多个条件
    [Fact]
    public void Combined_Conditions_Examples()
    {
        // 使用 && 组合多个条件
        _mockUserService.Setup(x => x.UpdateUserAge(
            It.Is<int>(id => id > 0),                 // ID 必须大于 0
            It.Is<int>(age => age >= 18 && age < 100) // 年龄必须在 18-100 之间
        )).Returns(true);

        // 测试
        var result1 = _mockUserService.Object.UpdateUserAge(1, 25);  // 返回 true
        var result2 = _mockUserService.Object.UpdateUserAge(1, 150); // 返回 false（默认值）
    }
}

```

让我详细解释这些基础属性匹配器：

1. **It.IsAny< T>()** - 匹配任意值
```csharp
// 匹配任意整数
_mockUserService.Setup(x => x.GetUser(It.IsAny<int>()))
```
- 这是最基础的匹配器，表示可以匹配任何类型为 T 的值
- 常用于你不关心具体参数值的情况
- 例如：`It.IsAny<string>()`，`It.IsAny<int>()`，`It.IsAny<User>()`

2. **It.Is< T>()** - 条件匹配
```csharp
// 匹配符合特定条件的值
_mockUserService.Setup(x => x.GetUser(It.Is<int>(id => id > 0)))
```
- 用于设置更具体的匹配条件
- 接受一个 Lambda 表达式作为条件
- 常用于需要验证参数值的场景

3. **It.IsIn< T>()** - 匹配指定集合中的值
```csharp
// 匹配指定的几个值中的任意一个
_mockUserService.Setup(x => x.GetUser(It.IsIn<int>(1, 2, 3)))
```
- 用于匹配参数是否在指定的值列表中
- 有对应的 `It.IsNotIn<T>()` 用于相反的情况

4. **It.IsRegex()** - 正则表达式匹配
```csharp
// 使用正则表达式匹配字符串
_mockUserService.Setup(x => x.FindUsers(
    It.IsRegex(@"^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$")))
```
- 用于使用正则表达式匹配字符串参数
- 常用于需要验证字符串格式的场景

5. **组合条件**
```csharp
// 组合多个条件
_mockUserService.Setup(x => x.UpdateUserAge(
    It.Is< int >(id => id > 0),
    It.Is< int>(age => age >= 18 && age < 100)
))
```
- 可以在一个 Setup 中组合多个条件
- 使用 Lambda 表达式组合复杂的逻辑

6. **一些实用技巧**：
   - 可以使用 `Returns` 设置返回值
   - 可以使用 `Throws` 抛出异常
   - 可以使用 `Callback` 执行自定义操作
   - 默认情况下，未设置的调用会返回类型的默认值

这些是最常用的匹配器，它们可以帮助你在不同场景下精确地控制模拟对象的行为。你想了解哪些具体的使用场景，或者需要更多的示例吗？

***
***
***
## 进阶

我很乐意帮你了解 Moq 框架的使用。我会创建一个包含详细示例的教程。



```csharp
// 首先需要安装 Moq NuGet 包
// dotnet add package Moq

using Moq;
using Xunit;
using System;
using System.Collections.Generic;
using System.Linq;

// 1. 首先定义一些示例接口和类
public interface IUserRepository
{
    User GetById(int id);
    bool Save(User user);
    List<User> GetAllUsers();
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserService
{
    private readonly IUserRepository _userRepository;

    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }

    public User GetUserById(int id)
    {
        return _userRepository.GetById(id);
    }

    public bool CreateUser(User user)
    {
        if (string.IsNullOrEmpty(user.Name) || string.IsNullOrEmpty(user.Email))
            return false;

        return _userRepository.Save(user);
    }
}

// 2. 基础示例：创建 Mock 对象并设置返回值
public class BasicMoqTests
{
    [Fact]
    public void GetUserById_ShouldReturnUser()
    {
        // 准备
        var mockRepo = new Mock<IUserRepository>();
        var expectedUser = new User { Id = 1, Name = "张三", Email = "zhangsan@example.com" };
        
        mockRepo.Setup(repo => repo.GetById(1)).Returns(expectedUser);
        
        var userService = new UserService(mockRepo.Object);

        // 执行
        var result = userService.GetUserById(1);

        // 验证
        Assert.Equal(expectedUser.Id, result.Id);
        Assert.Equal(expectedUser.Name, result.Name);
    }
}

// 3. 验证方法调用
public class VerificationTests
{
    [Fact]
    public void CreateUser_ShouldCallSaveMethod()
    {
        // 准备
        var mockRepo = new Mock<IUserRepository>();
        var userService = new UserService(mockRepo.Object);
        var user = new User { Id = 1, Name = "张三", Email = "zhangsan@example.com" };

        mockRepo.Setup(repo => repo.Save(It.IsAny<User>())).Returns(true);

        // 执行
        userService.CreateUser(user);

        // 验证 Save 方法被调用
        mockRepo.Verify(repo => repo.Save(It.IsAny<User>()), Times.Once());
    }
}

// 4. 使用 It.Is 进行参数匹配
public class ParameterMatchingTests
{
    [Fact]
    public void GetById_WithSpecificParameters()
    {
        var mockRepo = new Mock<IUserRepository>();
        
        mockRepo.Setup(repo => repo.GetById(It.Is<int>(id => id > 0)))
               .Returns(new User { Id = 1, Name = "张三" });

        mockRepo.Setup(repo => repo.GetById(It.Is<int>(id => id <= 0)))
               .Returns((User)null);

        var userService = new UserService(mockRepo.Object);

        var validUser = userService.GetUserById(1);
        var invalidUser = userService.GetUserById(-1);

        Assert.NotNull(validUser);
        Assert.Null(invalidUser);
    }
}

// 5. 设置异常抛出
public class ExceptionTests
{
    [Fact]
    public void GetById_ShouldThrowException()
    {
        var mockRepo = new Mock<IUserRepository>();
        
        mockRepo.Setup(repo => repo.GetById(It.Is<int>(id => id == -1)))
               .Throws(new ArgumentException("Invalid ID"));

        var userService = new UserService(mockRepo.Object);

        Assert.Throws<ArgumentException>(() => userService.GetUserById(-1));
    }
}

// 6. 设置属性
public class PropertySetupTests
{
    [Fact]
    public void SetupProperty()
    {
        var mockRepo = new Mock<IUserRepository>();
        var users = new List<User> 
        { 
            new User { Id = 1, Name = "张三" },
            new User { Id = 2, Name = "李四" }
        };

        mockRepo.Setup(repo => repo.GetAllUsers())
               .Returns(users);

        var result = mockRepo.Object.GetAllUsers();
        
        Assert.Equal(2, result.Count);
    }
}

// 7. 设置回调
public class CallbackTests
{
    [Fact]
    public void SaveUser_WithCallback()
    {
        var mockRepo = new Mock<IUserRepository>();
        var user = new User { Id = 1, Name = "张三" };
        var wasCalled = false;

        mockRepo.Setup(repo => repo.Save(It.IsAny<User>()))
               .Callback<User>(u => wasCalled = true)
               .Returns(true);

        var result = mockRepo.Object.Save(user);

        Assert.True(wasCalled);
        Assert.True(result);
    }
}

// 8. 序列化返回值
public class SequenceTests
{
    [Fact]
    public void GetById_ReturnsSequence()
    {
        var mockRepo = new Mock<IUserRepository>();
        
        mockRepo.SetupSequence(repo => repo.GetById(It.IsAny<int>()))
               .Returns(new User { Id = 1, Name = "张三" })
               .Returns(new User { Id = 2, Name = "李四" })
               .Returns((User)null);

        var userService = new UserService(mockRepo.Object);

        var firstCall = userService.GetUserById(1);
        var secondCall = userService.GetUserById(1);
        var thirdCall = userService.GetUserById(1);

        Assert.Equal("张三", firstCall.Name);
        Assert.Equal("李四", secondCall.Name);
        Assert.Null(thirdCall);
    }
}

```

这个教程涵盖了 Moq 的主要使用场景，让我为你解释一下关键部分：

1. **基础设置**：
   - 使用 `new Mock<T>()` 创建模拟对象
   - 使用 `.Object` 获取模拟实例
   - 使用 `Setup()` 配置行为

2. **常用功能**：
   - 设置返回值（Returns）
   - 验证方法调用（Verify）
   - 参数匹配（It.Is, It.IsAny）
   - 异常处理（Throws）
   - 属性设置
   - 回调函数（Callback）
   - 序列化返回值（SetupSequence）

3. **验证方法**：
   - Times.Once()：验证方法被调用一次
   - Times.Never()：验证方法从未被调用
   - Times.Exactly(n)：验证方法被调用确切的次数

你想了解这些功能中的哪一个具体细节吗？或者你有特定的测试场景需要帮助？

***
***
***
问：
```
// 3.2 使用 It.IsAny<T>() 匹配任意参数 mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>())) .Returns(10); // 3.3 根据条件返回不同的值 mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>())) .Returns(1);

int result1 = calculator.Add(5, 3); // 返回 10 int result2 = calculator.Add(1, 1); 你上面的方法里面我理解 calculator.Add(5, 3); 和calculator.Add(1, 1); 同时都满足下面两个条件，如何决定是哪一个的，是按照顺序决定的吗 mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>())) .Returns(10); // 3.3 根据条件返回不同的值 mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>())) .Returns(1);
```
是的，这是一个很好的问题！在 Moq 中，当多个 Setup 都匹配同一个方法调用时，Moq 会使用"最具体"的匹配规则，而不是按照代码编写的顺序。让我通过例子详细解释这个机制：

```csharp
public interface ICalculator
{
    int Add(int a, int b);
}

public class MatchingRulesTests
{
    [Fact]
    public void DemonstrateMatchingRules()
    {
        var mockCalculator = new Mock<ICalculator>();

        // 1. 最general的规则（最不具体）
        mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()))
                     .Returns(10);

        // 2. 更具体的规则
        mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>()))
                     .Returns(1);

        // 3. 最具体的规则
        mockCalculator.Setup(calc => calc.Add(5, It.IsAny<int>()))
                     .Returns(100);

        var calculator = mockCalculator.Object;

        // 测试不同情况
        int result1 = calculator.Add(5, 3);    // 返回 100，因为完全匹配了第3个规则
        int result2 = calculator.Add(1, 1);    // 返回 1，因为匹配了第2个规则
        int result3 = calculator.Add(-1, 1);   // 返回 10，因为只匹配了第1个规则

        Assert.Equal(100, result1);  // 5完全匹配
        Assert.Equal(1, result2);    // 1 > 0 匹配
        Assert.Equal(10, result3);   // 只匹配通用规则
    }

    [Fact]
    public void MoreExamplesOfPriority()
    {
        var mockCalculator = new Mock<ICalculator>();

        // 按照具体程度从低到高的顺序排列
        mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()))
                     .Returns(0);  // 最低优先级

        mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>()))
                     .Returns(1);  // 中等优先级

        mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 10), It.IsAny<int>()))
                     .Returns(2);  // 更高优先级

        mockCalculator.Setup(calc => calc.Add(15, It.IsAny<int>()))
                     .Returns(3);  // 最高优先级

        var calculator = mockCalculator.Object;

        // 测试不同的输入值
        Assert.Equal(0, calculator.Add(-5, 1));   // 只匹配 IsAny
        Assert.Equal(1, calculator.Add(5, 1));    // 匹配 x > 0
        Assert.Equal(2, calculator.Add(11, 1));   // 匹配 x > 10
        Assert.Equal(3, calculator.Add(15, 1));   // 完全匹配 15
    }

    [Fact]
    public void ComplexMatchingRules()
    {
        var mockCalculator = new Mock<ICalculator>();

        // 从不具体到具体的规则
        mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()))
                     .Returns(100);  // 最基础规则

        mockCalculator.Setup(calc => calc.Add(
            It.Is<int>(x => x > 0), 
            It.Is<int>(y => y > 0)))
                     .Returns(200);  // 两个参数都有条件

        mockCalculator.Setup(calc => calc.Add(
            It.Is<int>(x => x > 10 && x < 20), 
            It.Is<int>(y => y > 0)))
                     .Returns(300);  // 更具体的条件

        mockCalculator.Setup(calc => calc.Add(15, 5))
                     .Returns(400);  // 最具体的匹配

        var calculator = mockCalculator.Object;

        // 测试各种情况
        Assert.Equal(100, calculator.Add(-1, -1));  // 只匹配最基础规则
        Assert.Equal(200, calculator.Add(5, 5));    // 匹配第二个规则
        Assert.Equal(300, calculator.Add(15, 1));   // 匹配第三个规则
        Assert.Equal(400, calculator.Add(15, 5));   // 匹配最具体的规则
    }
}

```

Moq 的匹配规则优先级如下（从高到低）：

1. **精确值匹配**
   ```csharp
   mockCalculator.Setup(calc => calc.Add(5, 3))  // 最高优先级
   ```

2. **具体条件匹配**
   ```csharp
   mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 10), It.IsAny<int>()))  // 次高优先级
   ```

3. **一般条件匹配**
   ```csharp
   mockCalculator.Setup(calc => calc.Add(It.Is<int>(x => x > 0), It.IsAny<int>()))  // 中等优先级
   ```

4. **任意值匹配**
   ```csharp
   mockCalculator.Setup(calc => calc.Add(It.IsAny<int>(), It.IsAny<int>()))  // 最低优先级
   ```

所以在你的例子中：
```csharp
calculator.Add(5, 3);  // 返回 1，因为匹配了 x > 0 的规则，这个规则比 IsAny 更具体
calculator.Add(1, 1);  // 返回 1，同样原因
```

重要注意点：
1. Moq 总是选择最具体的匹配规则
2. 规则的定义顺序并不重要，重要的是规则的具体程度
3. 当有多个同样具体的规则时，最后定义的规则会生效
4. 如果没有匹配的规则，Moq 会返回该类型的默认值

这个机制让我们可以定义非常精确的模拟行为，从通用情况到特殊情况都可以覆盖到。你需要我举更多的例子来说明某些特定场景吗？
***
***
***
问：能继续帮我深入讲解一下属性设置相关的用法吗？包含一些复杂的属性设置情况

好的，我来详细讲解 Moq 中关于属性设置的各种用法，从基础到复杂的情况。

```csharp
// 首先定义一些接口和类用于演示
public interface IUserProfile
{
    string Name { get; set; }
    int Age { get; set; }
    bool IsActive { get; }
    UserSettings Settings { get; set; }
    List<string> Roles { get; set; }
    IDictionary<string, string> Preferences { get; set; }
}

public class UserSettings
{
    public string Language { get; set; }
    public string Theme { get; set; }
    public bool NotificationsEnabled { get; set; }
}

public class PropertySetupTests
{
    // 1. 基础属性设置
    [Fact]
    public void BasicPropertySetup()
    {
        var mockProfile = new Mock<IUserProfile>();

        // 1.1 设置只读属性
        mockProfile.Setup(x => x.IsActive).Returns(true);

        // 1.2 设置可读写属性的固定值
        mockProfile.Setup(x => x.Name).Returns("张三");
        
        // 1.3 设置可追踪的属性（允许读写）
        mockProfile.SetupProperty(x => x.Age);
        mockProfile.SetupProperty(x => x.Name, "默认名字"); // 带默认值

        // 1.4 设置所有属性为可追踪
        mockProfile.SetupAllProperties();

        var profile = mockProfile.Object;

        // 测试
        Assert.True(profile.IsActive);
        Assert.Equal("张三", profile.Name);

        // 可以改变可追踪属性的值
        profile.Age = 25;
        Assert.Equal(25, profile.Age);
    }

    // 2. 属性值基于条件返回
    [Fact]
    public void ConditionalPropertyValues()
    {
        var mockProfile = new Mock<IUserProfile>();
        var isInitialized = false;

        // 2.1 基于条件返回不同的属性值
        mockProfile.Setup(x => x.Name)
                  .Returns(() => isInitialized ? "已初始化" : "未初始化");

        // 2.2 属性值依赖于其他属性
        mockProfile.Setup(x => x.Age).Returns(25);
        mockProfile.Setup(x => x.IsActive)
                  .Returns(() => mockProfile.Object.Age >= 18);

        var profile = mockProfile.Object;

        // 测试条件属性
        Assert.Equal("未初始化", profile.Name);
        isInitialized = true;
        Assert.Equal("已初始化", profile.Name);

        // 测试依赖属性
        Assert.True(profile.IsActive); // 因为 Age = 25 >= 18
    }

    // 3. 复杂对象属性设置
    [Fact]
    public void ComplexPropertySetup()
    {
        var mockProfile = new Mock<IUserProfile>();

        // 3.1 设置复杂对象属性
        mockProfile.Setup(x => x.Settings)
                  .Returns(new UserSettings 
                  { 
                      Language = "zh-CN",
                      Theme = "dark",
                      NotificationsEnabled = true
                  });

        // 3.2 设置集合属性
        mockProfile.Setup(x => x.Roles)
                  .Returns(new List<string> { "user", "admin" });

        // 3.3 设置字典属性
        mockProfile.Setup(x => x.Preferences)
                  .Returns(new Dictionary<string, string>
                  {
                      { "timezone", "UTC+8" },
                      { "currency", "CNY" }
                  });

        var profile = mockProfile.Object;

        // 测试复杂属性
        Assert.Equal("zh-CN", profile.Settings.Language);
        Assert.Contains("admin", profile.Roles);
        Assert.Equal("CNY", profile.Preferences["currency"]);
    }

    // 4. 属性变化监控和验证
    [Fact]
    public void PropertyChangeVerification()
    {
        var mockProfile = new Mock<IUserProfile>();
        mockProfile.SetupAllProperties();

        // 4.1 设置属性并验证
        var profile = mockProfile.Object;
        profile.Name = "张三";

        // 验证属性是否被设置
        mockProfile.VerifySet(x => x.Name = "张三", Times.Once());

        // 验证属性是否被获取
        mockProfile.VerifyGet(x => x.Name, Times.Once());

        // 4.2 验证属性从未被访问
        mockProfile.VerifyGet(x => x.Age, Times.Never());
    }

    // 5. 高级属性设置场景
    [Fact]
    public void AdvancedPropertyScenarios()
    {
        var mockProfile = new Mock<IUserProfile>();

        // 5.1 属性访问触发回调
        var propertyAccessCount = 0;
        mockProfile.Setup(x => x.Name)
                  .Returns("张三")
                  .Callback(() => propertyAccessCount++);

        // 5.2 属性设置时的验证
        mockProfile.SetupSet<string>(x => x.Name = It.Is<string>(s => s.Length > 2))
                  .Callback<string>(value => Console.WriteLine($"Name set to: {value}"));

        // 5.3 属性抛出异常
        mockProfile.Setup(x => x.Age)
                  .Throws<InvalidOperationException>();

        var profile = mockProfile.Object;

        // 测试回调
        var name = profile.Name;
        Assert.Equal(1, propertyAccessCount);

        // 测试属性设置验证
        profile.Name = "李四";  // 会触发回调

        // 测试异常
        Assert.Throws<InvalidOperationException>(() => profile.Age);
    }

    // 6. 动态属性值
    [Fact]
    public void DynamicPropertyValues()
    {
        var mockProfile = new Mock<IUserProfile>();
        var random = new Random();

        // 6.1 每次访问返回不同的值
        mockProfile.Setup(x => x.Age)
                  .Returns(() => random.Next(18, 60));

        // 6.2 基于访问次数返回不同值
        var accessCount = 0;
        mockProfile.Setup(x => x.Name)
                  .Returns(() => $"用户{++accessCount}");

        var profile = mockProfile.Object;

        // 测试动态值
        var age1 = profile.Age;
        var age2 = profile.Age;
        Assert.NotEqual(age1, age2); // 很可能不相等

        Assert.Equal("用户1", profile.Name);
        Assert.Equal("用户2", profile.Name);
    }
}

```

让我详细解释这些属性设置的不同方式：

1. **基础属性设置**
```csharp
// 固定返回值
mockProfile.Setup(x => x.Name).Returns("张三");

// 可追踪属性（可读写）
mockProfile.SetupProperty(x => x.Age);

// 所有属性可追踪
mockProfile.SetupAllProperties();
```

2. **条件属性值**
```csharp
// 基于条件返回不同值
mockProfile.Setup(x => x.Name)
          .Returns(() => isInitialized ? "已初始化" : "未初始化");

// 属性值依赖其他属性
mockProfile.Setup(x => x.IsActive)
          .Returns(() => mockProfile.Object.Age >= 18);
```

3. **复杂对象属性**
```csharp
// 设置复杂对象
mockProfile.Setup(x => x.Settings)
          .Returns(new UserSettings { ... });

// 设置集合
mockProfile.Setup(x => x.Roles)
          .Returns(new List<string> { ... });
```

4. **属性验证**
```csharp
// 验证属性设置
mockProfile.VerifySet(x => x.Name = "张三", Times.Once());

// 验证属性获取
mockProfile.VerifyGet(x => x.Name, Times.Once());
```

5. **高级场景**
```csharp
// 属性访问回调
mockProfile.Setup(x => x.Name)
          .Returns("张三")
          .Callback(() => propertyAccessCount++);

// 属性设置验证
mockProfile.SetupSet<string>(x => x.Name = It.Is<string>(s => s.Length > 2));

// 属性抛出异常
mockProfile.Setup(x => x.Age)
          .Throws<InvalidOperationException>();
```

6. **动态属性值**
```csharp
// 随机值
mockProfile.Setup(x => x.Age)
          .Returns(() => random.Next(18, 60));

// 基于访问次数
mockProfile.Setup(x => x.Name)
          .Returns(() => $"用户{++accessCount}");
```

一些重要的注意点：
1. `Setup` 用于设置只读属性或固定值
2. `SetupProperty` 用于可读写属性
3. `SetupAllProperties` 使所有属性可读写
4. 可以使用 `Returns` 返回固定值或动态值
5. 可以使用 `Callback` 在属性被访问时执行额外操作
6. 可以使用 `VerifyGet` 和 `VerifySet` 验证属性的访问情况

这些属性设置方法可以根据需要组合使用，创建出非常灵活的模拟对象。你想了解其中某个特定场景的更多细节吗？