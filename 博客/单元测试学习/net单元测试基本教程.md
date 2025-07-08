# 单元测试完全入门指南

让我从最基础的概念开始，系统地为您介绍单元测试的所有基础知识。

## 1. 什么是单元测试？

### 1.1 基本概念

单元测试是一种软件测试方法，用来测试软件中的最小可测试单元（通常是一个方法或函数）是否按预期工作。

### 1.2 为什么需要单元测试？

csharp

Apply to ProjectServi...

// 假设我们有一个简单的计算器类

```
public class Calculator

{

    public int Add(int a, int b)

    {

        return a + b;

    }

    public int Divide(int a, int b)

    {

        if (b == 0)

            throw new DivideByZeroException("除数不能为零");

        return a / b;

    }

}
```

没有测试的问题：

- 😰 不知道代码是否正确工作

- 😰 修改代码后不知道是否破坏了原有功能

- 😰 发现问题时已经在生产环境

- 😰 调试困难，不知道问题出在哪里

有测试的好处：

- 😊 确保代码按预期工作

- 😊 重构时有安全网保护

- 😊 早期发现问题

- 😊 代码质量更高

## 2. 测试框架基础

在.NET中，常用的测试框架有：

- NUnit - 功能丰富，社区活跃

- xUnit - 现代化，微软推荐

- MSTest - 微软官方，VS集成好

我们以NUnit为例来学习（概念在其他框架中都是相似的）。

### 2.1 创建第一个测试项目

bash

Apply to ProjectServi...

Run

# 创建测试项目

dotnet new nunit -n MyProject.Tests

# 安装必要的包

dotnet add package NUnit

dotnet add package NUnit3TestAdapter

dotnet add package Microsoft.NET.Test.Sdk

## 3. 测试的基本结构和标记

### 3.1 [Test] 标记 - 标识一个测试方法

```
csharp

Apply to ProjectServi...

using NUnit.Framework;

public class CalculatorTests

{

    [Test]  // 👈 这个标记告诉测试框架：这是一个测试方法

    public void Add_TwoPositiveNumbers_ShouldReturnSum()

    {

        // 测试代码写在这里

    }

}
```

[Test]标记的作用：

- 告诉测试运行器这是一个需要执行的测试

- 没有这个标记的方法不会被当作测试运行

- 测试方法必须是public，无参数，返回void或Task

### 3.2 测试类的基本结构

```
csharp

Apply to ProjectServi...

using NUnit.Framework;

[TestFixture]  // 👈 可选，标识这是一个测试类

public class CalculatorTests

{

    private Calculator _calculator;  // 👈 被测试的对象

    [SetUp]  // 👈 每个测试方法运行前都会执行

    public void Setup()

    {

        _calculator = new Calculator();

        Console.WriteLine("准备测试...");

    }

    [TearDown]  // 👈 每个测试方法运行后都会执行

    public void TearDown()

    {

        Console.WriteLine("清理测试...");

        // 清理资源，如关闭文件、数据库连接等

    }

    [OneTimeSetUp]  // 👈 整个测试类开始前执行一次

    public void OneTimeSetup()

    {

        Console.WriteLine("测试类开始执行");

    }

    [OneTimeTearDown]  // 👈 整个测试类结束后执行一次

    public void OneTimeTearDown()

    {

        Console.WriteLine("测试类执行完毕");

    }

    [Test]

    public void Add_TwoNumbers_ShouldReturnSum()

    {

        // 测试代码

    }

}
```

### 3.3 执行顺序示例

如果我们有两个测试方法，执行顺序是：

```
text

Apply to ProjectServi...

1. OneTimeSetUp (执行一次)

2. SetUp (第一个测试前)

3. 第一个测试方法

4. TearDown (第一个测试后)

5. SetUp (第二个测试前)

6. 第二个测试方法

7. TearDown (第二个测试后)

8. OneTimeTearDown (执行一次)

```
## 4. AAA模式 - 测试的黄金法则

