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



