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

* ### 建立测试的表和表分区

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
  KEY `readtime` (`read_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

在测试的test表中，主键为id和read\_time，设置read\_time为主键是为了能够以read\_time作为表分区的关键字段，同时建立了一个表分区p20170801 ，该分区的值为read\_time的天数小于2017-08-01。

```
ALTER TABLE test PARTITION BY RANGE (TO_DAYS(read_time))
(
    PARTITION p20170801 VALUES LESS THAN (TO_DAYS('2017-08-01'))
)
```

执行上述语句后，我们可以在naviCat中的test表“设计表-》选项”中查看到分区信息，如图所示：

![](/assets/partion_create.png)

* ### 创建追加表分区的存储过程

  创建一个Set\_Partition存储过程，完成表后面每半个月追加一次表空间的功能，输入参数为数据库名称和表名称。

    CREATE DEFINER = `root`@`%` PROCEDURE `NewProc`(in dbname varchar(512), in tablename varchar(512))
    begin

    /* 事务回滚，其实放这里没什么作用，ALTER TABLE是隐式提交，回滚不了的。*/
    declare exit handler for sqlexception rollback;
    start TRANSACTION;

    set @_dbname = dbname;
    set @_tablename = tablename;

    /* 到系统表查出这个表的最大分区，得到最大分区的日期。在创建分区的时候，名称就以日期格式存放，方便后面维护 */
    set @maxpartition = Concat("select REPLACE(partition_name,'p','') into @P12_Name from 
       INFORMATION_SCHEMA.PARTITIONS where TABLE_SCHEMA='",@_dbname,"' and table_name='",
       @_tablename,"' order by partition_ordinal_position DESC limit 1;");
    PREPARE stmt2 FROM @maxpartition;
    EXECUTE stmt2;

/_ 判断最大分区的时间段，如果是前半个月的，那么根据情况需要加13,14,15,16天  
   如果是后半个月的，那么直接加15天。 +0 是为了把日期都格式化成YYYYMMDD这样的格式_/  
    IF \(DAY\(@P12\_Name\)&lt;=15\) THEN  
       CASE day\(LAST\_DAY\(@P12\_name\)\)  
          WHEN 31 THEN set @Max\_date= date\(DATE\_ADD\(@P12\_Name+0,INTERVAL 16 DAY\)\)+0 ;  
          WHEN 30 THEN set @Max\_date= date\(DATE\_ADD\(@P12\_Name+0,INTERVAL 15 DAY\)\)+0 ;  
          WHEN 29 THEN set @Max\_date= date\(DATE\_ADD\(@P12\_Name+0,INTERVAL 14 DAY\)\)+0 ;   
          WHEN 28 THEN set @Max\_date= date\(DATE\_ADD\(@P12\_Name+0,INTERVAL 13 DAY\)\)+0 ;   
       END CASE;  
    ELSE  
       set @Max\_date= date\(DATE\_ADD\(@P12\_Name+0, INTERVAL 15 DAY\)\)+0;  
    END IF;

/_ 修改表，在最大分区的后面增加一个分区，时间范围加半个月 _/  
    SET @s1=concat\('ALTER TABLE ',@\_tablename,' ADD PARTITION \(PARTITION p',  
    @Max\_date,' VALUES LESS THAN \(TO\_DAYS \(''',date\(@Max\_date\),'''\)\)\)'\);  
    PREPARE stmt2 FROM @s1;  
    EXECUTE stmt2;  
    DEALLOCATE PREPARE stmt2;

/_ 提交 _/  
    COMMIT ;  
 end;



