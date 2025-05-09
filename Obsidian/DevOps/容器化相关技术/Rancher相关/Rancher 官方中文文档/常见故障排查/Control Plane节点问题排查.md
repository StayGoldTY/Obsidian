本节适用于具有 `controlplane` 角色的节点。

## 检查 Controlplane 容器是否正在运行[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/controlplane/_index#%E6%A3%80%E6%9F%A5-controlplane-%E5%AE%B9%E5%99%A8%E6%98%AF%E5%90%A6%E6%AD%A3%E5%9C%A8%E8%BF%90%E8%A1%8C "Direct link to heading")

在具有`controlplane`角色的节点上启动了三个特定的容器

- `kube-apiserver`
- `kube-controller-manager`
- `kube-scheduler`

这三个容器的正常情况应该是 **Up** 状态。 并且 **Up** 状态应该是长时间运行，通过下面命令可以进行检查：

```
docker ps -a -f=name='kube-apiserver|kube-controller-manager|kube-scheduler'



输出示例：

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

26c7159abbcc rancher/hyperkube:v1.11.5-rancher1 "/opt/rke-tools/en..." 3 hours ago Up 3 hours kube-apiserver

f3d287ca4549 rancher/hyperkube:v1.11.5-rancher1 "/opt/rke-tools/en..." 3 hours ago Up 3 hours kube-scheduler

bdf3898b8063 rancher/hyperkube:v1.11.5-rancher1 "/opt/rke-tools/en..." 3 hours ago Up 3 hours kube-controller-manager
```



## Controlplane 容器日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/controlplane/_index#controlplane-%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97 "Direct link to heading")

> **注意：** 如果是 `controlplane` 角色的节点， `kube-controller-manager` 和 `kube-scheduler` 会通过选举选举出 leader 节点。 只有 leader 节点会记录当前操作的日志信息。查看 [Kubernetes leader 选举](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-resources/_index)可以知道如何查看当前的 leader 节点。

通过下面命令查看容器日志信息可以查看到可能包含的错误信息：

docker logs kube-apiserver

docker logs kube-controller-manager

docker logs kube-scheduler