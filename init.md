<!-- 本书的定位是短篇，科普向，外加一点点专业知识。 -->
一般来说，Linux是对以Linux为内核的操作系统的统称，与Windows操作系统和MacOS操作系统属并列关系。  
Linux从最上层到最底层可以划分为如下五个部分：
- 应用软件(微信，qq等)
- 图形化桌面环境(Desktop Environment)
- GNU工具集(你可能听说过ls 等命令)
- Linux内核
- 计算机硬件

这里简述一下Linux内核所负责的四部分职责。

- 系统内存管理
- 系统软件进程管理
- 硬件设备管理
- 文件系统管理

## TODO linux启动过程(init等)

在「早期用户空间」的最终环节里，真正的根文件系统被挂载好后，就会替换掉原来的伪根文件系统。接着 /sbin/init 被执行，同样也替换掉原来的 /init 进程。Arch 御用的 pid为1的 进程 是 systemd (简体中文). 传统的为SysVinit[https://wiki.archlinux.org/index.php/SysVinit]

https://wiki.archlinux.org/index.php/Arch_boot_process_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
https://wiki.archlinux.org/index.php/Init
https://zhuanlan.zhihu.com/p/50367649
https://unix.stackexchange.com/questions/13290/init-process-ancestor-of-all-processes
https://superuser.com/questions/377572/what-is-the-main-purpose-of-the-swapper-process-in-unix


《待考证》
pid0有几个常见的名字,swapper/scheduler/idle 。前两个名字都是Unix历史原因，可不理会。实际上大多数Unix已不进行swap交换。在linux,swapper不交换任何东西，它仅仅是一个idle进程，可以将其视作linux内核的一部分。pid0进程是运行cpu_idle（）的空闲进程。 它只是无限循环地无所作为。 它的存在使得总是有一个准备好要计划的任务。

pid为1的/sbin/init 以及 kthreadd均有pid0（内核）创造

## 系统内存管理
除了真实的物理内存(内存条)，Linux还存在一个交换空间(swap space)的概念。交换空间是在你的硬盘设备上提前分配好的一块区域。假如运行Dota2需要4G的内存空间，但是你的内存条只有2G,不够运行Dota2.那么此时如果你有已经分配好的交换空间，就可以使用2G交换空间 + 2G物理内存的方式运行Dota2.真实的物理内存和交换空间两部分加在一起统称为`虚拟内存`，Linux内核即提供对虚拟内存的管理。  

虚拟内存会按组划分为很多块，这些块的术语称为页面(page)。内核会维护一个内存页面表，用来指明各个页面的存放位置，哪些在物理内存中，哪些在交换空间中。  

Linux内核会记录哪些页面正在使用，并且会把一段时间内未访问的，在物理内存中的页面移动到交换空间里，这个过程称为换出(swapping out)，即使此时物理内存仍有空余，换出的过程也会进行。当程序需要访问一个已经被换出到交换空间的页面时，则内核需要将这个页面换入到物理内存中。如果此时物理内存均已用满，没有空闲的空间，则内核必须选择一个已有的页面进行换出，为需要换入的新页面腾挪位置。显然换入/换出的过程会耗费时间，拖慢程序的运行速度，但是在物理内存不足时，使用交换空间至少可以保证程序可以运行。  

在2020年，绝大多数个人主机内存均已很充足，网络上也有个人主机再无需设置交换空间的声音。但是还是建议在使用Linux时，要设置交换空间。一是因为有一些程序如果没有交换空间可能无法运行，二是因为个人主机需要交换空间的存在才可以支持休眠等功能。

## 系统软件进程管理

Linux中运行的程序一般称为进程。内核对进程采取相应的管理和控制。为了尽可能充分地压榨 CPU 性能，内核使用调度器，通过一定的优先级算法将 CPU 按照时间动态的分配给各个进程。让我们感觉就像所有进程都在同时使用 CPU 一样。 