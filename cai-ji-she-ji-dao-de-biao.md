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

* ### 能耗类型表

  能耗类型表，主要是在电表的能耗进行逻辑上分类，这样以后就可以按照能耗类型进行统计。

      CREATE TABLE `usage` (
        `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
        `type` tinyint(4) DEFAULT NULL COMMENT '能耗类型',
        `name` char(255) DEFAULT NULL COMMENT '能耗名称',
        `description` varchar(255) DEFAULT NULL COMMENT '描述信息',
        PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='电能的用途的分类';

* ### 网关表

网关表主要用于通讯的表，通讯协议暂时有modbus和mqtt协议两种协议的形式，如果网关通过modbus来上传数据，那么我们就对每个是modbus协议的网关建立通讯链路，然后再根据电表的modbus地址表的信息定时轮询该链路上modbus地址值。

    CREATE TABLE `gw` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `gw_id` int(11) NOT NULL COMMENT 'id号',
      `type` enum('mqtt','modbus') NOT NULL DEFAULT 'modbus' COMMENT '通讯协议',
      `name` varchar(50) DEFAULT NULL COMMENT '网关型号或者安装位置信息',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='网关的配置主要用于通讯，网关数就是通讯链路数';

* ### modbus网关协议配置

该表是针对网关的上传协议是modbus来的，其中gw\_id字段来源与网关表的gw\_id，而IP地址是modbus从站的IP地址，端口号默认值502，也是modbus的默认端口号。

    CREATE TABLE `gw_modbus` (
      `id` int(11) NOT NULL,
      `gw_id` int(11) NOT NULL COMMENT '网关id',
      `IP` varchar(20) NOT NULL COMMENT 'modbus通讯的地址',
      `Port` int(10) unsigned DEFAULT '502' COMMENT 'modbus通讯的端口号',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='电表在网关中的通讯参数配置';

* ### 电表基础信息表

该表是整个数据库表中的桥梁作用，其中关联的gw\_id是网关的id，也就是通讯功能的表；area_id是区域id，也就是该电表的地理位置_信息；usage

    CREATE TABLE `ammeter_basic` (
      `id` int(11) NOT NULL,
      `ammeter_id` varchar(255) NOT NULL COMMENT '电表号',
      `gw_id` int(11) NOT NULL COMMENT '网关id',
      `area_id` int(11) NOT NULL COMMENT '区域id',
      `usage_id` int(11) DEFAULT NULL COMMENT '能耗分类id',
      `desc` varchar(255) DEFAULT NULL COMMENT '描述信息',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='电表的基本信息表';





