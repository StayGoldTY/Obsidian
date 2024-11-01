在 CentOS 系统中，如果 Docker 相关文件占用过大，可以通过以下步骤来安全清理 Docker 的文件和日志，从而释放磁盘空间。以下是常见的清理方式：

### 1. 清理无用的容器、镜像和数据卷
Docker 容器、镜像、网络和数据卷在使用过程中可能会积累无用的资源，你可以通过以下命令清理它们。

#### 1.1 停止并移除无用的容器
首先，查看所有容器：
```sh
docker ps -a
```
然后停止并移除所有无用的容器：
```sh
docker container prune
```
这将移除所有处于 `Exited` 状态的容器。

#### 1.2 移除未使用的镜像
有些镜像不再使用，但占用了大量空间，可以通过以下命令移除未使用的镜像：
```sh
docker image prune -a
```
这将移除所有没有被容器使用的镜像。需要注意的是，这可能会使某些需要的镜像重新下载。

#### 1.3 移除未使用的网络
如果 Docker 网络也不再使用，可以通过以下命令移除：
```sh
docker network prune
```

#### 1.4 移除未使用的数据卷
数据卷也会占用大量空间，可以通过以下命令移除未使用的数据卷：
```sh
docker volume prune
```
这将移除所有没有挂载到容器的数据卷。

#### 1.5 全部清理
可以通过以下命令来进行全量清理：
```sh
docker system prune -a --volumes
```
这将清理掉未使用的镜像、未使用的数据卷、容器和网络。

### 2. 清理 Docker 日志
Docker 的容器日志文件也可能会占用大量的磁盘空间，特别是长时间运行的容器。

#### 2.1 检查 Docker 容器日志
Docker 的日志文件通常位于 `/var/lib/docker/containers/{container-id}/*.log`，你可以通过以下命令查看某个日志的大小：
```sh
sudo du -sh /var/lib/docker/containers/*/*.log
```

#### 2.2 清理日志文件
如果某些日志文件占用空间较大，可以清空这些日志：
```sh
sudo truncate -s 0 /var/lib/docker/containers/{container-id}/*.log
```
将 `{container-id}` 替换为实际容器的 ID，这样可以将日志文件清空而不删除文件本身。

#### 2.3 设置日志轮转
可以通过配置 Docker 的守护进程来限制日志文件大小，从而避免日志文件无限增长。

在 `/etc/docker/daemon.json` 中添加以下配置：
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
这将限制每个日志文件的大小为 10MB，并最多保留 3 个轮转日志文件。配置完成后，重新启动 Docker 服务：
```sh
sudo systemctl restart docker
```

### 3. 删除无用的 Docker Builder 缓存
Docker 的 BuildKit 会缓存构建过程中产生的中间镜像层，这些缓存占用空间，可以通过以下命令来清理：
```sh
docker builder prune
```
这将清除构建过程中产生的中间层。

### 4. 检查并删除旧的 Docker 容器卷目录
有时候容器的数据卷残留在 `/var/lib/docker/volumes`，可以使用以下命令来查看哪些卷占用了最多的空间：
```sh
sudo du -sh /var/lib/docker/volumes/*
```
找到不再使用的数据卷后，可以手动删除。

### 注意事项
- **备份数据**：在执行任何删除操作前，确保已经备份了重要的数据，以防止误删。
- **小心使用 `-a` 参数**：`docker system prune -a` 会移除所有未使用的镜像，这可能会导致一些镜像需要重新拉取。
- **适度清理**：清理得太频繁可能导致重复下载镜像，反而浪费网络流量和时间。

这样做可以帮助你有效地减少 Docker 在 CentOS 系统中的磁盘占用。

***
### 问：如何安全的清理所有docker容器的日志
要安全地清理所有 Docker 容器的日志，可以通过以下方法来操作。主要是通过清空日志文件来释放磁盘空间，同时确保容器本身不受影响。以下是安全清理所有 Docker 容器日志的步骤：

### 1. 手动清理所有容器日志
Docker 日志文件通常位于 `/var/lib/docker/containers/{container-id}/*.log`。您可以使用 `truncate` 命令清空所有容器日志文件，而不会影响容器的运行。

1.1 查找所有日志文件路径并逐个清理
```sh
sudo find /var/lib/docker/containers/ -name "*.log" -exec truncate -s 0 {} \;
```
上述命令会递归查找 `/var/lib/docker/containers/` 目录中的所有日志文件 (`*.log`)，并将它们的大小设为 0，从而清空日志内容。

- `find`：查找目录下的所有日志文件
- `truncate -s 0 {}`：将日志文件大小设为 0，相当于清空文件内容

### 2. 使用 Shell 脚本清理日志
如果希望定期执行日志清理操作，可以使用 Shell 脚本。如下的脚本会清理 Docker 所有容器的日志：

```sh
#!/bin/bash
LOG_FILES=$(find /var/lib/docker/containers/ -name "*.log")
for LOG_FILE in $LOG_FILES
do
    truncate -s 0 $LOG_FILE
    echo "Cleared: $LOG_FILE"
done
```
将该脚本保存为 `clear_docker_logs.sh`，然后使用 `chmod` 命令给脚本执行权限：
```sh
chmod +x clear_docker_logs.sh
```
然后可以执行脚本：
```sh
sudo ./clear_docker_logs.sh
```

