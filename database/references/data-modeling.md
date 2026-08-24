# 数据建模与规范

数据建模方法论、行业规范与电力业务示例（含配电网、新能源场景）。

## 一、ER 建模要点

### 1. 实体识别与关系表达

```
[变电站] 1 ──── N [配电变压器] N ──── 1 [台区] 1 ──── N [用户]
   │                                        │
   └── N [工单]                             └── N [计量表]
```

- 1:N 用外键挂在 N 侧
- N:M 必须拆中间表（如 用户↔角色 拆 user_role）
- 1:1 极少见，多因拆分宽表，注意确认是否真需要

### 2. 建模误区规避

| 误区 | 后果 | 正确做法 |
|------|------|---------|
| 万能大宽表 | 字段爆炸、扩展困难 | 按业务域拆表，用 JOIN 关联 |
| 过度范式化 | 查询需要 7 表 JOIN，性能差 | 冗余少量高频字段（如 供电所名称）换查询性能 |
| 用 JSON 存结构数据 | 无法索引、统计困难 | 拆成标准字段或子表 |
| 表名/字段名无规范 | 团队协作灾难 | 严格遵循命名规范 |
| 状态字段存中文 | 变更枚举要改数据 | 存数字码 + 注释 + 数据字典 |

## 二、命名与字典规范

### 库表字段命名规范

```
库名：  [业务域]_[子域]       如 power_dispatch / new_energy_project
表名：  [业务实体]           如 substation / transformer / work_order
字段：  [修饰词_][业务名词]   如 is_deleted / total_capacity_kva
外键：  [引用表名]_id         如 org_id / project_id
索引：  idx_[表]_[字段组合]   如 idx_transformer_org
唯一键： uk_[表]_[业务键]      如 uk_work_order_no
```

### 数据字典模板

| 字段名 | 类型 | 允许值/范围 | 是否必填 | 默认值 | 业务含义 | 示例 |
|--------|------|------------|---------|--------|---------|------|
| status | TINYINT | 0待处理/1处理中/2完成/3逾期 | 是 | 0 | 工单状态 | 2 |
| voltage_level | VARCHAR(16) | 0.4kV/10kV/35kV/110kV... | 是 | - | 电压等级 | 10kV |

**要求**：每张业务表配套数据字典；枚举字段枚举值变化时同步更新字典，不直接改数据。

## 三、电力行业建模示例

### 场景 1：配电网台账（馈线-配变-台区-用户）

```sql
-- 馈线
CREATE TABLE feeder (
    id            BIGINT PRIMARY KEY,
    feeder_no     VARCHAR(32) NOT NULL UNIQUE COMMENT '馈线编号',
    substation_id BIGINT NOT NULL COMMENT '所属变电站',
    voltage_level VARCHAR(16) NOT NULL COMMENT '电压等级 10kV/35kV',
    length_km     DECIMAL(10,3) COMMENT '线路长度(km)',
    status        TINYINT DEFAULT 1 COMMENT '1-运行 0-停运'
);

-- 台区
CREATE TABLE station_area (
    id           BIGINT PRIMARY KEY,
    area_no      VARCHAR(32) NOT NULL UNIQUE COMMENT '台区编号',
    feeder_id    BIGINT NOT NULL COMMENT '所属馈线',
    name         VARCHAR(128) NOT NULL,
    capacity_kva DECIMAL(10,2) COMMENT '台区容量(kVA)',
    longitude    DECIMAL(10,6), latitude DECIMAL(10,6)
);

-- 用户（与台区 1:N）
CREATE TABLE power_user (
    id          BIGINT PRIMARY KEY,
    user_no     VARCHAR(32) NOT NULL UNIQUE COMMENT '户号',
    area_id     BIGINT NOT NULL COMMENT '所属台区',
    user_type   TINYINT COMMENT '1-居民 2-工商业 3-农业',
    create_date DATE
);
```

### 场景 2：新能源项目开发

```sql
-- 新能源项目主表
CREATE TABLE new_energy_project (
    id            BIGINT PRIMARY KEY,
    project_no    VARCHAR(32) NOT NULL UNIQUE COMMENT '项目编号',
    project_name  VARCHAR(256) NOT NULL COMMENT '项目名称',
    energy_type   TINYINT COMMENT '1-光伏 2-风电 3-储能 4-充电桩',
    capacity_mw   DECIMAL(12,3) COMMENT '装机容量(MW)',
    status        TINYINT COMMENT '0-储备 1-可研 2-核准 3-在建 4-并网 5-运营',
    invest_amount DECIMAL(18,2) COMMENT '投资金额(元)',
    start_date    DATE, grid_date DATE
);

-- 项目进度里程碑（1:N）
CREATE TABLE project_milestone (
    id         BIGINT PRIMARY KEY,
    project_id BIGINT NOT NULL,
    stage      TINYINT COMMENT '里程碑阶段',
    plan_date  DATE COMMENT '计划完成日',
    actual_date DATE COMMENT '实际完成日',
    remark     VARCHAR(512)
);
```

### 场景 3：工单/逾期跟踪（与工作台数据表呼应）

```sql
CREATE TABLE work_order (
    id           BIGINT PRIMARY KEY,
    order_no     VARCHAR(32) NOT NULL UNIQUE,
    biz_type     TINYINT COMMENT '1-抢修 2-巡检 3-业扩 4-咨询',
    owner_id     BIGINT COMMENT '责任人',
    due_date     DATE NOT NULL COMMENT '截止日（逾期判断依据）',
    finish_date  DATE COMMENT '完成日',
    status       TINYINT COMMENT '0-待办 1-进行中 2-已完成 3-已逾期',
    created_at   TIMESTAMPTZ DEFAULT now()
);
-- 逾期指标：due_date < CURRENT_DATE AND status IN (0,1) 即视为逾期
```

## 四、建模评审问题清单

1. 实体是否齐全？是否遗漏业务名词？
2. 关系方向是否正确（外键挂载侧）？
3. 字段命名是否遵循规范？状态字段是否枚举化？
4. 是否违背了"可计算不存储"原则？
5. 每个高频查询路径是否有索引支撑？
6. 数据字典是否完整（含枚举含义）？
7. 是否考虑了数据增长（分区、归档、软删）？
