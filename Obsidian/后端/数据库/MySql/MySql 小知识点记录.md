## ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)类似这种报错using password: YES 和 using password: NO是什么区别
using password: YES 和 using password: NO 表示在尝试登录MySQL时是否使用了密码:

1. using password: YES 表示在登录时提供了密码,但提供的密码是错误的,导致了"Access denied"的错误[3]。这说明MySQL服务器启用了密码验证,但客户端提供的密码无法通过验证。

2. using password: NO 表示在登录时没有提供密码,直接尝试登录[1][3]。如果MySQL服务器要求密码验证,没有提供密码自然也会导致"Access denied"的错误。

综上,两者的区别在于一个是使用了错误密码,另一个是根本没提供密码,但效果都是无法通过MySQL的密码验证,被拒绝访问。

正确的做法是:
1. 如果MySQL要求密码验证,就必须提供正确的密码
2. 如果希望无密码访问,可以在MySQL的配置中关闭密码验证(不推荐)
3. 使用 FLUSH PRIVILEGES 语句确保修改的密码或权限设置生效[1]

