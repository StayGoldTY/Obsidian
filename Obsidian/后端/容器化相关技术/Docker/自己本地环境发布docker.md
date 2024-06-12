# 1.先用vs2022发布对应的程序
 直接右键发布即可，配置一般如下：
 ![[Pasted image 20240422140733.png]]
# 2.构建对应的容器

在publis的目录下面运行
```
docker build -t your-app-name .
```
`docker build -t your-app-name .` 命令中的 `.` 表示在当前目录下构建 Docker 镜像。

***

当您运行这个命令时，Docker 会在当前目录中查找 `Dockerfile` 文件，并使用该文件中的指令来构建镜像。

因此，在运行 `docker build` 命令之前，您需要确保：

1. 您已经导航到包含 `Dockerfile` 文件的目录，通常是您的 .NET Core 项目的根目录。

2. `Dockerfile` 文件存在于当前目录中，并且包含了构建镜像所需的所有指令。

3. 您的 .NET Core 项目的所有必需文件（如源代码、配置文件等）也存在于当前目录或其子目录中，以便 Docker 可以将它们复制到镜像中。

例如，假设您的项目目录结构如下：

```
MyProject/
    ├── MyProject.csproj
    ├── Program.cs
    ├── Dockerfile
    └── ...
```

在这种情况下，您需要在终端或命令提示符下导航到 `MyProject` 目录，然后运行 `docker build -t your-app-name .` 命令。

```bash
cd MyProject
docker build -t your-app-name .
```

这将指示 Docker 在当前目录（`MyProject`）中查找 `Dockerfile` 文件，并使用该文件来构建镜像。

请确保您在正确的目录下运行 `docker build` 命令，以便 Docker 可以找到 `Dockerfile` 文件和项目的其他必需文件。

# 3.运行docker run 命令来运行容器
根据提供的 DaemonSet YAML 文件，可以生成以下 Docker 运行命令：

```bash
docker run -d --name adatpter -p 290:290 -v "C:\Windows\Fonts:/usr/share/fonts" -v "D:\data\files:/app/Files" -v "D:\data\tomcat\webapps\BatchScanSeal\pdf:/opt"  -e BaseDb="Server=192.168.128.146,30755;Database=hnstdb1208;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e ConnStr="Server=192.168.128.146,30755;Database=ZJK_GongGai;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e DestFileRoot="/opt" -e NationwideBuildingMarketBaseDBConnStr="Server=192.168.128.146,30755;Database=NationwideBuildingMarketBaseDB;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e NationwideBuildingMarketBaseDBTmpConnStr="Server=192.168.128.146,30755;Database=NationwideBuildingMarketBaseDBTmp;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e OldSystemStr="Server=192.168.128.146,30755;Database=TestDB;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e REDIS_CONNECTION_STRING="hnstp@127.0.0.1:6379::0"  -e Register_Conn="127.0.0.1:8500" -e Surging_Server_IP="127.0.0.1" -e TengLongDb="Server=192.168.128.146,30755;Database=HN_DataBase;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" --restart always st-server:latest
```

这个 Docker 运行命令包括了以下内容：

- `-d`：在后台运行容器。
- `--name`：指定容器名称为 `adatpter`。
- `-p 290:290`：将容器的 290 端口映射到主机的 290 端口。
- `-v`：挂载宿主机的目录到容器中，包括字体目录、文件目录、PDF 目录和时区目录。
- `-e`：设置容器的环境变量，包括各种数据库连接字符串、Redis 连接字符串等。
- `--restart always`：设置容器在退出时总是自动重启。
- 最后一行指定要运行的镜像为 `registry.cn-shenzhen.aliyuncs.com/lekima/hainan-adapter:latest8`。

请注意，这个命令假设你已经在宿主机上创建了相应的目录，并且镜像已经存在于本地或可以从指定的镜像仓库中拉取。你可以根据实际情况对命令进行调整。

***

问：服务已启动。 按下Ctrl + C关闭。 服务端启动成功，03/01/2024 07:42:33。 Unhandled exception. Autofac.Core.DependencyResolutionException: An exception was thrown while activating λ:System.Object. ---> System.AggregateException: One or more errors occurred. (One or more errors occurred. (Connection refused (127.0.0.1:8500))) ---> System.AggregateException: One or more errors occurred. (Connection refused (127.0.0.1:8500)) ---> System.Net.Http.HttpRequestException: Connection refused (127.0.0.1:8500) ---> System.Net.Sockets.SocketException (111): Connection refused

部署后测试发现，docker连接不了本机的consul和redis，以为docker本身就是一个隔离的环境，要在运行consul的时候加选项或者配置redis.conf，让consul和redis都可以通过网络地址来访问
所以命令修改如下
```
docker run -d --name adatpter -p 290:290 -v "C:\Windows\Fonts:/usr/share/fonts" -v "D:\data\files:/app/Files" -v "D:\data\tomcat\webapps\BatchScanSeal\pdf:/opt"  -e BaseDb="Server=192.168.128.146,30755;Database=hnstdb1208;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e ConnStr="Server=192.168.128.146,30755;Database=ZJK_GongGai;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e DestFileRoot="/opt" -e NationwideBuildingMarketBaseDBConnStr="Server=192.168.128.146,30755;Database=NationwideBuildingMarketBaseDB;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e NationwideBuildingMarketBaseDBTmpConnStr="Server=192.168.128.146,30755;Database=NationwideBuildingMarketBaseDBTmp;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" -e OldSystemStr="Server=192.168.128.146,30755;Database=TestDB;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=Tru e;" -e REDIS_CONNECTION_STRING="hnstp@10.2.101.243:6379::0"  -e Register_Conn="10.2.101.243:8500" -e Surging_Server_IP="0.0.0.0" -e TengLongDb="Server=192.168.128.146,30755;Database=HN_DataBase;User ID=sa;Password=sz&jz@we9Ln;Trusted_Connection=False;MultipleActiveResultSets=true;TrustServerCertificate=True;Encrypt=True;" --restart always st-server:latest
```