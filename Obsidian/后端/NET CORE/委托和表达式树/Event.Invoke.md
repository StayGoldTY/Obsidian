是的，在 C# 中，当你调用事件的 `Invoke` 方法时，实际上是在调用所有订阅该事件的方法。以下是一个详细的解释和示例。

### 事件调用机制

1. **事件定义**：定义事件时，实际上是定义了一个基于委托的事件，这个事件可以有多个订阅者。
2. **订阅事件**：订阅事件的对象将其方法添加到事件的调用列表中。
3. **触发事件**：调用事件的 `Invoke` 方法会依次调用所有订阅该事件的方法。

### 示例

以下是一个示例，展示如何定义、订阅和触发事件：

```csharp
using System;

public delegate void NotifyHandler(string message);

public class Notifier
{
    // 定义一个事件
    public event NotifyHandler OnNotify;

    // 方法，用于触发事件
    public void Notify(string message)
    {
        // 调用事件的 Invoke 方法，触发所有订阅者
        OnNotify?.Invoke(message);
    }
}

public class Subscriber
{
    private string _name;

    public Subscriber(string name)
    {
        _name = name;
    }

    // 订阅事件的方法
    public void Subscribe(Notifier notifier)
    {
        notifier.OnNotify += OnNotified;
    }

    // 事件处理方法
    private void OnNotified(string message)
    {
        Console.WriteLine($"{_name} received message: {message}");
    }
}

public class Program
{
    public static void Main()
    {
        Notifier notifier = new Notifier();
        
        Subscriber subscriber1 = new Subscriber("Subscriber 1");
        Subscriber subscriber2 = new Subscriber("Subscriber 2");

        // 订阅事件
        subscriber1.Subscribe(notifier);
        subscriber2.Subscribe(notifier);

        // 触发事件
        notifier.Notify("Hello, World!");
    }
}
```

### 代码解释

1. **定义事件**：
    ```csharp
    public event NotifyHandler OnNotify;
    ```
    在 `Notifier` 类中定义了一个事件 `OnNotify`，它使用 `NotifyHandler` 委托类型。

2. **触发事件**：
    ```csharp
    public void Notify(string message)
    {
        OnNotify?.Invoke(message);
    }
    ```
    `Notify` 方法使用 `OnNotify?.Invoke(message);` 触发事件，调用所有订阅者的方法。`OnNotify?` 用于检查是否有任何方法订阅了该事件，避免空引用异常。

3. **订阅事件**：
    ```csharp
    notifier.OnNotify += OnNotified;
    ```
    `Subscriber` 类中的 `Subscribe` 方法订阅了 `Notifier` 的 `OnNotify` 事件，并将 `OnNotified` 方法作为事件处理程序。

4. **事件处理方法**：
    ```csharp
    private void OnNotified(string message)
    {
        Console.WriteLine($"{_name} received message: {message}");
    }
    ```
    这是一个事件处理方法，当事件被触发时，这个方法将被调用。

### 输出

当运行上述程序时，输出将会是：
```
Subscriber 1 received message: Hello, World!
Subscriber 2 received message: Hello, World!
```

这表明事件 `OnNotify` 被触发时，所有订阅该事件的方法都被调用了。通过调用 `Invoke` 方法，可以确保所有订阅者都会收到通知并执行相应的处理逻辑。