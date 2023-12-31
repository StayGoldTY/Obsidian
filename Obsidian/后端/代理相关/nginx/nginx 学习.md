昨天配置ngInx来访问rancher学习到了很多知识
最开始我对于nginx的认识只是简单的做一个转发或者负载均衡，gzip之类的了解
我直接访问的地址是 [https://sys.hizj.net:10008/，然后通过nginx代理的地址是https://10.10.115.115:8443]
我操作了很多次rancher界面可以代理出来，但是登录的时候访问的还是https://10.10.115.115:8443/xx/xx相关的url，结果导致每次都不能登录
继续了解资料后发现nginx用法远超乎我的想象
对于上面问题的解决最主要的是配置：
```
proxy_set_header Host $host:10008; 
proxy_set_header X-Real-IP $remote_addr; 
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
proxy_set_header X-Forwarded-Proto $scheme; 
proxy_pass http://rancher-server:port/;
```
**其中的关键点配置了 Proxy Header,而Rancher 能自己识别出原始访问地址的关键就在于:
Rancher 内部对这些 Proxy Header 做了特殊处理!

把自己内部的会把所有自己的请求都转换成 Proxy Header 相关的请求
之后在显示登录界面,使用资源链接等地方,Rancher 会优先使用这些「原始请求值」,而不是代理请求自己带过来的信息。

所以可以总结为:

1. Rancher 支持解析代理头获取原始请求详情
2. Rancher 会优先使用原始请求的值,而不是代理请求值

就是这样的一套内部处理机制,让 Rancher 可以还原真实的用户访问信息,而不会被代理干扰。

***

如果一个反向代理的后端服务内部没有处理 Proxy Header,那么即使代理服务器发送了这些 Header,后端服务也无法获取和识别原始请求信息。

举个例子,我们没修改的 Nginx 反向代理一个普通应用:

```
客户端 -> Nginx代理 -> 普通应用服务
```

1. 客户端发起请求到 Nginx
2. Nginx 加入 Proxy Header ,转发请求到应用服务
3. 普通应用服务 收到请求,但无法解析 Proxy Header
4. 所以应用服务认为请求来自代理服务器,无法还原真实客户端源

结果就是应用服务获取到的用户IP和访问地址,还是代理 Nginx 自己的信息。

所以,必须应用服务内部代码支持解析和使用 Proxy Header,这样才能获取原始请求细节, Ranch er正是做了这层逻辑处理。 普通应用服务如果没有这部分逻辑,就无法正确地处理被代理的请求。

获取原始请求信息,依赖于应用服务内部对 Proxy Header 的支持。

***

### 复录 常见反向代理的 Proxy Header 
 ```
 常见反向代理的 Proxy Header 主要有:

**1. X-Forwarded-For**

存储发送请求的客户端的原始 IP 地址。目的是获取真实请求源IP。

**2. X-Forwarded-Proto**

原始的请求协议,HTTP 还是 HTTPS。

**3. X-Forwarded-Port** 

原始请求的客户端端口号。

**4. X-Forwarded-Host**

原始的请求 Host,可以获取客户端请求访问的原始域名地址。  

**5. X-Forwarded-Prefix**

请求URI中原始的 URL 前缀部分。

**6. X-Real-IP**

功能与 X-Forwarded-For 类似,都是用来表示原始客户端 IP。

此外 Proxy Header 中还可以包含上游代理加上一些请求或响应的定制字段。反应代理链中不同节点的信息。

需要注意的是,没有统一的规范。各代理可自行决定设置哪些 Header,包含什么信息,格式不定。服务端需要灵活兼容解析。
```