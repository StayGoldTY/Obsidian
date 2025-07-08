# Moq 完整使用指南

## 1. 什么是Moq？

Moq 是.NET平台上最流行的Mock框架，它允许我们创建"假"对象来替代真实的依赖项，从而实现单元测试的隔离。

### 为什么需要Mock？

- 隔离测试 - 只测试当前类的逻辑，不依赖外部系统

- 控制依赖 - 可以精确控制依赖项的行为

- 提高速度 - 避免真实的数据库、网络调用等

- 测试边界情况 - 可以模拟异常、特殊返回值等

## 2. 安装和基础设置

### 安装Moq


```

# 在您的测试项目中安装

Install-Package Moq

Install-Package Microsoft.NET.Test.Sdk

Install-Package xunit

Install-Package xunit.runner.visualstudio
```

### 基础项目结构


```
HAINAN.Server.Tests/

├── Services/

│   └── ProjectServiceTests.cs

├── Controllers/

│   └── ProjectControllerTests.cs

└── HAINAN.Server.Tests.csproj
```

## 3. Moq基础用法

### 3.1 创建Mock对象

```


// 最基本的Mock创建

var mockRepository = new Mock<IBaseRepository>();

// 获取Mock对象的实例

IBaseRepository repository = mockRepository.Object;
``````
### 3.2 Setup方法 - 设置Mock行为

```
public class ProjectServiceTests

{

    [Test]

    public void GetBuildingInfo_WithValidId_ShouldReturnBuildingInfo()

    {

        // Arrange

        var mockRepo = new Mock<IBaseRepository>();

        var expectedBuilding = new BuildingInfoModel 

        { 

            MainId = "test-id",

            BuildingName = "测试建筑",

            BuildingType = "住宅"

        };

        // Setup - 设置Mock的行为

        mockRepo.Setup(r => r.FindEntity<BuildingInfoModel>(It.IsAny<Expression<Func<BuildingInfoModel, bool>>>()))
.ReturnsAsync(expectedBuilding);

        var service = new ProjectService(mockRepo.Object);

        // Act

        var result = await service.GetBuildingInfo("test-id");

        // Assert

        Assert.NotNull(result);

        Assert.Equal("test-id", result.MainId);

        Assert.Equal("测试建筑", result.BuildingName);

    }

}
```

## 4. 参数匹配 - It类的使用

### 4.1 It.IsAny< T>() - 匹配任何值

```


// 匹配任何字符串参数

mockRepo.Setup(r => r.FindEntity<User>(It.IsAny<Expression<Func<User, bool>>>()))

       .ReturnsAsync(new User());

// 匹配任何整数参数

mockService.Setup(s => s.GetById(It.IsAny<int>()))

          .Returns(new User());
```



### 4.2 It.Is< T>() - 条件匹配

```
// 只匹配特定条件的参数

mockRepo.Setup(r => r.FindEntity< User>(It.Is< string>(id => id.Length > 5)))

       .ReturnsAsync(new User());

// 匹配特定值

mockService.Setup(s => s.GetById(It.Is< int>(id => id > 0)))

          .Returns(new User());

```
### 4.3 具体值匹配

```


// 只匹配特定的具体值

mockRepo.Setup(r => r.FindEntity<User>("specific-id"))

       .ReturnsAsync(new User { Id = "specific-id" });
```

## 5. 返回值设置

### 5.1 Returns - 返回固定值



```
mockService.Setup(s => s.GetUserName())

          .Returns("张三");

mockService.Setup(s => s.GetUser(It.IsAny<string>()))

          .Returns(new User { Name = "张三" });
```

### 5.2 ReturnsAsync - 返回异步值



```
mockRepo.Setup(r => r.FindEntityAsync<User>(It.IsAny<string>()))

       .ReturnsAsync(new User());

// 也可以这样写

mockRepo.Setup(r => r.FindEntityAsync<User>(It.IsAny<string>()))

       .Returns(Task.FromResult(new User()));
