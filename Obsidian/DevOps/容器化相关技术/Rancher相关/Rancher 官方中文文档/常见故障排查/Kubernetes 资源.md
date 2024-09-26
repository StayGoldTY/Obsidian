此页面上列出的命令/步骤可用于检查中最重要的 Kubernetes 资源。

确保您配置了正确的 kubeconfig（例如，Rancher 高可用时，`export KUBECONFIG=$PWD/kube_config_cluster.yml`）或者通过 Rancher UI 使用内嵌的 kubectl。

## 节点[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E8%8A%82%E7%82%B9 "Direct link to heading")

### 获取节点[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E8%8E%B7%E5%8F%96%E8%8A%82%E7%82%B9 "Direct link to heading")

运行以下命令并检查以下内容：

- 集群中的所有节点都应列出，确保没有任何节点丢失。
- 所有节点都应处于`Ready`状态（如果未处于`Ready`状态，请使用`docker logs kubelet`检查该节点上的`kubelet`容器日志）
- 检查所有节点是否是正确的版本。
- 检查 OS/内核/Docker 值是否按预期显示（如果不正确可能是由于升级了 OS/内核/Docker 而引起的问题）

kubectl get nodes -o wide

Copy

输出示例：
```
NAME STATUS ROLES AGE VERSION INTERNAL-IP EXTERNAL-IP OS-IMAGE KERNEL-VERSION CONTAINER-RUNTIME

controlplane-0 Ready controlplane 31m v1.13.5 138.68.188.91 <none> Ubuntu 18.04.2 LTS 4.15.0-47-generic docker://18.9.5

etcd-0 Ready etcd 31m v1.13.5 138.68.180.33 <none> Ubuntu 18.04.2 LTS 4.15.0-47-generic docker://18.9.5

worker-0 Ready worker 30m v1.13.5 139.59.179.88 <none> Ubuntu 18.04.2 LTS 4.15.0-47-generic docker://18.9.5

Copy

```

### 获取节点 Conditions[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E8%8E%B7%E5%8F%96%E8%8A%82%E7%82%B9-conditions "Direct link to heading")

