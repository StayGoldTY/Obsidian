```
void handle(final Handler handler) {
  if (handler instanceof ReportHandler) {
    // 生成报告
    ((ReportHandler)handler).report();
    return;
  }
  
  if (handler instanceof NotificationHandler) {
    // 发送通知
    ((NotificationHandler)handler).sendNotification();
  }
  ...
}
```
 上面的例子为什么违反了里氏替换原则，能详细跟我讲解一下吗？
## 回答
上面的代码违反了 **里氏替换原则（Liskov Substitution Principle，LSP）**，因为它在使用 `instanceof` 来判断具体的子类类型，然后根据不同的子类类型执行不同的操作。这种做法的问题是，如果有新的子类加入，方法就需要修改，从而导致了违反了里氏替换原则。

### 里氏替换原则的定义

里氏替换原则是面向对象设计的五大原则之一，定义如下：

> 如果一个子类型可以替换掉它的父类型，并且程序的行为不变，那么这个子类型就遵循了里氏替换原则。

简单来说，里氏替换原则要求：子类可以替换父类并且程序逻辑不出错。因此，父类所定义的行为应当对所有子类适用，子类无需修改父类代码的实现就可以正常工作。

### 问题分析

在你提供的代码中，`handle` 方法通过 `instanceof` 来判断 `handler` 是 `ReportHandler` 还是 `NotificationHandler`，并根据不同的类型进行不同的处理。这导致了一个问题：

1. **强制要求子类知道自己的具体类型**：
    
    - 如果有新的 `Handler` 子类添加进来（比如 `EmailHandler`），`handle` 方法就需要修改。这显然违反了开闭原则（对扩展开放，对修改封闭）。
    - 由于 `instanceof` 的使用，这就强制要求每个子类必须根据类型判断来执行特定逻辑，违反了里氏替换原则的本意：父类引用能够替代子类实例，而不需要了解具体类型。
2. **违背了子类行为的多态性**：
    
    - 理论上，你希望 `handler` 被当作一个 `Handler` 来处理，而不需要关注它具体是哪种 `Handler` 的实现。
    - 但是在这个方法中，父类 `Handler` 被迫知道和处理它的具体子类（`ReportHandler`、`NotificationHandler` 等），这使得代码无法利用多态的特性。

### 解决方法

要遵守里氏替换原则，可以利用 **多态性** 和 **方法重写** 来处理不同的 `Handler` 类型，而不是使用 `instanceof`。

一个更符合里氏替换原则的做法是：

```java
abstract class Handler {
    abstract void handle();
}

class ReportHandler extends Handler {
    @Override
    void handle() {
        // 生成报告
        report();
    }

    void report() {
        // 生成报告的具体实现
    }
}

class NotificationHandler extends Handler {
    @Override
    void handle() {
        // 发送通知
        sendNotification();
    }

    void sendNotification() {
        // 发送通知的具体实现
    }
}

void handle(final Handler handler) {
    handler.handle();  // 通过多态直接调用子类的 handle 方法
}
```

### 为什么这样更好？

1. **多态性**：
    
    - 通过重写 `handle` 方法，每个具体的 `Handler` 子类负责自己的行为。这样你不需要通过 `instanceof` 来判断类型，而是通过多态来动态分发行为。
2. **扩展性**：
    
    - 如果以后增加新的 `Handler` 子类（比如 `EmailHandler`），只需要实现自己的 `handle` 方法，而 `handle` 方法本身不需要修改。这符合开闭原则。
3. **简洁性**：
    
    - 代码更简洁，避免了冗长的类型判断逻辑，也更容易理解和维护。

总结来说，原代码通过 `instanceof` 判断具体类型，导致了违反里氏替换原则。通过利用多态和方法重写，可以让代码更符合里氏替换原则，提高代码的扩展性和可维护性。

***
***
***
问：还是不太明白，里氏替换原则最核心的其实就是子类替换父类后不会引起任何异常和错误，上面代码为什么子类替换父类后会不符合上面的原则的

