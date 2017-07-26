# Mysql的事件

大约如果有100个电表的话，每分钟采集一条数据存储到数据库中实时表，这样每天大约有15万条数据，这样半个月就有200万条数据，而msyql对于单表达到200万条以上的查询性能会下降很多，所以需要做表分区。由于数据较大，所以决定做定时任务每隔15天执行存储过程自动进行添加分区。

* ### 开启事件

用来执行定时任务，都是被动执行的，事件是因为时间到了触发执行的，所以我们需要开启事情，并设置好计划任务。

查看是否开启：

```SQL
show variables like 'event_scheduler';
```

默认mysql是关闭的

![](/assets/event.png)

如果显示OFF,则输入以下语句开启：

```SQL
set global event_scheduler = on;
```

**虽然这里用set global event\_scheduler = on语句开启了事件，但是每次重启电脑。或重启mysql服务后，会发现，事件自动关闭（event\_scheduler=OFF），所以想让事件一直保持开启，最好修改配置文件，让mysql服务启动的时候开启时间，只需要在C:\ProgramData\MySQL\MySQL Server 5.7\my.ini**配置文件的**\[mysqld\]部分**加上_**event\_scheduler=ON，如下所示：**_

![](/assets/event_default.png)

重启Mysql，此时再调用查询语句，就会发现mysql自动开启了事件

```
show variables like 'event_scheduler';
```

![](/assets/event_on.png)

* ### 创建计划事件

### 



