1. 先备份rancher
```
docker run --name busybox-backup-2024-01-09 --volumes-from 819c26791480 -v $PWD:/backup busybox tar pzcvf /backup/rancher-data-backup-2.5.9-2024-01-09.tar.gz /var/lib/rancher

```
2. 通过 docker inspect   xxxx 来反推出大概的docker的运行命令
```
docker run -d  --name /rancher  --restart=unless-stopped   -p 8080:80 -p 8443:443  -v /docker_volume/rancher_home/auditlog:/var/log/auditlog  -v /docker_volume/rancher_home/rancher:/var/lib/rancher   rancher/rancher:v2.7.1

```
3. 拉取你需要更新的rancher版本镜像
  ```
  docker pull rancher/rancher:v2.7.1

```
4. 停止之前rancher的容器，并删除
 ```
 docker stop 819c26791480
  952  docker rm -f 819c26791480

```
  5. 运行新的镜像
  ```
  docker run -d --privileged  --name /rancher  --restart=unless-stopped   -p 8080:80 -p 8443:443  -v /docker_volume/rancher_home/auditlog:/var/log/auditlog  -v /docker_volume/rancher_home/rancher:/var/lib/rancher   rancher/rancher:v2.7.1

```
6.  解决各种报错
情况一：
```
[Error checking if image [rancher/rke-tools:v0.1.90] exists on host [10.111.155.115]: error during connect: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.41/images/rancher/rke-tools:v0.1.90/json": can not build dialer to [c-rkvdg:m-d3bc217ce9d9]]
```
解决：下载对应的镜像 
```
docker pull rancher/rke-tools:v0.1.90

```
情况二：没有对应的agent，执行对应的agent的注册命令
```
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.7.1 --server https://10.111.155.115:8443 --token n5s5f6l9t58nwbz6hf8lgzpnvfmszmfmzfsw9qj4pcchgw9kknnvsn --ca-checksum 61d92dd21fc8812b46fe973a34775c1d749f62842d4bfdbd500e9196625d7952 --etcd --controlplane --worker

```

情况三：注册后依然报错
```
Template system-library-rancher-monitoring incompatible with rancher version or cluster's [c-rkvdg] kubernetes version
```
这个是最棘手的，弄了很久。
看官方的解释大概是因为k8s版本升级高于1.22版本后就和之前的图表功能不兼容了
具体细节在这里
https://github.com/rancher/rancher/issues/37039
我用下面的方法解决的：
```
kubectl edit clusters.management.cattle.io c-rkvdg
然后把对应的地方都如下设置一下
spec:
  enableClusterAlerting: false
  enableClusterMonitoring: false


```
并且把找到包含错误消息的部分：

```
- lastUpdateTime: "2023-03-28T03:29:01Z"
  message: template system-library-rancher-monitoring incompatible with rancher
    version or cluster's [c-qmh8k] kubernetes version
  reason: Error
  status: "False"
```

将其替换为以下部分：

```
- lastUpdateTime: "2023-03-28T03:29:01Z"
  status: "True"
  type: PrometheusOperatorDeployed
```

保存更改并退出编辑器。
问题解决

情况四：不知道什么原因突然报错：
```
2024/01/16 00:39:57 [INFO] Waiting for server to become available: Get "[https://127.0.0.1:6444/version?timeout=15m0s](https://127.0.0.1:6444/version?timeout=15m0s)": dial tcp 127.0.0.1:6444: connect: connection refused
```
https://github.com/rancher/rancher/issues/36238
https://github.com/rancher/rancher/issues/36238#issuecomment-1232690242