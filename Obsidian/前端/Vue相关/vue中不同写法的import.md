```
##import {kgUtils} from './kgUtils'; 
import kgUtils from './kgUtils';
vue中上面两种写法的区别是什么
```


在 Vue.js 或任何其他 JavaScript 环境中，以下两种导入写法是有区别的：

1. `import {kgUtils} from './kgUtils';`
2. `import kgUtils from './kgUtils';`

这两种写法分别用于不同的导出方式。具体解释如下：

1. `import {kgUtils} from './kgUtils';`

   这种写法是用于导入命名导出（named export）。即 `kgUtils` 模块文件中有一个明确的导出对象或函数 `kgUtils`。

   ```js
   // kgUtils.js
   export const kgUtils = {
     // 一些工具函数
   };
   ```

   或者：

   ```js
   // kgUtils.js
   export function kgUtils() {
     // 一些工具函数
   }
   ```

   在这种情况下，你可以通过 `{ kgUtils }` 的方式导入 `kgUtils`。

2. `import kgUtils from './kgUtils';`

   这种写法是用于导入默认导出（default export）。即 `kgUtils` 模块文件中有一个默认导出对象、函数或类。

   ```js
   // kgUtils.js
   const kgUtils = {
     // 一些工具函数
   };

   export default kgUtils;
   ```

   或者：

   ```js
   // kgUtils.js
   export default function kgUtils() {
     // 一些工具函数
   }
   ```

   在这种情况下，你可以直接导入默认导出的对象、函数或类，并且可以使用任意名称来引用它。

### 总结

- 使用 `import {kgUtils} from './kgUtils';` 是针对命名导出，在导入时需要使用大括号并且名称要与导出的名称一致。
- 使用 `import kgUtils from './kgUtils';` 是针对默认导出，在导入时不需要使用大括号，名称可以随意。

选择哪种导入方式取决于你在模块中如何导出内容。

***
## 问：默认导出只能有一个是吧
是的，默认导出（default export）在一个模块中只能有一个。而命名导出（named export）可以有多个。

### 默认导出

在一个模块中只能有一个默认导出，使用 `export default` 进行导出：

```js
// kgUtils.js
const kgUtils = {
  // 一些工具函数
};

export default kgUtils;
```

你可以使用任意名称导入这个默认导出：

```js
import utils from './kgUtils';  // 这里的名称可以随意
console.log(utils);
```

### 命名导出

一个模块可以有多个命名导出，使用 `export` 进行导出：

```js
// kgUtils.js
export const kgUtils = {
  // 一些工具函数
};

export const anotherUtil = {
  // 其他工具函数
};
```

导入命名导出时需要使用大括号并且名称要与导出的名称一致：

```js
import { kgUtils, anotherUtil } from './kgUtils';
console.log(kgUtils);
console.log(anotherUtil);
```

你也可以混合使用默认导出和命名导出：

```js
// kgUtils.js
const kgUtils = {
  // 一些工具函数
};

const anotherUtil = {
  // 其他工具函数
};

export default kgUtils;
export { anotherUtil };
```

导入时：

```js
import utils, { anotherUtil } from './kgUtils';
console.log(utils);
console.log(anotherUtil);
```

总结：

- 每个模块只能有一个默认导出。
- 每个模块可以有多个命名导出。
- 导入默认导出时不需要大括号，可以使用任意名称。
- 导入命名导出时需要大括号，并且名称必须与导出的名称一致。