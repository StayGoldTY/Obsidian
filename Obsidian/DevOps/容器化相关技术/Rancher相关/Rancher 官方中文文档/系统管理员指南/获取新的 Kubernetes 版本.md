_从 2.3.0 版起可用_

RKE 元数据功能允许您在发布新版本的 Kubernetes 后，无需升级 Rancher，立即为集群配置元数据。此功能对于使用 Kubernetes 的补丁版本非常有用，例如，如果您希望在仅支持 Kubernetes v1.14.6 的 Rancher Server 版本中，将业务集群升级到 Kubernetes v1.14.7。

> **注意：** Kubernetes API 可能在次要版本之间发生变化。因此，我们不支持获取 Kubernetes 次要版本，例如在 Rancher Server 当前仅支持 v1.14。如果想要使用 Kubernetes v1.15。您需要升级 Rancher Server 以添加对新的次要 Kubernetes 版本的支持。

Rancher 的 Kubernetes 元数据包含了 Rancher 配置 [RKE 集群](https://docs.rancher.cn/docs/rancher2.5/cluster-provisioning/rke-clusters/_index)时可以使用的 Kubernetes 版本信息。Rancher 定期同步数据，并为**系统镜像** **服务选项**和**插件模板**创建自定义资源定义（CRD）。因此，当新的 Kubernetes 版本与 Rancher Server 版本兼容时，Kubernetes 元数据使 Rancher 可以使用新的 Kubernetes 版本来配置集群。元数据概述了 [Rancher Kubernetes Engine](https://docs.rancher.cn/docs/rke/_index)（RKE）用于部署各种 Kubernetes 版本的信息。

下表描述了受定期数据同步影响的 CRD。

> **注意：** 只有管理员可以编辑元数据 CRD。除非明确需要，否则建议不要更新现有对象。

|资源|描述|RancherAPI URL|
|:--|:--|:--|
|系统镜像|用于通过 RKE 部署 Kubernetes 集群的系统镜像列表。|`<RANCHER_SERVER_URL>/v3/rkek8ssystemimages`|
|服务选项|传递给 Kubernetes 组件的默认参数，如`kube-api`、`scheduler`、`kubelet`、`kube-proxy`和`kube-controller-manager`等|`<RANCHER_SERVER_URL>/v3/rkek8sserviceoptions`|
|插件模板|用于部署插件组件的 YAML 定义，如 Canal、Calico、Flannel、Weave、Kube-dns、CoreDNS、`metrics-server`、`nginx-ingress`|`<RANCHER_SERVER_URL>/v3/rkeaddons`|

管理员可以将 RKE 元数据设置配置为执行以下操作：

- 刷新 Kubernetes 元数据，如果一个新的 Kubernetes 补丁版本出来了，用户希望 Rancher 在不升级 Rancher 的情况下为集群提供最新版本的 Kubernetes。
- 更改 Rancher 用于同步元数据的 URL，如果需要让 Rancher 从本地的端点同步而不是与 GitHub 同步，这对于离线环境非常有用。
- 防止 Rancher 自动同步元数据，这是禁止 Rancher 中使用新的和不受支持的 Kubernetes 版本的一种方法。

## 刷新 Kubernetes 元数据[#](https://docs.rancher.cn/docs/rancher2.5/admin-settings/k8s-metadata/_index#%E5%88%B7%E6%96%B0-kubernetes-%E5%85%83%E6%95%B0%E6%8D%AE "Direct link to heading")

默认情况下，刷新 Kubernetes 元数据的选项可供系统管理员使用，也可供具有**管理集群驱动**[全局权限](https://docs.rancher.cn/docs/rancher2.5/admin-settings/rbac/global-permissions/_index)的任何用户使用。

要强制 Rancher 刷新 Kubernetes 元数据，可以在**工具 > 驱动管理 > 刷新 Kubernetes 元数据**下执行手动刷新操作。

您可以通过将 `refresh-interval-minutes`设置为 `0`（见下文的代码示例），将 Rancher 配置为只在需要时刷新元数据，并在需要时使用该按钮手动执行元数据刷新。

## 配置元数据同步[#](https://docs.rancher.cn/docs/rancher2.5/admin-settings/k8s-metadata/_index#%E9%85%8D%E7%BD%AE%E5%85%83%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5 "Direct link to heading")

> 只有管理员可以更改这些设置。

RKE 元数据配置控制 Rancher 同步元数据的频率以及从何处下载数据。您可以从 Rancher UI 中的设置或通过 API `v3/settings/rke-metadata-config` 配置元数据。

元数据的配置方式取决于 Rancher 版本。

- Rancher v2.4+
- Rancher v2.3

1. 转到**全局**视图并单击**系统设置**选项卡。
    
2. 转到**rke-metadata-config**部分。单击**省略号(…)**并单击**升级**
    
3. 您可以选择填写以下参数：
    
    - `refresh-interval-minutes`：这是 Rancher 等待同步元数据的时间。若要禁用定期刷新，请将`refresh-interval-minutes`设置为 0。
    - `url`： 这是 Rancher 从中获取数据的 HTTP 路径。该路径必须是 JSON 文件的直接路径。例如，Rancher v2.4 的默认 URL 是`https://releases.rancher.com/kontainer-driver-metadata/release-v2.4/data.json`。

如果没有离线环境，则无需指定 Rancher 获取元数据的 URL，因为默认设置是从 [Rancher 的元数据存储库中提取](https://releases.rancher.com/kontainer-driver-metadata/release-v2.4/data.json)。

但是，如果您有[离线环境](https://docs.rancher.cn/docs/rancher2.5/admin-settings/k8s-metadata/_index#%E7%A6%BB%E7%BA%BF%E7%8E%AF%E5%A2%83)需求，则需要将 Kubernetes 元数据仓库镜像到 Rancher 可用的位置。然后，您需要更改 URL 以指向 JSON 文件的新位置。

## 离线环境[#](https://docs.rancher.cn/docs/rancher2.5/admin-settings/k8s-metadata/_index#%E7%A6%BB%E7%BA%BF%E7%8E%AF%E5%A2%83 "Direct link to heading")

Rancher Server 会定期刷新并下载`rke-metadata-config`中配置定元数据。如果新的元数据中包含当前 Rancher Server 版本支持的新的 Kubernetes 版本元数据。则用户可以在不升级 Rancher 的情况下，开始使用这些新的 Kubernetes 版本。有关 Kubernetes 和 Rancher 版本的兼容性表，请参阅[服务条款](https://rancher.com/support-maintenance-terms/all-supported-versions/)。

如果您有一个离线环境，则可能无法从 Rancher 的 Git 代码库中自动定期刷新 Kubernetes 元数据。在这种情况下，应该禁用定期刷新以防止在日志中显示相关错误。或者，您可以配置`rke-metadata-config`，以便 Rancher 可以与 RKE 元数据的本地副本同步。

若要将 Rancher 与 RKE 元数据的本地镜像同步，管理员将通过更新`rke-metadata-config`以指向镜像的仓库。请参见[配置元数据同步](https://docs.rancher.cn/docs/rancher2.5/admin-settings/k8s-metadata/_index#%E9%85%8D%E7%BD%AE%E5%85%83%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5)。

在将新的 Kubernetes 版本加载到 Rancher Server 中之后，需要执行其他步骤才能使用它们启动集群。Rancher 需要访问更新的系统镜像。虽然元数据设置只能由系统管理员更改，但任何用户都可以下载 Rancher 系统镜像并为它们准备一个私有镜像仓库。

1. 要下载私有镜像仓库所需的镜像，请单击 Rancher UI 左下角的 Rancher Server 版本。
2. 下载 Linux 或 Windows 操作系统的特定镜像列表。
3. 下载`rancher-images.txt`。
4. 使用与[离线安装](https://docs.rancher.cn/docs/rancher2.5/installation/other-installation-methods/air-gap/populate-private-registry/_index)过程中相同的步骤准备私有镜像仓库，但不要使用发布公告页面中的`rancher-images.txt`，而是使用从前面步骤获得的镜像列表。

**结果：** Rancher 的离线安装现在可以同步 Kubernetes 元数据了。如果在新版本的 Kubernetes 发布后，更新了本地的 RKE 元数据 Git 库，并且在私有镜像仓库中同步了新的镜像，那么您可以在无需升级 Rancher Server 版本的情况下使用新版本的 Kubernetes 配置集群。