


          
# Rancher 镜像拉取问题解决总结



还有一种补充一下对于`rancher/mirrored-coredns-coredns` 和 `rancher/mirrored-pause`
因为通过 `docker run` 启动的 Rancher 镜像中内置了 `rancher/mirrored-coredns-coredns` 和 `rancher/mirrored-pause` 这两个镜像的 tar 包，这样在离线环境上也能自动启动这两个服务。

但是这两个镜像的离线 tar 包默认存储在了 `/var/lib/rancher/k3s/agent/images/k3s-airgap-images.tar`。

当通过 docker run 来启动 Rancher 时，通过 -v 将 Rancher 的数据目录(/var/lib/rancher) 映射到本地之后，导致了容器中的 /var/lib/rancher 使用的是主机的空目录。也就是说新启动的 Rancher 中缺少了这个离线的 tar 包，而且，`CATTLE_SYSTEM_DEFAULT_REGISTRY` 对这两个 pod 不生效。现阶段国内网络使用 docker hub 有问题，所以导致了 `rancher/mirrored-coredns-coredns` 和 `rancher/mirrored-pause` 拉取失败。

为了解决这个问题，可以：

1. 取消映射的目录，这样就可以使用容器内置的离线 tar 包启动这两个服务。
2. 手动从镜像中获取 `/var/lib/rancher/k3s/agent/images/k3s-airgap-images.tar`，并传到对应目录下，例如：

`root@ksd:~# mkdir -p /data/rancher/k3s/agent/images/ root@ksd:~# docker run --rm --entrypoint "" -v $(pwd):/output rancher/rancher:v2.9.0 cp /var/lib/rancher/k3s/agent/images/k3s-airgap-images.tar /output/k3s-airgap-images.tar root@ksd:~# ls docker.sh  k3s-airgap-images.tar  snap root@ksd:~# cp k3s-airgap-images.tar /data/rancher/k3s/agent/images/ root@ksd:~# docker run -d --restart=unless-stopped --privileged \     -p 10080:80 -p 10443:443  \     -e CATTLE_SYSTEM_DEFAULT_REGISTRY=registry.cn-hangzhou.aliyuncs.com \     -e CATTLE_BOOTSTRAP_PASSWORD=rancher \     -v /data/rancher:/var/lib/rancher \     registry.cn-hangzhou.aliyuncs.com/rancher/rancher:v2.9.0 8b62637c72cfb4b3704d5811f8e9bbd37bd05a93c9155465e776b5e0481f5465`


## 下面的也很重要
同样也是通过本地下载coredns对应的版本，然后
```
# 拉取你需要的确切镜像名，这里以官方名为例
docker pull rancher/mirrored-coredns-coredns:1.12.0

# 将镜像保存为 tar 文件
docker save -o coredns-1.12.0.tar rancher/mirrored-coredns-coredns:1.12.0
```
**然后还是复制到下面的/data/rancher/k3s/agent/images/目录即可
```
rm -rf /data/rancher

mkdir -p /data/rancher/k3s/agent/images
cp -r /data/rancher-backup/k3s/agent/images/* /data/rancher/k3s/agent/images/
```