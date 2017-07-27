## 表设计

在采集服务中涉及到需要存储的信息有区域信息、能耗分类信息和网关通讯配置以及实时表和历史表数据等，下面就是该服务涉及到的表的结构说明。

* ### 区域表

区域表主要是存储采集的位置信息的，也就是电表的位置信息，是一个树形结构的信息表，存储了节点id和父节点id和名称信息。

    CREATE TABLE `area` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `node_id` int(11) NOT NULL COMMENT '节点id',
      `parent_id` int(11) NOT NULL COMMENT '父节点id',
      `name` varchar(255) NOT NULL COMMENT '节点名称',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;



