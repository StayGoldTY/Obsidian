本节包含对具有`etcd`角色的节点进行故障排查的命令和技巧。

## 检查 etcd 容器是否正在运行[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5-etcd-%E5%AE%B9%E5%99%A8%E6%98%AF%E5%90%A6%E6%AD%A3%E5%9C%A8%E8%BF%90%E8%A1%8C "Direct link to heading")

etcd 的容器的状态应为**Up**。**Up**之后显示的持续时间是容器运行的时间。

docker ps -a -f=name=etcd$

Copy

输出示例:

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

605a124503b9 rancher/coreos-etcd:v3.2.18 "/usr/local/bin/et..." 2 hours ago Up 2 hours etcd

Copy

## etcd 容器日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#etcd-%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97 "Direct link to heading")

容器的日志记录可以包含有关可能出现的问题的信息。

docker logs etcd

Copy

|日志|说明|
|---|---|
|`health check for peer xxx could not connect: dial tcp IP:2380: getsockopt: connection refused`|无法建立与这个 IP 的 2380 端口进行连接。检查 etcd 容器是否在那个 IP 的主机上运行。|
|`xxx is starting a new election at term x`|etcd 集群已经失去了法定人数，正在尝试建立新的领导者。当大多数运行 etcd 的节点出现故障或无法访问时，可能会发生这种情况。|
|`connection error: desc = "transport: Error while dialing dial tcp 0.0.0.0:2379: i/o timeout"; Reconnecting to {0.0.0.0:2379 0 <nil>}`|主机防火墙阻止了网络通信。|
|`rafthttp: request cluster ID mismatch`|运行着 etcd 实例并记录`rafthttp: request cluster ID mismatch`的节点正在尝试加入另一个由其他成员构成的集群。应该从集群中删除这个节点，然后重新添加。|
|`rafthttp: failed to find member`|集群状态 (`/var/lib/etcd`) 包含错误信息，无法加入集群。应该从集群中删除这个节点，并删除状态目录，然后重新添加。|

## etcd 集群和连接性检查[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#etcd-%E9%9B%86%E7%BE%A4%E5%92%8C%E8%BF%9E%E6%8E%A5%E6%80%A7%E6%A3%80%E6%9F%A5 "Direct link to heading")

etcd 监听的地址取决于运行 etcd 的主机的地址配置。如果为运行 etcd 的主机配置了内部地址，则需要显式指定`etcdctl`的端点。如果有任何命令响应`Error: context deadline exceeded`，则代表 etcd 实例不正常（仲裁丢失或该实例未正确加入集群）

### 检查所有节点上的 etcd 成员[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E4%B8%8A%E7%9A%84-etcd-%E6%88%90%E5%91%98 "Direct link to heading")

输出应包含所有具有 etcd 角色的节点，并且所有节点上的输出应相同。

命令：

docker exec etcd etcdctl member list

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list"

Copy

输出示例:

xxx, started, etcd-xxx, https://IP:2380, https://IP:2379,https://IP:4001

xxx, started, etcd-xxx, https://IP:2380, https://IP:2379,https://IP:4001

xxx, started, etcd-xxx, https://IP:2380, https://IP:2379,https://IP:4001

Copy

### 检查端点状态[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E7%AB%AF%E7%82%B9%E7%8A%B6%E6%80%81 "Direct link to heading")

`RAFT TERM`的值应相等，`RAFT INDEX`的距离不应太远。

命令：

