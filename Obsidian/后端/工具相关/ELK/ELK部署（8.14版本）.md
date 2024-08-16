### 1.基本上都是通过rancher来部署，配置对应正确镜像即可。

### 2.这个地方有一个大坑，就是如果通过rancher挂载各种方式都挂载不上，直接把对应的目录下面的所有文件都拷贝的需要挂载的目录下面，如何在进行目录挂载


### 3.坑2 报如下错误
```
{"@timestamp":"2024-06-06T08:40:14.261Z", "log.level": "WARN", "message":"master not discovered yet, this node has not previously joined a bootstrapped cluster, and this node must discover master-eligible nodes [elasticsearch-79bd544dbd-h9d6m] to bootstrap a cluster: have discovered [{elasticsearch-744ddcb74d-25q5p}{imo9rxglQNClh-VEdZT50A}{LIfbj2ULTWiiXGUIPNawQA}{elasticsearch-744ddcb74d-25q5p}{10.42.0.81}{10.42.0.81:9300}{cdfhilmrstw}{8.14.0}{7000099-8505000}]; discovery will continue using [127.0.0.1:9300, 127.0.0.1:9301, 127.0.0.1:9302, 127.0.0.1:9303, 127.0.0.1:9304, 127.0.0.1:9305] from hosts providers and [{elasticsearch-744ddcb74d-25q5p}{imo9rxglQNClh-VEdZT50A}{LIfbj2ULTWiiXGUIPNawQA}{elasticsearch-744ddcb74d-25q5p}{10.42.0.81}{10.42.0.81:9300}{cdfhilmrstw}{8.14.0}{7000099-8505000}] from last-known cluster state; node term 0, last-accepted version 0 in term 0; for troubleshooting guidance, see https://www.elastic.co/guide/en/elasticsearch/reference/8.14/discovery-troubleshooting.html", "ecs.version": "1.2.0","service.name":"ES_ECS","event.dataset":"elasticsearch.server","process.thread.name":"elasticsearch[elasticsearch-744ddcb74d-25q5p][cluster_coordination]
```

这条日志信息显示，Elasticsearch节点未发现主节点，并且该节点尚未加入引导的集群。要解决这个问题，你可以尝试以下步骤：

1. **检查配置文件**：确保所有节点的配置文件（如`elasticsearch.yml`）中正确配置了集群名称和主节点的相关信息。

2. **网络连接**：确保各节点之间的网络连接正常，尤其是确保配置的IP地址和端口是正确的，并且没有被防火墙阻挡。

3. **引导节点**：在集群初始化时，需要至少一个引导节点。如果这是新的集群，请确保有一个节点被配置为引导节点。可以在`elasticsearch.yml`中设置`discovery.seed_hosts`或`cluster.initial_master_nodes`。

4. **节点角色**：检查每个节点的角色配置，确保至少有一个节点被配置为主节点（master-eligible）。

5. **日志详细信息**：查看更多的日志详细信息，以便了解更具体的错误原因。

6. **参考官方文档**：查看[Elasticsearch的官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/8.14/discovery-troubleshooting.html)中的故障排除指南，获取更多的故障排除步骤。

日志中的主要信息如下：
- 当前节点未加入引导的集群，需要发现主节点。
- 已经发现了一些节点，但仍然需要进一步的主节点发现。
- 继续使用配置的IP和端口进行主节点发现。

通过上述步骤，你应该能够解决Elasticsearch节点无法发现主节点的问题。如果问题依然存在，可以提供更多的日志信息进行进一步分析。


