在 Rancher 和 Kubernetes 中创建 Ingress 时，如果需要对 HTTP 通信进行加密，您必须提供包含 TLS 私钥和证书的密文。该私钥和证书用于加密和解密通过 Ingress 进行的通信。您可以导航到某个[项目](https://docs.rancher.cn/docs/rancher2.5/cluster-admin/projects-and-namespaces/_index)或命名空间](/docs/rancher2.5/cluster-admin/projects-and-namespaces/_index)，然后上传证书，以使证书可供 Ingress 使用。然后，您可以将证书添加到 Ingress 中。

## 添加证书[#](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/certificates/_index#%E6%B7%BB%E5%8A%A0%E8%AF%81%E4%B9%A6 "Direct link to heading")

您可以将 SSL 证书添加到项目，命名空间或同时添加到两者。如果添加的是项目范围的证书，您将可以在该项目下所有命名空间里使用该证书。

> **先决条件：** 您必须具有 TLS 私钥和证书。

1. 从“全局”视图中，选择要在其中部署 Ingress 的项目。
    
2. 从主菜单中，选择 **资源 > 密文 > 证书列表** 。单击**添加证书**。（对于 v2.3 之前的 Rancher，请单击 **资源 > 证书** 。）
    
3. 输入证书的**名称**。
    
    > **注意：** Kubernetes 将 SSL 证书分类为[secret](https://kubernetes.io/docs/concepts/configuration/secret/)，并且规定命名空间中的两个 secrets 不能具有相同的名称。因此，为防止冲突，请确保您的 SSL 证书与该命名空间中的其他证书，镜像库凭证和密文有不同的名称。
    
4. 选择证书的**作用域**。
    
    - **此项目所有命名空间：** 该证书可用于项目中任何命名空间中的任何工作负载。
        
    - **单个命名空间：** 该证书仅可用于一个命名空间中的工作负载。如果选择此选项，请从下拉列表中选择一个**命名空间**，或单击**创建新的命名空间**以将证书添加到新创建的命名空间中。
        
5. 在**私钥**里，将证书的私钥复制并粘贴到文本框中（包括头部和尾部），或单击**从文件读取**以浏览到文件系统上的私钥。如果可能的话，我们建议使用**从文件读取**，以减少出错的可能性。
    
    私钥文件以 `.key` 扩展名结尾。
    
6. 在**证书**里，将证书复制并粘贴到文本框中（包括头部和尾部），或单击**从文件读取**以浏览到文件系统上的证书。如果可能的话，我们建议使用**从文件读取**，以减少出错的可能性。
    
    证书文件以 `.crt` 扩展名结尾。
    

**结果：** 您的证书已添加到项目或命名空间。现在，您可以将其添加到工作负载中。

- 如果将 SSL 证书添加到项目，则该证书可用于在任何项目命名空间的工作负载。
- 如果您将 SSL 证书添加到命名空间，则该证书仅可用于该命名空间的工作负载。
- 您的证书已添加到 **资源 > 密文 > 证书列表** 视图中。（对于 v2.3 之前的 Rancher，它已添加到 **资源 > 证书** 中。）

## 添加 Ingress[#](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/certificates/_index#%E6%B7%BB%E5%8A%A0-ingress "Direct link to heading")

现在，您可以在当前项目或命名空间中创建 Ingress 时添加证书。有关更多信息，请参阅[添加 Ingress](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/load-balancers-and-ingress/ingress/_index)。