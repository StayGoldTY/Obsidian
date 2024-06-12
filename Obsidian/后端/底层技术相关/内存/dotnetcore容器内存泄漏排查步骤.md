# 1.进入容器
```
docker exec -it -u root /bin/bash
```

# 2.在容器里面安装相关的工具
理论上用dotnet-dump是最便捷的，但是我的容器里面通过dotnet-dump collect -p <process_id> 一直报错提示权限问题，所以不用这个，用下面的这种（这里有一个窍门就是一般主进程的pid是 1）
```
--先用下面的命令看能不能找到对应的路径
find / -name createdump
/usr/share/dotnet/shared/Microsoft.NETCore.App/8.0.4/createdump
找到路径后直接通过命令存储到指定路径
/usr/share/dotnet/shared/Microsoft.NETCore.App/8.0.4/createdump -f /app/Files/dump00403.dmp 1
```
# 3.得到转存文件后分析(服务器上面)
```
--通过dotnet-dump来分析文件
--安装dotnet-dump
dotnet tool install --global dotnet-dump
--先把对应的工具添加到环境变量中来
export PATH=$PATH:/root/.dotnet/tools
--进入分析模式
dotnet-dump analyze /app/Files/dump0414.dmp
--进一步分析
用上面的命令能得到详细的内存泄漏的完整的content记录
 db 7ea7e6357928 -c 65536

--类似下面的命令可以把gcroot的结果输出到本地文件来
dotnet-dump analyze D:\dump\dump0506.dmp --command "gcroot 7f8ef3dca410" > D:\dump\gcroot_output.txt

```

# 4.进一步分析
```
通过dumpheap stat得到如下结果
7efe7badbe78   100,676  16,108,160 Castle.Proxies.WorkConclusionModelProxy
7efe73ca1920   124,628  24,925,600 HAINAN.IModules.Adaptee.Model.Org.PersonalUserModel
7efe7669dca0   385,250  27,738,000 Microsoft.EntityFrameworkCore.Infrastructure.Internal.LazyLoader
7efe7752c098     3,707  29,200,288 System.Collections.Generic.Dictionary<Microsoft.Extensions.DependencyInjection.ServiceLookup.ServiceCacheKey, System.Object>+Entry[]
7efe6d6b89e0   488,504  30,928,952 System.Int32[]
7efe73e39460   376,195  36,114,720 Microsoft.EntityFrameworkCore.ChangeTracking.Internal.InternalEntityEntry
7efe6d7963b8     5,375  38,731,046 System.Char[]
7efe6e55cce0    89,345 176,101,584 System.Byte[]
7efe6d6bd7c8 5,041,634 273,120,470 System.String
Total 12,489,103 objects, 1,069,157,434 bytes

Fragmented blocks larger than 0.5 MB:
         Address           Size      Followed By
    7e80d1469a10        958,008     7e80d1553848 HAINAN.DTO.System.SysLogParam 

```

```
dumpheap -type System.String或者 dumpheap -mt
当使用 dumpheap -mt 命令查看某个类型的对象时,如果该类型的对象数量非常多,可以添加额外的过滤条件来缩小结果范围,以便更好地分析问题。以下是一些常用的过滤方式:

1. 根据对象大小过滤:
   dumpheap -mt 7efe6d6bd7c8 -min 1000
   这个命令会只显示大小大于等于1000字节的对象。可以根据需要调整 -min 后面的大小阈值。

2. 根据对象地址范围过滤:
   dumpheap -mt 7efe6d6bd7c8 -startAtLowerBound 0x7efe6d000000 -endAtUpperBound 0x7efe6dffffff
   这个命令会只显示地址在 0x7efe6d000000 到 0x7efe6dffffff 之间的对象。可以根据实际情况调整起始和结束地址。

3. 根据年龄段过滤:
   dumpheap -mt 7efe6d6bd7c8 -generations 2
   这个命令会只显示第2代及以上的对象。可以将 -generations 后的数字改为0、1、2、3等,分别对应不同的GC代。

4. 根据引用类型过滤:
   dumpheap -mt 7efe6d6bd7c8 -type System.Int32[]
   这个命令会只显示 System.String 类型中包含 System.Int32[] 类型引用的对象。可以将 -type 后面的类型改为其他感兴趣的类型。

可以组合使用上述过滤条件,例如:
dumpheap -mt 7efe6d6bd7c8 -min 1000 -generations 2 -type System.Int32[]
这个命令会显示第2代及以上、大小大于等于1000字节、包含 System.Int32[] 引用的 System.String 对象。

除了 dumpheap 命令外,还可以使用 gcroot 命令来查看对象的引用链,例如:
gcroot -all 0x7efe6d8f5678
这个命令会显示地址为 0x7efe6d8f5678 的对象的所有引用链。

通过合理使用过滤条件和其他命令,可以更精准地分析内存中的对象,找出可能存在问题的地方。
```