### 3. 设置 Docker 日志轮转
为了避免日志文件无限制增长，可以通过配置 Docker 的守护进程进行日志轮转设置。这将自动限制日志大小，减少手动清理的频率。

编辑 Docker 的守护进程配置文件 `/etc/docker/daemon.json`，添加以下内容：
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
- `"max-size": "10m"`：将每个日志文件的最大大小限制为 10MB。
- `"max-file": "3"`：最多保留 3 个日志文件（包括历史轮转的日志文件）。

配置完成后，重新启动 Docker 服务以应用更改：
```sh
sudo systemctl restart docker
```
此设置会确保 Docker 自动进行日志轮转，防止日志文件无限制地占用磁盘空间。

### 4. 使用 Docker CLI 清理日志
从 Docker 版本 1.13 开始，可以使用 Docker 的命令行工具来管理日志。如果您希望通过 Docker 命令清理日志，可以使用以下命令。

列出所有容器的 ID：
```sh
docker ps -q
```
然后遍历这些容器并使用 `truncate` 命令清空日志：
```sh
docker ps -q | xargs -I {} sh -c 'truncate -s 0 /var/lib/docker/containers/{}/{}.log'
```
这条命令会将所有容器的日志文件清空。

### 注意事项
1. **不要删除日志文件本身**：Docker 期望日志文件的存在，因此直接删除日志文件可能会导致容器运行出现问题。推荐使用 `truncate` 命令清空日志，而不是删除文件。
2. **生产环境谨慎操作**：在生产环境中进行日志清理时，建议在低峰期进行，并确认备份重要数据，以防影响业务。
3. **定期清理**：通过设置日志轮转以及定期清理日志，可以有效地防止磁盘空间被日志占满。

通过这些方法，您可以安全地清理 Docker 容器的日志，并控制日志文件的增长，确保系统的磁盘空间不会因为 Docker 日志而被耗尽。

***
### 问：我想要每两个星期晚上12点定时执行 docker image prune -a 这个命令，具体要如何处理
要每两个星期晚上12点定时执行 `docker image prune -a` 命令，最简单的方法是使用 Linux 系统的 `cron` 定时任务来安排任务。在 CentOS 上，你可以按照以下步骤来设置这个定时任务：

### 1. 编辑 Crontab 文件
使用 `crontab` 命令来编辑定时任务：
```sh
sudo crontab -e
```
这会打开当前用户的 `crontab` 文件。请注意，如果是清理 Docker 镜像等需要权限的操作，建议使用 `sudo` 编辑 root 用户的 `crontab` 文件，以确保权限问题不会导致任务失败。

### 2. 添加定时任务条目
在 `crontab` 文件中添加如下行，以设置每两个星期的周六午夜 12:00 执行 `docker image prune -a` 命令：

```sh
0 0 */14 * * /usr/bin/docker image prune -a -f
```

- `0 0 */14 * *`：表示每隔 14 天的晚上 12:00 执行任务。
  - `0`：分钟为 0
  - `0`：小时为 0（即午夜 12:00）
  - `*/14`：每隔 14 天
  - `*`：每个月
  - `*`：每周的任何一天

`-f` 参数用于在执行时不需要交互式确认，以便 `cron` 可以自动执行任务。

### 3. 确定 Docker 命令路径
在 `crontab` 中执行命令时需要确保命令的路径正确。通常 `docker` 命令位于 `/usr/bin/docker`，你可以通过以下命令来确认 Docker 命令的路径：

```sh
which docker
```
这条命令会显示 Docker 可执行文件的完整路径，例如 `/usr/bin/docker`。

确保在 `crontab` 中使用的是正确的完整路径，例如 `/usr/bin/docker`。

### 4. 保存并退出
编辑完 `crontab` 文件后，保存并退出。对于大多数 Linux 系统，保存的快捷键是 `Ctrl + O`，然后按 `Enter`，再按 `Ctrl + X` 退出编辑器。

### 5. 验证 Crontab 任务是否添加成功
你可以通过以下命令查看已经添加的 `crontab` 任务：
```sh
sudo crontab -l
```
这将列出所有的 `crontab` 任务，确保你添加的任务已经存在并且格式正确。

### 示例：完整的 `crontab` 条目
```
0 0 */14 * * /usr/bin/docker image prune -a -f
```

### 注意事项
1. **`-f` 参数**：`docker image prune -a` 命令会要求确认删除操作，因此在 `cron` 中执行时必须加上 `-f` 参数以跳过确认。
2. **权限问题**：如果是 root 用户的任务，则可以通过 `sudo crontab -e` 来编辑。如果使用非 root 用户，请确保该用户对 Docker 有足够的权限。
3. **日志记录**：如果想要查看这个任务的执行日志，可以将输出重定向到一个日志文件，例如：
   ```sh
   0 0 */14 * * /usr/bin/docker image prune -a -f >> /var/log/docker-prune.log 2>&1
   ```
   这样可以将任务的输出结果保存在 `/var/log/docker-prune.log` 文件中，以便日后查看任务执行的状态和结果。

这样设置好后，Docker 镜像清理任务将在每两个星期的午夜自动运行，以释放磁盘空间。