# 第三课 多节点安装OpenStack Kilo版本
## 准备工作
### 实验环境
* 实验系统：Windows 10
* 虚拟环境：VirtualBox 5.0
* Linux 版本：CentOS 7
* OpenStack 版本：Kilo

### 节点分配
* controller ：一块桥接网卡，2G内存，50G存储（动态分配）
* compute-node-01 ：一块桥接网卡，1G内存，50G存储（动态分配）
* network-node-01 ：一块桥接网卡，一块host-only网卡，1G内存，50G存储（动态分配）
* block-node-01 ：一块桥接网卡，1G内存，50G存储（动态分配）
* object-node-01 ：一块桥接网卡，1G内存，50G存储（动态分配）
* object-node-02 ：一块桥接网卡，1G内存，50G存储（动态分配）

### 组件部署
**controller 节点**
 * keystone   
 * glance   
 * horizon  
 * neutron  
  * server 
  * ml2   
  * clientwhich 
 * nova         
  * api        
  * cnductor  
  * console    
  * novaporxy   
  * ccert     
 * cinder      
  * api       
  * schedule    
 * swift      
  * proxy    
  * client     
 * mariadb      
 * rabbitMQ    
 * httpServer  
 * memcached     
 * ntpd          

**compute-node-01 节点**
* nova
 * compute
 * qumu
* neutron
 * openvswitch
 * ml2

**network-node-01 节点**
* neutron
 * openvswitch
 * ml2

**block-node-01 节点**
* cinfer
 * lvm2
 * volume

**object-node-01  节点**
*  rsync
*  swift
 * accout
 * object
 * container

**object-node-02  节点**
* rsync
* swift
 * account
 * object
 * container

### 配置主机名
vi /etc/hosts

192.168.1.30 controller

192.168.1.31 compute-node-01

192.168.1.32 network-node-01

192.168.1.33 block-node-01

192.168.1.34 object-node-01

192.168.1.35 object-node-02

注：也可以在安装时打开网络，默认dhcp获取的ip也是可以的，只要在同一个网段即可，主机名也可以在此时设置，安装好后添加到/etc/hosts 文件里即可。


## 安装
### 配置基础组件（在所有节点都要执行）
1.安装基础组件及源

      yum install ntp -y

      yum install yum-plugin-priorities -y

      yum install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm -y

      yum install http://rdo.fedorapeople.org/openstack-kilo/rdo-release-kilo.rpm -y

      yum upgrade

      yum install openstack-selinux -y

注:`yum install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm -y`这个命令中的链接如果失效，可以寻找x86_64/e/下的epel-release-7-*.noarch.rpm文件进行替换

2.停止防火墙服务

      systemctl stop firewalld.service

      systemctl disable firewalld.service

    


