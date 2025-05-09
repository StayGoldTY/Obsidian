直接重启，重启后大概率会报错，类似下面的错误
hn-prod-2024.11.15 no_shard_available_action_exception
是因为副本切片没有激活，但是如果是单机的话，其实是没有副本切片的，直接运行下面命令,一键批量设置所有索引副本数为 0,问题解决
```
PUT /_settings
{
  "index": {
    "number_of_replicas": 0
  }
}

```