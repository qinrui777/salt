#### returner 是什么，有什么用？

默认所有minion返回的值都会发送到master端，我们可以看到，returner就是让Minion把返回的值发给其它地方，如MySQL,MongoDB、Redis、Memcache等。  
通过Return我们可以对SaltStack的每次操作进行记录，对以后日志审计提供了数据来源。目前官方已经支持30种Return数据存储与接口，我们可以很方便地配置与使用它。
当然也支持自己定义的Return。

查看对应minion上支持的returner类型
`salt 'minion1' sys.list_returners`  

<img src="https://github.com/qinrui777/salt/blob/master/images/salt_returner_01.png" width="600">



**master** 上 `salt 'minion1' cmd.run "echo hahahaha" --return syslog`

**minion1** 上 `tailf /var/log/syslog`  

<img src="https://github.com/qinrui777/salt/blob/master/images/salt_return_syslog01.png" width="600">

#### 配置returner
minion 上的 /etc/salt/minion



操作之前，我们需要在我们的minion端安装python的redis模块
```
wget –no-check-certificate https://pypi.python.org/packages/source/r/redis/redis-2.8.0.tar.gz
tar -zvxf redis-2.8.0.tar.gz
mv redis-2.8.0 python-redis-2.8.0
cd python-redis-2.8.0
python setup.py install
```
