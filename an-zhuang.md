## 安装mysql注意事项

在选择mysql安装的时候，要确定操作系统是32位还是64位操作系统，同时要注意操作系统是windows还是linux，安装成功后有些地方还是需要修改。

* ### 更改MySQL数据库存储路径

首先在windows的服务中停止掉mysql，在cmd中键入如下命令就可以停止mysql的服务了。

```
sc stop MySQL57
```

![](/assets/stop_mysql.png)

将C:\ProgramData\MySQL\MySQL Server 5.7\Data目录下的文件全部拷贝至一个比较大的磁盘中，如拷贝至F盘中

![](/assets/move_mysql.png)

打开C:\ProgramData\MySQL\MySQL Server 5.7\my.ini文件，修改里面的datadir项为迁移后的地址，

![](/assets/datadir.png)

重启mysql就可以了

```
sc start MySQL57
```

* ### 开启事件任务

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

* ### 开启创建函数权限

  如果在create function的时候有 1418的错语的时候，可以直接在配置文件my.cnf中添加如下行\[mysqld\] log\_bin\_trust\_function\_creators=1;

![](/assets/1418.png)

### 授权远程连接数据库

在安装mysql的机器上运行：

```
mysql -u root -p
```

然后输入数据库root用户的密码:luomi，再给远程授权

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'luomi' WITH GRANT OPTION;
```



