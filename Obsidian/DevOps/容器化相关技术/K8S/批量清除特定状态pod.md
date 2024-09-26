问：如何能批量清除kubectl get pod --all-namespaces 里面状态是evicted或者error的pod

## 回答

要批量清除状态为 `Evicted` 或 `Error` 的 Pod，可以结合 `kubectl get pod` 和 `kubectl delete pod` 命令，并通过 `grep`、`awk` 等命令来筛选 Pod 的状态。

以下是清理状态为 `Evicted` 或 `Error` 的 Pod 的步骤：

### 1. 列出所有命名空间中状态为 `Evicted` 或 `Error` 的 Pod：
你可以先执行以下命令，查看状态为 `Evicted` 或 `Error` 的 Pod：

```bash
kubectl get pod --all-namespaces --field-selector=status.phase=Failed
```

### 2. 批量删除状态为 `Evicted` 或 `Error` 的 Pod：
通过结合 `grep` 和 `awk` 来筛选 Pod 名称并删除这些 Pod。

#### 方式一：删除 `Evicted` 的 Pod
```bash
kubectl get pod --all-namespaces --field-selector=status.phase=Failed | grep Evicted | awk '{print $2 " -n " $1}' | xargs -I {} kubectl delete pod {}
```

#### 方式二：删除 `Error` 的 Pod
```bash
kubectl get pod --all-namespaces --field-selector=status.phase=Failed | grep Error | awk '{print $2 " -n " $1}' | xargs -I {} kubectl delete pod {}
```

#### 方式三：同时删除 `Evicted` 和 `Error` 的 Pod
你也可以将 `Evicted` 和 `Error` 状态的 Pod 一起处理：

```bash
kubectl get pod --all-namespaces --field-selector=status.phase=Failed | grep -E "Evicted|Error" | awk '{print $2 " -n " $1}' | xargs -I {} kubectl delete pod {}
```

### 说明：
- `kubectl get pod --all-namespaces --field-selector=status.phase=Failed`：列出所有命名空间中状态为 `Failed` 的 Pod。
- `grep Evicted` 或 `grep Error`：筛选出状态为 `Evicted` 或 `Error` 的 Pod。
- `awk '{print $2 " -n " $1}'`：提取命名空间和 Pod 名称（`$1` 是命名空间，`$2` 是 Pod 名称）。
- `xargs -I {} kubectl delete pod {}`：批量删除筛选出的 Pod。

通过这种方式，你可以批量删除所有状态为 `Evicted` 或 `Error` 的 Pod。

***
问：我是在rancher里面的kubectl命令行执行的，执行后如下提示 kubectl get pod --all-namespaces --field-selector=status.phase=Failed | grep -E "Evicted|Error" | awk '{print $2 " -n " $1}' | xargs -I {} kubectl delete pod {} bash: awk: command not found

## 回答
由于 Rancher 环境中没有安装 `awk`，我们可以避免使用它，并直接通过 `kubectl` 的内置功能来完成删除操作。

可以使用 `grep` 和 `cut` 来替代 `awk`，实现批量删除 `Evicted` 或 `Error` 状态的 Pod。以下是修改后的命令：

### 1. 删除 `Evicted` 状态的 Pod

```bash
kubectl get pod --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{" -n "}{.metadata.namespace}{" "}{.status.reason}{"\n"}{end}' | grep Evicted | cut -d' ' -f1,2 | xargs -I {} kubectl delete pod {}
```

### 2. 删除 `Error` 状态的 Pod

```bash
kubectl get pod --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{" -n "}{.metadata.namespace}{" "}{.status.reason}{"\n"}{end}' | grep Error | cut -d' ' -f1,2 | xargs -I {} kubectl delete pod {}
```

### 3. 同时删除 `Evicted` 和 `Error` 状态的 Pod

```bash
kubectl get pod --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{" -n "}{.metadata.namespace}{" "}{.status.reason}{"\n"}{end}' | grep -E "Evicted|Error" | cut -d' ' -f1,2 | xargs -I {} kubectl delete pod {}
```

### 解释：
- `grep Evicted`：筛选出状态为 `Evicted` 的 Pod。
- `cut -d' ' -f1,2`：使用 `cut` 提取 Pod 名称和命名空间（第1列和第2列）。
- `xargs -I {} kubectl delete pod {}`：删除筛选出的 Pod。

这种方法避免使用 `awk`，仅依赖 `grep` 和 `cut`，在 Rancher 的 `kubectl` 命令行环境下也能正常工作。

***
执行上面的命令报错
```
kubectl get pod --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{" -n "}{.metadata.namespace}{" "}{.status.reason}{"\n"}{end}' | grep -E "Evicted|Error" | cut -d' ' -f1,2 | xargs -I {} kubectl delete pod {} Error from server (NotFound): pods "pushprox-kube-controller-manager-proxy-64f6dc94c6-ckf6r -n" not found Error from server (NotFound): pods "pushprox-kube-controller-manager-proxy-64f6dc94c6-dq98f -n" not found Error from server (NotFound): pods "pushprox-kube-etcd-proxy-55544d768d-gfld4 -n" not found Error from server (NotFound): pods "pushprox-kube-etcd-proxy-55544d768d-mgkq5 -n" not found Error from server (NotFound): pods "pushprox-kube-proxy-proxy-85f89bcc4d-l6sz8 -n" not found
```

## 回答

从错误信息来看，命令的输出在 `-n` 命名空间部分出现了问题，导致 `kubectl delete pod` 识别出错。这个错误是由于 Pod 名称和命名空间没有正确传递给 `kubectl delete pod`。

问题的根源可能是 `cut` 或输出处理时未能正确拆分 Pod 名称和命名空间。可以通过调整 `jsonpath` 的输出格式，确保 `-n <namespace>` 的格式正确传递给删除命令。

### 修正后的命令

```bash
kubectl get pod --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.metadata.namespace}{" "}{.status.reason}{"\n"}{end}' | grep -E "Evicted|Error" | while read pod namespace _; do kubectl delete pod "$pod" -n "$namespace"; done
```

### 解释：
1. `kubectl get pod --all-namespaces -o jsonpath=...`：获取所有 Pod 的名称、命名空间和状态。
2. `grep -E "Evicted|Error"`：过滤出状态为 `Evicted` 或 `Error` 的 Pod。
3. `while read pod namespace _`：使用 `while` 循环逐行读取 Pod 名称和命名空间，并忽略第三个字段（即 `reason`）。
4. `kubectl delete pod "$pod" -n "$namespace"`：正确地删除 Pod，确保 `-n` 参数和命名空间被正确传递。

这个命令确保了 Pod 名称和命名空间的正确传递，避免了之前的问题。