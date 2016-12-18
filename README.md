README
===========================
该文件是用来描述关于Linux namespaces 用法，更好的理解docker是如何应用linux kernal的namespaces去实现的虚拟化技术， reference http://coolshell.cn/articles/17010.html
#简介
Linux Namespace是Linux提供的一种内核级别环境隔离的方法。Linux namespaces有以下几种：
![image](https://github.com/shu-frank-csdn/Linux-namespaces-study/blob/master/linux-namespaces.png)

主要是三个系统调用：

* clone() – 实现线程的系统调用，用来创建一个新的进程，并可以通过设计上述参数达到隔离。
* unshare() – 使某进程脱离某个namespace。
* setns() – 把某进程加入到某个namespace

#文件介绍
* uts.c uts namespace ，子进程hostname 变成hello
* pid.c pid namesapce, 会发现inside container 的pid是1，因为传统的UNIX系统中，PID是1的进程是init,它是所有进程的父进程。如果某个子进程脱离了父进程（父进程没有wait它），那么init就会负责回收资源并结束这个子进程。所以，要做到进程空间的隔离，首先要创建出PID为1的进程.</br>但是在进程中输入ps,top等命令，还是可以看到所有的进程，说明还是没有完全隔离。因为，ps,top会去读/proc文件系统，还要对proc文件系统进行隔离。
* pid.mount.c linux系统中/proc目录是一种虚拟文件系统，存储的是当前内核运行状态的特殊文件。本文件中，启用了mount namespace并在子进程中重新mount了/proc文件系统。所以进程只有2个。
