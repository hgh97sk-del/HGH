# 采集2.0 核心业务表速查

按域分类。字段列为常用关键字段；完整字段以《2.0业务表结构.xls》为准。表名可直接用于 SQL。

## 一、档案类表（SG_MIS 营销档案）

| 表名 | 中文名 | 关键字段 |
|------|--------|---------|
| SG_MIS.CUST | 能源客户 | CUST_ID, BP_ID, MGT_ORG_CODE, CUST_NO, CUST_NAME |
| SG_MIS.ELEC_CONS_CUST | 用电户 | ELEC_CONS_CUST_ID, CUST_NO, CUST_NAME, SCY_CAP(保安负荷容量), MGT_ORG_CODE |
| SG_MIS.GPC | 发电户 | GPC_ID, CUST_NO, PLANT_TYPE(电站类型), GC_STAT(发电客户状态) |
| SG_MIS.GEN_CONS_RELA | 发用电户关联 | GEN_CONS_RELA_ID, CUST_ID, GEN_CUST_ID, VALID_FLAG |
| SG_MIS.INST_ELEC_CONS | 计量点-用电 | INST_ID, INST_NO, PIPELINE_ID, DIST_STA_ID, CNTRL_STA_ID, BILLING_UNIT_NO, MR_UNIT_NO |
| SG_MIS.SRV_LOC | 连接对象 | SRV_LOC_ID, SRV_LOC_ADDR, SRV_LOC_NAME, SRV_KIND |
| SG_MIS.CONN_GEN_POWER | 连接发电 | CONN_GEN_POWER_ID, PROTECT_MODE, SRV_KIND |
| SG_MIS.DIST_STA | 配送站（台区/公变） | DIST_STA_ID, 公专标志, 配送站容量 |
| SG_MIS.CNTRL_STA | 枢纽站 | CNTRL_STA_ID, 调压设备数量, 接入管线标识, 设备容量 |
| SG_MIS.PIPELINE | 管线 | PIPELINE_ID, 管线编码, 管线长度, 管线承压, 管线规格 |
| SG_MIS.ACQ_TRML | 采集终端 | 终端资产编号, DEV_MODE, 终端类型 |
| SG_MIS.ELEC_METER | 电能表 | 资产编号, 电能表类型, 额定电压/电流, 准确度等级 |
| SG_MIS.IT / IT_RUN | 互感器 / 互感器运行 | 资产编号, TA/TV 变比 |
| SG_MIS.METER_BOX | 计量箱（柜、屏） | 计量箱资产编号 |
| SG_MIS.MGT_ORG | 管理单位 | MGT_ORG_CODE, MGT_ORG_NAME, DIST_LV(区域层级), 上级管理单位编码 |
| SG_MIS.CODE_CLS / CODE_CLS_VAL | 代码分类 / 代码分类值 | CODE_CLS_ID, CODE_CLS_VAL, CODE_CLS_VAL_NAME |
| SG_MIS.WK_ORDER | 工单 | WK_ORDER_ID, WK_ORDER_NO, WK_ORDER_NAME, WK_ORDER_STAT(01运行/02挂起/03作废), WK_ORDER_TYPE |
| SG_MIS.MR_DATA | 抄表数据 | M_R_DATA_ID, PLAN_NO, MGT_ORG_CODE, CUST_NO, QTY_CHARG_YM(量费年月) |

### 自建档案（C_ 前缀）
| 表名 | 中文名 | 说明 |
|------|--------|------|
| C_IOT_POINT_SELF_BUILT | 自建物联点 | 物联点状态/类型 |
| C_INVERTE | 逆变器档案 | 设备标识、资产编号、状态 |
| C_ACQ_DEV_SELF_BUILT | 自建采集设备 | 设备分类、档案来源 |
| C_CUST_DEV | 客户侧设备 | 类型、类别、运行状态 |
| C_SENSOR | 传感器 | 配网侧设备 |
| C_BREAKER | 智能断路器 | 配网侧 |
| C_BUS_BAR | 母线 | 上级设备资产编号 |
| C_MODULE | 模组 | 资产编号、运行状态 |

### 全量档案（A_ARCH_ 前缀，SGAMI_ARCH schema）
| 表名 | 中文名 | 说明 |
|------|--------|------|
| A_ARCH_CONS_FULL_INFO | 用电户全量档案 | 含网格、区域层级等宽表 |
| A_ARCH_GPC_FULL_INFO | 发电量全量档案 | GEN_MODE, EC_DATE(并网日期), CTRT_CAP(合同容量), INST_CAP(装机容量), ACCS_CAP(接入容量) |
| A_ARCH_INST_FULL_INFO | 安装点全量档案 | 安装点宽表 |
| A_ARCH_METER_FULL_INFO | 电表全量档案 | 电能表宽表 |
| A_ARCH_TMNL_FULL_INFO | 终端全量档案 | 终端宽表 |
| A_ARCH_DIST_FULL_INFO | 台区全量档案统计 | 台区宽表 |
| A_ARCH_INVERTE_FULL_INFO | 逆变器宽表 | 光伏场景 |
| A_ARCH_LOW_VOLT_CUST | 低压拓扑 | 台区-户关系 |
| A_ARCH_MID_VOLT_CUST | 中压拓扑 | 馈线-台区关系 |

## 二、感知类表（P_ 前缀）

| 表名 | 中文名 | 关键字段 |
|------|--------|---------|
| P_DATA_ENTITY_DUAL | 二元数据实体 | DATA_ENTITY_ID, ASSET_NO, DEV_ID, SAMP_TYPE(采样方式), VALID_DATE, INVALID_DATE |
| P_DATA_ENTITY_TERNARY | 三元数据实体 | DATA_ENTITY_ID, IOT_POINT_NO, ASSET_NO, DEV_ID, SAMP_TYPE |
| P_COLL_TASK | 采集任务明细 | 采集任务标识, 采集策略标识, 物联关系标识, 任务起止时间, 终端资产号 |
| P_COLL_STRATEGY | 采集策略 | 策略类型, 采集周期 |
| P_COLL_STRATEGY_RELA_OBJ | 采集策略关联对象 | 关联对象标志/类别 |
| P_COLL_DATA | 采集数据项 | 采集数据分类 |
| P_MPEA_PARA | 抄表参数表 | 下发状态 |
| P_TELEMETER_RELA | 遥测关系 | 遥测点编号, 设备资产编号, 数据实体标识 |
| P_CONTROL_RELA | 控制关系 | 遥控点编号, 设备资产编号 |
| P_TASK_EXEC_REC | 任务执行记录 | 任务执行流水号, 执行结果 |
| P_PARA_SET / P_PARA_ABNOR | 参数设置 / 参数异常 | 参数类型, 设备资产编号 |

## 三、量测类表（E_ 前缀，采集数据）

| 表名 | 中文名 | 关键字段 |
|------|--------|---------|
| E_MP_READ_DAY | 日冻结电能示值 | DATA_ENTITY_ID, DATA_DATE, MGT_ORG_CODE, COMP_RTO(综合倍率), PAP_R/PAP_R1~R6, RAP_R..., 一~四象限无功 |
| E_MP_DEMAND_DAY | 日冻结需量 | DATA_ENTITY_ID, DATA_DATE, COLL_TIME, PAP_DEMAND(正向有功总最大需量), PAP_DEMAND_TIME |
| E_MP_USTAT_DAY | 日冻结电压统计数据 | 数据实体标识, A 相电压最大值等 |
| E_MP_VSTAT_MON | 月冻结电压统计数据 | 电压统计月冻结 |
| E_MP_PHASE_READ_DAY | 日冻结分相电能示值 | 分相正/反向有/无功电能示值 |
| E_MP_UNBALANCE_DAY/MON | 日/月不平衡度越限累计时间 | 电压/流不平衡度越限累计时间 |
| E_CALC_MEAS_EN_DAY | 计算点(总加组)日冻结电能量 | 计算点标识, 日最大/小有功功率 |
| E_GAS_METER_READ_DAY/MON | 日/月冻结气表读数 | 数据实体标识, 结算日累计流量 |
| E_WATER_METER_READ_MON | 月冻结水表读数 | 当前累计流量, 结算日累计流量 |
| E_TMNL_COMM_FLOW_MON | 月冻结终端通信流量 | 当日通信流量 |
| E_METER_PURCHASE_DAY | 日冻结电能表购电信息 | 购电次数, 剩余金额 |