## 回答
你理解得很对，**里氏替换原则**的核心思想确实是：子类替换父类后，程序的行为应该正常，不会引起任何异常或者错误。

现在让我通过具体的例子详细解释一下上面代码违反里氏替换原则的原因。

### **原代码分析**

```java
void handle(final Handler handler) {
  if (handler instanceof ReportHandler) {
    // 生成报告
    ((ReportHandler)handler).report();
    return;
  }
  
  if (handler instanceof NotificationHandler) {
    // 发送通知
    ((NotificationHandler)handler).sendNotification();
  }
  ...
}
```

这段代码的关键问题是，`handle` 方法在运行时需要判断 `handler` 的具体类型（`ReportHandler` 或 `NotificationHandler`），然后根据不同的类型执行不同的操作。

假设你将一个新的 `Handler` 类型（例如 `EmailHandler`）添加到系统中，代码将需要修改，增加 `instanceof EmailHandler` 的判断语句。**问题就在这里**：每次你添加新的子类时，`handle` 方法的代码就必须修改。这就是为什么它违反了里氏替换原则。

### **里氏替换原则的核心要求**

里氏替换原则的核心要求是：当一个子类实例替代父类实例时，程序不应改变其原本的行为。换句话说，代码不应该关心对象的具体类型，而应该依赖于父类提供的通用接口或抽象方法来执行操作。

#### 如果遵循里氏替换原则：

- 如果有新的 `Handler` 类型（比如 `EmailHandler`），你应该能够直接将它作为 `Handler` 类型传入 `handle` 方法。
- `handle` 方法应该依赖于 `Handler` 类型本身提供的接口，而不需要通过 `instanceof` 来判断具体的子类类型。
- 子类的行为应该是由子类本身定义的，父类不需要关心子类的具体实现。

### **为什么这段代码违反了里氏替换原则**

假设我们定义了一个新的子类 `EmailHandler`，它也继承自 `Handler`：

```java
class EmailHandler extends Handler {
    void sendEmail() {
        // 发送邮件的具体实现
    }
}
```

然后我们传入 `EmailHandler` 实例：

```java
Handler handler = new EmailHandler();
handle(handler);
```

如果 `handle` 方法中的代码使用了 `instanceof` 来判断 `handler` 的具体类型，并且没有为 `EmailHandler` 提供特定的处理逻辑，那么就会导致以下问题：

- 由于 `handle` 方法无法识别 `EmailHandler` 类型，它无法执行特定的操作。
- 你必须修改 `handle` 方法，增加 `instanceof EmailHandler` 的判断，从而违反了里氏替换原则，不能保证添加新的子类后，原有的 `handle` 方法行为不变。

### **违反里氏替换原则的本质**

本质上，违反里氏替换原则的原因是 **父类方法依赖于子类的具体类型**。在面向对象设计中，我们希望能够通过父类的引用来调用子类的行为，而不需要知道具体是哪一个子类。

当你使用 `instanceof` 判断子类类型时，`handle` 方法并没有利用多态，而是依赖于明确的子类类型来执行不同的操作。这样，如果子类变动（比如增加新的子类），你就必须修改父类的方法，这显然违反了开闭原则和里氏替换原则。

### **正确的做法**

要遵循里氏替换原则，我们应该依赖多态性，让每个子类实现自己的行为，而不是通过 `instanceof` 来判断类型：

```java
abstract class Handler {
    abstract void handle();
}

class ReportHandler extends Handler {
    @Override
    void handle() {
        // 生成报告的逻辑
        report();
    }

    void report() {
        // 报告生成的具体实现
    }
}

class NotificationHandler extends Handler {
    @Override
    void handle() {
        // 发送通知的逻辑
        sendNotification();
    }

    void sendNotification() {
        // 发送通知的具体实现
    }
}

class EmailHandler extends Handler {
    @Override
    void handle() {
        // 发送邮件的逻辑
        sendEmail();
    }

    void sendEmail() {
        // 发送邮件的具体实现
    }
}

void handle(final Handler handler) {
    handler.handle();  // 使用多态，调用具体子类的 handle 方法
}
```

### **总结**