我原先的yml文件如下
```
cluster.name: "docker-cluster" network.host: 0.0.0.0 http.host: 0.0.0.0 #----------------------- BEGIN SECURITY AUTO CONFIGURATION ----------------------- # # The following settings, TLS certificates, and keys have been automatically # generated to configure Elasticsearch security features on 06-06-2024 08:30:11 # # -------------------------------------------------------------------------------- # Enable security features xpack.security.enabled: false xpack.security.enrollment.enabled: true # Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents xpack.security.http.ssl: enabled: false keystore.path: certs/http.p12 # Enable encryption and mutual authentication between cluster nodes xpack.security.transport.ssl: enabled: false verification_mode: certificate keystore.path: certs/transport.p12 truststore.path: certs/transport.p12 # Create a new cluster with the current node only # Additional nodes can still join the cluster later cluster.initial_master_nodes: ["elasticsearch-79bd544dbd-h9d6m"] #----------------------- END SECURITY AUTO CONFIGURATION -------------------------
```

修改后如下：
```
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.host: 0.0.0.0

#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically      
# generated to configure Elasticsearch security features on 06-06-2024 08:30:11
#
# --------------------------------------------------------------------------------

# Enable security features
xpack.security.enabled: false
xpack.security.enrollment.enabled: false

# Disable SSL since security is disabled
xpack.security.http.ssl.enabled: false
xpack.security.transport.ssl.enabled: false

# Initial master nodes
cluster.initial_master_nodes: ["elasticsearch-79bd544dbd-h9d6m"]

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------

# Ensure this node can be a master node
node.master: true
node.data: true

```

