```
scp root@<IP_OF_LINUX_MACHINE>:/etc/rancher/k3s/k3s.yaml ~/.kube/config

vi ~/.kube/config
```

手动下载helm.
**手动下载 Helm 二进制文件**

Helm 的官方二进制文件发布在 GitHub 的 Release 页面上。你可以直接从这里下载对应你操作系统和架构的二进制包，然后手动安装。这种方法不依赖于那个安装脚本，而是直接获取最终的可执行文件。

1. **访问 Helm Release 页面：** 在浏览器中打开 Helm 的 GitHub Release 页面：`https://github.com/helm/helm/releases`
    
2. **选择版本并下载：** 找到你想要的 Helm 版本（通常选择最新的稳定版本）。在对应版本的 Assets (资产) 中，找到适用于 **Linux amd64** 的压缩包，通常命名为 `helm-vX.Y.Z-linux-amd64.tar.gz` (其中 `X.Y.Z` 是版本号)。
    
    你可以通过其他网络条件较好的电脑下载这个文件，或者使用支持断点续传、多线程下载的工具来尝试下载。
    
3. **上传并解压文件：** 将下载好的 `helm-vX.Y.Z-linux-amd64.tar.gz` 文件上传到你的服务器。然后解压：
    
    Bash
    
    ```
    tar -zxvf helm-vX.Y.Z-linux-amd64.tar.gz
    # 解压后通常会得到一个 linux-amd64 的文件夹
    ```
    
4. **安装 Helm 二进制文件：** 解压后的文件夹中会有一个 `helm` 可执行文件。将它移动到系统的 PATH 环境变量包含的目录中，例如 `/usr/local/bin/`。
    
    Bash
    
    ```
    sudo mv linux-amd64/helm /usr/local/bin/
    ```
    
5. **验证安装：** 关闭当前终端或执行 `bash` 命令重新加载环境变量，然后运行：
    
    Bash
    
    ```
    helm version
    ```
    
    如果能正确显示版本信息，说明安装成功。


手动处理`cert-manager


- **下载 `cert-manager.yaml` 文件：** 在 v1.17.2 版本的 Assets 列表中，找到 `cert-manager.yaml` 这个文件，下载它。 你可以尝试使用 `curl` 命令下载到你的服务器的 `/root` 目录：
    
    Bash
    
    ```
    curl -L -o /root/cert-manager.yaml https://github.com/cert-manager/cert-manager/releases/download/v1.17.2/cert-manager.yaml
    ```
    
    如果直接下载困难，也请尝试使用其他方式（浏览器、下载工具等）下载后，上传到 `/root` 目录。
    
- **使用 `kubectl apply` 安装 Cert-Manager：** 这个命令会一步到位地安装 Cert-Manager 的所有组件（包括 CRD 和 Deployment）。
    
    Bash
    
    ```
    kubectl apply -f /root/cert-manager.yaml
    ```
    
    **解释：**
    
    - `kubectl apply -f`: 使用指定的文件来创建或更新 Kubernetes 资源。
    - `/root/cert-manager.yaml`: 你下载的 `cert-manager.yaml` 文件的本地路径。
- **监控 Cert-Manager Pod 状态：** 安装命令执行后，等待几分钟，然后查看 Cert-Manager 的 Pod 是否正常启动：
    
    Bash
    
    ```
    kubectl get pods -n cert-manager
    ```
    
    你应该看到 `cert-manager-xxx`, `cert-manager-cainjector-xxx`, `cert-manager-webhook-xxx` 这几个 Pod 都处于 `Running` 状态。**注意，使用 `kubectl apply` 方式安装，命名空间不再是 `cert-manager`，而是文件内部指定的命名空间，通常也是 `cert-manager`。** 如果你运行 `kubectl get pods -A` 能看到这几个 Pod 在某个命名空间运行，也是可以的。

最后运行命令如下，用国内的地址
```
helm install rancher rancher-latest/rancher \  
  --namespace cattle-system \  
  --set hostname=rancher.tc.org \  
  --set replicas=1 \  
  --set bootstrapPassword=Pkpm@wh2021_1 \  
  --set systemDefaultRegistry="registry.cn-hangzhou.aliyuncs.com" \  
  --set ingress.ingressClassName=traefik # K3s 默认使用 Traefik
```

--删除上面安装的rancher
```
kubectl delete pod rancher-f77c4c655-qxlkq -n cattle-system --force --grace-period=0

kubectl delete deployment rancher -n cattle-system

kubectl delete job rancher-post-delete -n cattle-system

helm uninstall rancher -n cattle-system
--下面两个是完全清除对应的目录，让电脑还原到没有安装rancher状态。
rm -rf /var/lib/rancher 
rm -rf /etc/rancher
--这一条是针对于我映射的-v /data/rancher:/var/lib/rancher 
rm -rf /data/rancher

mkdir -p /data/rancher/k3s/agent/images
cp -r /data/rancher-backup/k3s/agent/images/* /data/rancher/k3s/agent/images/

```

#### 使用清理脚本

使用也非常简单，只需要运行一个 K8s 的 job 即可：

`# git clone https://github.com/rancher/rancher-cleanup.git # cd rancher-cleanup # kubectl create -f deploy/rancher-cleanup.yaml`

运行的同时可以试试查看清理的进度和日志：

`kubectl -n kube-system logs -l job-name=cleanup-job -f`

#### [](https://forums.rancher.cn/t/rancher/1686#h-4)验证

- 使用 `kubectl create -f deploy/verify.yaml` 运行一个 job。
    
- 使用 `kubectl -n kube-system logs -l job-name=verify-job -f` 查看日志，输出应该为空（除了 deprecated 警告）
    
- 使用 `kubectl -n kube-system logs -l job-name=verify-job -f | grep -v "is deprecated"` | 检查完成的日志 grep -v “deprecated”，这将排除 deprecated 警告。