1. **里氏替换原则**要求子类可以替代父类，而程序的行为不应该发生任何变化。
2. 在原代码中，`handle` 方法依赖于 `instanceof` 来判断 `handler` 的具体类型，这导致了每次新增子类时都需要修改 `handle` 方法，从而违反了里氏替换原则。
3. 通过多态（即子类重写父类方法），我们可以避免在父类方法中做类型判断，让程序能够在不修改代码的情况下添加新的子类，符合里氏替换原则。

***
***
***
问：看了你上面说的说明，我理解的lsp是这样的，lsp即是要求我们子类不要做出超出父类行为的方法，要子类应该是父类的延申和扩展，而不应该修改父类的方法

## 回答
你理解的 **里氏替换原则（LSP）** 有一些核心的理解是正确的，但还需要一些补充和澄清。让我来详细解释一下 LSP 的本意。

### **里氏替换原则（LSP）的核心定义**

里氏替换原则要求：**子类对象应该能够替代父类对象，而程序的行为不会改变**。

这意味着：

1. **子类的行为应该是父类行为的拓展，而不是改变父类的行为**。
2. 当使用父类类型的引用时，子类应该能够替代父类，而不需要修改任何使用该父类的代码。

### **子类不应改变父类的预期行为**

里氏替换原则的关键是：**子类不仅应该能够扩展父类的功能，还应该确保不会违背父类的预期行为**。这就是说，子类不能做出超出父类预期的行为。

#### 举个例子：

假设我们有一个 `Bird` 类，它有一个方法 `fly()`，表示鸟类的飞行：

```java
class Bird {
    void fly() {
        System.out.println("Flying...");
    }
}
```

然后，我们有一个子类 `Penguin`，它也继承了 `Bird` 类，但是企鹅并不能飞：

```java
class Penguin extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}
```

在这个例子中，**`Penguin` 的 `fly` 方法违背了父类 `Bird` 的行为**，因为父类 `Bird` 期望所有的鸟类都能飞。如果我们用 `Bird` 类型的引用来操作 `Penguin` 对象，程序就会抛出异常，这显然不符合预期。

为了遵守里氏替换原则，我们应该避免子类改变父类的方法行为，或者违反父类的方法预期。如果一个子类不具备父类的某些行为，**应该通过不同的设计来避免让子类去做这些不符合预期的事情**。

### **LSP的关键要点**

1. **方法签名一致**：子类应该遵循父类方法的签名和行为，不能改变方法的输入输出或者副作用。也就是说，**子类重写的方法应当提供与父类方法相同的功能，或者提供更加精确的实现**。
    
2. **增强而非改变行为**：子类可以通过重写方法来增强父类的行为，但不能改变父类方法的原有语义。例如，`Bird` 类的 `fly()` 方法可以被 `Penguin` 类重写，但应该避免做出 `throw` 异常这种违反行为的设计。
    
3. **子类可以增加自己的特有行为**：子类可以在保留父类行为的基础上，增加更多的特性或方法，而不破坏父类的功能。
    

### **回到原代码的分析**

现在回到你的原始问题，为什么 `instanceof` 的判断会违反 LSP。

```java
void handle(final Handler handler) {
  if (handler instanceof ReportHandler) {
    // 生成报告
    ((ReportHandler)handler).report();
    return;
  }
  
  if (handler instanceof NotificationHandler) {
    // 发送通知
    ((NotificationHandler)handler).sendNotification();
  }
  ...
}
```

在这里，`handle` 方法通过 `instanceof` 判断具体的子类类型来决定不同的操作，这导致了违反里氏替换原则：

- **问题**：`handle` 方法要求知道并依赖具体的子类类型（`ReportHandler`、`NotificationHandler`），而这正是违反了多态的设计思想。如果我们新增一个 `EmailHandler`，那么 `handle` 方法就必须修改，增加对 `EmailHandler` 的判断，这使得 `handle` 方法不再能够透明地处理任何 `Handler` 的子类。
    