```

### 5.3 Returns with Callback - 基于参数返回不同值


```
mockRepo.Setup(r => r.FindEntity<User>(It.IsAny<string>()))

       .Returns<string>(id => new User { Id = id });

// 更复杂的逻辑

mockService.Setup(s => s.CalculateScore(It.IsAny<int>()))

          .Returns<int>(score => score > 60 ? "及格" : "不及格");
```


## 6. 验证Mock调用 - Verify

### 6.1 验证方法被调用



```
[Test]

public void DeleteProject_ShouldCallRepositoryDelete()

{

    // Arrange

    var mockRepo = new Mock<IBaseRepository>();

    var service = new ProjectService(mockRepo.Object);

    // Act

    service.DeleteProject("test-id");

    // Assert - 验证方法被调用了

    mockRepo.Verify(r => r.Delete<Project>(It.IsAny<string>()), Times.Once);

}
```

### 6.2 验证调用次数



```
// 验证被调用了一次

mockRepo.Verify(r => r.Save(), Times.Once);

// 验证从未被调用

mockRepo.Verify(r => r.Delete(It.IsAny<string>()), Times.Never);

// 验证被调用了至少一次

mockRepo.Verify(r => r.Update(It.IsAny<User>()), Times.AtLeastOnce);

// 验证被调用了最多两次

mockRepo.Verify(r => r.Find(It.IsAny<string>()), Times.AtMost(2));

// 验证被调用了确切的次数

mockRepo.Verify(r => r.Log(It.IsAny<string>()), Times.Exactly(3));
```

### 6.3 验证具体参数



```
[Test]

public void UpdateUser_ShouldCallWithCorrectParameters()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    var service = new UserService(mockRepo.Object);

    var user = new User { Id = "123", Name = "张三" };

    // Act

    service.UpdateUser(user);

    // Assert - 验证使用正确的参数调用

    mockRepo.Verify(r => r.Update(It.Is<User>(u => u.Id == "123" && u.Name == "张三")), 

                   Times.Once);

}
```

## 7. 异常处理测试

### 7.1 Setup抛出异常

```

[Test]

public void GetUser_WhenRepositoryThrows_ShouldHandleException()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    mockRepo.Setup(r => r.FindById(It.IsAny<string>()))

           .Throws<DatabaseException>();

    var service = new UserService(mockRepo.Object);

    // Act & Assert

    Assert.Throws<DatabaseException>(() => service.GetUser("test-id"));

}

[Test]

public async Task GetUserAsync_WhenRepositoryThrows_ShouldHandleException()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    mockRepo.Setup(r => r.FindByIdAsync(It.IsAny<string>()))

           .ThrowsAsync(new DatabaseException("数据库连接失败"));

    var service = new UserService(mockRepo.Object);

    // Act & Assert

    await Assert.ThrowsAsync<DatabaseException>(() => service.GetUserAsync("test-id"));

}
```
## 8. 属性Mock

### 8.1 Setup属性



```
# 在您的测试项目中安装

Install-Package Moq

Install-Package Microsoft.NET.Test.Sdk

Install-Package xunit

Install-Package xunit.runner.visualstudio
```

### 8.2 自动属性

```
HAINAN.Server.Tests/

├── Services/

│   └── ProjectServiceTests.cs

├── Controllers/

│   └── ProjectControllerTests.cs

└── HAINAN.Server.Tests.csproj
```

## 9. 回调函数 - Callback

### 9.1 基础回调



```
// 最基本的Mock创建

var mockRepository = new Mock<IBaseRepository>();

// 获取Mock对象的实例

IBaseRepository repository = mockRepository.Object;
```

### 9.2 复杂回调

```

public class ProjectServiceTests

{

    [Test]

    public void GetBuildingInfo_WithValidId_ShouldReturnBuildingInfo()

