salt 每次运行任务是都会将作业发布到pub-sub总线，minion会对任务做出相应，为了区分不同任务，salt master每次发布一个任务都会为该任务创建一个jobid,


master端默认会缓存 **24** 小时内的所有job的详细操作，缓存目录为   
`saltmaster:/var/cache/salt/master/jobs`

minion端每次执行任务都会在     
`minion1:/var/cache/salt/minion/proc`  
目录创建以jobid为名称的文件。任务执行完毕后文件会被删除

例子： 
`salt 'minion1' cmd.run "sleep 10"    `                ##master上

`string /var/cache/salt/minion/proc/20180524131140750483 `   ##minion上，201805xxxx 也就是jobid


####  管理job
#####  方法一：salt-run 命令  
来管理job,这种方式其实是通过runner系统对job管理  

`salt --async '*' test.ping `                  //查看执行jobid
`salt -v '*' test.ping  `                      
Executing job with jid 20180524130412535360

`salt-run jobs.lookup_jid 20180524130359383439`

#####  方法二：salt util模块管理
另一种管理job方式就是通过salt util模块管理  
先执行一个长时间执行的命令   
```
salt 'minion1' cmd.run "sleep 50;echo hello"   
Exiting gracefully on Ctrl-c
```

打开另一个master窗口 
`salt '*' saltutil.find_job 20180524132040889672`
`salt '*' saltutil.kill_job 20180524132040889672`

######  saltutil模块中的job管理方法

saltutil.running #查看minion当前正在运行的jobs  
saltutil.find_job<jid> #查看指定jid的job(minion正在运行的jobs)  
saltutil.signal_job<jid> <single> #给指定的jid进程发送信号  
saltutil.term_job <jid> #终止指定的jid进程(信号为15)  
saltutil.kill_job <jid> #终止指定的jid进程(信号为9)  
salt runner中的job管理方法  
salt-run jobs.active#查看所有minion当前正在运行的jobs(在所有minions上运行saltutil.running)  
salt-run jobs.lookup_jid<jid>

结束正在执行的任务    
salt 'minion1' cmd.run "ping www.baidu.com"     ##master上 
找到该任务   salt-run jobs.active    
    
salt 'minion1' saltutil.term_job 20180525125237055724   
