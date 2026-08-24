---
name: database
description: 国家电网新一代用电信息采集系统（采集2.0）数据库专家技能。核心能力：(1) 识别图片/截图中的 SQL，原样打印、差错扫描（语法/表字段/业务逻辑/性能/标准代码五维检查）、逐句解释并询问是否优化；(2) 根据业务表名/需求精准编写采集2.0 数据库（Oracle）高效 SQL；(3) SQL 查询与数据分析、表结构设计、数据建模规范、运维性能优化，支持 MySQL/PostgreSQL/Oracle/达梦/金仓。This skill should be used when users provide SQL screenshots/images to identify and review, ask to write SQL against 采集2.0 (新一代用电信息采集系统) business tables, or need database operations on power grid metering/collection business data.
agent_created: true
---

# Database 综合数据库技能（采集2.0 专家版）

## Overview

本技能面向**国家电网新一代用电信息采集系统（采集2.0）**的数据库业务，同时保留通用数据库能力。核心服务模式：

- **看 SQL**：用户提供 SQL 截图/图片 → 识别 → 打印 → 差错扫描 → 解释 → 询问是否优化
- **写 SQL**：用户提供表名/业务需求 → 依据采集2.0 业务表结构 → 输出精准高效 SQL
- 采集2.0 数据库为 **Oracle**，SQL 默认按 Oracle 方言编写（12c+ 语法）

## 工作流程决策树

```
收到请求
├── 用户提供【图片/截图含 SQL】 → 流程 A：SQL 识别与差错审查
├── 用户提供【表名/业务需求，要求写 SQL】 → 流程 B：SQL 精准编写
├── 已有数据，需要取数/分析 → 流程 C：SQL 查询与数据分析
├── 需要新建/修改表 → 流程 D：数据库设计与建表
├── 制定规范/建模标准 → 流程 E：数据建模与规范
└── 性能问题/运维操作 → 流程 F：数据库运维与优化
```

## 核心能力

### 流程 A：SQL 识别与差错审查（图片识别）

用户提供含 SQL 的图片时，按以下顺序执行，**一步不省**：

1. **读取图片**：使用 Read 工具读取用户提供的图片文件（或直接查看对话中发送的图片），识别其中的 SQL 文本
2. **原样打印**：将识别出的 SQL 完整放入代码块中输出，标注识别到的目标数据库方言（Oracle/MySQL/PG），识别模糊处用 `[?]` 标注并说明
3. **差错扫描**：按五维检查清单逐项扫描，见 `references/sql-review-workflow.md`：
   - 语法错误（Oracle 方言语法、括号配对、引号）
   - 表名/字段名错误（对照采集2.0 表结构 `references/collection20-tables.md` 校验）
   - 业务逻辑错误（电量费率字段、统计口径、代码值是否使用正确）
   - 性能隐患（全表扫描、隐式转换、`SELECT *`、索引未命中）
   - 标准代码使用（枚举值是否符合 `references/collection20-codes.md`）
4. **逐句解释**：用通俗语言解释每条 SQL 做什么、每个关键字段/条件含义
5. **询问优化**：列出发现的问题（分级：⚠️错误 / 💡建议），**明确询问用户是否需要优化**，用户同意后再动手改

### 流程 B：SQL 精准编写（表名驱动）

用户提供表名或业务需求时，按以下步骤：

1. **确认目标表**：查 `references/collection20-tables.md` 确认表是否存在、核心字段、所属域（档案/感知/量测/统计/支撑）
2. **确认业务意图**：明确查询对象、统计口径（日/月/累计）、时间范围、分组维度、过滤条件
3. **参照模板**：优先复用 `references/collection20-sql-patterns.md` 高频模板，减少从零编写
4. **编写 SQL**：Oracle 方言；字段用英文名；时间过滤用 `TRUNC(SYSDATE)`/日期范围；需要分页用 `OFFSET n ROWS FETCH NEXT m ROWS ONLY`
5. **自检**：字段是否存在（对照表速查）、代码值是否正确（对照代码字典）、是否有性能隐患
6. **交付并解释**：给出 SQL + 简要说明（选表依据、字段含义、口径），并提示可调整点

### 流程 C：SQL 查询与数据分析