每个测试都应该遵循AAA模式：

### 4.1 Arrange（准备）- 准备测试数据

csharp

Apply to ProjectServi...

```
[Test]

public void Add_TwoNumbers_ShouldReturnSum()

{

    // Arrange - 准备阶段

    var calculator = new Calculator();  // 创建被测试对象

    int number1 = 5;                   // 准备输入数据

    int number2 = 3;

    int expectedResult = 8;            // 准备期望结果

    // Act 和 Assert 会在下面介绍

}

```
### 4.2 Act（执行）- 执行被测试的方法

csharp
```

Apply to ProjectServi...

[Test]

public void Add_TwoNumbers_ShouldReturnSum()

{

    // Arrange

    var calculator = new Calculator();

    int number1 = 5;

    int number2 = 3;

    int expectedResult = 8;

    // Act - 执行阶段

    int actualResult = calculator.Add(number1, number2);  // 👈 执行被测试的方法

    // Assert 在下面

}
```

### 4.3 Assert（断言）- 验证结果


```
[Test]

public void Add_TwoNumbers_ShouldReturnSum()

{

    // Arrange

    var calculator = new Calculator();

    int number1 = 5;

    int number2 = 3;

    int expectedResult = 8;

    // Act

    int actualResult = calculator.Add(number1, number2);

    // Assert - 验证阶段

    Assert.AreEqual(expectedResult, actualResult);  // 👈 验证结果是否符合预期

}
```

## 5. 断言（Assert）- 验证测试结果

断言是测试的核心，用来验证实际结果是否符合预期。

### 5.1 基本断言

```

[Test]

public void BasicAssertions_Examples()

{

    // 验证相等

    Assert.AreEqual(5, 2 + 3);              // 期望值, 实际值

    Assert.AreEqual("Hello", "He" + "llo");

    // 验证不相等

    Assert.AreNotEqual(5, 2 + 2);

    // 验证为真/假

    Assert.IsTrue(5 > 3);

    Assert.IsFalse(5 < 3);

    // 验证null

    string nullString = null;

    Assert.IsNull(nullString);

    Assert.IsNotNull("not null");

    // 验证类型

    object obj = "Hello";

    Assert.IsInstanceOf<string>(obj);

}
```
### 5.2 数值断言



```
[Test]

public void NumericAssertions_Examples()

{

    // 验证数值在某个范围内（浮点数比较常用）

    Assert.AreEqual(0.333, 1.0/3.0, 0.001);  // 允许0.001的误差

    // 验证大于/小于

    Assert.Greater(10, 5);

    Assert.GreaterOrEqual(10, 10);

    Assert.Less(5, 10);

    Assert.LessOrEqual(5, 5);

}
```

### 5.3 字符串断言


```
[Test]

public void StringAssertions_Examples()

{

    string text = "Hello World";

    // 包含

    Assert.That(text, Does.Contain("World"));

    // 开始/结束

    Assert.That(text, Does.StartWith("Hello"));

    Assert.That(text, Does.EndWith("World"));

    // 忽略大小写

    Assert.That(text, Does.Contain("WORLD").IgnoreCase);

    // 正则表达式

    Assert.That(text, Does.Match(@"Hello\s+World"));

}
```
### 5.4 集合断言

```

[Test]

public void CollectionAssertions_Examples()

{

    var numbers = new List<int> { 1, 2, 3, 4, 5 };

    // 包含元素

    Assert.That(numbers, Does.Contain(3));

    // 集合相等（顺序和元素都要相同）

    var expected = new List<int> { 1, 2, 3, 4, 5 };

    Assert.That(numbers, Is.EqualTo(expected));

    // 集合等价（元素相同，顺序可以不同）

    var unordered = new List<int> { 5, 3, 1, 4, 2 };

    Assert.That(numbers, Is.EquivalentTo(unordered));

    // 集合大小

    Assert.That(numbers, Has.Count.EqualTo(5));

    Assert.That(numbers, Is.Not.Empty);

    // 所有元素满足条件

    Assert.That(numbers, Is.All.GreaterThan(0));

}

```

