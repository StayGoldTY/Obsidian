
每个文件都有一个 inode，用于存储文件的创建者和创建日期等元信息。inode 也会占用磁盘空间，众多的小缓存文件很容易导致 inode 资源耗尽。此外，在 inode 已用完但磁盘未满的情况下，也无法在磁盘上创建新文件。

在 KubeSphere 中，对 inode 使用率的监控可以帮助您清楚地了解集群 inode 的使用率，从而提前检测到此类情况。该机制提示用户及时清理临时文件，防止集群因 inode 耗尽而无法工作。

fs.file-max=1000000

# max-file 表示系统级别的能够打开的文件句柄的数量， 一般如果遇到文件句柄达到上限时，会碰到 "Too many open files"或者Socket/File: Can’t open so many files等错误。

看到too many open files可能想到fs.file-max参数，

max-file：表示系统级别的能够打开的文件句柄的数量。是对整个系统的限制，并不是针对用户的。
ulimit -n：控制进程级别（比如 Nginx 进程、MySQL 进程）能够打开的文件句柄的数量。提供对 shell 及其启动的进程的可用文件句柄的控制。这是进程级别的。
一边情况下，服务器上的 ulimit 都需要我们自己设置，而不能使用系统默认的，否则会出现文件描述符耗尽的问题。文件句柄达到上限之后的常见错误有：Too many open files 或者 Socket/File: Can’t open so many files 等。

查看 max-file：

$ sysctl -a | grep 'fs.file-max'
fs.file-max = 6553560
 
$ cat /proc/sys/fs/file-max
6553560
设置的方式有两种，一种是临时生效，重启后恢复默认。另一种永久生效。

file-max 的修改：

$ echo 6553560 > /proc/sys/fs/file-max
$ sysctl -w "fs.file-max=34166"
// 以上 2 种重启机器后会恢复为默认值
或
$ echo "fs.file-max = 6553560" >> /etc/sysctl.conf
// 立即生效，此方式永久生效
$ sysctl -p 
ulimit open files 修改：

// 这只是在当前终端有效，退出之后，open files 又变为默认值。当然也可以写到 /etc/profile 中，因为每次登录终端时，都会自动执行 /etc/profile
$ ulimit -HSn 65535
或
// 加入以下配置，重启即可生效
$ vim /etc/security/limits.conf  
* soft nofile 65535 
* hard nofile 65535

// 如果需要设置当前用户 session 立即生效，可以执行：
$ ulimit -n 65535 

解释说明
文件句柄：在 Linux 环境中，任何事物都是用文件来表示，设备是文件，目录是文件，socket 也是文件。用来表示所处理对象的接口和唯一接口就是文件。应用程序在读/写一个文件时，首先需要打开这个文件，打开的过程其实质就是在进程与文件之间建立起连接，句柄的作用就是唯一标识此连接。此后对文件的读/写时，目标文件就由这个句柄作为代表。最后关闭文件其实就是释放这个句柄的过程，使得进程与文件之间的连接断开。

看到too many open files可能想到fs.file-max参数，其实还受下面参数影响：

fs.inotify.max_queued_events：表示调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值的事件被丢弃，但会触发IN_Q_OVERFLOW事件。

fs.inotify.max_user_instances：表示每一个real user ID可创建的inotify instatnces的数量上限，默认128.

fs.inotify.max_user_watches：表示同一用户同时可以添加的watch数目（watch一般是针对目录，决定了同时同一用户可以监控的目录数量）

建议修改系统默认参数，方法如下（vi /etc/sysctl.conf）：

fs.inotify.max_user_instances=8192

注意: max_queued_events 是inotify管理的队列的最大长度，文件系统变化越频繁，这个值就应该越大。如果你在日志中看到Event Queue Overflow，说明max_queued_events太小需要调整参数后再次使用。

关于重启inotify配置max_user_watches无效被恢复默认值8192的正确修改方法

一般网上修改方法就是直接修改文件：

/proc/sys/fs/inotify/max_user_watches

或者修改方法：

sysctl -w fs.inotify.max_user_watches="99999999"

但是这些修改后，Linux系统重启inotify配置max_user_watches无效被恢复默认值8192，这个可能很多的新手不是很明白，这个不详细讲解，有空大家去了解下:sysctl

Linux系统重启inotify配置max_user_watches无效被恢复默认值8192的正确修改方法为：

vim /etc/sysctl.conf 

注意添加的内容：

fs.inotify.max_user_watches=99999999（你想设置的值）

haole ，好了，很简单。。

inotify watch 耗尽
每个 linux 进程可以持有多个 fd，每个 inotify 类型的 fd 可以 watch 多个目录，每个用户下所有进程 inotify 类型的 fd 可以 watch 的总目录数有个最大限制，这个限制可以通过内核参数配置: fs.inotify.max_user_watches

查看最大 inotify watch 数:

$ cat /proc/sys/fs/inotify/max_user_watches 
8192
使用下面的脚本查看当前有 inotify watch 类型 fd 的进程以及每个 fd watch 的目录数量，降序输出，带总数统计:



如果看到总 watch 数比较大，接近最大限制，可以修改内核参数调高下这个限制。 临时调整:





