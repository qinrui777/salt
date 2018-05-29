它的核心是写sls(SaLt State file)文件,sls文件默认格式是YAML格式(以后会支持XML)，并默认使用jinja模板.
YAML与XML类似，是一种简单的适合用来传输数据的格式，而jinja是根据django的模板语言发展而来的语言，简单并强大，支持for if 等循环判断。

salt state主要用来描述系统，软性，服务，配置文件应该出于的状态，常常被称为配置管理！


state文件默认是放在/srv/salt中，它与你的master配置文件中的file_roots设置有关
vim /etc/salt/master
```
file_roots:
  base:
    - /srv/salt
```

#/srv/salt/apahce.sls
```
apache:             ##state ID，全文件唯一,如果模块没跟-name 参数，会将默认用的ID作为 -name
 pkg:               ##模块
   #- name: apache  ##函数参数，可以省略
   - installed      ##函数
 service:           ##模块
   - running        ##函数
  #- name: apache   ##函数参数，这个是省略的，也可以写上
   - require:       ##依赖系统
     - pkg: apache  ##表示依赖id为apache的pkg状态
```

> 可以使用names语句一次性声明多个states来覆盖ID语句

####  简单实例：

`/srv/salt/example.sls`
```
remove vim:
  pkg.installed:
    - names:
      - vim
      - curl
    - fire_event: True
```  
`salt 'minion1' state.apply examples`

master端通过执行命令进行数据同步，检测配置是否正确，并没有真正的推送数据   
salt '*' state.highstate  **-v test=true**
> 当State运行在test模式下时，Salt并不会改变系统上的资源。当一个资源和期望的一样时，会返回True；否则就会返回None通知用户。这个功能可以用于定期对线上业务主机的配置文件进行健康检查，以发现不合规的配置变更。这些变更可能并不全都适合直接做自动化的纠正，存在引发服务故障的可能性。  

> salt 'minion1' cmd.run 'apt-get install tree' -v test=true 程执行命令时该参数没有用


#### 多环境
（比如 base、 prod 、dev 、uat）    
salt -N STOCK state.sls  **saltenv='prod'**  nginx.stock


####  state执行顺序

state的执行时无序，那个无序是指执行我们写的那个sls是无序的，正是因为那个无序，salt保证每次执行的顺序是一样的，就加入了state order，在说它之前看看High Data(高级数据？)和Low Data(低级数据？)，高级数据我理解的就是我们编写sls文件的数据，低级数据就是经过render和parser编译过的数据。

查看highdata

`salt '*' state.show_highstate`  

查看lowdata  

`salt '*' state.show_lowstate`

通过查看lowdata我们发现里面有一个字段order,因为salt默认会自动设置order，从10000开始。可以通过设置master配置文件参数state_auto_order: False来关闭

Order的设定：
* include 被include的文件Order靠前，先执行

* 手动定义order字段，如
  apache:
    pkg:
  - installed
  - order: 1
 order的数字越小越先执行从1开始，-1是最后执行


