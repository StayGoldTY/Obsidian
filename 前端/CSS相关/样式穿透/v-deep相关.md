在 Vue 中,v-deep 是用来攻破样式作用域的。


默认情况下,Vue 的样式作用域会限制 CSS 只对当前组件生效。这可以避免不同组件的样式相互影响。

但是有时我们需要写一个全局样式,对所有的子组件都生效。这时可以使用 /deep/ 或者 v-deep 来破坏作用域。

例如:

```css
/* 这条规则只会应用到当前组件 */ 
.title {
  color: red;
}

/* 这条规则会应用到当前组件和子组件 */
::v-deep .title { 
  color: red;
}
```

v-deep 的工作原理是:

1. 它会把选择器转换成 /deep/ .title 的形式

2. /deep/ 是一种specially scoped style,可以破坏作用域规则

3. 这样CSS规则就可以应用到子组件了



需要注意的是:


- v-deep 和 /deep/ 在 Vue 3 中已被废弃,不推荐使用

- 应该通过全局样式和 CSS 继承来避免层叠样式

- 如果必须使用,可以换成 deep() selector 的函数形式
尽可能通过别的方式解决样式作用域问题

所以简单来说,v-deep 的作用是破坏 Vue 的样式作用域,使其影响子组件。但它已被废弃,我们应该寻找别的方式来解决全局样式的问题。

## <font color=#A35568>对v-deep只会影响当前组件和它的子组件,不会影响全局所有的组件。</font>

v-deep破坏的是当前组件的样式作用域,使样式向下渗透到子组件。但它不会破坏全局作用域,不会影响到其他没有嵌套关系的组件。


## <font color=#A35568>几种避免层叠样式的实现方式,我举些例子详细说明下:</font>

1. 使用全局样式

在公共样式文件中定义通用类或规则:

```css
/* common.css */

.text-center {
  text-align: center; 
}

.text-red {
  color: red;
}
```

然后在需要的组件中导入这个全局样式文件即可使用这些通用样式类。

2. 使用 CSS 继承

利用 CSS 继承,可以通过定义父组件样式,自动传递给子组件:

```css
/* Parent.vue */
.parent {
  color: red;
}
```

```css
/* Child.vue */

/* 子组件会继承父组件的 color 样式 */
```

3. 使用 CSS 变量

在 :root 处定义全局 CSS 变量:

```css
/* common.css */
:root {
  --primary-color: red;
}
```

子组件引用该变量:

```css
/* Child.vue */ 
.title {
  color: var(--primary-color); 
}
```

4. 通过 Props 传递样式

父组件:

```js
export default {
  props: ['titleClass']  
}
```

子组件:

```html
<!-- Parent.vue -->
<Child :titleClass="{ color: 'red' }" />
```

```js
// Child.vue

<h2 :class="titleClass">Title</h2> 
```



## <font color=#A35568>deep() 函数用法:</font>
```
css

Copy code

:deep(.title) {   color: red; }`

js

Copy code

const stylesheet = {   '.title': {     color: 'red'   } } // 使用 :deep(stylesheet['.title']) {   color: red;  }`
```


通过将选择器传递给 :deep() 函数生成样式,来代替 ::v-deep。





