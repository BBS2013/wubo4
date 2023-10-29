# linux是怎么工作的

# 第一部分

## 1.Linux系统

![image-20231026202851964](D:\github\学习记录\linux how to work.assets\image-20231026202851964-1698323336947-1-1698323344114-3.png)

## 1.2linxu系统构成

<font color='orange'>Linux操作系统主要分为三层</font>。如图所示，最底层是硬件系统，包括内存和中央处理器（用于计算和从内存中读写数据），此外硬盘和网络接口也是硬件系统的一部分。

硬件系统之上是内核，它是操作系统的核心。内核是运行在内存中的软件，它向中央处理器发送指令。内核管理硬件系统，是硬件系统和应用程序之间进行通信的接口。

进程是指计算机中运行的所有程序，由内核统一管理，它们组成了最顶层，称为用户空间（userspace）。（另一个更确切的术语是用户进程，无论它们是否直接和用户交互。例如，所有的Web服务器都是以用户进程的形式运行的。）

## 1.3**内核和用户进程**之间最主要的区别是：

内核在内核模式（kernel mode）中运行，而用户进程则在用户模式（user mode）中运行。

在内核模式中运行的代码可以不受限地访问中央处理器和内存，这种模式功能强大，但也非常危险，因为内核进程可以轻而易举地使整个系统崩溃。那些只有内核可以访问的空间我们称为内核空间（kernelspace）。

相对于内核模式，用户模式对内存和中央处理器的访问有一定程度的限制，可访问的内存空间通常很小，对CPU的操作也很安全。用户空间指的是那些用户进程能够访问的内存空间。如果一个用户进程出错并崩溃的话，其导致的后果也相对有限，并且能够被内核清理掉。例如，如果你的Web浏览器崩溃了，不会影响到你正在运行的其他程序。



### 内核负责管理以下四个方面。

进程：内核决定哪个进程可以使用CPU。

内存：内核管理所有的内存，为进程分配内存，管理进程间的共享内存以及空闲内存。

设备驱动程序：作为硬件系统（如磁盘）和进程之间的接口，内核负责操控硬件设备。

系统调用和支持：进程通常使用系统调用和内核进行通信。



### 进程管理

进程管理涉及进程的启动、暂停、恢复和终止。

内核负责上下文切换。我们来看看下面的场景，以便理解它的工作原理。

1. CPU为每个进程计时，到时即停止进程，并切换至内核模式，由内核接管CPU控制权。
2. 内核记录下当前CPU和内存的状态信息，这些信息在恢复被停止的进程时需要用到。
3. 内核执行上一个时间段内的任务（如从输入输出设备获得数据，磁盘读写操作等）。
4. 内核准备执行下一个进程，从准备就绪的进程中选择一个执行。
5. 内核为新进程准备CPU和内存。
6. 内核将新进程执行的时间段通知CPU。7. 内核将CPU切换至用户模式，将CPU控制权移交给新进程。

内核是在什么时候运行的。答案就是，<font color='red'>**内核是在上下文切换时的时间段间隙中运行的**。</font>



### <font color='red'>内存管理</font>

内核在上下文切换过程中管理内存,

内核需要自己的专有内存空间，其他的用户进程无法访问；

每个用户进程有自己的专有内存空间；

一个进程不能访问另一个进程的专有内存空间；

用户进程之间可以共享内存；

用户进程的某些内存空间可以是只读的；

通过使用磁盘交换，系统可以使用比实际内存容量更多的内存空间。

新型的CPU提供了MMU（Memory Management Unit，内存管理单元），MMU使用了一种叫作虚拟内存的内存访问机制，即进程不是直接访问内存的实际物理地址，而是通过内核使得进程看起来可以使用整个系统的内存。当进程访问内存的时候，MMU截获访问请求，然后通过内存映射表将要访问的内存地址转换为实际的物理地址。内核需要初始化、维护和更新这个地址映射表。例如，在上下文切换时，<font color='orange'>内核将内存映射表从被移出进程转给被移入进程使用。</font>



### 系统调用和支持

当你在终端窗口中输入ls时，终端窗口中的shell调用fork()创建一个shell的副本，然后该副本调用exec(ls)来运行ls。

系统调用通常使用括号来标记。exec(ls)



### 用户空间：

内核分配给用户进程的内存我们称之为用户空间。因为一个进程简单说就是内存中的一个状态。用户空间也可以指所有用户进程占用的所有内存。



# 第二部分 

根据系统启动的大体顺序，更深入地介绍从设备管理到网络配置的各个部分。



Linux的启动顺序主要分为以下步骤：

1. 加载BIOS的硬件信息，执行BIOS的内置程序。
2. 读取MBR（master root record）中boot loader的引导信息。
3. 加载内核kernel boot到内存中。
4. 内核执行／sbin／init，并加载／etc／inittab，执行rc.sysinit进行初始化。
5. 启动核心的外挂模块／etc／modules.conf。
6. 按照启动级别（默认是5）执行／et／rc5.d下的脚本。
7. 执行／bin／login程序。





BIOS的硬件信息是按照以下步骤加载的：