    {

        // Arrange

        var mockRepo = new Mock<IBaseRepository>();

        var expectedBuilding = new BuildingInfoModel 

        { 

            MainId = "test-id",

            BuildingName = "测试建筑",

            BuildingType = "住宅"

        };

        // Setup - 设置Mock的行为

        mockRepo.Setup(r => r.FindEntity<BuildingInfoModel>(It.IsAny<Expression<Func<BuildingInfoModel, bool>>>()))

               .ReturnsAsync(expectedBuilding);

        var service = new ProjectService(mockRepo.Object);

        // Act

        var result = await service.GetBuildingInfo("test-id");

        // Assert

        Assert.NotNull(result);

        Assert.Equal("test-id", result.MainId);

        Assert.Equal("测试建筑", result.BuildingName);

    }

}
```
## 10. 序列返回 - SetupSequence



```
// 匹配任何字符串参数

mockRepo.Setup(r => r.FindEntity<User>(It.IsAny<Expression<Func<User, bool>>>()))

       .ReturnsAsync(new User());

// 匹配任何整数参数

mockService.Setup(s => s.GetById(It.IsAny<int>()))

          .Returns(new User());
```

## 11. 实际项目示例 - 基于您的HAINAN项目

### 11.1 测试ProjectService的完整示例
```

// 只匹配特定条件的参数

mockRepo.Setup(r => r.FindEntity<User>(It.Is<string>(id => id.Length > 5)))

       .ReturnsAsync(new User());

// 匹配特定值

mockService.Setup(s => s.GetById(It.Is<int>(id => id > 0)))

          .Returns(new User());
```

### 11.2 测试更复杂的业务逻辑


```
// 只匹配特定的具体值

mockRepo.Setup(r => r.FindEntity<User>("specific-id"))

       .ReturnsAsync(new User { Id = "specific-id" });
```

## 12. Mock行为模式

### 12.1 Strict vs Loose Mock



```
mockService.Setup(s => s.GetUserName())

          .Returns("张三");

mockService.Setup(s => s.GetUser(It.IsAny<string>()))

          .Returns(new User { Name = "张三" });
```
### 12.2 DefaultValue设置


```
mockRepo.Setup(r => r.FindEntityAsync<User>(It.IsAny<string>()))

       .ReturnsAsync(new User());

// 也可以这样写

mockRepo.Setup(r => r.FindEntityAsync<User>(It.IsAny<string>()))

       .Returns(Task.FromResult(new User()));
```

## 13. 高级特性

### 13.1 Protected方法Mock


```
mockRepo.Setup(r => r.FindEntity<User>(It.IsAny<string>()))

       .Returns<string>(id => new User { Id = id });

// 更复杂的逻辑

mockService.Setup(s => s.CalculateScore(It.IsAny<int>()))

          .Returns<int>(score => score > 60 ? "及格" : "不及格");
```

### 13.2 事件Mock
```
[Test]

public void DeleteProject_ShouldCallRepositoryDelete()

{

    // Arrange

    var mockRepo = new Mock<IBaseRepository>();

    var service = new ProjectService(mockRepo.Object);

    // Act

    service.DeleteProject("test-id");

    // Assert - 验证方法被调用了

    mockRepo.Verify(r => r.Delete<Project>(It.IsAny<string>()), Times.Once);

}
```

## 14. 最佳实践

### 14.1 测试命名约定


```
// 验证被调用了一次

mockRepo.Verify(r => r.Save(), Times.Once);

// 验证从未被调用

mockRepo.Verify(r => r.Delete(It.IsAny<string>()), Times.Never);

// 验证被调用了至少一次

mockRepo.Verify(r => r.Update(It.IsAny<User>()), Times.AtLeastOnce);

// 验证被调用了最多两次

mockRepo.Verify(r => r.Find(It.IsAny<string>()), Times.AtMost(2));
// 验证被调用了确切的次数

mockRepo.Verify(r => r.Log(It.IsAny<string>()), Times.Exactly(3));
```



### 14.2 AAA模式（Arrange-Act-Assert）

```

