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

该表是整个数据库表中的桥梁作用，其中关联的gwid是网关的id，也就是通讯功能的表；areaid是区域id，也就是该电表的地理位置信息；usage\_id是电表的能耗类型。这几个字段在以后的查询统计和通讯方面都十分重要。

    CREATE TABLE `ammeter_basic` (
      `id` int(11) NOT NULL,
      `ammeter_id` varchar(255) NOT NULL COMMENT '电表号',
      `gw_id` int(11) NOT NULL COMMENT '网关id',
      `area_id` int(11) NOT NULL COMMENT '区域id',
      `usage_id` int(11) DEFAULT NULL COMMENT '能耗分类id',
      `desc` varchar(255) DEFAULT NULL COMMENT '描述信息',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='电表的基本信息表';

* ### modbus协议的电表地址表

该表是配置电表的各个信息项在modbus协议上传的网关的位置信息，我们通过modbus 协议向网关查询该电表的信息项。

    CREATE TABLE `ammeter_mdaddr` (
      `ID` int(64) unsigned NOT NULL AUTO_INCREMENT,
      `ammeter_id` int(11) NOT NULL COMMENT '关联ammeter_basic中的id',
      `Slave_id` int(11) DEFAULT NULL COMMENT 'modbus的单元号',
      `FC` tinyint(4) DEFAULT NULL COMMENT 'modbus功能码',
      `va_addr` bigint(64) DEFAULT NULL COMMENT 'A相电压地址',
      `vb_addr` bigint(64) DEFAULT NULL COMMENT 'B相电压地址',
      `vc_addr` bigint(64) DEFAULT NULL COMMENT 'C相电压地址',
      `ca_addr` bigint(64) DEFAULT NULL COMMENT 'A相电流地址',
      `cb_addr` bigint(64) DEFAULT NULL COMMENT 'B相电流地址',
      `cc_addr` bigint(64) DEFAULT NULL COMMENT 'C相电流地址',
      `pat_addr` bigint(64) DEFAULT NULL COMMENT '正向有功地址',
      `prt_addr` bigint(64) DEFAULT NULL COMMENT '正向无功地址',
      `nat_addr` bigint(64) DEFAULT NULL COMMENT '反向有功地址',
      `nrt_addr` bigint(64) DEFAULT NULL COMMENT '反向无功地址',
      `cos_a_addr` bigint(64) DEFAULT NULL COMMENT 'A相功率因素地址',
      `cos_b_addr` bigint(64) DEFAULT NULL COMMENT 'B相功率因素地址',
      `cos_c_addr` bigint(64) DEFAULT NULL COMMENT 'C相功率因素地址',
      `pat_sharp_addr` bigint(64) DEFAULT NULL COMMENT '正向有功平地址',
      `pat_peek_addr` bigint(64) DEFAULT NULL COMMENT '正向有功尖地址',
      `pat_flat_addr` bigint(64) DEFAULT NULL COMMENT '正向有功峰地址',
      `pat_valley_addr` bigint(64) DEFAULT NULL COMMENT '正向有功谷地址',
      `hz_addr` bigint(64) DEFAULT NULL COMMENT '电网频率地址',
      `v_rate` float DEFAULT NULL COMMENT '电压倍数',
      `c_rate` float DEFAULT NULL COMMENT '电流倍数',
      PRIMARY KEY (`ID`),
      KEY `node_id_idx` (`ID`),
      KEY `ammater_id` (`ammeter_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='电表在网关中的modbus位置信息';



    CREATE TABLE `ammeter_real` (
      `ID` bigint(64) unsigned NOT NULL AUTO_INCREMENT,
      `ammater_id` varchar(255) NOT NULL COMMENT '电表号',
      `va` float DEFAULT NULL COMMENT 'A相电压',
      `vb` float DEFAULT NULL COMMENT 'B相电压',
      `vc` float DEFAULT NULL COMMENT 'C相电压',
      `ca` float DEFAULT NULL COMMENT 'A相电流',
      `cb` float DEFAULT NULL COMMENT 'B相电流',
      `cc` float DEFAULT NULL COMMENT 'C相电流',
      `pat` float DEFAULT NULL COMMENT '正向有功总电量',
      `prt` float DEFAULT NULL COMMENT '正向无功总电量',
      `nat` float DEFAULT NULL COMMENT '反向有功总电量',
      `nrt` float DEFAULT NULL COMMENT '反向无功总电量',
      `pat_sharp` float DEFAULT NULL COMMENT '正向有功平',
      `pat_peek` float DEFAULT NULL COMMENT '正向有功尖',
      `pat_flat` float DEFAULT NULL COMMENT '正向有功峰',
      `pat_valley` float DEFAULT NULL COMMENT '正向有功谷',
      `cos_a` float DEFAULT NULL COMMENT 'A相功率因数',
      `cos_b` float DEFAULT NULL COMMENT 'B相功率因数',
      `cos_c` float DEFAULT NULL COMMENT 'C相功率因数',
      `hz` float DEFAULT NULL COMMENT '电网频率',
      `read_time` datetime NOT NULL COMMENT '读取时间',
      PRIMARY KEY (`ID`,`read_time`),
      KEY `read_time` (`read_time`),
      KEY `e_num` (`ammater_id`),
      KEY `enum_and_readtime` (`read_time`,`ammater_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=17023 DEFAULT CHARSET=utf8 COMMENT='data';