***
还有如下配置
将xpack.security.enabled: 和  
xpack.security.[http](https://so.csdn.net/so/search?q=http&spm=1001.2101.3001.7020).ssl:enabled: 设置为false即可

但是[配置文件](https://so.csdn.net/so/search?q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&spm=1001.2101.3001.7020)没有上述选项，添加后，重新启动仍然访问不大

后面网上找到在配置文件添加 http.host: 0.0.0.0
***

坑3：ES虽然正确启动了，但是都提示没有主节点，正常的方法应该是去配置主节点相关的内容，简单的做法是直接先设置单节点即可

之前的报错信息类似下面这样,主要观察有类似字段master_not_discovered_exception
```
http://101.40.70.165:40757/_cat/nodes?v 返回如下： {"error":{"root_cause":[{"type":"master_not_discovered_exception","reason":null}],"type":"master_not_discovered_exception","reason":null},"status":503}
```
单节点设置类似下面这样主要是 discovery.type: single-node
```
cluster.name: "docker-cluster"
node.name: "node-1"
network.host: 0.0.0.0
http.host: 0.0.0.0

# 安全配置
xpack.security.enabled: false
xpack.security.enrollment.enabled: true
xpack.security.http.ssl.enabled: false
xpack.security.transport.ssl.enabled: false

# 节点角色配置
node.roles: [ master, data ]

# 单节点配置
discovery.type: single-node

```

***

## 坑4：Kibana 提示如下
```
[WARN ][plugins.usageCollection.usage-collection.collector-set] ResponseError: illegal_argument_exception Root causes: illegal_argument_exception: node [elasticsearch-6c8c7d4877-48852] does not have the remote cluster client role enabled at KibanaTransport.request (/usr/share/kibana/node_modules/@elastic/transport/lib/Transport.js:492:27) at processTicksAndRejections (node:internal/process/task_queues:95:5) at KibanaTransport.request (/usr/share/kibana/node_modules/@kbn/core-elasticsearch-client-server-internal/src/create_transport.js:51:16) at ClientTraced.SearchApi [as search] (/usr/share/kibana/node_modules/@elastic/elasticsearch/lib/api/api/search.js:66:12) at fetchClusters (/usr/share/kibana/node_modules/@kbn/monitoring-plugin/server/lib/alerts/fetch_clusters.js:47:20) at UsageCollector.fetch (/usr/share/kibana/node_modules/@kbn/monitoring-plugin/server/kibana_monitoring/collectors/get_usage_collector.js:99:24) at CollectorSet.fetchCollector (/usr/share/kibana/node_modules/@kbn/usage-collection-plugin/server/collector/collector_set.js:141:24) at fetch_monitoring (/usr/share/kibana/node_modules/@kbn/usage-collection-plugin/server/collector/collector_set.js:175:103)
```

从日志中可以看出，Kibana 试图访问一个没有启用远程集群客户端角色的 Elasticsearch 节点。`illegal_argument_exception` 错误提示节点 `elasticsearch-6c8c7d4877-48852` 没有启用远程集群客户端角色。

### 解决方法

需要启用 Elasticsearch 节点的远程集群客户端角色。

### 修改 Elasticsearch 配置

在 `elasticsearch.yml` 配置文件中，添加 `remote_cluster_client` 角色。以下是更新后的配置文件：

```yaml
cluster.name: "docker-cluster"
node.name: "node-1"
network.host: 0.0.0.0
http.host: 0.0.0.0

# 安全配置
xpack.security.enabled: false
xpack.security.enrollment.enabled: true
xpack.security.http.ssl.enabled: false
xpack.security.transport.ssl.enabled: false

# 节点角色配置
node.roles: [ master, data, remote_cluster_client ] #主要是这个地方

# 单节点配置
discovery.type: single-node
```

***

## 坑5 配置后提示如下
```
[plugins.eventLog] eventLog initialization operation failed and will be retried: createDataStreamIfNotExists; 2 more times; error: error creating data stream: illegal_state_exception Root causes: illegal_state_exception: index, alias, and data stream names need to be unique, but the following duplicates were found [data stream [.kibana-event-log-ds] conflicts with index] [2024-06-07T01:07:04.808+00:00][INFO ][plugins.eventLog] Creating datastream .kibana-event-log-ds [2024-06-07T01:07:04.817+00:00][WARN ][plugins.eventLog] eventLog initialization operation failed and will be retried: createDataStreamIfNotExists; 1 more times; error: error creating data stream: illegal_state_exception
```
从错误信息来看，Kibana 试图创建一个名为 `.kibana-event-log-ds` 的数据流（data stream），但这个名字已经被一个现有的索引或别名使用了。这导致了 `illegal_state_exception` 错误。

### 解决方法

1. **检查和删除冲突的索引或别名**：
   你需要检查 Elasticsearch 中是否存在名为 `.kibana-event-log-ds` 的索引或别名，并删除它们。

2. **创建数据流**：
   确认删除冲突的索引或别名后，Kibana 将能够成功创建所需的数据流。

### 具体步骤

1. **列出现有的索引和别名**：
   首先，列出所有现有的索引和别名，看看是否存在冲突的名字。

   列出索引：
   ```sh
   curl -X GET "http://101.40.70.165:40757/_cat/indices?v"
   ```

   列出别名：
   ```sh
   curl -X GET "http://101.40.70.165:40757/_cat/aliases?v"
   ```

2. **删除冲突的索引或别名**：
   如果存在名为 `.kibana-event-log-ds` 的索引或别名，需要删除它们。

   删除索引：
   ```sh
   curl -X DELETE "http://101.40.70.165:40757/.kibana-event-log-ds"
   ```

   删除别名（如果存在）：
   ```sh
   curl -X DELETE "http://101.40.70.165:40757/_alias/.kibana-event-log-ds"
   ```

3. **重启 Kibana**：
   在删除冲突的索引或别名后，重新启动 Kibana：

   ```sh
   docker-compose restart kibana
   ```

### 示例命令

以下是一些具体的命令示例：

列出所有索引：
```sh
curl -X GET "http://101.40.70.165:40757/_cat/indices?v"
```

列出所有别名：
```sh
curl -X GET "http://101.40.70.165:40757/_cat/aliases?v"
```

删除冲突的索引：
```sh
curl -X DELETE "http://101.40.70.165:40757/.kibana-event-log-ds"
```

删除冲突的别名（如果存在）：
```sh
curl -X DELETE "http://101.40.70.165:40757/_alias/.kibana-event-log-ds"
```

### 重启服务

重启 Kibana 服务：
```sh
docker-compose restart kibana
```

通过这些步骤，你应该能够解决数据流名称冲突的问题，让 Kibana 正常启动并连接到 Elasticsearch。如果你按照这些步骤操作后仍然遇到问题，请提供更多的日志信息以便进一步排查。


从返回的结果来看，当前没有名为 `.kibana-event-log-ds` 的数据流。这意味着问题并不是由现有的数据流引起的。那么，我们需要排查是否存在其他原因导致 Kibana 无法创建 `.kibana-event-log-ds` 数据流。

### 全面检查并清理环境

1. **检查和删除可能存在的冲突索引**：
   尽管我们没有在 `/_cat/indices` 中看到 `.kibana-event-log-ds`，但可以尝试直接删除它，确保没有残留的索引。

   ```sh
   curl -X DELETE "http://101.40.70.165:40757/.kibana-event-log-ds"
   ```

2. **检查并删除可能存在的别名**：
   确保没有名为 `.kibana-event-log-ds` 的别名。

   ```sh
   curl -X DELETE "http://101.40.70.165:40757/_alias/.kibana-event-log-ds"
   ```

3. **重新启动 Elasticsearch 和 Kibana**：
   确保所有的更改都已应用，重新启动服务。

   ```sh
   docker-compose down
   docker-compose up -d
   ```

### 确保 Kibana 配置正确

确认 Kibana 的 `kibana.yml` 配置文件中没有错误。以下是一个基本的 `kibana.yml` 配置示例：

```yaml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://101.40.70.165:40757"]
```

### 清理 Kibana 索引和重新初始化

如果上述步骤仍然不能解决问题，可以尝试清理 Kibana 的索引并重新初始化。

1. **删除 Kibana 索引**：
   Kibana 会在启动时自动重新创建这些索引。

   ```sh
   curl -X DELETE "http://101.40.70.165:40757/.kibana_*"
   curl -X DELETE "http://101.40.70.165:40757/.tasks"
   curl -X DELETE "http://101.40.70.165:40757/.kibana_task_manager_*"
   ```

2. **重启 Kibana**：
   删除索引后，重新启动 Kibana。

   ```sh
   docker-compose restart kibana
   ```

### 总结

通过上述步骤，你可以确保删除任何可能存在的冲突索引或别名，清理并重新初始化 Kibana 的环境。如果问题依然存在，请提供更多的错误日志信息，以便进一步排查问题。

附录，kibana.yml的配置
```
#
# ** THIS IS AN AUTO-GENERATED FILE **
#
# Default Kibana configuration for docker target
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://101.40.70.165:40757" ]
monitoring.ui.container.elasticsearch.enabled: true

# Set encryption keys to avoid warnings and ensure functionality
xpack.encryptedSavedObjects.encryptionKey: "W1xV8y9fH6N1gY4oR2mS6kN3wL3tC6yZ"
xpack.reporting.encryptionKey: "mQ5xN8yF7kL2tD3bX5zV2rH4jG3nY6rQ"
xpack.screenshotting.browser.chromium.disableSandbox: true

# Additional encryption key to avoid session invalidation warnings
xpack.security.encryptionKey: "something_at_least_32_characters_long"

# CORS and CSP settings
server.customResponseHeaders:
  Cross-Origin-Opener-Policy: "same-origin" 
  Cross-Origin-Embedder-Policy: "require-corp"
  Content-Security-Policy: "script-src 'self' 'unsafe-inline';"

# SSL Verification Mode (replace deprecated rejectUnauthorized)
xpack.actions.ssl.verificationMode: "none"

# Reporting roles behavior
xpack.reporting.roles.enabled: false

# Asset Manager settings
server.publicBaseUrl: "http://101.40.70.165"
i18n.locale: "zh-CN"  //这个设置让其变成中文

```

最后终于成功了


