你现在遇到的错误：

```
fatal: unable to access 'https://192.168.128.146:776/root/scene-bim-server.git/': SSL peer certificate or SSH remote key was not OK
```

说明这次**已经成功连接到了服务器的 776 端口**，但出现了**SSL 证书验证失败**的问题，通常是因为 GitLab 使用了**自签名证书（self-signed certificate）**，而 Git 默认不信任它。

---

## ✅ 解决方案一：临时关闭 SSL 验证（快速测试用）

你可以直接添加环境变量跳过证书校验（**不推荐长期使用**）：

```bash
GIT_SSL_NO_VERIFY=true git clone https://192.168.128.146:776/root/scene-bim-server.git
```

或者在 Windows PowerShell 中执行：

```powershell
$env:GIT_SSL_NO_VERIFY=1
git clone https://192.168.128.146:776/root/scene-bim-server.git
```

---

## ✅ 解决方案二：为 Git 配置永久跳过证书检查（**针对当前仓库**）

```bash
git config --global http."https://192.168.128.146:776/".sslVerify false
```

然后再执行：

```bash
git clone https://192.168.128.146:776/root/scene-bim-server.git
```

---

## ✅ 解决方案三（推荐）：使用 **SSH 克隆地址**

如果你有 GitLab 的 SSH Key 配置，可以切换为更安全可靠的 **SSH 克隆方式**，无需走 HTTPS，也就不存在证书问题：

```bash
git@192.168.128.146:root/scene-bim-server.git
```

> 前提是：你已经在 GitLab 用户的设置里添加了你的公钥（`~/.ssh/id_rsa.pub`），并且服务端开放了 SSH（默认端口 22）。

---

## ✅ 补充建议（最佳实践）

- **长期建议配置正规证书**（例如使用 Let's Encrypt 或内网颁发的 CA）；
    
- 或者将 GitLab 配置一个 Nginx 反向代理监听 `443` 并由它处理 HTTPS 与证书；
    
- 或者使用 SSH 协议克隆项目，省心又安全。
    

---

需要我帮你一步步配置 SSH Key 登录 GitLab 或生成证书的 Nginx 配置，也可以告诉我。