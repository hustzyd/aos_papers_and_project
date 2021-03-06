
# Virtual machine monitors: Current technology and future trends

## 背景介绍

虚拟机技术的发展经历了很多次的波折。在1960年左右，计算机都是大型机，硬件的资源十分昂贵。为了对这种硬件资源进行高效的复用，诞生了基于软件层面的虚拟机管理技术（VMM）。后来，到了上世纪八十年代，随着多任务操作系统的诞生以及硬件成本的下降，VMM技术逐渐退出了历史舞台。

然而，VMM这项技术并没有消失。二十世纪末，斯坦福的研究学者开始关注用VMM来解决硬件和操作系统的兼容问题，尝试利用VMM技术对复杂的硬件体系结构进行改进，以支持现有的操作系统。这个想法诞生了迄今为止仍然是虚拟机领域霸主的VMware公司。新世纪初，随着计算机硬件成本的进一步下降，机器的数量迅速增加，管理人员的管理任务也变得复杂起来。此外，单台计算机的运算能力也不断提升，功能和服务不断增多，增大了单台计算机崩溃的可能性，管理成本进一步提高。为了减少系统崩溃，系统管理员开始“主动降低硬件水平”，尝试每台机器运行一个应用程序。这反过来又增加了硬件需求，造成了巨大的成本和管理开销。这时，VMM技术再次出现在人们的视野，将曾经在许多物理机器上运行的应用程序移动到虚拟机中，并将这些虚拟机整合到几个物理平台上，可以提高使用效率并降低空间和管理成本。由此，VMM技术的发展再次迅猛起来，并得到了很多大公司的资本和技术支持。

## 虚拟机技术

VMM技术再次进入人们的视野之后，就不再是之前单纯的出于硬件复用的考虑，更多是出于运行效率、安全性和方便管理的考虑。VMM技术的主要架构设计如图所示，核心的需求是在硬件和操作系统之间提供一个中间层，用于给操作系统提供一个硬件操作的接口。
![image](./pics/vmm0.png)

从技术角度讲，VMM必须能够将硬件接口导出到虚拟机中的软件，该硬件大致等同于物理硬件。此外，VMM还需保持对物理硬件的控制，并保留在虚拟机和物理硬件的能力。出于这个技术目的，有很多不同的技术实现方案。在评估这些权衡时，VMM的中心设计目标是兼容性，性能和简单性。兼容性显然非常重要，因为VMM的主要优势在于其运行传统软件的能力。性能这个衡量虚拟化开销的目标是以与软件在物理机上的运行开销对比的。当然，简单性尤其重要，因为VMM故障可能会导致计算机上运行的所有虚拟机发生故障。当然，虚拟机还有重要的一点就是安全性，隔离性保证攻击者不能轻易进行虚拟机逃逸以攻击Host OS。

## CPU虚拟化

CPU硬件虚拟化技术允许虚拟机中的指令能在CPU上直接运行，可以将其概述为“直接执行”机制。这要求CPU能在非特权模式下运行虚拟机的特权和非特权代码，而VMM以特权模式运行。因此，当虚拟机尝试执行特权操作时，CPU会陷入VMM，VMM模拟VMM管理的虚拟机状态上的特权操作。因此，设计的关键是如何让VMM安全、透明地执行虚拟机中的代码。

### 挑战

这篇文章发表于2005年，文章认为主要的挑战是大多数的CPU并不支持虚拟化特性，包括流行的X86架构。x86不支持虚拟化特性，并且中断和特权指令的处理机制都不满足VMM的设计需求。

### 技术

有几种技术解决了如何在不能虚拟化的CPU上实现VMM，最常见的是半虚拟化，将“直接执行”机制与快速二进制翻译相结合。通过半虚拟化，VMM通过用简单虚拟化，替换原始指令集的非虚拟化部分，来定义虚拟机接口。尽管必须将操作系统移植到虚拟机中运行，但大多数正常应用程序仍可以不加修改地运行。最后VMware参与推动这项技术，核心就是对将特权指令进行二进制转换。在大多数现代操作系统中，运行正常应用程序的处理器模式都是可虚拟化的，因此可以使用直接执行来运行；而二进制转换器用来运行不可虚拟化的特权模式。最终是设计一个与硬件相匹配的高性能虚拟机，从而保持了软件的全面兼容性。

