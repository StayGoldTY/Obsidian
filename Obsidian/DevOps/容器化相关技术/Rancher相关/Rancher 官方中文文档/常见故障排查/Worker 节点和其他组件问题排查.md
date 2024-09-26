本节适用于每个节点，因为它包括在具有任何角色的节点上运行的组件。

## 检查容器是否正在运行[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/worker-and-generic/_index#%E6%A3%80%E6%9F%A5%E5%AE%B9%E5%99%A8%E6%98%AF%E5%90%A6%E6%AD%A3%E5%9C%A8%E8%BF%90%E8%A1%8C "Direct link to heading")

`worker`节点上应该运行以下两个容器：

- kubelet
- kube-proxy

这些容器的正常情况应该是 **Up** 状态。 并且 **Up** 状态应该是长时间运行，通过下面命令可以进行检查：

```
docker ps -a -f=name='kubelet|kube-proxy'

Copy

输出示例：

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

158d0dcc33a5 rancher/hyperkube:v1.11.5-rancher1 "/opt/rke-tools/en..." 3 hours ago Up 3 hours kube-proxy

a30717ecfb55 rancher/hyperkube:v1.11.5-rancher1 "/opt/rke-tools/en..." 3 hours ago Up 3 hours kubelet

Copy
```

## 容器日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/worker-and-generic/_index#%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97 "Direct link to heading")

通过下面命令查看容器日志信息可以查看到可能包含的错误信息：

docker logs kubelet

docker logs kube-proxy

