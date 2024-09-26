`nginx-proxy` 容器部署在除了`controlplane`角色的所有节点上。他通过动态生成 NGINX 的配置，从而提供对`controlplane`角色节点的访问。

## 检查容器是否正在运行[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/nginx-proxy/_index#%E6%A3%80%E6%9F%A5%E5%AE%B9%E5%99%A8%E6%98%AF%E5%90%A6%E6%AD%A3%E5%9C%A8%E8%BF%90%E8%A1%8C "Direct link to heading")

nginx-proxy 容器在正常情况应该是 **Up** 状态。 并且 **Up** 状态应该是长时间运行，通过下面命令可以进行检查：

```
docker ps -a -f=name=nginx-proxy

Copy

输出示例：

docker ps -a -f=name=nginx-proxy

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

c3e933687c0e rancher/rke-tools:v0.1.15 "nginx-proxy CP_HO..." 3 hours ago Up 3 hours nginx-proxy

Copy
```

## 检查动态生成的 NGINX 配置[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/nginx-proxy/_index#%E6%A3%80%E6%9F%A5%E5%8A%A8%E6%80%81%E7%94%9F%E6%88%90%E7%9A%84-nginx-%E9%85%8D%E7%BD%AE "Direct link to heading")

生成的配置应包括具有`controlplane`角色的节点的 IP 地址。 可以使用以下命令检查配置：

```
docker exec nginx-proxy cat /etc/nginx/nginx.conf

Copy

输出示例：

error_log stderr notice;

worker_processes auto;

events {

multi_accept on;

use epoll;

worker_connections 1024;

}

stream {

upstream kube_apiserver {

server ip_of_controlplane_node1:6443;

server ip_of_controlplane_node2:6443;

}

server {

listen 6443;

proxy_pass kube_apiserver;

proxy_timeout 30;

proxy_connect_timeout 2s;

}

}

```


## nginx-proxy 容器日志[#](https://docs.rancher.cn/docs/rancher2.5/troubleshooting/kubernetes-components/nginx-proxy/_index#nginx-proxy-%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97 "Direct link to heading")

通过下面命令查看容器日志信息可以查看到可能包含的`nginx-proxy`错误信息：

docker logs nginx-proxy

