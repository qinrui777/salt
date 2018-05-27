### SALT是什么,有什么用？
一个基础设施管理工具，类似于Puppet 、 Chef 、 Ansible ,两大主要功能：1 、远程执行命令 ；2 、配置管理（salt state system）

### SALT特性有哪些？
- 基于Python语言编写的
- 采用了ZeroMQ这样一个消息中间件（消息发送、接收速度很快，大概ssh的40倍）
- C/S架构（另外也有 No agent 、多master两种方式，使用较多的是C/S架构）
- 支持多平台 linux 、windows 

### 基本概念
上面说到一般用C/S架构，那么server端在salt中叫做 **master** ,client端叫做 **minion**


### 常用模块或者叫系统
- **Grains**
- **Pillar**
- **Targeting**
- **Returner**
- **Runners**
- **Event**
- **Reactor**


### 利用Vagrant快速搭建 **demo** 环境

**前提：** 已经安装 **VirtualBox** 和 **Vagrant**

下载demo环境所需的Vagrantfile和其他文件
`git clone https://github.com/UtahDave/salt-vagrant-demo.git`
进入下载的目录
`cd salt-vagrant-demo`
利用该目录的Vagrantfile启动demo环境，这个过程需要下载一些东西，需要几分钟时间
`vagrant up`
环境搭建好后，可在VirtualBox看到，集群有一个master、两个minion 
> 该demo环境会自动做好安装、设置配置文件、认证等一下，可方便联系salt命令，如果是生产环境或者想要自行安装、配置等，可忽略




###  常用命令
