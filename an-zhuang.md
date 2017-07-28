## 安装mysql注意事项

在选择mysql安装的时候，要确定操作系统是32位还是64位操作系统，同时要注意操作系统是windows还是linux，安装成功后有些地方还是需要修改。

* ### 更改MySQL数据库存储路径

首先在windows的服务中停止掉mysql，在cmd中键入如下命令就可以停止mysql的服务了。

```
sc stop MySQL57
```

![](/assets/stop_mysql.png)



