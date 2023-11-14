问题一、如果已经在windows中开启了Hyper-V，启动Docker QuickStart Terminal的时候就会出现。

Running pre-create checks...
Errorwithpre-create check:"This computer is running Hyper-V. VirtualBox won't boot a 64bits VM when Hyper-V is activated. Either use Hyper-V as a driver, or disable the Hyper-V hypervisor. (To skip this check, use --virtualbox-no-vtx-check)"
Looks like something went wronginstep ´Checkingifmachinedefaultexists´...Press any key tocontinue...

解决方案：
在添加windows功能里面吧Hyper-V功能关闭，重启电脑

问题二： Waiting for an IP
全部重新初始化之后，断网启动Docker QuickStart

问题三： Error checking TLS connection: ssh command error: command : ip addr show



今天安装docker部署的时候总是再报这个错误。

报错的原因是初始化的时候出错了。

在docker 安装目录下有一个文件，如下图所示



将它复制到你电脑用户名目录下生成.docker 的文件夹中，如下图



在将第一次报错后初始化的这两个文件删除



将你的网络关掉，重启docker



这样就可以了，只有第一次初始化的时候要关闭网络，我就是因为没有关闭所以初始化错误。