## 四、统计类表（A_ 前缀，核心分析表）

### 电量统计
| 表名 | 中文名 | 说明/关键字段 |
|------|--------|--------------|
| A_BA_CONS_ENERGY_DAY/MON | 用户日/月电量 | CUST_ID, CUST_NO, DATA_DATE, CUST_CLS, ENERGY1~6(6费率电量) |
| A_BA_INST_ENERGY_DAY/MON | 计量点日/月电量 | PAP_E1~E6, RAP_E1~E6, PRP_E, RRP_E |
| A_BA_METER_ENERGY_DAY/MON | 电能表日/月电量 | METER_ID, ASSET_NO, 各费率示值(本次/上次)+电量 |
| A_BA_METER_PHASE_ENERGY_DAY | 电能表分相日电量 | A/B/C 相正反向有功电量 |
| A_BA_INST_PHASE_ENERGY_DAY | 计量点分相日电量 | 分相正/反有/无功电量 + 来源标识 |
| A_BA_IND_ENERGY_DAY/MON | 行业日/月电量 | IND_CLS(行业分类), ENERGY_TYPE, IND_ENERGY |
| A_BA_ORG_ENERGY_DAY | 供电单位电量日统计 | GPQ(发电电量), GCPQ(上网电量), DIST_LV |
| A_BA_VOLTAGE_ENERGY_DAY/MON | 承压日/月电量 | 各电压等级电量 |

### 负荷与特征
| 表名 | 中文名 | 说明 |
|------|--------|------|
| A_BA_LOAD_ORG_STAT_DAY | 单位负荷统计表 | MAX_LOAD_DAY/MIN_LOAD_DAY/MAV_LOAD_DAY |
| A_BA_LOAD_ORG_STAT_HOUR | 单位负荷统计表(小时) | LOAD_HOUR 当前负荷 |
| A_BA_CONS_TRAIT_DAY/MON | 用户行为日/月特征 | MAX_P, MIN_P, AVG_P, 负荷率, 峰谷差率, MAX_PQ |
| A_BA_METER_TRAIT_DAY/MON | 电能表运行日/月特征 | 电压/电流最大/小及不平衡度, 功率最大/小 |
| A_BA_DIST_TRAIT_DAY/MON | 配送站(公变)运行日/月特征 | 各相电压/电流极值, LOAD_MAX_RATE_A/B/C(负载率), AVG_UA/B/C, 渗透率 |

### 采集质量
| 表名 | 中文名 | 说明 |
|------|--------|------|
| A_COLL_DEV_STAT_DAY/MON | 采集成功率日/月统计 | 成功率统计 |
| A_COLL_COMPLETE_STAT_DAY/MON | 采集完整率日/月统计 | 完整率 |
| A_COLL_TIMELY_STAT_DAY/MON | 采集及时率日/月统计 | 及时率 |
| A_COLL_ABNOR_STAT_DAY/MON | 采集异常率日/月统计 | 异常率 |
| A_COLL_*_FAIL_LIST | 各类失败列表 | 电能示值/需量/曲线失败，供补召 |
| A_COLL_TMNL_FLOW_STAT_DAY/MON | 终端流量日/月统计 | UP/DOWN_GPRS_FLOW |

