## 采集服务部署

基于node.js开发的采集服务程序，暂时只支持modbus TCP协议来对网关进行采集，并存储到Mysql数据库中，同时我们开发了对采集服务的配置程序集成在一个应用里面，下面主要是针对采集服务在windows端的配置工作。

### 1.安装node.js

32 位安装包下载地址 :[https://nodejs.org/dist/v6.10.1/node-v6.10.1-x86.msi](https://nodejs.org/dist/v6.10.1/node-v6.10.1-x86.msi)

64 位安装包下载地址 :[https://nodejs.org/dist/v6.10.1/node-v6.10.1-x64.msi](https://nodejs.org/dist/v6.10.1/node-v6.10.1-x64.msi)

本文实例以 v6.10.1版本为例，其他版本类似， 安装步骤：

步骤 1 : 双击下载后的安装包，如下所示：

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step1.png "install-node-msi-version-on-windows-step1")

步骤 2 : 点击以上的Run\(运行\)，将出现如下界面：

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step2.png "install-node-msi-version-on-windows-step2")

步骤 3 : 勾选接受协议选项，点击 next（下一步） 按钮 :

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step3.png "install-node-msi-version-on-windows-step3")

步骤 4 : Node.js默认安装目录为 "C:\Program Files\nodejs\" , 你可以修改目录，并点击 next（下一步）：

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step4.png "install-node-msi-version-on-windows-step4")

步骤 5 : 点击树形图标来选择你需要的安装模式 , 然后点击下一步 next（下一步）

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step5.png "install-node-msi-version-on-windows-step5")

步骤 6 :点击 Install（安装） 开始安装Node.js。你也可以点击 Back（返回）来修改先前的配置。 然后并点击 next（下一步）：

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step6.png "install-node-msi-version-on-windows-step6")

安装过程：

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step7.png "install-node-msi-version-on-windows-step7")

点击 Finish（完成）按钮退出安装向导。

![](http://www.runoob.com/wp-content/uploads/2014/03/install-node-msi-version-on-windows-step8.png "install-node-msi-version-on-windows-step8")

检测Node.js是否安装成功

![](/assets/node_install2.png)