### 未来发展

当然，预期的发展目标就是在CPU直接实现硬件级别的虚拟化。AMD和Intel都宣布了即将在自家的产品上实现虚拟化特性，当然牙膏厂也并没有实验，2005年就推出了相关技术。

## 内存虚拟化

内存虚拟化的传统实现是VMM在物理机器上维护一个影子内存，用于映射虚拟机中的内存数据，以便对虚拟机内存进行精确管理。当虚拟机中运行的操作系统在其页面表中建立映射时，VMM检测到这些更改并在相应的影子页表项中建立一个映射，指向硬件存储器中的实际页面位置。当虚拟机进行内存读写时，硬件使用影子页表进行内存转换，这样VMM就可以控制每个虚拟机正在使用的内存。与传统操作系统的虚拟存储系统设计一样，VMM可以将虚拟机分页到磁盘，以便分配给虚拟机的内存可以超过硬件的物理内存大小，这样的设计可以让VMM高效使用硬件资源。VMM可以根据需要动态控制每个虚拟机获得的内存量。

### 挑战

VMM的虚拟内存子系统会不断控制进入虚拟机中的活跃内存页，并且必须通过将虚拟机的一部分分配给磁盘来定期回收部分内存。然而，对于这样一套内存管理机制，在虚拟机（GuestOS）中运行的操作系统可能比VMM的虚拟内存系统具有更多的信息，这些信息可以被用于更好地管理分页。例如，一个GuestOS可能会注意到创建页面的过程已经退出，这意味着该页面可能不会再被访问，而在硬件级别上运行的VMM不会看到这一点，并可能会将该页面分页。为了解决这个问题，VMware的ESX Server采用了类似半虚拟化的方法，其中在GuestOS内运行的气球进程可以与VMM进行通信。当VMM想要从虚拟机中读取内存时，它要求“气球程序”处理分配更多的内存，实质上是“膨胀”了这个过程。然后，GuestOS使用其关于页面替换的深入信息来选择要提供给扩展进程的页面，并将这个信息传递给VMM用于内存页面管理。

### 未来发展

未来的发展自然是将虚拟内存管理的这套影子内存系统放在硬件上进行实现

## I/O虚拟化

30年前，IBM大型机的I/O子系统使用基于通道的架构，其中通过与单独的通道处理器进行通信来访问I/O设备。通过使用通道处理器，VMM可以安全地将I/O设备访问直接导出到虚拟机,虚拟化开销非常低。

### 挑战
VMware Workstation开发了图所示的托管体系结构。在此体系结构中，虚拟化层使用主机操作系统（HostOS）的设备驱动程序来访问设备。
![image](./pics/vmm1.png)

另一个问题是，现代操作系统（如Windows和Linux）没有资源管理支持来为虚拟机提供性能隔离和服务保证，而这是许多服务器环境所需的功能。 ESX Server采用更传统的VMM方法，无需主机操作系统即可直接在硬件上运行。除了复杂的调度和资源管理外，ESX Server还为网络和存储设备提供高度优化的I/O子系统。

### 未来发展

使用硬件来进行高性能的I/O虚拟化。

## 总结

最后，作者研究了虚拟机的特性和使用场景。虚拟机的一大特性就是安全性和隔离性，由于对外提供服务的是虚拟机，安全性保证了黑客获取虚拟机权限后不会进一步逃逸从而影响到物理机器，因此这个特性是虚拟机未来的一大应用方向和改进点。其次，作者注意到，越来越多的厂商开始利用虚拟机进行软件分发，诸如oracle这种庞大的软件，厂商只需要提供给用于一个配置好的虚拟机镜像，用户实现了“开箱即用”而不用再去繁琐的配置环境。这一点很有感触，这也是今天容器技术快速发展的一个重要推动力。