## 6. 异常测试

### 6.1 验证抛出异常

```


[Test]

public void Divide_ByZero_ShouldThrowException()

{

    // Arrange

    var calculator = new Calculator();

    // Act & Assert - 验证会抛出特定异常

    Assert.Throws<DivideByZeroException>(() => calculator.Divide(10, 0));

}

[Test]

public void Divide_ByZero_ShouldThrowExceptionWithMessage()

{

    // Arrange

    var calculator = new Calculator();

    // Act & Assert - 验证异常消息

    var exception = Assert.Throws<DivideByZeroException>(() => calculator.Divide(10, 0));

    Assert.AreEqual("除数不能为零", exception.Message);

}
```

### 6.2 验证不抛出异常

```
[Test]

public void Divide_ValidNumbers_ShouldNotThrowException()

{

    // Arrange

    var calculator = new Calculator();

    // Act & Assert - 验证不会抛出异常

    Assert.DoesNotThrow(() => calculator.Divide(10, 2));

}
```

## 7. 测试数据和参数化测试

### 7.1 TestCase - 一个方法测试多组数据
```


[TestCase(2, 3, 5)]      // 第一组测试数据

[TestCase(10, 15, 25)]   // 第二组测试数据

[TestCase(-1, 1, 0)]     // 第三组测试数据

[TestCase(0, 0, 0)]      // 第四组测试数据

public void Add_DifferentNumbers_ShouldReturnCorrectSum(int a, int b, int expected)

{

    // Arrange

    var calculator = new Calculator();

    // Act

    int result = calculator.Add(a, b);

    // Assert

    Assert.AreEqual(expected, result);

}
```

执行结果：这个测试方法会执行4次，每次使用不同的参数。

### 7.2 TestCaseSource - 从外部数据源获取测试数据

```


public class CalculatorTests

{

    // 定义测试数据源

    private static object[] AddTestCases =

    {

        new object[] { 2, 3, 5 },

        new object[] { 10, 15, 25 },

        new object[] { -1, 1, 0 },

        new object[] { 0, 0, 0 }

    };

    [TestCaseSource(nameof(AddTestCases))]

    public void Add_WithTestCaseSource_ShouldReturnCorrectSum(int a, int b, int expected)

    {

        // Arrange

        var calculator = new Calculator();

        // Act

        int result = calculator.Add(a, b);

        // Assert

        Assert.AreEqual(expected, result);

    }

}
```

### 7.3 Values - 组合测试
```


[Test]

public void Add_CombinedValues_ShouldWork(

    [Values(1, 2, 3)] int a,

    [Values(10, 20)] int b)

{

    // 这会生成6个测试：

    // (1,10), (1,20), (2,10), (2,20), (3,10), (3,20)

    var calculator = new Calculator();

    int result = calculator.Add(a, b);

    Assert.Greater(result, 0);  // 简单验证结果大于0

}
```

## 8. 测试的命名约定

### 8.1 好的测试命名

```


// 格式：方法名_测试条件_期望结果

[Test]

public void Add_TwoPositiveNumbers_ShouldReturnSum() { }

[Test]

public void Add_PositiveAndNegativeNumber_ShouldReturnDifference() { }

[Test]

public void Divide_ByZero_ShouldThrowDivideByZeroException() { }

[Test]

public void GetUser_WithValidId_ShouldReturnUser() { }

[Test]

public void GetUser_WithInvalidId_ShouldReturnNull() { }
```

### 8.2 测试类的组织



```
// 按功能分组

public class CalculatorAddTests { }      // 加法相关测试

public class CalculatorDivideTests { }   // 除法相关测试

// 或者按类分组

public class CalculatorTests { }         // Calculator类的所有测试

public class UserServiceTests { }       // UserService类的所有测试
```

## 9. 完整的实际示例

