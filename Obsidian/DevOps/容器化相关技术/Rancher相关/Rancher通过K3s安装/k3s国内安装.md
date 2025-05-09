```
# curl –sfL \
     https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | \
     INSTALL_K3S_MIRROR=cn sh -s - \
     --system-default-registry "registry.cn-hangzhou.aliyuncs.com"

[INFO]  Finding release for channel stable
[INFO]  Using v1.25.3+k3s1 as release
[INFO]  Downloading hash rancher-mirror.rancher.cn/k3s/v1.25.3-k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary rancher-mirror.rancher.cn/k3s/v1.25.3-k3s1/k3s
[INFO]  Verifying binary download
...
...
[INFO]  systemd: Starting k3s
```