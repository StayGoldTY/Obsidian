首先在ES里面设置属性
确保在elasticsearch.yml配置文件中启用了安全功能:
`xpack.security.enabled: true`


在kibana.yml配置文件中,添加以下配置,使用至少32个字符的加密key:

`xpack.encryptedSavedObjects.encryptionKey: '你的32个字符以上的加密key'`


### 步骤 1：设置 `elastic` 用户信息

如果你还没有重置 `elastic` 用户的密码，请先重置密码：

#### 进入 Elasticsearch 容器

sh

复制代码

`docker exec -it elasticsearch-container /bin/bash`

#### 运行重置密码命令

sh

复制代码

`bin/elasticsearch-reset-password -u elastic`

按照提示输入 `y` 并记录生成的新密码。

### 重置 `kibana_system` 用户的密码

#### 进入 Elasticsearch 容器

sh

复制代码

`docker exec -it elasticsearch-container /bin/bash`

#### 运行重置密码命令

sh

复制代码

`bin/elasticsearch-reset-password -u kibana_system`

#### 编辑 `kibana.yml` 文件

使用 `vi` 或 `nano` 编辑器打开 `kibana.yml` 文件：

sh

复制代码

`vi /usr/share/kibana/config/kibana.yml`

#### 更新 `kibana_system` 用户密码

找到以下配置项并更新为新密码：

yaml

复制代码

`elasticsearch.username: "kibana_system" elasticsearch.password: "new_password"`

或者直接在kibana容器环境变量里面设置

ELASTICSEARCH_USERNAME

ELASTICSEARCH_PASSWORD
![[Pasted image 20240614105521.png]]

***
设置后发现用kibana_system`登录会提示没有权限

用`elastic`用户会登录提示
```
Error: [config validation of [elasticsearch].username]: value of "elastic" is forbidden. This is a superuser account that cannot write to system indices that Kibana needs to function. Use a service account token instead. Learn more: https://www.elastic.co/guide/en/elasticsearch/reference/8.0/service-accounts.html
```
因为在 Elasticsearch 8.0 及以上版本中，默认情况下 `elastic` 超级用户不能用于 Kibana，因为该用户不能写入 Kibana 所需的系统索引。你需要使用服务账户令牌（Service Account Token）来进行身份验证。
***


### 步骤 2：设置 服务账户令牌（Service Account Token）来进行身份验证

在 Elasticsearch 8.0 及以上版本中，默认情况下 `elastic` 超级用户不能用于 Kibana，因为该用户不能写入 Kibana 所需的系统索引。你需要使用服务账户令牌（Service Account Token）来进行身份验证。

以下是如何生成和配置服务账户令牌的步骤：

### 生成服务账户令牌

1. **进入 Elasticsearch 容器**：
   ```sh
   docker exec -it elasticsearch-container /bin/bash
   ```

2. **生成服务账户令牌**：
   ```sh
  bin/elasticsearch-service-tokens create elastic/kibana my-token
   ```

这个命令会生成一个服务账户令牌。请记录下生成的令牌，你需要在 Kibana 配置中使用它。

### 配置 Kibana 使用服务账户令牌

1. **进入 Kibana 容器**：
   ```sh
   docker exec -it kibana-container /bin/bash
   ```

2. **编辑 `kibana.yml` 文件**：
   使用 `vi` 或 `nano` 编辑器打开 `kibana.yml` 文件：
   ```sh
   vi /usr/share/kibana/config/kibana.yml
   ```

3. **更新配置**：
   在 `kibana.yml` 文件中添加以下配置，使用你生成的服务账户令牌：
   ```yaml
   elasticsearch.serviceAccountToken: "AAEAAWVsYXN0aWMva2liYW5hL215LXRva2VuOnp2SjVVT0pwVEFtNkxSS0xKSWx2a0E"
   ```

   替换 `"YOUR_GENERATED_TOKEN"` 为你生成的服务账户令牌。

4. **保存并退出编辑器**。

### 重启 Kibana

1. **退出容器**：
   ```sh
   exit
   ```

2. **重启 Kibana 容器**：
   ```sh
   docker restart kibana-container
   ```

### 验证

完成以上步骤后，Kibana 应该能够正常启动并连接到 Elasticsearch。

如果还有其他问题或需要进一步帮助，请随时告知。