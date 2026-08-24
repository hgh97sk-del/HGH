# 采集2.0 系统全景（新一代用电信息采集系统）

来源：《GW-MD01~03 物理模型设计说明书 V0.996》与《2.0业务表结构.xls》

## 一、系统定位

国家电网**新一代用电信息采集系统（采集2.0）**，数据库为 **Oracle**。核心职责：对电能表、采集终端、配电设备等对象进行数据采集、计量、统计分析，支撑抄表、线损、异常诊断、停复电分析、分布式电源管理等业务。

## 二、物理模型四大域

| 域 | 实体数 | 职责 | 表前缀 |
|----|-------|------|--------|
| 档案域 | 107 | 营销/自建设备档案：客户、计量点、设备、管网、台区 | SG_MIS、C_、A_ARCH_ |
| 感知域 | 27 | 采集配置：数据实体、采集任务/策略、参数、遥测/控制关系 | P_ |
| 量测域 | 133 | 采集数据：日冻结示值/需量、曲线、事件 | E_ |
| 业务域 | 362 | 统计分析：电量、负荷、线损、异常、停电、采集质量 | A_ |

### 档案域 8 个子域
档案关系拓展（管理单位上级/子集）、档案管理（自建物联点/逆变器/客户侧设备/传感器等）、公共档案（代码分类/管理单位）、管网档案（管线/枢纽站/配送站/调压设备）、计量支撑档案（工单/网格/电厂/设备装拆）、客户档案（能源客户/用电户/发电户/计量点/连接对象）、设备运行档案（测量点/物联点/计度器示数）、设备资产档案（电能表/采集终端/互感器/通信设备）

### 量测域 7 个子域
量测数据、电能表事件、计算点(总加组)事件、智能断路器事件、终端事件、终端/电能表事件、智能物联电表模组事件

### 业务域 16 个子域
基础分析、数据采集管理、档案分析、设备装接、控制管理、采集/用电异常管理、停电分析、线损管理、时钟管理、分布式电源管理、反窃电管理、设备升级管理、终端在线监测、设备运行状态监测、采集质量管理、台区负荷监测

## 三、表名前缀体系（重要）

| 前缀 | 含义 | 示例 |
|------|------|------|
| `SG_MIS` | 营销基础档案（同步自营销系统） | SG_MIS.CUST 能源客户、SG_MIS.ELEC_CONS_CUST 用电户、SG_MIS.INST_ELEC_CONS 计量点、SG_MIS.WK_ORDER 工单 |
| `C_` | 采集自建档案 | C_IOT_POINT_SELF_BUILT 自建物联点、C_INVERTE 逆变器、C_BUS_BAR 母线 |
| `A_ARCH_` | 全量档案信息（SGAMI_ARCH schema） | A_ARCH_CONS_FULL_INFO 用电户全量档案、A_ARCH_INST_FULL_INFO 安装点全量档案 |
| `P_` | 感知域配置 | P_DATA_ENTITY_DUAL 二元数据实体、P_COLL_TASK 采集任务、P_COLL_STRATEGY 采集策略 |
| `E_` | 量测域采集数据 | E_MP_READ_DAY 日冻结电能示值、E_MP_DEMAND_DAY 日冻结需量 |
| `A_BA_` | 基础分析统计 | A_BA_CONS_ENERGY_DAY 用户日电量、A_BA_INST_ENERGY_MON 计量点月电量 |
| `A_COLL_` | 采集质量管理 | A_COLL_DEV_STAT_DAY 采集成功率日统计、A_COLL_COMPLETE_STAT_DAY 采集完整率 |
| `A_LL_` | 线损管理 | A_LL_DIST_STAT_DAY 配送站线损日统计 |
| `A_PO_` | 停电分析 | A_PO_AREA_DET 区域停复电明细、A_PO_ACCU_REC 停电事件准确性分析 |
| `A_ABNOR_` | 异常管理 | A_ABNOR_INFO 异常结果表、A_ABNOR_DIAG_REC 异常诊断记录 |
| `A_LM_` | 停电/负荷监测 | A_LM_USER_FREQ_DUR_STAT_DAY 用户停电次数时长统计 |
| `S_` | 系统支撑 | S_INTERFACE_LOG 接口日志、S_USER_LOGIN_LOG 登录日志、S_COMM_PROTOCOL 通信协议 |
| `IEOM` | 运维作业平台 | IEOM.A_IEOM_WKST_POOL_INFO 作业基础信息 |