- **LSP的原则**：在 LSP 的原则下，**`handle` 方法应该依赖于父类的抽象接口，而不是具体的子类类型**。只要是 `Handler` 类型的对象，`handle` 方法就应该能够处理，不需要知道该对象是 `ReportHandler`、`NotificationHandler` 还是 `EmailHandler`。
    

### **符合LSP的代码设计**

为了遵循 LSP，我们应该让每个子类实现自己特定的行为，而不依赖于 `instanceof` 判断。可以通过多态和方法重写来实现：

```java
abstract class Handler {
    abstract void handle();
}

class ReportHandler extends Handler {
    @Override
    void handle() {
        // 生成报告
        report();
    }

    void report() {
        // 生成报告的具体实现
    }
}

class NotificationHandler extends Handler {
    @Override
    void handle() {
        // 发送通知
        sendNotification();
    }

    void sendNotification() {
        // 发送通知的具体实现
    }
}

class EmailHandler extends Handler {
    @Override
    void handle() {
        // 发送邮件
        sendEmail();
    }

    void sendEmail() {
        // 发送邮件的具体实现
    }
}

void handle(final Handler handler) {
    handler.handle();  // 使用多态，调用具体子类的 handle 方法
}
```

- **为什么符合LSP**：现在，`handle` 方法依赖于 `Handler` 类型的多态性，不需要修改代码来处理新的子类。每个子类都可以扩展自己的行为，而不会破坏父类的预期功能。`handle` 方法对于所有 `Handler` 类型的对象都是透明的，它只依赖于 `Handler` 提供的抽象行为。

### **总结**

- **里氏替换原则的核心**是：**子类可以替代父类，并且程序的行为不应改变**。这意味着子类不应该做出改变父类行为的行为，也不应该引入不符合父类预期的异常或错误。
- **子类应扩展父类行为**，而不是改变它。当子类提供新的功能时，它应该通过继承父类的接口或者方法来实现，确保父类代码能够透明地适配子类，而不需要通过类型判断（如 `instanceof`）来干预。

***
***
***
问：如你上面说的，那不是子类应该尽量不用override来改写父类方法了，还是说可以使用override但是要保证override的方法改写后行为一致

## 回答
你理解得非常接近！实际上，**子类使用 `override` 来重写父类方法是完全可以的**，但前提是要确保重写后的行为依然符合父类的预期和行为契约。因此，**重写父类方法时，必须保证子类的方法行为与父类的一致**，或者是更加精确和合理的扩展，而不是完全改变父类方法的预期。

### **可以使用 `override`，但要保证行为一致**

**里氏替换原则（LSP）**并不要求子类不能重写父类的方法，而是要求子类重写的方法 **不能违背父类方法的预期行为**。如果子类通过 `override` 重写父类的方法，应该确保：

1. **不会改变父类方法的原有行为语义**。也就是说，父类定义了什么行为，子类可以增加更多的功能，但不能改变原有的核心行为。
    
2. **对外暴露的接口保持一致**，即子类的方法签名、参数、返回值等都应该与父类方法一致，或者提供一个更加具体的实现，而不应做出与父类方法完全不一致的行为。
    

### **举个例子：父类和子类的 `override`**

假设有一个简单的例子，父类 `Bird` 有一个方法 `fly()`，表示鸟类的飞行，子类 `Penguin` 也继承了 `Bird` 类并重写了 `fly()` 方法。

#### 父类 `Bird` 和子类 `Penguin` 示例：

```java
class Bird {
    void fly() {
        System.out.println("Bird is flying...");
    }
}

class Penguin extends Bird {
    @Override
    void fly() {
        // Penguin can't fly, but we can override and handle it differently
        System.out.println("Penguin can't fly!");
    }
}
```

这里，子类 `Penguin` 重写了父类的 `fly()` 方法，**但它并没有完全改变父类方法的行为**，而是根据自己的特性做了适当的处理：打印出“Penguin can't fly”而不是“Bird is flying”。这种做法是**符合LSP的**，因为它没有违反父类的行为契约。

#### 关键点：

