```
docker run -d --restart=unless-stopped \  
  -p 8080:80 -p 8443:443 \  
  --privileged \  
  -v /root/rancher-certs/FULL_CHAIN.pem:/etc/rancher/ssl/cert.pem \  
  -v /root/rancher-certs/PRIVATE_KEY.pem:/etc/rancher/ssl/key.pem \  
  -v /root/rancher-certs/CA_CERTS.pem:/etc/rancher/ssl/cacerts.pem \  
  -v /data/rancher:/var/lib/rancher \  
  -e CATTLE_SYSTEM_DEFAULT_REGISTRY=registry.cn-hangzhou.aliyuncs.com \  
  -e CATTLE_SERVER_URL=https://rancher.tc.org:8443 \  
  -e CATTLE_BOOTSTRAP_PASSWORD=Pkpm@wh2021_1 \  
  --name rancher \  
  registry.cn-hangzhou.aliyuncs.com/rancher/rancher:v2.11.1
```

k3s安装命令
```
helm install rancher rancher-latest/rancher \  
  --namespace cattle-system \  
  --set hostname=rancher.tc.org \  
  --set replicas=1 \  
  --set bootstrapPassword=Pkpm@wh2021_1 \  
  --set systemDefaultRegistry="registry.cn-hangzhou.aliyuncs.com" \  
  --set ingress.ingressClassName=traefik \  
  --set ingress.ports.http=8080 \  
  --set ingress.ports.https=8443
```