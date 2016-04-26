# 第二课 openstack底层技术及通用组件讲解
## 1.计算虚拟化相关技术介绍
### CPU特权级
CPU指中央处理器，是一台电脑上最核心的组件，它的功能主要是解释计算机的指令以及处理计算机软件中的数据，在操作系统中做的任何操作，比如打开一个网页或world 文档之类的操作，落到底层时都需要CPU进行计算处理，而这样的操作显然不能随便让哪个程序去做，否则硬件很容易被一些黑客程序去攻破，系统的稳定性无从谈起，因此就自然引出“特权级”的概念，最关键，最核心的一些操作智能由最高权限的程序来执行，这样就可以做到集中管理，并且能够减少有限资源的访问和使用的冲突。

以X86架构的CPU来说，它定义了一个值叫环Ring，从里到外四个环分别是Ring0，Ring1，Ring2，Ring3，也就是分别对应着四个特权级，Ring0是最高的，Ring3是最低的。CPU执行每条指令时都会对这条指令所具有的特权级做相应的检查，这样就能做到集中管理并减少资源访问使用冲突。

### 内核态与用户态
当程序运行在Ring3特权级时称为“用户态”，反之，当程序运行在Ring0特权级上的时候成为“内核态”。操作系统的代码运行在最高级别的Ring0上，而应用程序的代码运行在最低级别Ring3上。运行在用户态的程序是不能做一些底层操作的，比如需要访问磁盘，读写内存，这些操作都需要通过调用内核态的代码才能够执行。

计算虚拟化最直观的体现就是在一台服务器上同时运行多个操作系统，而这些操作系统资源池是由一个数组的操作系统和若干个客户操作系统组成的。一个服务器是无法做到同时运行多个操作系统的，因为数组操作系统是工作在内核态，而整个内核态是被数组操作系统独占的，而其它的客户操作系统就没办法使用了，但客户操作系统是不知道的，所以它以前执行什么指令，现在还是执行什么指令，但这样肯定是不行的，因为它没有权限，也就无法做的一个服务器同时运行多个操作系统。

虚拟化主要的难题就是解决多个CPU怎么虚拟的问题，因为只有多个CPU才能有多个特权级，这样才能保证每个操作系统都运行在它自己的特权级上面，所以才叫做计算虚拟化，而不是服务虚拟化。

### hypervisor（VMM）
一种运行在基础物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件。也可叫做VMM（virtual machine monitor），即虚拟机监视器。

对于数组操作系统来说，它认为hypervisor是一个普通的运行在它上面的一个驱动程序而已，而对于客户操作系统（VM）来说，它认为它就是运行在它自己的硬件上面，也就是说所有的VM需要的硬件都是由hypervisor虚拟化提供的。

### hypervisor类型：半虚拟化
* 半虚拟化
  - 对客户操作系统（VM）的内核进行修改，将运行在Ring 0 上的指令转为调用hypervisor

虚拟化管理程序（hypervisor）有两种主要的实现方式，一是半虚拟化，设计思路是：让客户操作系统知道自己是一个虚拟机，运行在Ring3特权级上，需要把客户操作系统的内核代码进行修改，由原来的调用Ring0特权级的指令修改为调用hypervisor，这种半虚拟化只能虚拟Linux操作系统。

### hypervisor类型：全虚拟化
* 硬件复制全虚拟化
  - Intel VT和AMD-V技术
  - 客户操作系统可以直接使用Ring 0 而无需修改
  - 查看CPU是否支持：
    + grep "vmx" /proc/cpuinfo
    + grep "svm" /proc/cpuinfo

第二种实现的方式是全虚拟化，其全称为硬件辅助全虚拟化，因为只有CPU具备Intel VT或AMD-V技术才能实现虚拟化。

Vmware Workstation,VirualBox为非硬件辅助全虚拟化，其主要对CPU进行虚拟化模拟，对客户操作系统而言，它认为自己运行在自己的硬件之上。

全虚拟化的实现方式的性能大大优于半虚拟化实现方式的性能，实际生产一般采用全虚拟化的方式以实现计算虚拟化。

### 计算虚拟化的其他实现方式
* 操作系统虚拟化
  - 允许操作系统内核拥有彼此隔离和分割的多用户空间实例。这些用户空间实例，也称之为容器
  - 基于linux内核的namespace、chroot、cgroup实现

hypervisor这种计算虚拟化的实现方式可以提供包括操作系统内核在内的一个完整的系统镜像，这种技术可以为用户分配虚拟化后的CPU、内存、存储等资源，并且都是跟其它用户相互独立的，但是这种技术是一种重量级虚拟化实现技术，因为它提供了一个完整的操作系统，所有的应用都是基于这个操作系统运行的。

除了hypervisor这种重量级的虚拟化实现技术还有轻量级的虚拟化实现技术，称之为操作系统虚拟化，操作系统虚拟化允许操作系统内核拥有彼此隔离和分割的多用户空间实例，这些用户空间实例，也称之为容器。容器可以为应用程序提供一个隔离的运行空间，每个容器内都包含一个用户独享的完整的环境空间，并且对一个容器内的变动是不会影响其它容器的运行环境的。