在使用 `dumpheap -stat` 命令输出的结果中，"方法表地址"（Method Table, 简称 MT）指的是输出结果中的第一列内容。

当你执行 `dumpheap -stat` 命令时，会得到一个表格，这个表格通常包括以下几列：

1. **MT（Method Table）**：这是每个类型的方法表地址，用于标识CLR中的类型信息。这个地址是唯一的，对于每种类型来说都不相同。这就是我们所说的"方法表地址"。
    
2. **Count**：表示堆中该类型对象的实例数量。
    
3. **TotalSize**：表示堆中该类型所有对象实例所占用的总内存大小。
    
4. **ClassName**：表示对象的类型名称。
    

例如，如果你看到这样的输出：

`MT        Count    TotalSize ClassName 00007ff9a3b416f0  500       8000      System.String`

这里，`00007ff9a3b416f0` 就是 `System.String` 类型的方法表地址。当你想要使用 `dumpheap -mt` 命令来列出特定类型的所有对象实例时，就需要使用这个地址作为参数。例如：

shell

`> dumpheap -mt 00007ff9a3b416f0`

这个命令会列出堆中所有 `System.String` 类型的对象实例。之后，如果你想进一步分析某个特定实例，可以从这个命令的输出中获取对象的地址，然后使用 `gcroot` 命令来查找该对象的根引用。
当你使用 `dumpheap -mt` 命令列出了特定类型的所有对象实例后，你会得到一个包含对象地址的列表。这个列表通常包含以下几列：

1. **Address**：对象的内存地址，这是你将用于 `gcroot` 命令的参数。
    
2. **MT**：对象的方法表地址，与 `dumpheap -stat` 输出中的方法表地址相同。
    
3. **Size**：对象所占用的内存大小。
    
4. **ClassName**：对象的类型名称。
    

例如，如果你执行了以下命令：

shell

`> dumpheap -mt 00007ff9a3b416f0`

你可能会得到类似这样的输出：

`Address       MT            Size     ClassName 0000020c7b3ed8a8 00007ff9a3b416f0  84       System.String 0000020c7b3ed8b0 00007ff9a3b416f0  24       System.String ...`

在这个例子中，`0000020c7b3ed8a8` 是一个 `System.String` 对象的内存地址。
我们已经知道如何使用 `dumpheap -mt` 命令来列出特定类型的所有对象实例，并且知道了如何从输出中找到对象的地址。下一步，我们将使用 `gcroot` 命令来查找对象的根引用，以帮助我们理解为什么这些对象没有被垃圾回收器（GC）回收，这对于诊断内存泄漏问题非常重要。

### 使用 `gcroot` 命令查找对象的根引用

1. **选择对象地址**：从 `dumpheap -mt` 命令的输出中，选择你想进一步分析的对象实例的地址。
    
2. **执行 `gcroot` 命令**：使用 `gcroot <对象地址>` 命令来查找该对象的根引用。这个命令会显示从根到该对象的所有引用路径，帮助你理解为什么该对象还在内存中。
    
3. **分析 `gcroot` 输出**：`gcroot` 命令的输出可能会显示多个引用路径。每个路径都表示从一个GC根对象到目标对象的引用链。GC根对象通常包括静态变量、本地变量、线程栈上的对象等。
    
4. **识别潜在的内存泄漏**：通过分析引用路径，你可以识别出哪些对象或引用导致了对象无法被GC回收。例如，一个静态集合可能持有对象的引用，导致这些对象即使不再需要也无法被回收。
    

### 示例

假设我们从 `dumpheap -mt` 命令的输出中选择了一个 `System.String` 对象的地址为 `0000020c7b3ed8a8`，我们可以这样使用 `gcroot` 命令：

shell

`> gcroot 0000020c7b3ed8a8`

假设输出显示这个字符串对象被一个静态字段引用，那么这可能是一个内存泄漏的线索。静态字段生命周期很长，如果它们持有不再需要的对象引用，那么这些对象就不会被GC回收。

### 总结

通过 `dumpheap -mt` 和 `gcroot` 命令的结合使用，我们可以有效地诊断和分析.NET应用程序中的内存泄漏问题。首先，使用 `dumpheap -mt` 命令找到占用内存较多的对象类型和实例，然后使用 `gcroot` 命令查找这些对象的根引用，从而理解为什么这些对象没有被垃圾回收器回收。这种方法可以帮助开发者定位内存泄漏的原因，并采取相应的措施来解决问题。