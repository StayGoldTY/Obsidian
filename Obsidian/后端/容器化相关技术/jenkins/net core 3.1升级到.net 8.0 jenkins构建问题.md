问：jenkins构建的时候报错 /var/jenkins_home/.nuget/packages/microsoft.extensions.configuration.binder/8.0.1/buildTransitive/netstandard2.0/Microsoft.Extensions.Configuration.Binder.targets(28,9): warning : Microsoft.Extensions.Configuration.Binder doesn't support netcoreapp3.1 and has not been tested with it. Consider upgrading your TargetFramework to net6.0 or later. You may also set <SuppressTfmSupportBuildWarnings>true</SuppressTfmSupportBuildWarnings> in the project file to ignore this warning and attempt to run in this unsupported configuration at your own risk. [/var/jenkins_home/workspace/stgate/Surging.ApiGateway/Surging.ApiGateway.csproj] /var/jenkins_home/tools/io.jenkins.plugins.dotnet.DotNetSDK/net31/sdk/3.1.416/Sdks/Microsoft.NET.Sdk/targets/Microsoft.PackageDependencyResolution.targets(241,5): error NETSDK1005: Assets file '/var/jenkins_home/workspace/stgate/Surging.ApiGateway/obj/project.assets.json' doesn't have a target for '.NETCoreApp,Version=v3.1'. Ensure that restore has run and that you have included 'netcoreapp3.1' in the TargetFrameworks for your project. [/var/jenkins_home/workspace/stgate/Surging.ApiGateway/Surging.ApiGateway.csproj] Build step 'Execute shell' marked build as failure Finished: FAILURE

发现是自己构建目录中对于的.net 8.0的sdk缺失，理论上可以用jenKins全部配置工具安装的，但是我安装了几次都不成功，后面自己现在解压到服务器里面来安装的

***
问：dockerfile里面 RUN apt-get update 这一步构建很慢，大概二三十分钟才能构建完成，完全没有用到之前配置的，类似阿里云的镜像地址，我猜测是原镜像缓存了.net core 8.0sdk后，和配置的不匹配，理论上要一起换到合适8.0的地址
```
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging. FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build #FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-bionic #FROM mcr.microsoft.com/dotnet/core/sdk:3.1 WORKDIR /app COPY . . RUN echo "deb http://mirrors.aliyun.com/debian buster main contrib non-free \ deb-src http://mirrors.aliyun.com/debian buster main contrib non-free \ deb http://mirrors.aliyun.com/debian buster-updates main contrib non-free \ deb-src http://mirrors.aliyun.com/debian buster-updates main contrib non-free \ deb http://mirrors.aliyun.com/debian-security buster/updates main contrib non-free \ deb-src http://mirrors.aliyun.com/debian-security buster/updates main contrib non-free \ deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib \ deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib" >
```

后面构建完成后发现报错
```
cs:line 84 fail: Surging.Core.CPlatform.Runtime.Client.Implementation.RemoteInvokeService[0] 2024-03-01T15:20:22.157383862Z 发起请求中发生了错误，服务Id：HAINAN.IModules.Adaptee.CommonService.IAccountService.randomImage_key。错误信息：The type initializer for 'Gdip' threw an exception.|InnerException:System.Drawing.Common is not supported on non-Windows platforms. See https://aka.ms/systemdrawingnonwindows for more information. Surging.Core.CPlatform.Exceptions.CPlatformCommunicationException: The type initializer for 'Gdip' threw an exception.|InnerException:System.Drawing.Common is not supported on non-Windows platforms. See https://aka.ms/systemdrawingnonwindows for more information.
```

你遇到的错误是因为 `System.Drawing.Common` 在非 Windows 平台上的限制。从 .NET Core 3.0 开始，`System.Drawing.Common` 依赖于本地库 `libgdiplus`，但它在 Linux 和 macOS 上的支持有限。从 .NET 6 开始，默认情况下，`System.Drawing.Common` 只在 Windows 上受支持。尝试在 Linux 或 macOS 上使用可能会导致异常，特别是如果未正确安装或配置 `libgdiplus`。

错误消息 "The type initializer for 'Gdip' threw an exception." 通常表示 GDI+ 的初始化失败，而 "System.Drawing.Common is not supported on non-Windows platforms" 表明你正在非 Windows 平台上遇到此问题。

### 解决方案

#### 1. 在 Linux 上安装 libgdiplus
如果你的应用在 Linux 上运行，确保已经安装了 `libgdiplus`。你可以通过包管理器安装它。例如，在基于 Debian 的系统（如 Ubuntu）上，你可以使用以下命令：

```bash
apt-get update
apt-get install -y libgdiplus
```

在 Dockerfile 中，确保在运行应用之前添加这些命令。

#### 2. 使用兼容的库
如果可能，考虑使用其他兼容非 Windows 平台的库，如 `ImageSharp`，这是一个跨平台的完全托管图像处理库。

#### 3. 设置运行时配置
对于 .NET 6 或更高版本的应用，如果你需要在非 Windows 平台上使用 `System.Drawing.Common`，可以通过项目文件或运行时配置文件来配置运行时以支持它。在你的项目文件中，添加以下运行时配置：

```xml
<ItemGroup>
  <RuntimeHostConfigurationOption Include="System.Drawing.EnableUnixSupport" Value="true" />
</ItemGroup>
```

这告诉运行时在 Unix 系统上启用 `System.Drawing` 的支持。

### 注意
即使在安装了 `libgdiplus` 之后，`System.Drawing.Common` 在非 Windows 平台上的某些功能仍然可能受限，或者表现不同。测试应用的相关功能以确保它们符合你的预期。

更多信息和解决方案可以在官方文档 [System.Drawing.Common not supported on non-Windows platforms](https://aka.ms/systemdrawingnonwindows) 中找到。


就是因为要适配`System.Drawing` 所以才加上构建那么慢的几个配置，而且显示构建成功，其实那几个配置还是没有正确配置过去，只是系统自动跳过了。
所以我直接在代码里面换掉了使用`System.Drawing`的部分代码用了其他兼容linux的nuget包来解决，让构建速度加快了很多

***

构建成功后到服务器运行发现第三个问题
```
Surging.Core.CPlatform.Runtime.Server.Implementation.DefaultServiceExecutor[0] 2024-03-02T04:36:00.343171053Z 执行本地逻辑时候发生了错误。 SixLabors.Fonts.FontException: Cannot use the default value type instance to create a font. 2024-03-02T04:36:00.343181530Z at SixLabors.Fonts.FontsThrowHelper.ThrowDefaultInstance() 2024-03-02T04:36:00.343185489Z at SixLabors.Fonts.FontFamily.CreateFont(Single size, FontStyle style) at Common.Lib.Helper.RandImageHelper.generate(String charCode, Int32 length) in /var/jenkins_home/workspace/hainan-serveradapter/Common/Common.Lib/Helper/RandomImageHelper.cs:line 79 2024-03-02T04:36:00.343193197Z at HAINAN.Modules.Adaptee.CommonService.AccountService.randomImage(String key) in /var/jenkins_home/workspace/hainan-serveradapter/Modules/HAINAN.Modules.Adaptee/CommonService/AccountService.cs:line 924 at (Object, Object[])
```
容器里面没有自带的字体，只能把系统的字体映射到容器里面来
/usr/share/fonts  映射到容器同样路径下面