## 四、物理表规范

### 字段定义格式（说明书标准）
每个物理表含：约束（主键）说明 + 字段表（中文名称/英文名称/类型/非空/注释）

### 常用类型
- 编码/标识类：`VARCHAR2(16)`（如 MGT_ORG_CODE）、`VARCHAR2(32)`（编号类）
- 名称类：`VARCHAR2(128)` / `VARCHAR2(256)`
- 标志类：`NUMBER(1)`（0/1）
- 枚举类：`NUMBER(3)` / `VARCHAR2(8)`
- 电量/金额类：`NUMBER(16,2)` 或 `NUMBER(18,2)`

### 通用公共字段
| 字段 | 含义 | 说明 |
|------|------|------|
| MGT_ORG_CODE | 管理单位编码 | 几乎每表必有，01总部~05所 |
| DATA_ENTITY_ID | 数据实体标识 | 采集数据表主键，关联 P_DATA_ENTITY |
| DATA_DATE | 数据日期 | 日数据；月数据用 DATA_DATE 存月份 |
| STAT_DATE | 统计日期 | 统计表使用 |
| CALC_TIME | 计算时间 | 统计计算完成时间 |
| CUST_NO | 客户编号 | 营销标准编号 |
| ASSET_NO | 资产编号 | 设备资产编号 |
| VALID_FLAG | 有效标志 | 1有效 0无效 |

## 五、核心业务概念

### 1. 数据实体（DATA_ENTITY）— 采集组织核心
- **二元数据实体** P_DATA_ENTITY_DUAL：`计量设备 + 采样方式` 二元组，如"某电能表 + 日冻结示值"
- **三元数据实体** P_DATA_ENTITY_TERNARY：`计量设备 + 采样方式 + 物联点` 三元组
- 统一视图：`VW_P_DATA_ENTITY`（二元+三元合并）
- 所有量测数据表（E_*）以 DATA_ENTITY_ID 关联数据实体

### 2. 计量点（INST_ELEC_CONS）
- 连接"房产-设备-合同"的桥梁，计费参数的总代表
- 一个计量点对应一个行业（电/水/气/热）；INST_ID 安装点标识
- 安装点分类 instCls：01 用电客户 / 02 关口（台区/变电站/电厂关口）/ 03 发电上网关口

### 3. 管网层级概念
- **配送站 DIST_STA**：即台区/公变，配送站容量、公专标志
- **枢纽站 CNTRL_STA**：变电站级，接入管线、调压设备
- **管线 PIPELINE**：线路，含管线长度、承压、规格
- 关系：枢纽站—管线（CNTRL_STA_PLINE_RELA）、管线—配送站（PLINE_DIST_STA_RELA）

### 4. 设备体系
- 采集终端 ACQ_TRML（专变终端/集中器/采集器/融合终端TTU 等）
- 电能表 ELEC_METER、互感器 IT、计量箱 METER_BOX、物联卡 IO_TCARD
- 物联点 IOT_POINT、测量点 MEAS_POINT、计算点（总加组）CALC_MEAS_POINT

### 5. 电量与费率体系
- 正/反向 × 有/无功 四个方向：PAP/PRP/RAP/RRP
- **6 费率**：R1尖 / R2峰 / R3平 / R4谷 / R5 / R6
- 示值（R 开头，止码=本次、起码=上次 _S）、电量（E 开头）、需量（D 开头）
- 例：PAP_R 正向有功总示值、PAP_E1 尖峰电量、PAP_DEMAND 正向有功总最大需量
- 示值数据类型 readDataType：1正向有功 2正向无功 3反向有功 4反向无功

### 6. 采集质量指标
- **成功率**（A_COLL_DEV_STAT_*）：采集成功数/应采数
- **完整率**（A_COLL_COMPLETE_STAT_*）：数据完整程度
- **及时率**（A_COLL_TIMELY_STAT_*）：按时到达比例
- 失败列表（A_COLL_*_FAIL_LIST）：失败档案，供补召

## 六、三库版本说明

- `GW-MD01 基础采集分册` V0.996（2023-10）为最新版，V0.995（2021）为旧版
- `2.0业务表结构.xls` 中的表与说明书 V0.996 一致；表名前缀（SGAMI_STAT/SGAMI_PROV/SGAMI_SUPPORT/IEOM）为 Oracle schema 名
- 数据字典视图（ALL_TABLES/ALL_TAB_COLUMNS/ALL_INDEXES 等）可用于反向查看表结构
