### Event system

我们知道Master与Minion是基于ZMQ通信的，他们通信我们看来是消息队列，对它们来说这些消息就是一些事件，什么事件应该做什么，是salt基本已经预设好了。
我们学习它的事件系统来完成一些自定义的行为，后面的反应系统就是基于事件系统的。一条消息其实就是一个事件，事件通常是一个字典数据，这个字典数据通常包含tag,
这个tag是用来区分用途过滤消息的，详见绿大-https://groups.google.com/forum/#!topic/saltstack-users-cn/wXVE4ydnnzc ,让我们来看看这些事件。


####  监听event（monitor event）

#####  方法一：
```
wget https://raw.githubusercontent.com/saltstack/salt/develop/tests/eventlisten.py
python eventlisten.py
```
再打开一个master 终端，执行 `salt 'minion1' test.ping `

观察第一个master终端的输出如下：



> Minion: python2.6 eventlisten.py -n minion <minion-id>   ##捕捉minion端的需要额外参数，minion-id是该Minion的id

##### 方法二：
可以通过以下命令查看event事件，然后再打开一个终端执行任务
`salt-run state.event pretty=True`


##### 方法三：

`vim monitor_event.py`
```
import salt.utils.event
event = salt.utils.event.MasterEvent('/var/run/salt/master')
for eachevent in event.iter_events(full=True):    //用迭代器一直查看事件
    print eachevent
    print "------"
```
`python monitor_event.py`
 
 
####  发送事件（fire event）

- Master发给minion  
`salt '*' event.fire "{'data': 'some message'}" "tag"`   
前面必须是字符串包住的字典，后面是tag,如果你的minion在监听event，你会看到这条event的

- Minion发给minion  
`salt-call event.fire_master 'some message' 'tag'`
前面数据类型没有要求，后面是tag,去master那看看收到了没有

- Minion发给自己
`salt-call event.fire "{'data': 'some message'}" 'tag'`
前面必须是字符串包住的字典，后面是tag


###   Reactor system 
反应系统是基于事件系统的，它的作用是当master收到来自minion的特殊事件后就触发某些动作，比如minion上线后发送一个init事件，master收到后，对其应用init的状态文件
是master端进程，实际上 reactor系统需要监听事件总线 （event bus）以确定它需要执行什么，minion没有反应系统。


####  配置reactor

1.修改master配置文件或者在/etc/salt/master.d/中建立reactor.conf,内容
```
reactor:
 - 'testtag':                    ##接收到的tag
   - /srv/reactor/start.sls
- /srv/reactor/monitor.sls
 - 'test1*tag':                  ##接收到的tag，支持通配符
   - /srv/reactor/other.sls
```
2.建立reactor响应sls文件
/srv/reacter/start.sls文件内容
```
{% if data['id'] == 'mysql1' %}
delete_file:
 cmd.cmd.run:
   - tgt: 'G@os:CentOS'
- expr_form: compound
- arg:
 - rm -rf /tmp/*
{% endif %}
```
/srv/reactor/other.sls文件内容
```
{% if data['data']['state'] == 'refresh' %}
overstate_run:
 runner.state.over
{% endif %}
```

下面来解释一下这两个文件，reacter的sls文件是支持jinja的，所以第一行是通过jinja来判断，reacter的sls支持两个变量data和tag, 
data是接受事件的那个字典，tag就是事件的tag，所以第一行的判断就很好理解了，第二行是id，可以随意起，第三行是要运行的执行模块或者runner，
如果是执行模块，以cmd.开始，如果是runner则以runner.开始，可执行模块执行需要target，所以- tat:后面跟的就是可执行模块的target,- expr_form
指的target的匹配方式，- arg是只执行模块函数的参数，runner一般不需要这些。
所以第一个示例相当于执行了salt -C 'G@mysql1' cmd.run 'rm -rf /tmp/*' 第二个相当于执行了 salt-run state.over
