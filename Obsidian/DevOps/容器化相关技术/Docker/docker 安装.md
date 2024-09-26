# 下面是以arm64  centos环境安装为例



具体步骤如下:

1. 选择适合Ubuntu系统和架构的Docker tar文件进行下载,例如:

```
https://download.docker.com/linux/static/stable/aarch64/docker-20.10.12.tgz(ARM64)

https://download.docker.com/linux/static/stable/x86_64/docker-20.10.12.tgz(AMD64)
```
![[docker-24.0.7.tgz]]
2. 解压缩tar文件:

```
tar xzvf docker-24.0.7.tgz
```

3. 将解压后的docker目录移动到/usr/bin/下: 

```
sudo mv docker/* /usr/bin/
```

4. 配置docker.service让其能让systemctl来进行管理docker
这个错误提示 docker.service 服务单元文件没有找到,很可能是之前的步骤里面忘记创建了。

解决方法是先确认是否在 /etc/systemd/system/ 目录下创建了 docker.service 文件:

```bash
ls /etc/systemd/system/docker.service
```

如果该文件不存在,就需要回到安装步骤中,创建 systemd service 文件:

```bash
sudo nano /etc/systemd/system/docker.service
```

在文件中填入正确的内容:

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Other options

[Install]
WantedBy=multi-user.target
```

保存退出后,再次尝试启动:

```bash
sudo systemctl start docker
```

现在应该能够正常运行 docker 了。

检查一下服务状态:

```bash
sudo systemctl status docker
```

