最近公司拉取报错
```
docker pull nginx:1.25.1 1.25.1: Pulling from library/nginx 648e0aadf75a: Waiting 262696647b70: Waiting e66d0270d23f: Waiting 55ac49bd649c: Waiting cbf42f5a00d2: Waiting 8015f365966b: Waiting 4cadff8bc2aa: Waiting error pulling image configuration: Get "https://production.cloudflare.docker.com/registry-v2/docker/registry/v2/blobs/sha256/89/89da1fb6dcb964dd35c3f41b7b93ffc35eaf20bc61f2e1335fea710a18424287/data?verify=1718113777-sXJPP2ioLtkR1vODb6RCIFPB%2Bsc%3D": dial tcp 23.234.30.58:443: i/o timeout
```
修改daemon.json如下即可
```
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.m.daocloud.io",
    "http://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://docker.nju.edu.cn",
    "https://docker.mirrors.sjtug.sjtu.edu.cn",
    "https://mirror.aliyuncs.com"
  ],
  "insecure-registries": [
    "registry.docker-cn.com",
    "docker.mirrors.ustc.edu.cn"
  ],
  "dns": ["8.8.8.8", "8.8.4.4"]
}

```