- 子类重写了父类的方法，**但并没有改变父类方法的原有预期**，只是根据实际情况进行了处理。
- 如果 `Penguin` 的 `fly()` 方法抛出了异常或做出不符合“鸟类飞行”逻辑的行为，那就违反了里氏替换原则。

### **不符合 LSP 的例子**

假设我们有一个父类 `Shape`，它有一个方法 `draw()` 来绘制形状，而子类 `Circle` 和 `Square` 分别继承 `Shape` 并重写了 `draw()` 方法。如果子类 `Circle` 重写了 `draw()` 方法，并且做了一些改变，违反了父类的行为契约，那么就会违反 LSP。

```java
class Shape {
    void draw() {
        System.out.println("Drawing a shape...");
    }
}

class Circle extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing a circle...");
    }
}

class Square extends Shape {
    @Override
    void draw() {
        throw new UnsupportedOperationException("Square can't be drawn!");
    }
}
```

在上面的例子中，**`Square` 类重写了 `draw()` 方法，抛出了异常**，这违反了父类 `Shape` 的行为契约，因为父类的 `draw()` 方法没有抛出异常。任何使用 `Shape` 类型的代码都会期望调用 `draw()` 方法时不会抛出异常，而 `Square` 类却抛出了 `UnsupportedOperationException`。这种行为显然不符合里氏替换原则，因为 **子类引入了父类没有的异常行为**。

### **遵循 LSP 的正确方法**

为确保符合 LSP，我们可以避免 `Square` 类抛出异常，而是用一种适当的方式来处理这种不支持的行为。例如：

```java
class Shape {
    void draw() {
        System.out.println("Drawing a shape...");
    }
}

class Circle extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing a circle...");
    }
}

class Square extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing a square...");
    }
}
```

在这个版本中，`Square` 类的 `draw()` 方法就不会抛出异常，而是正常地完成它的功能（即绘制一个正方形），这种方式是符合里氏替换原则的。

### **总结**

1. **子类可以重写父类方法**，但必须保证重写后的方法不会破坏父类原有的行为预期，也就是说，**子类方法的行为要符合父类定义的契约**。
    
2. **行为一致性**：如果父类方法的行为是打印一条消息，子类可以重写并增强这个行为，但不能改变该行为的核心逻辑（比如抛出异常或做出完全不符合父类方法预期的改变）。
    
3. **增强而非改变**：子类可以通过重写方法来增强父类的方法，例如提供更具体的实现，但不能使行为变得不一致或不可预测。
    
4. **多态和继承的正确使用**：当使用父类引用指向子类对象时，应该能够透明地调用子类方法，而不需要知道具体是哪一个子类。

***
***
***
问：那么这里的关键是行为契约的定义是什么，怎么样去判断是否违反了行为契约

## 回答
**行为契约**（Behavioral Contract）是面向对象编程中的一个重要概念，特别是在遵循 **里氏替换原则（LSP）** 时，理解行为契约至关重要。简单来说，**行为契约**定义了一个类对外提供的行为承诺，即该类及其子类在继承和重写方法时，必须确保某些行为不会发生不一致或不符合预期的改变。

### **行为契约的定义**

行为契约可以理解为：

- **方法的功能预期**：父类对外界提供的功能接口，它定义了方法应该如何工作，输入和输出是什么样的。
- **副作用**：调用方法时可能产生的影响（例如，修改内部状态、输出结果、抛出异常等）。
- **不变式**：类在运行时应该保持的约束或条件。例如，父类的 `draw()` 方法可能有一个不变式，即该方法调用后，状态不应发生不一致的改变。

**行为契约**实际上是一种对外承诺，类通过方法签名、方法行为、输入输出以及状态变化等方面，明确承诺该方法如何执行。当子类继承父类时，它应该遵守这个契约，而不是打破它。

### **判断是否违反了行为契约**

判断是否违反行为契约的标准包括以下几个方面：

#### 1. **方法签名的一致性**

子类重写的方法不能改变父类方法的签名，包括方法名、参数类型、返回类型等。如果改变了这些，就会破坏契约，因为父类期望以某种方式与子类交互。

**例如**，如果父类的方法是：

