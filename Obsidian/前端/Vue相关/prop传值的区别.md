在 Vue 中，`<TopLine :showHistory=true>` 和 `<TopLine showHistory=true>` 这两种写法有着重要的区别：

1. **使用 `:`（即 v-bind）**:
   
   ```html
   <TopLine :showHistory=true>
   ```

   当你在 Vue 模板中使用 `:`，实际上是使用了 `v-bind` 的缩写。这意味着 `showHistory` 属性的值会被当作 JavaScript 表达式来解析。在这个例子中，`true` 是一个 JavaScript 布尔值。因此，`showHistory` 属性会被赋予布尔值 `true`。

2. **不使用 `:`**:
   
   ```html
   <TopLine showHistory=true>
   ```

   在不使用 `:` 的情况下，属性值被当作纯字符串来处理。这意味着不论你写的是 `true` 还是 `false`，都会被当作字符串而非布尔值。所以在这个例子中，`showHistory` 将得到字符串 `"true"` 而非布尔值。

总的来说，如果你想传递 JavaScript 表达式（如变量、布尔值、对象等），应该使用 `:`（或 `v-bind`）。如果你只是传递静态或字面的字符串，可以不用它。在你的示例中，如果 `showHistory` 属性期望的是一个布尔值，那么你应该使用第一种写法 `<TopLine :showHistory=true>`。