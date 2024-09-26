在 Rancher 中，**Ingress** 是 Kubernetes 集群中的一种资源类型，用于管理外部访问集群内服务的方式。Ingress 提供了一个统一的入口点，用来管理 HTTP 和 HTTPS 请求的路由，通过指定规则来将请求转发到集群中的特定服务（Service）。

### Ingress 的作用
- **URL 路由**：根据请求的 URL 路径或主机名，将流量路由到不同的 Kubernetes 服务。
- **TLS/HTTPS**：可以为服务配置 TLS 加密，通过使用证书实现 HTTPS 访问。
- **负载均衡**：可以配置外部访问的负载均衡，将请求分发到多个后端服务。
- **虚拟主机**：支持通过主机名来区分请求（例如：不同的域名指向不同的服务）。

### 例子说明
假设你有一个应用程序由多个服务组成，比如一个网站的前端和后端服务。你希望外部用户通过 `example.com` 访问你的前端，而后端服务则通过特定路径访问。Ingress 可以帮助你实现这个目标。

#### 1. 配置前端服务的 Ingress 规则
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"  # 使用 Nginx 作为 Ingress 控制器
spec:
  rules:
  - host: "example.com"  # 主机名（域名）
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: frontend-service  # 前端服务
            port:
              number: 80
```

这个配置表明：当用户访问 `example.com` 时，将流量转发到集群中的 `frontend-service` 服务上。

#### 2. 配置后端服务的 Ingress 规则
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: backend-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: "example.com"
    http:
      paths:
      - path: "/api"
        pathType: Prefix
        backend:
          service:
            name: backend-service  # 后端服务
            port:
              number: 80
```

这个配置表明：当用户访问 `example.com/api` 时，流量会被路由到 `backend-service` 服务。

### Ingress 控制器
需要注意的是，**Ingress 本身不做任何实际的流量管理**，它依赖于集群中运行的 **Ingress 控制器**（例如 Nginx Ingress Controller、Traefik 等）来处理实际的路由和流量转发。Rancher 中默认可以使用 Nginx 作为 Ingress 控制器。

### 总结
Ingress 是 Kubernetes 中用于管理外部流量进入集群的一种资源，通过定义规则，它能够控制如何将外部 HTTP 或 HTTPS 请求路由到集群中的服务。结合 Ingress 控制器，可以实现域名、路径、负载均衡和 TLS 加密等功能。