```java
class Shape {
    void draw() {
        // 默认的绘制形状行为
    }
}
```

那么子类不能把返回类型从 `void` 改成其他类型：

```java
class Square extends Shape {
    // 错误的重写：改变了返回类型
    int draw() {
        return 1;
    }
}
```

这样的重写就违反了契约。

#### 2. **输入参数的一致性**

子类重写父类方法时，**方法的输入参数必须与父类保持一致**，即使方法的功能可以稍微扩展，但不能改变对输入参数的要求。

**例如**，假设父类的方法是：

```java
class Shape {
    void setColor(String color) {
        // 设置颜色的默认实现
    }
}
```

子类不能改变输入参数的类型：

```java
class Square extends Shape {
    // 错误的重写：改变了输入参数类型
    void setColor(int colorCode) {
        // 处理颜色代码
    }
}
```

这种重写方式违反了行为契约，因为父类期望接收一个字符串作为颜色，而子类接收的是 `int` 类型的颜色代码。

#### 3. **行为的预期一致性**

子类不能改变父类方法的行为预期，包括**方法的执行逻辑、返回值**和**副作用**。

例如，父类的方法 `draw()` 可能是为了在屏幕上显示图形，而子类 `Circle` 重写时，应当保持这个功能。如果子类将其改为 `draw()` 后抛出异常或产生不一致的副作用，那么就违背了契约。

**例如**，父类的 `draw()` 方法：

```java
class Shape {
    void draw() {
        System.out.println("Drawing shape...");
    }
}
```

子类的错误重写：

```java
class Square extends Shape {
    @Override
    void draw() {
        throw new UnsupportedOperationException("Square can't be drawn!");
    }
}
```

这种情况就违反了行为契约，因为父类 `Shape` 的 `draw()` 方法没有抛出异常，而子类却抛出了异常，破坏了父类的行为预期。

#### 4. **异常的处理**

父类方法中定义了哪些可能的异常，子类在重写时不能去除这些异常，也不能引入新的未声明的异常。违反这一点就会破坏行为契约。

**例如**，父类方法：

```java
class Shape {
    void draw() throws IOException {
        // 绘制形状，可能会抛出IOException
    }
}
```

子类重写时不能去除 `IOException`：

```java
class Circle extends Shape {
    @Override
    void draw() {
        // 错误的重写：没有声明抛出IOException
    }
}
```

这种方式违反了契约，因为父类方法可能抛出 `IOException`，而子类的方法却没有声明。

#### 5. **状态变化的契约一致性**

子类应该遵循父类方法的行为约定，特别是在状态变化方面。比如，父类的方法可能要求在调用时保持某些状态的约束或前提条件，子类不能违反这些规则。

**例如**，假设父类有一个 `open()` 方法用于打开某个资源，并假设它会修改资源的状态：

```java
class Resource {
    boolean isOpen = false;
    
    void open() {
        if (!isOpen) {
            // 打开资源
            isOpen = true;
        }
    }
}
```

子类在重写时不能做出违反父类状态的改变：

```java
class SpecialResource extends Resource {
    @Override
    void open() {
        // 错误的重写：没有遵守父类对状态的约束
        isOpen = true;  // 不再检查是否已经打开
    }
}
```

在这个例子中，子类没有遵守父类对 `isOpen` 状态的检查，直接修改了状态，这违反了行为契约。

### **总结**

**行为契约**的核心是**父类为子类提供的预期行为**，子类应遵循父类的预期，而不是改变它。如果子类重写了父类的方法，但改变了方法的输入、输出、副作用或状态，或者抛出了新的异常，这样就违反了行为契约。

#### 如何判断是否违反行为契约：

1. 检查方法签名是否一致。
2. 确保子类没有改变父类方法的输入参数要求。
3. 确保方法的功能行为一致，且不引入不一致的副作用或异常。
4. 子类重写时是否遵循父类对状态的约定。
5. 子类是否正确声明了异常，或者不引入父类未声明的异常。

遵循这些规则，可以确保子类与父类之间的继承关系符合 **里氏替换原则（LSP）**，并且不违反行为契约。