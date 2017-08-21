## 采集服务部署

基于node.js开发的采集服务程序，暂时只支持modbus TCP协议来对网关进行采集，并存储到Mysql数据库中，同时我们开发了对采集服务的配置程序集成在一个应用里面，下面主要是针对采集服务在windows端的配置工作。

### 1.安装node.js

32 位安装包下载地址 :[https://nodejs.org/dist/v7.9.0/node-v7.9.0-x86.msi](https://nodejs.org/dist/v7.9.0/node-v7.9.0-x86.msi)

64 位安装包下载地址 :[https://nodejs.org/dist/v7.9.0/node-v7.9.1-x64.msi](https://nodejs.org/dist/v7.9.0/node-v7.9.1-x64.msi)

本文实例以 v7.9.0版本为例，其他版本类似， 安装步骤：

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

![](/assets/nodecheck.png)

## 2.拷贝采集程序

在安装完node环境和mysql，就可以部署采集服务程序了

部署其实很简单，只需要将collector\_ui文件夹拷贝到某个目录中，然后解压就可以了，并在终端启动来检测是否正确。

![](/assets/ui.png)在IE浏览器中就可以查看是否正常运行。敲入[http://localhost:5000就可以了，进入了系统的登录界面。](http://localhost:5000就可以了，进入了系统的登录界面。)

![](/assets/login.png)

## 3.利用NSSM部署服务

目前NSSM最新版的是2.23（[下载地址](http://nssm.cc/release/nssm-2.23.zip)），下载之后解压，根据你的系统选择32位和64位的版本，直接在nssm.exe 所在目录运行命令行，输入nssw install +你的服务名，例如：

```bash
nssm install collector_ui
```

之后会显示出GUI界面：

![](http://img.keenwon.com/2014/07/20140708222155_45820.png)

在Path 中选择你安装的node.exe所在位置，Startup directory 选择你的node应用的目录，Argument 输入你的启动文件，例如在D:/work/wangky/collector\_ui运行bin/www （在Startup directory目录执行node ./bin/www ）：

![](/assets/nssm_set.png)

点击Install Service：

![](/assets/nssmok.png)

之后运行：

```
nssm start collector_ui
```

服务已经启动，我刚才的bin/www文件，启动一个http服务器，监听3000端口，现在就可以打开`127.0.0.1:3000`访问了：

![](/assets/nssm_install8.png)

其他的设置可以参考[官方文档](http://nssm.cc/usage)。它的命令行操作也很简单：

```
nssm start <servicename>
nssm stop <servicename>
nssm restart <servicename>
```



