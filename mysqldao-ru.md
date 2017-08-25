## MySQL数据库导入

在安装好的数据库中，安装Navicat  for MySQL程序，以便进行数据库的表结构和事件函数等的导入。

![](/assets/createdb.png)  
在collector数据库中右键，弹出的菜单，点击“打开数据库”选项，打开数据库后就可以运行sql文件来导入数据库了。

在collector数据库中右键，弹出的菜单，点击“运行SQL文件”选项，弹出对话框，让你选择sql文件来导入。

![](/assets/importsql.png)

点击“开始”按钮就可以完成数据库的导入工作了，导入成功后就可以看到提示文字。

![](/assets/importdbok.png)



### 授权给root用户

权限问题，授权 给 root  所有sql 权限

```
mysql> grant all privileges on *.* to root@"%" identified by ".";
Query OK, 0 rows affected (0.00 sec)


mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```