1. 通电自检：系统通电之后，主板的BIOS运行POST（Power on self test）代码，检测系统外围的一些设备，比如CPU、内存、显卡、IO、键盘鼠标等。
2. MBR引导：检测通过后，根据BIOS里boot设置的启动顺序（光驱、硬盘、网盘），搜索相应的启动驱动器，并获取第一个启动设备的代号。读取第一个启动设备的MBR的引导加载程序（lilo、grub、spfdisk）启动信息，从MBR中加载启动引导管理器（grub），并运行该启动引导管理，进入grub启动引导阶段。





读取MBR中的boot loader的引导信息是Linux启动过程中的重要步骤之一。在MBR中，boot loader通常被存储在第一个扇区，大小为446字节，这部分主要包含操作系统的机器码。接下来的扇区中包含了分区表（Partitiontable）和主引导记录签名（0x55和0xAA）。

具体操作如下：

1. BIOS根据启动顺序从0柱面0磁道1扇区开始读取硬盘的MBR。
2. BIOS将读取到的MBR复制到0×7c00地址所在的物理内存中，这个内存地址通常被称为“bootloader”。
3. Bootloader是在操作系统内核运行之前运行的一段程序，它负责加载并启动内核。在Linux系统中，常见的bootloader有grub等。
4. Bootloader会根据设定的内核映像所在路径，系统读取内存映像，并进行解压缩操作。

除了boot loader，Linux系统中还有其他引导程序，例如LILO和SYSLINUX。

LILO是一种过去常用的引导程序，主要特点是简单、可靠，但它不能识别较大的硬盘和文件系统。

SYSLINUX是一个轻量级的引导程序，主要用于创建可引导的镜像或者嵌入式系统。



加载内核boot到内存中是Linux启动过程中的重要步骤之一。在Linux系统中，内核boot被存储在硬盘上的特定位置，例如在MBR中的boot loader会指向内核boot所在的位置。

当系统从boot loader启动后，它会读取内核boot所在的物理地址，并将其加载到内存中。这个过程通常由引导程序（如GRUB）完成，引导程序会根据配置文件中的设置，将内核boot加载到内存中。

具体来说，引导程序会先读取硬盘上的MBR，获取启动设备的分区信息，然后根据配置文件中的设置，选择要加载的内核boot，将其从硬盘上读取并加载到内存中。这个过程通常涉及到一些底层的操作，如设置内存管理单元（MMU）等。

一旦内核boot被加载到内存中，引导程序就会跳转到内核boot的入口点，开始执行内核的初始化代码。这个过程包括对硬件进行初始化、加载驱动程序、设置内存管理等操作，直到最后跳转到用户空间的操作系统环境中。

总之，加载内核boot到内存中是Linux启动过程中非常关键的一步，它涉及到一些底层的操作和硬件初始化的过程。



在Linux启动过程中，当内核执行／sbin／init程序时，它将加载并执行／etc／inittab文件中的内容。inittab文件用于初始化系统，并定义了在系统启动和运行过程中需要执行的各种任务和程序。

在inittab文件中，通常会定义一些系统级别的服务和程序，例如启动脚本、系统监视器、进程控制等等。当内核执行init程序时，它会按照inittab文件中的设置，逐行执行其中的命令和脚本。这些命令和脚本通常会涉及到一些系统级别的初始化操作，例如设置环境变量、加载模块、启动守护进程等等。

在执行完inittab文件中的所有命令和脚本后，系统会进入下一个启动级别（例如运行级别），并执行相应的脚本和程序。这些脚本和程序通常会涉及到一些更具体的初始化操作，例如启动登录界面、加载网络配置等等。

总之，内核执行／sbin／init程序时，会加载并执行／etc／inittab文件中的内容，以完成系统级别的初始化操作。这个过程涉及到一些底层的操作和配置，以确保系统能够正常运行并提供所需的服务。



启动核心的外挂模块是指Linux内核启动时加载的额外模块，这些模块可以扩展和增强内核的功能。在Linux系统中，这些模块通常存储在／lib／modules目录下，而对应的配置文件则存储在／etc／modules.conf文件中。

在Linux启动过程中，当内核执行／sbin／init程序时，它将加载并执行／etc／inittab文件中的内容。在执行完inittab文件中的所有命令和脚本后，系统会进入下一个启动级别（例如运行级别），并执行相应的脚本和程序。在这个过程中，系统会根据／etc／modules.conf文件中定义的模块列表来加载需要的外挂模块。

在／etc／modules.conf文件中，每一行代表一个模块，可以包括模块的名称、参数和选项。例如，以下是一些常见的外挂模块和其对应的配置选项：

1. e1000：这是Intel公司的一款网卡驱动模块，用于支持千兆网卡。可以在modules.conf文件中添加以下行来加载该模块：

```
e1000 crc32_api crc32c_intel耿 131072
```

1. ext4：这是Linux默认的文件系统类型之一，用于支持ext4文件系统。可以在modules.conf文件中添加以下行来加载该模块：

```
ext4 crc32c_generic_new 131072
```

1. ipv6：这是用于支持IPv6协议的模块，可以在modules.conf文件中添加以下行来加载该模块：

```
ipv6 1
```

总之，启动核心的外挂模块是通过在／etc／modules.conf文件中添加相应的模块名称和参数来实现加载和使用的。在Linux启动过程中，系统会根据配置文件的设置来自动加载需要的模块，以确保系统能够正常运行并提供所需的服务。







linux内核启动

# 第三部分

系统各部分的运行方式，并介绍一些基本技巧和常用的工具