让我们创建一个完整的例子来演示所有概念：

### 9.1 被测试的类

```


// 一个简单的用户服务类

public class UserService

{

    private readonly List<User> _users = new List<User>();

    public void AddUser(User user)

    {

        if (user == null)

            throw new ArgumentNullException(nameof(user));

        if (string.IsNullOrEmpty(user.Name))

            throw new ArgumentException("用户名不能为空", nameof(user));

        if (_users.Any(u => u.Email == user.Email))

            throw new InvalidOperationException("邮箱已存在");

        _users.Add(user);

    }

    public User GetUserByEmail(string email)

    {

        return _users.FirstOrDefault(u => u.Email == email);

    }

    public List<User> GetAllUsers()

    {

        return _users.ToList();

    }

    public int GetUserCount()

    {

        return _users.Count;

    }

}

public class User

{

    public string Name { get; set; }

    public string Email { get; set; }

    public int Age { get; set; }

}
```

### 9.2 完整的测试类
```

using NUnit.Framework;

using System;

using System.Collections.Generic;

[TestFixture]

public class UserServiceTests

{

    private UserService _userService;

    [SetUp]

    public void Setup()

    {

        _userService = new UserService();

    }

    #region AddUser测试

    [Test]

    public void AddUser_ValidUser_ShouldAddSuccessfully()

    {

        // Arrange

        var user = new User 

        { 

            Name = "张三", 

            Email = "zhangsan@test.com", 

            Age = 25 

        };

        // Act

        _userService.AddUser(user);

        // Assert

        Assert.AreEqual(1, _userService.GetUserCount());

        var addedUser = _userService.GetUserByEmail("zhangsan@test.com");

        Assert.IsNotNull(addedUser);

        Assert.AreEqual("张三", addedUser.Name);

    }

    [Test]

    public void AddUser_NullUser_ShouldThrowArgumentNullException()

    {

        // Act & Assert

        Assert.Throws<ArgumentNullException>(() => _userService.AddUser(null));

    }

    [TestCase("")]

    [TestCase("   ")]

    [TestCase(null)]

    public void AddUser_EmptyName_ShouldThrowArgumentException(string name)

    {

        // Arrange

        var user = new User { Name = name, Email = "test@test.com" };

        // Act & Assert

        var exception = Assert.Throws<ArgumentException>(() => _userService.AddUser(user));

        Assert.That(exception.Message, Does.Contain("用户名不能为空"));

    }

    [Test]

    public void AddUser_DuplicateEmail_ShouldThrowInvalidOperationException()

    {

        // Arrange

        var user1 = new User { Name = "张三", Email = "same@test.com" };

        var user2 = new User { Name = "李四", Email = "same@test.com" };

        _userService.AddUser(user1);

        // Act & Assert

        var exception = Assert.Throws<InvalidOperationException>(() => _userService.AddUser(user2));

        Assert.AreEqual("邮箱已存在", exception.Message);

    }

    #endregion

    #region GetUserByEmail测试

    [Test]

    public void GetUserByEmail_ExistingEmail_ShouldReturnUser()

    {

        // Arrange

        var user = new User { Name = "张三", Email = "zhangsan@test.com" };

        _userService.AddUser(user);

        // Act

        var result = _userService.GetUserByEmail("zhangsan@test.com");

        // Assert

        Assert.IsNotNull(result);

        Assert.AreEqual("张三", result.Name);

        Assert.AreEqual("zhangsan@test.com", result.Email);

    }

    [Test]

    public void GetUserByEmail_NonExistingEmail_ShouldReturnNull()

    {

        // Act

        var result = _userService.GetUserByEmail("notexist@test.com");

        // Assert

        Assert.IsNull(result);

    }

    #endregion

    #region GetAllUsers测试

    [Test]

    public void GetAllUsers_EmptyService_ShouldReturnEmptyList()

    {

        // Act

        var result = _userService.GetAllUsers();

        // Assert

        Assert.IsNotNull(result);

        Assert.IsEmpty(result);

    }

    [Test]

    public void GetAllUsers_WithMultipleUsers_ShouldReturnAllUsers()

    {

        // Arrange

        var user1 = new User { Name = "张三", Email = "zhang@test.com" };

        var user2 = new User { Name = "李四", Email = "li@test.com" };

        _userService.AddUser(user1);

        _userService.AddUser(user2);

        // Act

        var result = _userService.GetAllUsers();

        // Assert

        Assert.AreEqual(2, result.Count);

        Assert.That(result, Does.Contain(user1));

        Assert.That(result, Does.Contain(user2));

    }

    #endregion

    #region 集成测试示例

    [Test]

    public void UserService_CompleteWorkflow_ShouldWorkCorrectly()

    {

        // Arrange

        var users = new List<User>

        {

            new User { Name = "张三", Email = "zhang@test.com", Age = 25 },

            new User { Name = "李四", Email = "li@test.com", Age = 30 },

            new User { Name = "王五", Email = "wang@test.com", Age = 35 }

        };

        // Act - 添加用户

        foreach (var user in users)

        {

            _userService.AddUser(user);

        }

        // Assert - 验证添加结果

        Assert.AreEqual(3, _userService.GetUserCount());

        // Act - 查询用户

        var foundUser = _userService.GetUserByEmail("li@test.com");

        // Assert - 验证查询结果

        Assert.IsNotNull(foundUser);

        Assert.AreEqual("李四", foundUser.Name);

        Assert.AreEqual(30, foundUser.Age);

        // Act - 获取所有用户

        var allUsers = _userService.GetAllUsers();

        // Assert - 验证所有用户

        Assert.AreEqual(3, allUsers.Count);

        Assert.That(allUsers.Select(u => u.Name), 

                   Is.EquivalentTo(new[] { "张三", "李四", "王五" }));

    }

    #endregion

}
```