[Test]

public void UpdateUser_ShouldCallWithCorrectParameters()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    var service = new UserService(mockRepo.Object);

    var user = new User { Id = "123", Name = "张三" };

    // Act

    service.UpdateUser(user);

    // Assert - 验证使用正确的参数调用

    mockRepo.Verify(r => r.Update(It.Is<User>(u => u.Id == "123" && u.Name == "张三")), 

                   Times.Once);

}
```

### 14.3 避免过度Mock

```

[Test]

public void GetUser_WhenRepositoryThrows_ShouldHandleException()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    mockRepo.Setup(r => r.FindById(It.IsAny<string>()))

           .Throws<DatabaseException>();

    var service = new UserService(mockRepo.Object);

    // Act & Assert

    Assert.Throws<DatabaseException>(() => service.GetUser("test-id"));

}

[Test]

public async Task GetUserAsync_WhenRepositoryThrows_ShouldHandleException()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    mockRepo.Setup(r => r.FindByIdAsync(It.IsAny<string>()))

           .ThrowsAsync(new DatabaseException("数据库连接失败"));

    var service = new UserService(mockRepo.Object);

    // Act & Assert

    await Assert.ThrowsAsync<DatabaseException>(() => service.GetUserAsync("test-id"));

}
```
### 14.4 使用Builder模式创建测试数据



```
var mockUser = new Mock<IUser>();

// Setup属性getter

mockUser.Setup(u => u.Name).Returns("张三");

mockUser.Setup(u => u.Age).Returns(25);

// Setup属性setter

mockUser.SetupSet(u => u.Name = It.IsAny<string>());

// 验证属性被访问

mockUser.VerifyGet(u => u.Name, Times.Once);

mockUser.VerifySet(u => u.Name = "张三", Times.Once);
```

## 15. 常见错误和解决方案

### 15.1 Mock没有被正确设置

```
var mockUser = new Mock<IUser>();

// 设置自动属性行为

mockUser.SetupAllProperties();

// 现在可以直接设置和获取属性值

mockUser.Object.Name = "张三";

Assert.Equal("张三", mockUser.Object.Name);
```

### 15.2 异步方法Mock



```
[Test]

public void SaveUser_ShouldLogOperation()

{

    // Arrange

    var mockRepo = new Mock<IUserRepository>();

    var mockLogger = new Mock<ILogger>();

    var savedUsers = new List<User>();

    // Setup回调函数

    mockRepo.Setup(r => r.Save(It.IsAny<User>()))

           .Callback<User>(user => savedUsers.Add(user));

    var service = new UserService(mockRepo.Object, mockLogger.Object);

    // Act

    service.SaveUser(new User { Name = "张三" });

    // Assert

    Assert.Single(savedUsers);

    Assert.Equal("张三", savedUsers[0].Name);

}
```

### 15.3 Expression参数匹配



```
mockRepo.Setup(r => r.Update(It.IsAny< User>()))

       .Callback< User>(user => 

       {

           // 执行复杂的回调逻辑

           Console.WriteLine($"更新用户: {user.Name}");

           user.LastModified = DateTime.Now;

       })

       .Returns(true);
```

## 总结

Moq是一个功能强大且易用的Mock框架，通过合理使用Moq，您可以：

1. 编写更好的单元测试 - 隔离依赖，专注测试业务逻辑

2. 提高测试覆盖率 - 轻松测试各种边界情况和异常场景

3. 加快测试执行速度 - 避免真实的外部依赖

4. 增强代码质量 - 通过测试驱动开发改善代码设计

记住单元测试的核心原则：

- 快速 - 测试应该能快速执行

- 独立 - 测试之间不应相互依赖

- 可重复 - 每次运行都应得到相同结果

- 自验证 - 测试结果应该是明确的成功或失败

- 及时 - 测试应该与代码一起编写

希望这个详细的Moq教程能帮助您更好地编写单元测试！