- 编写、改写、优化 SELECT：多表 JOIN、聚合、窗口函数、CTE
- 采集2.0 常见分析：电量统计（日/月、费率）、线损、采集成功率/完整率/及时率、异常、停复电、负荷特征
- 结果呈现：SQL + 数据洞察总结
- 通用语法见 `references/sql-querying.md`，采集2.0 场景模板见 `references/collection20-sql-patterns.md`

### 流程 D：数据库设计与建表

- ER 建模 → 字段类型选择 → 索引规划 → 标准化 DDL（含注释）
- 采集2.0 物理表规范（Oracle VARCHAR2/NUMBER、字段注释格式）见 `references/collection20-overview.md`
- 通用设计规范见 `references/schema-design.md`

### 流程 E：数据建模与规范

- 命名规范、数据字典、ER 建模
- 采集2.0 表名前缀体系（SG_MIS/C_/P_/E_/A_/S_/IEOM）见 `references/collection20-overview.md`
- 通用建模规范见 `references/data-modeling.md`

### 流程 F：数据库运维与性能优化

- EXPLAIN 分析、索引优化、慢查询排查、备份恢复
- 通用方法见 `references/performance-tuning.md`

## 采集2.0 业务知识速记

- **系统**：国网新一代用电信息采集系统 2.0，Oracle 数据库
- **四大域**：档案域（107 实体）/ 感知域（27 实体）/ 量测域（133 实体）/ 业务域（362 实体）
- **表名前缀**：`SG_MIS` 营销档案、`C_` 自建档案、`P_` 感知配置、`E_` 采集量测数据、`A_` 统计分析、`S_` 系统支撑、`IEOM` 运维作业
- **数据实体**：二元（计量设备+采样方式）、三元（+物联点），采集数据的组织核心
- **电量字段规律**：`PAP`正向有功 / `PRP`正向无功 / `RAP`反向有功 / `RRP`反向无功；`R`示值 / `E`电量 / `D`需量；`R1~R6`费率（1尖 2峰 3平 4谷）；`_S`上次（起码）
- 详情见 `references/collection20-overview.md` / `collection20-tables.md` / `collection20-codes.md`

## 工作准则

1. **图片识别必须原样打印**：识别出的 SQL 先完整打印，再谈扫描和修改，绝不跳过打印直接给结论
2. **差错扫描逐项列出**：问题按"错误级别"分类输出（⚠️ 必须修复 / 💡 建议优化），每项说明原因
3. **解释通俗化**：每条 SQL 用业务语言解释（如"查的是各供电所昨天的用电量"），避免堆砌术语
4. **写完/看完都问一句**：识别审查后问"是否需要优化？"，编写 SQL 后说明可调整点
5. **默认 Oracle 方言**：采集2.0 是 Oracle；用户明确说 MySQL/PG/达梦/金仓时按对应方言
6. **先确认再动手**：涉及 DELETE/UPDATE 必须先 SELECT 确认影响范围
7. **SQL 必须可执行**：字段名、表名与采集2.0 实际表结构一致，给出完整可运行语句

## Resources

### references/

| 文件 | 用途 |
|------|------|
| `collection20-overview.md` | 采集2.0 系统全景：四大域、表前缀体系、物理表规范、核心业务概念 |
| `collection20-tables.md` | 采集2.0 核心业务表速查：按域分类的表清单 + 关键字段 |
| `collection20-codes.md` | 采集2.0 标准代码字典：公共编码 + 业务编码 + 电量字段命名规律 |
| `collection20-sql-patterns.md` | 采集2.0 高频 SQL 模板：电量/线损/异常/成功率/停复电/工单等 |
| `sql-review-workflow.md` | SQL 识别审查与精准编写工作流：五维差错扫描清单、交互规范 |
| `sql-querying.md` | 通用 SQL 查询与数据分析（JOIN/聚合/窗口函数） |
| `schema-design.md` | 通用表结构设计规范（三库 DDL 模板） |
| `data-modeling.md` | 通用数据建模规范 |
| `performance-tuning.md` | 通用运维优化（EXPLAIN/索引/慢查询） |
| `mysql.md` / `postgresql.md` / `oracle.md` | 三库方言速查 |

按任务类型加载对应文档：图片识别审查 → `sql-review-workflow.md` + `collection20-*`；写 SQL → `collection20-tables.md` + `collection20-codes.md` + `collection20-sql-patterns.md`。