## 10. 运行测试

### 10.1 在Visual Studio中运行

1. 打开"测试资源管理器"（Test Explorer）

2. 点击"运行所有测试"

3. 查看测试结果

### 10.2 在命令行中运行

```

Run

# 运行所有测试

dotnet test

# 运行特定测试类

dotnet test --filter "ClassName=UserServiceTests"

# 运行特定测试方法

dotnet test --filter "MethodName=AddUser_ValidUser_ShouldAddSuccessfully"

# 生成测试报告

dotnet test --logger "trx;LogFileName=test-results.trx"
```

## 11. 测试的最佳实践

### 11.1 FIRST原则

- Fast（快速） - 测试应该快速执行

- Independent（独立） - 测试之间不应相互依赖

- Repeatable（可重复） - 每次运行都应得到相同结果

- Self-Validating（自验证） - 测试结果应该是明确的Pass或Fail

- Timely（及时） - 测试应该与代码一起编写

### 11.2 测试覆盖率


```
// 好的测试应该覆盖：

// 1. 正常路径（Happy Path）

[Test] public void Add_ValidNumbers_ShouldReturnSum() { }

// 2. 边界条件

[Test] public void Add_ZeroValues_ShouldReturnZero() { }

[Test] public void Add_MaxValues_ShouldNotOverflow() { }

// 3. 异常情况

[Test] public void Add_NullInput_ShouldThrowException() { }

// 4. 业务规则

[Test] public void AddUser_DuplicateEmail_ShouldThrowException() { }
```

## 总结

现在您已经掌握了单元测试的基础知识：

1. [Test]标记 - 标识测试方法

2. Setup/TearDown - 测试的生命周期管理

3. AAA模式 - Arrange, Act, Assert的测试结构

4. 断言 - 验证测试结果的各种方法

5. 参数化测试 - 用不同数据测试同一逻辑

6. 异常测试 - 验证异常情况的处理

7. 测试命名 - 清晰描述测试意图

这些基础概念是编写高质量单元测试的基石。掌握了这些，您就可以开始为自己的代码编写测试了！