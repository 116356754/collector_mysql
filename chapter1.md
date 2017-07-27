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

有两种设定计划任务的方式：

1. AT 时间戳，用来完成单次的计划任务。

2. EVERY 时间（单位）的数量时间单位\[STARTS 时间戳\] \[ENDS时间戳\]，用来完成重复的计划任务。

在重复的计划任务中，时间（单位）的数量可以是任意非空（Not Null）的整数式，时间单位是关键词：YEAR，MONTH，DAY，HOUR，MINUTE 或者SECOND。

因为我们需要每半个月就需要创建一个表分区，所以我们需要创建EVERY类型的计划任务，任务的内容为调用存储过程，该存储过程是创建新的表分区。

![](/assets/cron_define.png)

每隔15天就需要调用一次存储过程，实现新增表分区的功能。

![](/assets/event_plan.png)

* ### 建立测试的表和数值表分区

建立分区的优点很多，主要有：

1. 可以把一些归类的数据放在一个分区中，可以减少服务器检查数据的数量加快查询。
2. 方便维护，通过删除分区来删除老的数据。
3. 根据查找条件，也就是where后面的条件，查找只查找相应的分区不用全部查找了。

但是使用表分区也有约束条件：

1. 每张表最大分区数为1024。
2. 所有的主键或者唯一索引必须被包含在分区表达式中。
3. 不能使用任何外键约束。

为了测试我们，新建了一个表test，DDL语句如下：

```SQL
CREATE TABLE `test` (
  `id` int(11) NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  `read_time` date NOT NULL,
  PRIMARY KEY (`id`,`read_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE (TO_DAYS(read_time))
(PARTITION p20170801 VALUES LESS THAN (736907) ENGINE = InnoDB)
```

在测试的test表中，主键为id和read\_time，设置read\_time为主键是为了能够以read\_time作为表分区的关键字段，同时建立了第一个表分区p20170801 ，该分区的值为read\_time的天数小于2017-08-01

