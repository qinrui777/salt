salt 中的两大数据系统，可类比为 两大兄弟，一个比较喜欢一直住一套房子（grains），一个喜欢不停的换房子(pillar) 
均可以用在salt的模块和其他组件中

电影《一路有你》 主演： 古天乐、黄奕、莫文蔚 。
- 男的（古）买房子就要住一辈子  
- 女的（莫）买房子是为了换大房子  


### Grains 
服务器的一些静态信息，这里强调的是静态，就是不会变的东西，比如说os是centos，如果不会变化，除非重新安装系统


##### 查看minion的全部静态变量
`salt '*' grains.items`
##### 显示grains的变量名称
` salt '*' grains.ls`
##### 显示某一个变量
`salt '*' grains.item os`
##### 直接获取内容
`salt '*' grains.get os` 

### 定义grains的三种方法
##### 1 、定义minion的grains

> 一般在master 上执行，也可以在minion上 用salt-call 来查看,比如 `salt-call  grains.ls`

可以写在/etc/salt/minion中格式如下,重启minion生效(systemctl restart salt-minion)
```
grains:
 roles:
   - webserver
   - memcache
 deployment: datacenter4
 cabinet: 13
 cab_u: 14-15
 ```
或者写在/etc/salt/grains中，格式如下
```
roles:
 - webserver
 - memcache
deployment: datacenter4
cabinet: 13
cab_u: 14-15
```
##### 2、 不重启minion端 刷新grains
1.修改minion配置文件  
cat /etc/salt/grains   
centos: node2  
test: node2  

2.master端刷新  
salt '*' saltutil.sync_grains   
node2.minion:


##### 3、定义在master端

* 在/srv/salt中建立_grains目录  
`mkdir /srv/salt/_grains`

* 编写grains文件，需要返回一个字典  
```
 vim test1.py
 def hello():                      ##函数名字无所谓，应该是所有函数都会运行
   agrain = {}
   agrain['hello'] = 'saltstack'
return agrain                   ##返回这个字典
```

* 同步到各个minion中去  
```
salt '*' saltutil.sync_grains
salt '*' saltutil.sync_all
salt '*' state.highstate
```  

* 验证
`salt '*' grains.item hello`


### Pillar  
可以指定一些信息到指定的minion上，不像grains一样是分发到所有Minion上的，它保存的数据可以是动态的,Pillar以sls来写的，格式是键值对

适用情景：
- 1.比较敏感的数据，比如密码，key等
- 2.特殊数据到特定Minion上
- 3.动态的内容
- 4.其他数据类型

#####  编写pillar数据  

1.指定pillar_roots，默认是/srv/pillar(可通过修改master配置文件修改),建立目录

mkdir /srv/pillar
cd /srv/pillar  
2.编辑一个pillar数据文件

vim test1.sls
name: 'salt'
 users:
   hadoop: 1000
redhat: 2000
ubuntu: 2001
3.建立top file指定minion到pillar数据文件  

 vim top.sls
 base:
   '*':
     - test1
##### 刷新Pillar数据  
`salt '*' saltutil.refresh_pillar`

测试一下
```sh
salt '*' pillar.get name
salt '*' pillar.item name
```

##### 在state中通过jinja使用pillar数据  
```sh
vim /srv/salt/user.sls
 {% for user, uid in pillar.get(’users’, {}).items() %}  ##pillar.get('users',{})可用pillar['users']代替，前者在没有得到值的情况下，赋默认值
 {{user}}:
   user.present:
     - uid: {{uid}}
 {% endfor %}
当然也可以不使用jinja模板

vim /srv/salt/user2.sls
{{ pillar.get('name','') }}:
 user.present:
   - uid: 2002
```
通过jinja模板配合grains指定pillar数据

/srv/pillar/pkg.sls
```sh
pkgs:
 {% if grains[’os_family’] == ’RedHat’ %}
 apache: httpd
 vim: vim-enhanced
 {% elif grains[’os_family’] == ’Debian’ %}
 apache: apache2
 vim: vim
 {% elif grains[’os’] == ’Arch’ %}
 apache: apache
 vim: vim
 {% endif %}
```

### 对比  
<img src="https://github.com/qinrui777/salt/blob/master/images/grains_pillar_compare.png" width="600">
