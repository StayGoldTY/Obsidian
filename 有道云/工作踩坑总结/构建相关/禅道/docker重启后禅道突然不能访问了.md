排查过程如下
1.查看容器日志
08:32:30.22 WARN ==> Apache: Waiting MySQL 1 seconds 08:32:30.23 WARN ==> Sentry: Waiting Apache 1 seconds 08:32:32.22 WARN ==> Apache: Waiting MySQL 2 seconds 08:32:32.23 WARN ==> Sentry: Waiting Apache 2 seconds 08:32:36.23 WARN ==> Apache: Waiting MySQL 4 seconds 08:32:36.23 WARN ==> Sentry: Waiting Apache 4 seconds 08:32:44.23 WARN ==> Apache: Waiting MySQL 8 seconds 08:32:44.24 WARN ==> Sentry: Waiting Apache 8 seconds 08:33:00.24 WARN ==> Apache: Waiting MySQL 16 seconds 08:33:00.24 WARN ==> Sentry: Waiting Apache 16 seconds 08:33:32.24 WARN ==> Apache: Waiting MySQL 32 seconds 08:33:32.24 ERROR ==> Apache Maximum number of retries reached! 08:33:32.24 ERROR ==> Apache Unable to connect to MySQL: 127.0.0.1:3306 08:33:32.24 WARN ==> Sentry: Waiting Apache 32 seconds
**一眼知道是mysql启动的问题

2.继续排查为什么mysql没有启动，期间试过很多方法都不行
后来看官方文档[https://www.zentao.net/book/zentaopms/1059.html]
![[Pasted image 20240220102706.png]]
需要增加启动内置mysql的环境变量才行
具体配置如下：

|ZBX_DBTLSCONNECT|required|
|ZBX_DB_ENCRYPTION|true|
|MYSQL_ROOT_PASSWORD|123456|
|MYSQL_INTERNAL|true|

3.恢复后发现新问题，就是之前的数据都不见了，固化的数据没有丢失，但是访问禅道却不行
自己用过各种方式来还原都不行，后面看文档应该是新版本镜像和老版本镜像的文件结构不一样，还原的时候要按照对应的文件结构要求来还原。
后面直接用跟之前一样的下面版本镜像解决了问题
```
easysoft/zentao:17.8|
```