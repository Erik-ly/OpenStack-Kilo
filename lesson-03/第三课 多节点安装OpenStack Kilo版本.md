# 第三课 多节点安装OpenStack Kilo版本
## 环境说明
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
+ controller 节点
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

+ compute-node-01 节点
* nova
 * compute
 * qumu
* neutron
 * openvswitch
 * ml2

+ network-node-01 节点
* neutron
 * openvswitch
 * ml2

+ block-node-01 节点
* cinfer
 * lvm2
 * volume

+ object-node-01  节点
*  rsync
*  swift
 * accout
 * object
 * container

+ object-node-02  节点
* rsync
* swift
 * account
 * object
 * container









