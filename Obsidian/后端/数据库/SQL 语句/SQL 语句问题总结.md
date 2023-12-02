**LEFT 和 WHERE 一起使用
```
SELECT *
FROM TableA a
LEFT JOIN TableB b ON a.Key = b.Key
WHERE b.SomeColumn = 'SomeValue'

```

上面的sql语句中,并不能保证返回TableA中的所有数据
因为SQL 执行的顺序是先执行  left join 再执行 where
执行left join 后b表中的数据会和A表生成一个新的表
然后新的表会用where语句里面的条件来匹配，这样匹配的结果就有可能把TableA中的语句直接过滤掉
所以更好的做法是：
*如果你想保留主表中所有的行，无论子表中是否有匹配的行，应该将只影响子表的条件移至 `ON` 子句，而不是放在 `WHERE` 子句中。这样，`WHERE` 子句只用于过滤主表中的行，而 `ON` 子句用于定义如何连接两个表。
