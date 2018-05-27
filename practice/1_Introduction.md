### SALT是什么,有什么用？
* 一个基础设施管理工具，类似于Puppet 、 Chef 、 Ansible
* 两大主要功能：1 、远程执行命令 ；2 、配置管理（salt state system）

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
 
环境搭建好后，可在VirtualBox看到，集群有一个master、两个minion ，如下图：  



<img src="https://github.com/qinrui777/salt/blob/master/images/salt_virtualbox.png" width="400">

|      ip      |role  |
|------------  |----- |
|192.168.50.10|master |
|192.168.50.11|minion1|
|192.168.50.12|minion2|

> 该demo环境会自动做好安装、设置配置文件、认证等一下，可方便联系salt命令，如果是生产环境或者想要自行安装、配置等，可忽略
> ![image](https://github.com/qinrui777/salt/blob/master/images/salt_virtualbox.png =250x250)

###  常用命令

登入master 所在虚拟机，登录minion 节点操作类似( vagrant ssh minion1 )  
`vagrant ssh master`

如果没有权限执行，可切换到 **root** 用户
sudo su

##### 命令格式 
`salt [options] '<target>' <function> [arguments]`  
`salt '*' test.ping     ##类似helloword`  

#####  salt-key 密钥管理，通常在master端执行
 ```
salt-key [options]
salt-key -L              ##查看所有minion-key的情况
salt-key -a <key-name>   ##接受某个minion-key
salt-key -d <key-name>   ##删除某个minion-key
salt-key -A              ##接受所有的minion-key
salt-key -D              ##删除所有的minion-key
```

#####  salt-call 该命令通常在minion上执行

minion自己执行可执行模块，不是通过master下发job

```
salt-call [options] <function> [arguments]
salt-call test.ping           ##自己执行test.ping命令
salt-call cmd.run 'ifconfig'  ##自己执行cmd.run函数
```

#####  salt-cp 分发文件到minion上
不支持目录分发，通常在master运行.后面在讲远程执行也会类似命令 **cp.get_file**  cp 模块中有拷贝文件、目录的功能直接使用
方法不一样，但殊途同归
```
salt-cp [options] '<target>' SOURCE DEST
salt-cp '*' testfile.html /tmp
salt-cp 'test*' index.html /tmp/a.html
```


##### target

* globbing 默认   
`salt 'test*' test.ping`
* regular expression 正则表达式  
`salt -E 'minion[1-2]' test.ping`
* list 列表  
`salt -L 'minion1,minion2' test.ping`
* grains  
`salt -G 'os:CentOS' test.ping`
```
#查看所有grains键/值
salt 'test*' grains.items
#查看所有grains项
salt 'test*' grains.ls
查看某个grains的值
salt 'test*' grains.item num_cpus
```
* pillar
* nodegroups 其实就是对Minion分组
可以在master 端 /etc/salt/master 中设置

* 子网 CDIR
salt -S ‘115.29.249.3/24’ test.ping
* 组个 Compound matchers -C
salt -C ‘G@os:xxx -N@dbNode’ test.ping

