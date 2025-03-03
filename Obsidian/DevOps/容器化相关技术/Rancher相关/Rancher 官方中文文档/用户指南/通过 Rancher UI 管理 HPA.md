_自 Rancher v2.3.0 起可用_

Rancher UI 支持创建，管理和删除 HPA。您可以将 CPU 或内存使用率配置为用于 HPA 自动扩缩容所使用的指标。

如果要创建可根据 CPU 和内存以外的其他指标进行扩展的 HPA，请参阅[配置 HPA 以使用 Prometheus 的自定义指标进行扩展](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/horitzontal-pod-autoscaler/manage-hpa-with-kubectl/_index)。

## 创建一个 HPA[#](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/horitzontal-pod-autoscaler/manage-hpa-with-rancher-ui/_index#%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA-hpa "Direct link to heading")

1. 从**全局**视图中，打开一个项目，将 HPA 部署到此项目中。
2. 单击**资源 > HPA**。
3. 单击**添加 HPA**。
4. 输入 HPA 的**名称**。
5. 为 HPA 选择一个**命名空间**。
6. 选择**工作负载**作为 HPA 的扩展目标。
7. 为 HPA 指定**最小比例**和**最大比例**。
8. 配置 HPA 的指标。您可以选择内存或 CPU 使用率作为度量标准，这将触发 HPA 弹性扩缩容。在**数量**字段中，输入将触发 HPA 扩缩容机制的工作负载内存或 CPU 使用率的百分比。要配置其他 HPA 指标，包括 Prometheus 可用的指标，您需要[使用 kubectl 管理 HPA](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/horitzontal-pod-autoscaler/manage-hpa-with-kubectl/_index)。
9. 单击**创建**以创建 HPA。

> **结果：** HPA 已部署到选定的命名空间。您可以从项目的资源> HPA 视图查看 HPA 的状态。

## 获取 HPA 指标和状态[#](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/horitzontal-pod-autoscaler/manage-hpa-with-rancher-ui/_index#%E8%8E%B7%E5%8F%96-hpa-%E6%8C%87%E6%A0%87%E5%92%8C%E7%8A%B6%E6%80%81 "Direct link to heading")

1. 从**全局**视图中，打开要查看的 HPA 资源所在的项目。
2. 单击**资源 > HPA**。**HPA**选项卡显示当前副本数。
3. 有关特定 HPA 的更多详细指标和状态，请单击 HPA 的名称。将会进入 HPA 详细信息页面。

## 删除 HPA[#](https://docs.rancher.cn/docs/rancher2.5/k8s-in-rancher/horitzontal-pod-autoscaler/manage-hpa-with-rancher-ui/_index#%E5%88%A0%E9%99%A4-hpa "Direct link to heading")

1. 从**全局**视图中，打开要从中删除 HPA 的项目。
2. 单击**资源> HPA”**。
3. 找到要删除的 HPA。
4. 单击**省略号（...）>删除**。
5. 单击**删除**以确认。

> **结果：** HPA 已从当前集群中删除。