运行以下命令以列出节点 Conditions，关于节点 Conditions 请查看[节点 Conditions](https://kubernetes.io/docs/concepts/architecture/nodes/#condition)

```
kubectl get nodes -o go-template='{{range .items}}{{$node := .}}{{range .status.conditions}}{{$node.metadata.name}}{{": "}}{{.type}}{{":"}}{{.status}}{{"\n"}}{{end}}{{end}}'
```

Copy

运行以下命令以列出节点有问题的 Conditions，关于节点 Conditions 请查看[节点 Conditions](https://kubernetes.io/docs/concepts/architecture/nodes/#condition)

```
kubectl get nodes -o go-template='{{range .items}}{{$node := .}}{{range .status.conditions}}{{if ne .type "Ready"}}{{if eq .status "True"}}{{$node.metadata.name}}{{": "}}{{.type}}{{":"}}{{.status}}{{"\n"}}{{end}}{{else}}{{if ne .status "True"}}{{$node.metadata.name}}{{": "}}{{.type}}{{": "}}{{.status}}{{"\n"}}{{end}}{{end}}{{end}}{{end}}'

Copy

输出示例：

worker-0: DiskPressure:True

Copy

```
## Kubernetes Leader 选举[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#kubernetes-leader-%E9%80%89%E4%B8%BE "Direct link to heading")

### Kubernetes Controller Manager Leader[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#kubernetes-controller-manager-leader "Direct link to heading")

leader 选举由选举程序决定。在确定了 leader 节点之后，leader 状态(`holderIdentity`) 将会保存在 `kube-controller-manager` endpoint 中 (在本示例中，leader 节点是`controlplane-0`)。

```
kubectl -n kube-system get endpoints kube-controller-manager -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'

{"holderIdentity":"controlplane-0_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","leaseDurationSeconds":15,"acquireTime":"2018-12-27T08:59:45Z","renewTime":"2018-12-27T09:44:57Z","leaderTransitions":0}>
```

Copy

### Kubernetes Scheduler Leader[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#kubernetes-scheduler-leader "Direct link to heading")

leader 选举由选举程序决定。在确定了 leader 节点之后，leader 状态(`holderIdentity`) 将会保存在 `kube-scheduler` endpoint 中 (在本示例中，leader 节点是 `controlplane-0`)。

```
kubectl -n kube-system get endpoints kube-scheduler -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'

{"holderIdentity":"controlplane-0_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","leaseDurationSeconds":15,"acquireTime":"2018-12-27T08:59:45Z","renewTime":"2018-12-27T09:44:57Z","leaderTransitions":0}>
```



## Ingress Controller[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#ingress-controller "Direct link to heading")

默认的 Ingress Controller 是 NGINX，并作为 DaemonSet 部署在`ingress-nginx` 命名空间中。只能将该 Pod 调度到具有`worker`角色的节点上。

检查 Pod 是否在所有节点上运行：

```
kubectl -n ingress-nginx get pods -o wide
示例输出：

kubectl -n ingress-nginx get pods -o wide

NAME READY STATUS RESTARTS AGE IP NODE

default-http-backend-797c5bc547-kwwlq 1/1 Running 0 17m x.x.x.x worker-1

nginx-ingress-controller-4qd64 1/1 Running 0 14m x.x.x.x worker-1

nginx-ingress-controller-8wxhm 1/1 Running 0 13m x.x.x.x worker-0

```

如果 Pod 无法运行（状态不是 **Running**，`Ready`状态未显示为`1/1`，或者您看到大量的`Restart`次数），请检查 Pod 的详细信息，日志和 namespaces 事件。

### 查看 Pod 详细信息[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-pod-%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF "Direct link to heading")

kubectl -n ingress-nginx describe pods -l app=ingress-nginx

Copy

### 查看 Pod 容器日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-pod-%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97 "Direct link to heading")

kubectl -n ingress-nginx logs -l app=ingress-nginx

Copy

### 查看 Namespace 事件[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-namespace-%E4%BA%8B%E4%BB%B6 "Direct link to heading")

kubectl -n ingress-nginx get events

Copy

### Debug 日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#debug-%E6%97%A5%E5%BF%97 "Direct link to heading")

执行下面命令开启 debug 日志：

kubectl -n ingress-nginx patch ds nginx-ingress-controller --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--v=5"}]'

Copy

### 检查配置[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%A3%80%E6%9F%A5%E9%85%8D%E7%BD%AE "Direct link to heading")

查看每个 Pod 中生成的配置：

kubectl -n ingress-nginx get pods -l app=ingress-nginx --no-headers -o custom-columns=.NAME:.metadata.name | while read pod; do kubectl -n ingress-nginx exec $pod -- cat /etc/nginx/nginx.conf; done

Copy

## Rancher Agents[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#rancher-agents "Direct link to heading")

Rancher 通过 cluster-agent 与集群进行通信（通过`cattle-cluster-agent`调用 Kubernetes API 与集群通讯），并且通过`cattle-node-agent`与节点进行通信。

### cattle-node-agent[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#cattle-node-agent "Direct link to heading")

检查是否每个节点都正常运行了 cattle-node-agent, 正确运行的状态应该是 **Running** 并且重启的次数应该不多。

kubectl -n cattle-system get pods -l app=cattle-agent -o wide

Copy

输出示例：

NAME READY STATUS RESTARTS AGE IP NODE

cattle-node-agent-4gc2p 1/1 Running 0 2h x.x.x.x worker-1

cattle-node-agent-8cxkk 1/1 Running 0 2h x.x.x.x etcd-1

cattle-node-agent-kzrlg 1/1 Running 0 2h x.x.x.x etcd-0

cattle-node-agent-nclz9 1/1 Running 0 2h x.x.x.x controlplane-0

cattle-node-agent-pwxp7 1/1 Running 0 2h x.x.x.x worker-0

cattle-node-agent-t5484 1/1 Running 0 2h x.x.x.x controlplane-1

cattle-node-agent-t8mtz 1/1 Running 0 2h x.x.x.x etcd-2

Copy

检查特定节点上 cattle-node-agent 或者所有节点上 cattle-node-agent pods 的日志是否有错误：

kubectl -n cattle-system logs -l app=cattle-agent

Copy

### cattle-cluster-agent[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#cattle-cluster-agent "Direct link to heading")

检查集群中是否正确运行了 cattle-cluster-agent，正确运行的状态应该是 **Running** 并且重启的次数应该不多。

kubectl -n cattle-system get pods -l app=cattle-cluster-agent -o wide

Copy

输出示例：

NAME READY STATUS RESTARTS AGE IP NODE

cattle-cluster-agent-54d7c6c54d-ht9h4 1/1 Running 0 2h x.x.x.x worker-1

Copy

检查 cattle-cluster-agent pod 的日志是否有错误:

kubectl -n cattle-system logs -l app=cattle-cluster-agent

Copy

## 其他[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E5%85%B6%E4%BB%96 "Direct link to heading")

### 检查所有 Pods/Jobs 的状态[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%A3%80%E6%9F%A5%E6%89%80%E6%9C%89-podsjobs-%E7%9A%84%E7%8A%B6%E6%80%81 "Direct link to heading")

执行以下命令进行检查集群中所有 Pod/Job 是否正常运行：

kubectl get pods --all-namespaces

Copy

Pods 和 Jobs 正确运行的状态应该是：

- **Running**，并且重启的次数应该不多。
- 或者是**Completed**，已经完成运行。

### 查看 Pod 详细信息[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-pod-%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF-1 "Direct link to heading")

kubectl describe pod POD_NAME -n NAMESPACE

Copy

### 查看 Pod 日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-pod-%E6%97%A5%E5%BF%97 "Direct link to heading")

kubectl logs POD_NAME -n NAMESPACE

Copy

如果 Job 状态是 **Completed** 则可以通过以下命令检查未完成原因：

### 查看 Job 详细信息[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-job-%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF "Direct link to heading")

kubectl describe job JOB_NAME -n NAMESPACE

Copy

### 查看 Job Pod 的日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E6%9F%A5%E7%9C%8B-job-pod-%E7%9A%84%E6%97%A5%E5%BF%97 "Direct link to heading")

kubectl logs -l job-name=JOB_NAME -n NAMESPACE

Copy

### 被驱逐的 Pod[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#%E8%A2%AB%E9%A9%B1%E9%80%90%E7%9A%84-pod "Direct link to heading")

Pod 会根据[驱逐信号](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#eviction-policy)中的描述进行驱逐。

查看被逐出的 Pod 列表（Pod 名称和命名空间）：

kubectl get pods --all-namespaces -o go-template='{{range .items}}{{if eq .status.phase "Failed"}}{{if eq .status.reason "Evicted"}}{{.metadata.name}}{{" "}}{{.metadata.namespace}}{{"\n"}}{{end}}{{end}}{{end}}'

Copy

要删除所有被驱逐的 Pod，

kubectl get pods --all-namespaces -o go-template='{{range .items}}{{if eq .status.phase "Failed"}}{{if eq .status.reason "Evicted"}}{{.metadata.name}}{{" "}}{{.metadata.namespace}}{{"\n"}}{{end}}{{end}}{{end}}' | while read epod enamespace; do kubectl -n $enamespace delete pod $epod; done

Copy

检查被逐出的 Pod，已调度节点的列表以及被驱逐的原因：

kubectl get pods --all-namespaces -o go-template='{{range .items}}{{if eq .status.phase "Failed"}}{{if eq .status.reason "Evicted"}}{{.metadata.name}}{{" "}}{{.metadata.namespace}}{{"\n"}}{{end}}{{end}}{{end}}' | while read epod enamespace; do kubectl -n $enamespace get pod $epod -o=custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,MSG:.status.message; done

Copy

### Job 的状态一直没有变更为 Completed[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index#job-%E7%9A%84%E7%8A%B6%E6%80%81%E4%B8%80%E7%9B%B4%E6%B2%A1%E6%9C%89%E5%8F%98%E6%9B%B4%E4%B8%BA-completed "Direct link to heading")

如果您启用了 Istio，而且部署了 Job 之后，Job 的状态一直没有变更为**Completed**，您需要参考[这些步骤](https://docs.rancher.cn/docs/rancher2.5/cluster-admin/tools/istio/setup/enable-istio-in-namespace/_index)手动添加 annotation。

因为 Istio Sidecarh 会无休止地运行，即使 Job 的任务完成了，它的状态也不能被视为**Completed**。上述步骤是在短期内处理这个问题的方法，它禁止了 Istio 和添加了 annotation 的 Pod 之间的通信。如果您使用了这种方式解决这个问题，这个 Job 就没有权限访问 service mesh，不能够用于集成测试。