一般基于Linux内核中的namespace实现用户空间的分割，通过chroot限制可以访问的文件，通过cgroup确定每个容器可以使用多少CPU，多少内存等系统资源。linux内核自带的有LXC容器，另外还有现在比较火的Docker，Docker还增加了一些管理功能。

### quemu
* 可以在一种架构（如PC机）下运行另一种架构（如ARM）下的操作系统和程序。
* x86架构，支持半虚拟化技术。
* 能让多个虚拟机使用同一镜像，并为每一个虚拟机配置个性化硬件环境（网卡、磁盘、图形适配器……）。
* qemu官方网站（http://www.qemu.org）.

### KVM
* KVM是开源软件，全称是kernel-based virtual machhine(基于内核的虚拟机)。
* 是x86架构且硬件支持虚拟化技术（如Intel VT或AMD-V）的linux全虚拟化解决方案。
* KVM还需要一个经过修改的QEMU软件（qemu-kvm），作为虚拟机上层控制和界面。
* KVM能让多个虚拟机使用同一镜像，并未每一个虚拟机配置个性化硬件环境（网卡、磁盘、图形适配器……）。
* 在主流的linux内核，如2.20以上的内核均已包含了KVM。

### hypervisor软件比较
![](https://github.com/Erik-ly/OpenStack-Kilo/blob/master/lesson-02/imagines/hypervisor-software.jpg)

### libvirt
* libvirt是一套免费、开源的支持linux下主流虚拟机管理程序的C函数库，其旨在为包括KVM在内的各种虚拟化管理程序提供一套方便、可靠的编程接口。
* 当前主流linux平台上默认的虚拟化管理工具virt-manager（图像化），virtsh（命令行模式）等均给予libvirt开发而成。
* libvirt关键名词解释：
  - 节点（Node）：一个物理机器，上面可能运行着多个虚拟客户机。Hypervisor和Domain都运行在Node之上。
  - 域（Domain）：是在Hypervisor上运行的一个客户机操作系统实例。域也被称为实例（instance，如亚马逊的AWS云计算服务中客户机就被成为实例）、客户机操作系统（guest OS）、虚拟机（virtual machine），它们都是指同一个概念。

![](https://github.com/Erik-ly/OpenStack-Kilo/blob/master/lesson-02/imagines/libvirt.jpg)

libvirt是基于驱动程序的架构来实现管理虚拟化软件的，比如它为KVM、LXC等各开发一套驱动程序，通过同样的接口调用不同的驱动程序来驱动KVM这样的管理虚拟化的程序，同时libvirt是作为中间的一个适配层屏蔽了底层的虚拟化管理程序的细节，为上层（比如OpenStack）提供统一的接口，通过libvirt，比如OpenStack就可以管理各种不同的虚拟化管理程序以及运行在这些虚拟化管理程序上的客户操作系统(VM)。

## 2.网络虚拟化相关技术介绍
### OSI七层模型
![](https://github.com/Erik-ly/OpenStack-Kilo/blob/master/lesson-02/imagines/OSI.jpg)

OSI七层模型是一个网络互联的模型，它定义了网络互联的基层框架。
网络虚拟化主要涉及的就是网络层和链路层。基于网卡和交换机可以将一台台服务器组成一个网，然后通过路由器把不同的子网串成一个更大的网，防火墙可以为这个网络提供安全保障。网络虚拟化也就是虚拟网卡、交换机、路由器、防火墙等。L2、L3分别代表链路层和网络层。

### 软件定义网路（SDN）
* SDN定义
  - 将网络的控制平面与数据转发平面进行分离，从而通过集中的控制器中的软件平台去实现可编程化控制底层硬件，实现对网络资源灵活的按需调配。

* SDN架构
  - 应用层：包括各种不同的业务和应用；
  - 控制层：主要负责处理数据平面资源的编排，维护网络拓扑、状态信息等；
  - 基础设施层：负责基于流表的数据处理、转发和状态收集。

SDN架构中最核心的是控制层，除了要提供API给上层的应用层，间接的控制底层硬件外，还负责基础设施层硬件的网络拓扑状态以及相关的数据编排等。

软件定义网络（SDN）与传统网络不同之处：

优势：

1.	硬件设备价格低廉，因为基础设施层的硬件设备功能单一(只负责数据转发)。
2.	整个网络的特性是由控制层的软件实现的，对网络的控制和运行都由控制层操作，升级更新比传统硬件容易。
3.	最核心特性就是对业务响应更快。因为所有的网络的控制都是通过控制层的软件来时时定义，但传统网络的业务发生变动之后就要把所有网络设备进行更改，比如路由器，交换机，防火墙等。

劣势：

1.	SDN技术还不成熟，稳定性较差。

2.	安全性比传统网络差，因为SDN的整个网络是通过控制层的软件控制的，如果黑客控制了网络层的软件，就会控制整个网络。

### Open vSwitch
* Open vSwitch 简称OVS。常用在虚拟化平台，为虚拟机提供二层交换功能。支持Xen/XenServer，KVM，VirtualBox多种虚拟化技术。
* Open vSwitch支持openflow协议。可以使用任何支持OpenFlow协议的控制器对OVS进行远程管理控制。

### Open vSwitch相关概念
* Bridge：Brigde代表一个以太网交换机（Switch），一个主机中可以创建一个或者多个Bridge设备。
* Port：端口与物理交换机的端口概念类似，每个Port都隶属于一个Bridge。
* Interface：连接到Port的网络接口设备。
* Controller：OpenFlow控制器。
* Datapath：负责执行数据交换，也就是把从接收端口收到的数据包在流表中进行匹配，并执行匹配到的动作。
* Flow table：每个datapath都和一个"flow table"关联，当datapath接收到数据之后，OVS会在flow table中查找可以匹配的flow，执行对应的操作，例如转发数据到另外的端口。

### Open vSwitch架构
![](https://github.com/Erik-ly/OpenStack-Kilo/blob/master/lesson-02/imagines/Open-vSwitch.jpg)

OVS架构主要分为三部分，第一部分是web控制器，可以用自带或第三方软件实现对OS进行管理控制。第二部分是用户态，用户态主要有两个主要的进程，一个是ovsdb-server，它是OS 的核心的守护进程，它实现了核心的虚拟交换机的功能，OS的配置会存在ovsdb数据库中，并且数据是持久性的。第三部分是内核态，用户态的所有操作最终都会转向内核态，然后再由内核态对数据进行处理。

### Open vSwitch常用组件及操作
* ovs-dpctl：命令行工具，用来配置交换机内核模块，可以控制转发规则。
  - ovs-dpctl dump-flows br0 #查看指定bridge上的datapath信息
* ovs-vsctl:主要是获取或者更改ovs-vswitch的配置信息，此工具操作的时候会更新ovsdb-server中的数据库。
  - ovs-vsctl add-br br0 #添加网桥
  - ovs-vsctl list-br #列出说有网桥
  - ovs-vsctl set-controller ovs-switch tcp:9.181.137.182:6633 #指定controller控制器
* ovs-ofctl:用来控制OVS作为OpenFlow交换机工作时候的流表内容

### Linux Bridge
* 桥接
  - 定义1：是指依据OSI网络模型的链路层的地址，对网络数据包进行转发的过程，工作在OSI的第二层。
  - 定义2：把一台机器上的若干个网络接口“连接”起来。
* Linux Bridge
  - linux上用来做二层协议交换的虚拟设备，与现实世界中的交换机功能相似。这个虚拟设备可以绑定若干个以太网接口设备，从而将它们桥接起来。
  - 安装：yum install bridge-utils -y
  - Linux Bridge配置命令：brctl（可以通过执行brctl help查看命令帮助）

## 3.OpenStack通用组件介绍
### Python相关说明
* OpenStack基于Python 2.7 版本开发。
* OpenStack Liberty版本才支持Python 3.
* 检查操作系统默认Python版本命令：python -v
* pip是一个安装和管理Python包的工具

### REST
* REST 是一中架构风格，其核心是面向资源
* 基于HTTP协议
* HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE
* GET用来获取资源，POST用来新建资源，PUT用来更新或新建资源,DELETE用来删除资源

### WSGI
* WSGI定义
  - WSGI的全称是Web Gateway Interface，这是一个规范，描述了web server如何与web application交互、web application如何处理请求
  - WSGI包含Server，Middleware，Application。WSGI server接收客户请求，传递给Middleware，Middleware根据相关配置路由给WSGI application处理

在OpenStack里，除了管理界面是基于apache来提供web服务的，其它的api都不是通过apache提供服务的。在之后的排查错误的时候，如果api不能启动，首先要查WSGI服务是否是起动的。

### Paste Deployment
* Paste Deployment（简称PD）是一个WSGI工具包
* 基于PD的应用配置文件，内容被分为很多段（section），PD只关心带有前缀的段，比如[app:main]或者[filter:errors]
* 一个section的内容是以键=值来标示的。#是一个注释。

### MariaDB
* MariaDB是一个采用Maria存储引擎的MySQL分支版本
* MariaDB完全兼容MySQL，包括API和客户端协议
* OpenStack的核心项目Keyston，Cinder，Glance，Neutron，Nova等均使用到它来存放相关持久数据

### RabbitMQ
* Message Queue（MQ）定义
  - MQ是消费-生产者模型的一个典型的代表，一端往消息队列中不断写入消息，而另一端则可以读取或者订阅队列中的消息
* MabbitMQ（官方地址：http://www.rabbitmq.com）是一个有erlang开发的基于AMQP协议（Advanced Message Queue Protocol）的开源实现。通常用于应用程序之间或者程序的不同组件之间通过消息来进行集成。
* 主要名词解释
  - 交换器（Exchange），它是发送消息的实体
  - 队列（Queue），这是接收消息的实体。
  - 绑定器（Bind），将交换器和队列连接起来，并且封装消息的路由信息。
* OpenStack中模块Cinder、Neutron、Nova等项目的内部组件之间的通信都是通过AMQP协议实现，消息由RabbitMQ作为中间件转发

