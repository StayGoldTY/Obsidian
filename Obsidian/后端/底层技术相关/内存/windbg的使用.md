### 1. 安装和配置 WinDbg

首先，确保已经安装了 WinDbg。它通常随 Windows SDK 一起提供，可以从 Microsoft 官方网站下载。

### 2. 安装 .NET Core 调试工具

为了使 WinDbg 能够理解 .NET Core 的内部结构，需要安装 `dotnet-sos` 工具。这可以通过以下命令完成：

```
dotnet tool install -g dotnet-sos
dotnet-sos install
```

这将安装 SOS 调试扩展，它是专门为 .NET Core 和 .NET 5+ 设计的。

### 3. 启动 WinDbg 并加载转储文件


### 4. 配置符号路径(这一步实际测试好像不用，可能是反编译的时候需要)

为了正确解析 .NET Core 应用程序的符号，需要配置符号路径。在 WinDbg 的命令行中输入：

plaintext

`.sympath+ SRV*你的符号存储路径*http://msdl.microsoft.com/download/symbols`

这样配置后，WinDbg 将使用 Microsoft 的符号服务器，并将符号缓存到本地指定的目录。

###  5.加载 SOS 调试扩展

在 WinDbg 中，加载 SOS 调试扩展以便能够调试 .NET Core 应用程序。输入以下命令：



`.load C:\Users\你的用户名\.dotnet\sos\sos.dll`
这个地方注意下面的路径名称你安装sos文件的时候一般会给你提示，如果不知道的话，可以用命令查找，一般是上面的标准命令，用户名换成自己的
请根据您的实际安装路径调整上述命令。

### 6. 开始调试

现在，您可以使用 SOS 提供的各种命令来分析和调试 .NET Core 转储文件。一些常用的命令包括：

- `!dumpheap -stat`：显示所有托管对象的统计信息。
- `!clrstack`：显示当前托管线程的调用堆栈。
- `!pe`：显示最近的异常对象。