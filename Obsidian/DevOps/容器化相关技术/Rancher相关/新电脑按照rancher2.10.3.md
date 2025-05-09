## 前提先安装具体要求生成对应的本地证书
## 如果需要重新安装清除命令
```
- **停止并删除容器**：
    
   
    
    `docker stop rancher 
    docker rm rancher`
    
- **备份并清理数据目录**：
    
   
    
    `cp -r /data/rancher /data/rancher-backup 
    rm -rf /data/rancher/*`
```

## 复制离线包
```
mkdir -p /data/rancher/k3s/agent/images/
cd /root/rancher-certs
cp k3s-airgap-images.tar /data/rancher/k3s/agent/images/
```
## 运行命令
```
docker run -d --restart=unless-stopped \  
  -p 8080:80 -p 8443:443 \  
  --privileged \  
  -v /root/rancher-certs/rancher.crt:/etc/rancher/ssl/cert.pem \  
  -v /root/rancher-certs/rancher.key:/etc/rancher/ssl/key.pem \  
  -v /root/rancher-certs/ca.crt:/etc/rancher/ssl/cacerts.pem \  
  -v /data/rancher:/var/lib/rancher \  
  -e CATTLE_SYSTEM_DEFAULT_REGISTRY=registry.cn-hangzhou.aliyuncs.com \  
  -e CATTLE_SERVER_URL=https://rancher.tc.org:8443 \  
  --name rancher \  
  rancher/rancher:latest
```