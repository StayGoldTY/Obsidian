立即调用函数表达式（IIFE）是 JavaScript 中的一种设计模式，用于创建一个立即执行的匿名函数，并且在该函数内形成一个私有作用域，避免污染全局命名空间。IIFE 通常用于模块化代码、插件开发和临时变量的封装。以下是 IIFE 的详细用法和示例。

### 基本语法

IIFE 的基本语法如下：

```javascript
(function() {
  // 这里是 IIFE 的函数体
  console.log("IIFE executed!");
})();
```

### 详细解释

1. **函数表达式**：
   ```javascript
   (function() { ... })
   ```
   这部分定义了一个匿名函数，并将其括在一对括号中，目的是将其转换为函数表达式。普通函数定义后面不会立即执行，而函数表达式则会在后面跟随一对括号，表示立即调用。

2. **立即调用**：
   ```javascript
   (function() { ... })();
   ```
   这部分的第二对括号 `()` 用于立即调用前面的匿名函数。

### 示例和用法

#### 1. 创建私有作用域

IIFE 最常见的用途是创建一个私有作用域，避免变量污染全局命名空间。

```javascript
(function() {
  var privateVariable = "I am private";
  console.log(privateVariable); // 输出: "I am private"
})();

console.log(typeof privateVariable); // 输出: "undefined"
```

#### 2. 模块化代码

IIFE 可以用于模块化代码，将相关的函数和变量封装在一起，形成一个模块。

```javascript
var myModule = (function() {
  var privateVariable = "I am private";

  function privateMethod() {
    console.log(privateVariable);
  }

  return {
    publicMethod: function() {
      privateMethod();
    }
  };
})();

myModule.publicMethod(); // 输出: "I am private"
```

#### 3. 传递参数

IIFE 可以接受参数，使其更加灵活。

```javascript
(function(global, $) {
  // 使用 global 和 $ 参数
  console.log(global); // 输出: window 对象
  console.log($); // 假设 jQuery 已加载，输出: jQuery 对象
})(window, jQuery);
```

#### 4. 使用 IIFE 创建单例对象

单例模式确保一个类只有一个实例，并提供一个访问它的全局访问点。

```javascript
var singleton = (function() {
  var instance;

  function createInstance() {
    var object = new Object("I am the instance");
    return object;
  }

  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

var instance1 = singleton.getInstance();
var instance2 = singleton.getInstance();

console.log(instance1 === instance2); // 输出: true
```

### 优点

1. **避免全局命名空间污染**：IIFE 创建了一个私有作用域，避免了变量和函数污染全局命名空间。
2. **数据封装**：可以将变量和函数封装在 IIFE 内部，使其无法从外部访问，提供数据隐私性。
3. **立即执行**：IIFE 在定义后立即执行，不需要显式调用。

### 实际应用

#### 在 ES5 时代的模块化

在 ES5 时代，没有模块系统，开发者通常使用 IIFE 来实现模块化代码。

```javascript
var myModule = (function() {
  var moduleName = "My Module";

  function sayHello() {
    console.log("Hello from " + moduleName);
  }

  return {
    sayHello: sayHello
  };
})();

myModule.sayHello(); // 输出: "Hello from My Module"
```

#### 在现代 JavaScript 中的替代方案

在现代 JavaScript 中，使用 ES6 模块系统可以替代 IIFE 实现模块化代码：

```javascript
// module.js
export function sayHello() {
  console.log("Hello from the module");
}

// main.js
import { sayHello } from './module.js';
sayHello(); // 输出: "Hello from the module"
```

尽管 ES6 模块系统已经普及，IIFE 仍然在一些特定场景中有其应用价值，尤其是在需要立即执行代码块或需要封装临时变量的情况下。

***
```
(function(global, $) { // 使用 global 和 $ 参数 
console.log(global); // 输出: window 对象 
console.log($); // 假设 jQuery 已加载，输出: jQuery 对象 
})(window, jQuery); 如果后面接（）是表示立即执行，这段代码最后的(window, jQuery)是什么作用的
```
在这段代码中，最后的 `(window, jQuery)` 是用于传递参数给 IIFE（立即调用函数表达式）的。让我们详细解释一下这个过程：

### 代码回顾

```javascript
(function(global, $) {
  // 使用 global 和 $ 参数
  console.log(global); // 输出: window 对象
  console.log($); // 假设 jQuery 已加载，输出: jQuery 对象
})(window, jQuery);
```

### 解释

1. **函数定义和调用**：
   - `(function(global, $) { ... })(window, jQuery);`
   - 这里定义了一个匿名函数 `(function(global, $) { ... })`，并立即调用它。

2. **传递参数**：
   - `window` 和 `jQuery` 被作为参数传递给这个立即调用的函数。

### 具体步骤

1. **定义匿名函数**：
   ```javascript
   (function(global, $) { 
     // 使用 global 和 $ 参数
     console.log(global); // 输出: window 对象
     console.log($); // 假设 jQuery 已加载，输出: jQuery 对象
   })
   ```
   这是一个匿名函数，接受两个参数 `global` 和 `$`。

2. **立即调用并传递参数**：
   ```javascript
   (window, jQuery);
   ```
   立即调用这个匿名函数，并传递两个实参 `window` 和 `jQuery`。

### 参数传递

- `window` 作为第一个参数传递给匿名函数中的 `global`。
- `jQuery` 作为第二个参数传递给匿名函数中的 `$`。

### 作用

通过这种方式，匿名函数内部的 `global` 变量指向全局的 `window` 对象，`$` 变量指向全局的 `jQuery` 对象。

### 实际意义

1. **创建局部变量**：
   这种写法将全局对象 `window` 和 `jQuery` 作为局部变量 `global` 和 `$` 传递给匿名函数，从而可以在匿名函数内部使用这两个全局对象，而不会污染全局命名空间。

2. **增强代码的可移植性和可读性**：
   通过使用 `global` 和 `$` 作为参数名称，可以提高代码的可读性，并且在不同的上下文中（如浏览器和 Node.js）更容易重用这段代码。

### 举例

假设你在一个浏览器环境中使用 jQuery，可以这样写：

```javascript
(function(global, $) {
  console.log(global); // 输出: window 对象
  console.log($); // 输出: jQuery 对象
  
  // 现在可以使用 $ 作为 jQuery 的别名
  $('body').css('background-color', 'red');
})(window, jQuery);
```

通过这种方式，你可以确保代码在执行时正确地引用了 `window` 和 `jQuery`，并且避免了对全局作用域的污染。