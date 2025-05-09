## 手动生成证书相关
```
## 这里有一个关键点就是openssl.conf里面的 basicConstraints = CA:TRUE
cd /root/rancher-certs
  openssl req -x509 -newkey rsa:2048 -nodes -keyout PRIVATE_KEY.pem -out cert.pem -days 365   -config openssl.cnf   -extensions v3_req # 明确指定使用配置文件中的 v3_req 扩展
  cp cert.pem FULL_CHAIN.pem
  cp cert.pem CA_CERTS.pem
  sudo cp /root/rancher-certs/CA_CERTS.pem /etc/pki/ca-trust/source/anchors/rancher-ca.pem
   sudo update-ca-trust extract
  curl -v https://rancher.tc.org:8443

```

## Openssl.conf
```
# /root/rancher-certs/openssl.cnf

[ req ]
default_bits        = 2048
prompt              = no  # 不会交互式询问信息
default_md          = sha256
distinguished_name  = req_distinguished_name
req_extensions      = v3_req # 指定包含 SAN 的扩展部分

[ req_distinguished_name ]
# 这里设置证书的基本信息，CN 最重要
countryName             = CN  # 国家代码 (可选)
stateOrProvinceName     = BeiJing # 省份 (可选)
localityName            = BeiJing # 城市 (可选)
organizationName        = MyOrg # 组织 (可选)
organizationalUnitName  = IT # 部门 (可选)
commonName              = rancher.tc.org # **重要：你的域名**
emailAddress            = admin@tc.org # (可选)

[ v3_req ]
# 这里指定 X.509 v3 扩展
basicConstraints = CA:TRUE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names # 引用下面的 SAN 列表

[ alt_names ]
# **重要：列出所有需要包含的域名和 IP 地址**
DNS.1 = rancher.tc.org
IP.1 = 10.111.155.178
# 如果还有其他域名或 IP，可以继续添加 DNS.2, IP.2 等
```

## kubectl get pod -n cattle-system | grep cluster-agentE0429 16:57:10.618464  100225 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify certificate: x509: certificate signed by unknown authority"


这个主要是把你通过什么方式安装的下面的对应的yaml复制到/root/.kube下面的config文件里面去比如：
rek2的就是/etc/rancher/rke2目录下面的rke2.yaml文件，k3s安装也是类似的

## 比较棘手的域名解析问题
 kubectl logs cattle-cluster-agent-65bb6467bc-kvqhh -n cattle-system -p
INFO: Environment: CATTLE_ADDRESS=10.42.0.2 CATTLE_CA_CHECKSUM=710fe21441cca293a6b0b157878418202cfdaecbc3ba662e3be466594f51654b CATTLE_CLUSTER=true CATTLE_CLUSTER_AGENT_PORT=tcp://10.43.151.237:80 CATTLE_CLUSTER_AGENT_PORT_443_TCP=tcp://10.43.151.237:443 CATTLE_CLUSTER_AGENT_PORT_443_TCP_ADDR=10.43.151.237 CATTLE_CLUSTER_AGENT_PORT_443_TCP_PORT=443 CATTLE_CLUSTER_AGENT_PORT_443_TCP_PROTO=tcp CATTLE_CLUSTER_AGENT_PORT_80_TCP=tcp://10.43.151.237:80 CATTLE_CLUSTER_AGENT_PORT_80_TCP_ADDR=10.43.151.237 CATTLE_CLUSTER_AGENT_PORT_80_TCP_PORT=80 CATTLE_CLUSTER_AGENT_PORT_80_TCP_PROTO=tcp CATTLE_CLUSTER_AGENT_SERVICE_HOST=10.43.151.237 CATTLE_CLUSTER_AGENT_SERVICE_PORT=80 CATTLE_CLUSTER_AGENT_SERVICE_PORT_HTTP=80 CATTLE_CLUSTER_AGENT_SERVICE_PORT_HTTPS_INTERNAL=443 CATTLE_CLUSTER_REGISTRY=registry.cn-hangzhou.aliyuncs.com CATTLE_CREDENTIAL_NAME=cattle-credentials-576af2aadc CATTLE_FEATURES=embedded-cluster-api=false,fleet=false,managed-system-upgrade-controller=false,multi-cluster-management=false,multi-cluster-management-agent=true,provisioningprebootstrap=false,provisioningv2=false,rke2=false,ui-sql-cache=false CATTLE_INGRESS_IP_DOMAIN=sslip.io CATTLE_INSTALL_UUID=5d07ed58-5bad-4169-b141-c9d121d5263c CATTLE_INTERNAL_ADDRESS= CATTLE_IS_RKE=false CATTLE_K8S_MANAGED=true CATTLE_NODE_NAME=cattle-cluster-agent-65bb6467bc-kvqhh CATTLE_RANCHER_PROVISIONING_CAPI_VERSION=106.0.0+up0.7.0 CATTLE_RANCHER_WEBHOOK_VERSION=106.0.1+up0.7.1 CATTLE_SERVER=https://rancher.tc.org:8443 CATTLE_SERVER_VERSION=v2.11.1
INFO: Using resolv.conf: search cattle-system.svc.cluster.local svc.cluster.local cluster.local nameserver 10.43.0.10 options ndots:5
ERROR: https://rancher.tc.org:8443/ping is not accessible (Could not resolve host: rancher.tc.org)


这个需要把对应的configmap里面的host加上对应的域名解析比如我这个地方要加上

```
kubectl edit configmap rke2-coredns-rke2-coredns -n kube-system
# --- 添加 hosts 插件 ---
        hosts {
          10.111.155.178 rancher.tc.org
          fallthrough
        }

```

后面其他问题在根据具体情况总结，耗时大概用了4到5天。