docker exec -e ETCDCTL_ENDPOINTS=$(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") etcd etcdctl endpoint status --write-out table

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd etcdctl endpoint status --endpoints=$(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") --write-out table

Copy

输出示例:

+-----------------+------------------+---------+---------+-----------+-----------+------------+

| ENDPOINT | ID | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |

+-----------------+------------------+---------+---------+-----------+-----------+------------+

| https://IP:2379 | 333ef673fc4add56 | 3.2.18 | 24 MB | false | 72 | 66887 |

| https://IP:2379 | 5feed52d940ce4cf | 3.2.18 | 24 MB | true | 72 | 66887 |

| https://IP:2379 | db6b3bdb559a848d | 3.2.18 | 25 MB | false | 72 | 66887 |

+-----------------+------------------+---------+---------+-----------+-----------+------------+

Copy

### 检查端点健康[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E7%AB%AF%E7%82%B9%E5%81%A5%E5%BA%B7 "Direct link to heading")

命令：

docker exec -e ETCDCTL_ENDPOINTS=$(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") etcd etcdctl endpoint health

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd etcdctl endpoint health --endpoints=$(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','")

Copy

输出示例:

https://IP:2379 is healthy: successfully committed proposal: took = 2.113189ms

https://IP:2379 is healthy: successfully committed proposal: took = 2.649963ms

https://IP:2379 is healthy: successfully committed proposal: took = 2.451201ms

Copy

### 检查端口 TCP / 2379 的连接[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E7%AB%AF%E5%8F%A3-tcp--2379-%E7%9A%84%E8%BF%9E%E6%8E%A5 "Direct link to heading")

命令：

for endpoint in $(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f5"); do

echo "Validating connection to ${endpoint}/health"

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -w "\n" --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) "${endpoint}/health"

done

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

for endpoint in $(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f5"); do

echo "Validating connection to ${endpoint}/health";

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -w "\n" --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) "${endpoint}/health"

done

Copy

输出示例：

Validating connection to https://IP:2379/health

{"health": "true"}

Validating connection to https://IP:2379/health

{"health": "true"}

Validating connection to https://IP:2379/health

{"health": "true"}

Copy

### 检查端口 TCP / 2380 的连接[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E7%AB%AF%E5%8F%A3-tcp--2380-%E7%9A%84%E8%BF%9E%E6%8E%A5 "Direct link to heading")

命令：

for endpoint in $(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f4"); do

echo "Validating connection to ${endpoint}/version";

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl --http1.1 -s -w "\n" --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) "${endpoint}/version"

done

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

for endpoint in $(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f4"); do

echo "Validating connection to ${endpoint}/version";

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl --http1.1 -s -w "\n" --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) "${endpoint}/version"

done

Copy

输出示例:

Validating connection to https://IP:2380/version

{"etcdserver":"3.2.18","etcdcluster":"3.2.0"}

Validating connection to https://IP:2380/version

{"etcdserver":"3.2.18","etcdcluster":"3.2.0"}

Validating connection to https://IP:2380/version

{"etcdserver":"3.2.18","etcdcluster":"3.2.0"}

Copy

## etcd 警报[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#etcd-%E8%AD%A6%E6%8A%A5 "Direct link to heading")

例如，etcd 空间不足时，etcd 将触发警报。

命令：

docker exec etcd etcdctl alarm list

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT alarm list"

Copy

触发 NOSPACE 警报时的示例输出:

memberID:x alarm:NOSPACE

memberID:x alarm:NOSPACE

memberID:x alarm:NOSPACE

Copy

## etcd 空间错误[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#etcd-%E7%A9%BA%E9%97%B4%E9%94%99%E8%AF%AF "Direct link to heading")

相关错误消息是`etcdserver: mvcc: database space exceeded`或`applying raft message exceeded backend quota`。警报`NOSPACE`将被触发。

解决方法：

### 压缩键空间[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E5%8E%8B%E7%BC%A9%E9%94%AE%E7%A9%BA%E9%97%B4 "Direct link to heading")

命令：

rev=$(docker exec etcd etcdctl endpoint status --write-out json | egrep -o '"revision":[0-9]*' | egrep -o '[0-9]*')

docker exec etcd etcdctl compact "$rev"

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

rev=$(docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT endpoint status --write-out json | egrep -o '\"revision\":[0-9]*' | egrep -o '[0-9]*'")

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT compact \"$rev\""

Copy

输出示例：

compacted revision xxx

Copy

### 对所有 etcd 成员进行碎片整理[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E5%AF%B9%E6%89%80%E6%9C%89-etcd-%E6%88%90%E5%91%98%E8%BF%9B%E8%A1%8C%E7%A2%8E%E7%89%87%E6%95%B4%E7%90%86 "Direct link to heading")

命令：

docker exec -e ETCDCTL_ENDPOINTS=$(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") etcd etcdctl defrag

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd sh -c "etcdctl defrag --endpoints=$(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','")"

Copy

输出示例：

Finished defragmenting etcd member[https://IP:2379]

Finished defragmenting etcd member[https://IP:2379]

Finished defragmenting etcd member[https://IP:2379]

Copy

### 检查端点状态[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%A3%80%E6%9F%A5%E7%AB%AF%E7%82%B9%E7%8A%B6%E6%80%81-1 "Direct link to heading")

命令：

docker exec -e ETCDCTL_ENDPOINTS=$(docker exec etcd /bin/sh -c "etcdctl member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") etcd etcdctl endpoint status --write-out table

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd sh -c "etcdctl endpoint status --endpoints=$(docker exec etcd /bin/sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT member list | cut -d, -f5 | sed -e 's/ //g' | paste -sd ','") --write-out table"

Copy

输出示例：

+-----------------+------------------+---------+---------+-----------+-----------+------------+

| ENDPOINT | ID | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |

+-----------------+------------------+---------+---------+-----------+-----------+------------+

| https://IP:2379 | e973e4419737125 | 3.2.18 | 553 kB | false | 32 | 2449410 |

| https://IP:2379 | 4a509c997b26c206 | 3.2.18 | 553 kB | false | 32 | 2449410 |

| https://IP:2379 | b217e736575e9dd3 | 3.2.18 | 553 kB | true | 32 | 2449410 |

+-----------------+------------------+---------+---------+-----------+-----------+------------+

Copy

### 解除告警[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E8%A7%A3%E9%99%A4%E5%91%8A%E8%AD%A6 "Direct link to heading")

确认压缩和碎片整理后 DB 大小减小后，需要解除该告警，以便 etcd 允许再次写入。

命令：

docker exec etcd etcdctl alarm list

docker exec etcd etcdctl alarm disarm

docker exec etcd etcdctl alarm list

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT alarm list"

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT alarm disarm"

docker exec etcd sh -c "etcdctl --endpoints=\$ETCDCTL_ENDPOINT alarm list"

Copy

输出示例：

docker exec etcd etcdctl alarm list

memberID:x alarm:NOSPACE

memberID:x alarm:NOSPACE

memberID:x alarm:NOSPACE

docker exec etcd etcdctl alarm disarm

docker exec etcd etcdctl alarm list

Copy

## 日志级别[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%97%A5%E5%BF%97%E7%BA%A7%E5%88%AB "Direct link to heading")

可以通过 API 动态更改 etcd 的日志级别。您可以使用以下命令配置调试日志记录。

命令：

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -XPUT -d '{"Level":"DEBUG"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) $(docker exec etcd printenv ETCDCTL_ENDPOINTS)/config/local/log

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -XPUT -d '{"Level":"DEBUG"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) $(docker exec etcd printenv ETCDCTL_ENDPOINT)/config/local/log

Copy

要将日志级别重置回默认值（INFO），可以使用以下命令。

命令：

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -XPUT -d '{"Level":"INFO"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) $(docker exec etcd printenv ETCDCTL_ENDPOINTS)/config/local/log

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker run --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro appropriate/curl -s -XPUT -d '{"Level":"INFO"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) $(docker exec etcd printenv ETCDCTL_ENDPOINT)/config/local/log

Copy

## etcd 内容[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#etcd-%E5%86%85%E5%AE%B9 "Direct link to heading")

如果要调查 etcd 的内容，则可以观看事件流或直接查询 etcd，请参见以下示例。

### 查看实时事件[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%9F%A5%E7%9C%8B%E5%AE%9E%E6%97%B6%E4%BA%8B%E4%BB%B6 "Direct link to heading")

命令：

docker exec etcd etcdctl watch --prefix /registry

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd etcdctl --endpoints=\$ETCDCTL_ENDPOINT watch --prefix /registry

Copy

如果只想查看受影响的键（而不是二进制数据），则可以附加 `| grep -a ^/registry` 命令仅过滤键。

### 直接查询 etcd[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E7%9B%B4%E6%8E%A5%E6%9F%A5%E8%AF%A2-etcd "Direct link to heading")

命令：

docker exec etcd etcdctl get /registry --prefix=true --keys-only

Copy

当使用低于 3.3.x 的 etcd 版本（Kubernetes 1.13.x 及更低版本）并且添加节点时指定了`--internal-address` 时的命令：

docker exec etcd etcdctl --endpoints=\$ETCDCTL_ENDPOINT get /registry --prefix=true --keys-only

Copy

您可以使用以下命令处理数据以获取每个键计数的摘要：

docker exec etcd etcdctl get /registry --prefix=true --keys-only | grep -v ^$ | awk -F'/' '{ if ($3 ~ /cattle.io/) {h[$3"/"$4]++} else { h[$3]++ }} END { for(k in h) print h[k], k }' | sort -nr

Copy

## 更换不健康的 etcd 节点[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/etcd/_index#%E6%9B%B4%E6%8D%A2%E4%B8%8D%E5%81%A5%E5%BA%B7%E7%9A%84-etcd-%E8%8A%82%E7%82%B9 "Direct link to heading")

当您的 etcd 集群中的某个节点不正常时，建议的方法是在将新的 etcd 节点添加到集群之前，先修复或删除出现故障或不正常的节点。