### 异常 / 线损 / 停电
| 表名 | 中文名 | 说明 |
|------|--------|------|
| A_ABNOR_INFO | 异常结果表 | ABNOR_ID, ABNOR_CATEG, ABNOR_TYPE, ASSET_NO, CUST_NO, 异常状态 |
| A_ABNOR_DIAG_REC | 异常诊断记录表 | ABNOR_ID, ABNOR_TYPE, HNDL_TYPE(处理环节), 处理结论/意见 |
| A_ABNOR_HEAD_MONITOR_DAY | 计量装置运行情况日监测明细 | 电能表维度电流过流监测 |
| A_LL_DIST_STAT_DAY | 配送站线损日统计 | 配送站标识, 统计日期, 线损率 |
| A_LL_DIST_STAT_HOUR | 分时线损统计 | 统计频度, 线损率 |
| A_LL_DIST_PHASE_STAT_DAY | 分相线损日统计 | 相别, 线损率 |
| A_PO_AREA_DET | 区域停复电明细 | PO_TIME(停电时间), PR_TIME(复电时间), PO_SCOPE(停电范围), AREA_NO |
| A_PO_ACCU_REC | 停电事件准确性分析记录 | 设备标识, 事件准确性 |
| A_LM_TMNL_METER_EVENT_DET | 终端和电表停上电事件明细 | POWER_OFF_TIME, POWER_ON_TIME, EVENT_TYPE |
| A_LM_USER_FREQ_DUR_STAT_DAY | 用户停电次数和时长统计 | CONS_TYPE, 停电次数/时长 |

## 五、支撑与作业类表

| 表名 | 中文名 | 说明 |
|------|--------|------|
| S_INTERFACE_LOG | 接口日志记录表 | INTERFACE_NAME, CALL_ST/ET, REQ_MSG |
| S_USER_LOGIN_LOG | 用户登录记录表 | OPERATOR_NAME, IP_ADDR, LOGIN_TIME, LOGIN_SRC |
| S_COMM_PROTOCOL / S_COMM_PROTOCOL_INFO | 通信协议(信息)表 | COMM_PRO_CODE, DEV_TYPE, MFR |
| S_METER_LABEL_RESULT | 电能表标签标记结果 | LABEL_NAME, LABEL_VALUE(如线损率) |
| IEOM.A_IEOM_WKST_POOL_INFO | 作业基础信息 | WORK_ORDER_NO, ORDER_TYPE(01采集异常/02用电异常...), ORDER_LV(优先级01I级/02II级) |
| IEOM.A_IEOM_WKST_POOL_RST | 作业子表 | WORK_ORDER_NO, ORDER_STATUS(01待派工/02待反馈...), DEV_SNS |
| IEOM.A_IEOM_KN_SP_FAULT | 异常类型树 | FAULT_CODE(第1位1采集异常2计量异常), FAULT_NAME_TREE |
| SGAMI_STAT.A_INST_APP_WORK_ORDER | 基于营销计划装接工单 | APP_NO, BUS_TYPE(业扩业务类型), FLOW_TYPE(01专变终端装接/02公变/03集中器), LINK_STATUS |
| SGAMI_STAT.A_INST_TMNL_DEBUG | 调试流程 | TMNL_ASSET_NO, DEBUG_LINK, DEBUG_RESULT |

## 六、Oracle 数据字典视图（系统自带）

ALL_TABLES / ALL_OBJECTS / ALL_TAB_COLUMNS / ALL_CONSTRAINTS / ALL_INDEXES / ALL_SYNONYMS / DBA_SOURCE / DBA_SCHEDULER_JOBS / DBA_DEPENDENCIES

可用于反向确认表结构、索引、约束（如：`SELECT * FROM ALL_TAB_COLUMNS WHERE TABLE_NAME='A_BA_CONS_ENERGY_DAY'`）。
