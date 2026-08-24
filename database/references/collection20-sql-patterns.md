# 采集2.0 高频 SQL 模板

默认 **Oracle 方言**（12c+）。模板字段均对照《2.0业务表结构.xls》与物理模型说明书 V0.996。使用时按需替换时间范围、管理单位、表名前缀（如需全库访问需带 schema 前缀如 `SGAMI_STAT.`）。

## 1. 用户日电量查询（A_BA_CONS_ENERGY_DAY）

```sql
-- 查询某省某日各用户日电量（含6费率）
SELECT CUST_NO,
       CUST_NAME,
       MGT_ORG_CODE,
       DATA_DATE,
       PAP_E1 AS ENERGY_F1,   -- 尖
       PAP_E2 AS ENERGY_F2,   -- 峰
       PAP_E3 AS ENERGY_F3,   -- 平
       PAP_E4 AS ENERGY_F4,   -- 谷
       PAP_E  AS ENERGY_TOTAL -- 正向有功总电量
FROM SGAMI_STAT.A_BA_CONS_ENERGY_DAY
WHERE MGT_ORG_CODE = '32'                -- 32=江苏
  AND DATA_DATE = TRUNC(SYSDATE) - 1     -- 昨日
ORDER BY ENERGY_TOTAL DESC;
```

## 2. 计量点日电量汇总（A_BA_INST_ENERGY_DAY）

```sql
-- 台区/单位维度正反向电量日汇总
SELECT MGT_ORG_CODE,
       SUM(PAP_E) AS TOTAL_FWD_ENERGY,  -- 正向有功总电量
       SUM(RAP_E) AS TOTAL_REV_ENERGY,  -- 反向有功总电量
       SUM(PRP_E) AS TOTAL_FWD_REACT,   -- 正向无功
       SUM(RRP_E) AS TOTAL_REV_REACT    -- 反向无功
FROM SGAMI_STAT.A_BA_INST_ENERGY_DAY
WHERE DATA_DATE = TRUNC(SYSDATE) - 1
GROUP BY MGT_ORG_CODE;
```

## 3. 供电单位电量日统计（A_BA_ORG_ENERGY_DAY）

```sql
-- 省级供电单位发电/上网电量统计（含区域层级与下级汇总标志）
SELECT MGT_ORG_CODE,
       MGT_ORG_NAME,
       DIST_LV,
       STAT_DATE,
       GPQ  AS GEN_ENERGY,   -- 发电电量
       GCPQ AS GRID_ENERGY   -- 上网电量
FROM SGAMI_STAT.A_BA_ORG_ENERGY_DAY
WHERE STAT_DATE = TRUNC(SYSDATE) - 1
  AND YN_FLAG = 1            -- 包含下级单位
ORDER BY DIST_LV, MGT_ORG_CODE;
```

## 4. 行业月电量统计（A_BA_IND_ENERGY_MON）

```sql
-- 行业维度月度用电量
SELECT IND_CLS,
       DATA_DATE,
       SUM(IND_ENERGY) AS TOTAL_ENERGY
FROM SGAMI_STAT.A_BA_IND_ENERGY_MON
WHERE DATA_DATE = TO_CHAR(TRUNC(SYSDATE, 'MM') - 1, 'YYYYMM')  -- 上月
GROUP BY IND_CLS, DATA_DATE
ORDER BY TOTAL_ENERGY DESC;
```

## 5. 配送站（公变）运行特征（A_BA_DIST_TRAIT_DAY）

```sql
-- 公变负载率与电压平衡度日特征
SELECT DIST_STA_ID,
       RESRC_SUPL_CODE,
       RESRC_SUPL_NAME,
       STAT_DATE,
       LOAD_MAX_RATE_A,  -- A相最大负载率
       LOAD_MAX_RATE_B,
       LOAD_MAX_RATE_C,
       AVG_UA, AVG_UB, AVG_UC
FROM SGAMI_STAT.A_BA_DIST_TRAIT_DAY
WHERE STAT_DATE = TRUNC(SYSDATE) - 1
  AND (LOAD_MAX_RATE_A > 0.8 OR LOAD_MAX_RATE_B > 0.8 OR LOAD_MAX_RATE_C > 0.8);  -- 重载预警
```

## 6. 用户行为月特征（A_BA_CONS_TRAIT_MON）

```sql
-- 用户月最大/最小/平均负荷与负荷率
SELECT CUST_NO, CUST_NAME, STAT_DATE,
       MAX_P, MAX_P_TIME,
       MIN_P, AVG_P,
       MAX_PQ,                       -- 最大视在功率
       ROUND(AVG_P / NULLIF(MAX_P,0), 4) AS LOAD_RATE  -- 负荷率
FROM SGAMI_STAT.A_BA_CONS_TRAIT_MON
WHERE MGT_ORG_CODE = '32'
  AND STAT_DATE = TO_CHAR(TRUNC(SYSDATE, 'MM') - 1, 'YYYYMM');
```

## 7. 异常结果查询（A_ABNOR_INFO）

```sql
-- 按异常大类查询异常明细（配合标准代码 abnorType）
SELECT ABNOR_ID, ABNOR_CATEG, ABNOR_TYPE,
       ASSET_NO, CUST_NO, CONS_TYPE,
       ABNOR_DESC, ABNOR_TIME, ABNOR_STATUS
FROM SGAMI_STAT.A_ABNOR_INFO
WHERE ABNOR_TYPE LIKE '02%'            -- 计量异常大类
  AND ABNOR_TIME >= TRUNC(SYSDATE) - 7 -- 近7天
ORDER BY ABNOR_TIME DESC;
```

## 8. 区域停复电分析（A_PO_AREA_DET）

```sql
-- 某区域停复电事件与停电时长
SELECT AREA_NO, PO_TIME, PR_TIME,
       ROUND((PR_TIME - PO_TIME) * 24, 2) AS OUTAGE_HOURS,  -- 停电时长(小时)
       PO_SCOPE, PO_FLAG
FROM SGAMI_STAT.A_PO_AREA_DET
WHERE PO_TIME >= TRUNC(SYSDATE) - 30
  AND MGT_ORG_CODE = '32'
ORDER BY PO_TIME DESC;
```

## 9. 线损统计（A_LL_DIST_STAT_DAY）

```sql
-- 配送站日线损率排名
SELECT DIST_STA_ID, STAT_DATE, LL_RATE  -- 线损率字段按实际列名调整
FROM SGAMI_STAT.A_LL_DIST_STAT_DAY
WHERE STAT_DATE = TRUNC(SYSDATE) - 1
ORDER BY LL_RATE DESC;
```

## 10. 日冻结电能示值最新值（E_MP_READ_DAY）

```sql
-- 取每只表最近一日冻结示值（正反向有功总）
SELECT DATA_ENTITY_ID, DATA_DATE,
       PAP_R, RAP_R, COMP_RTO
FROM SGAMI_PROV.E_MP_READ_DAY e
WHERE DATA_DATE = (
    SELECT MAX(DATA_DATE) FROM SGAMI_PROV.E_MP_READ_DAY
    WHERE DATA_ENTITY_ID = e.DATA_ENTITY_ID
)
ORDER BY DATA_ENTITY_ID;
```

## 11. 采集成功率统计（A_COLL_DEV_STAT_DAY）

```sql
-- 供电单位采集成功率日统计
SELECT MGT_ORG_CODE, MGT_ORG_NAME, STAT_DATE,
       SUCC_RATE,   -- 成功率（百分比，字段按实际）
       TOTAL_NUM, FAIL_NUM
FROM SGAMI_STAT.A_COLL_DEV_STAT_DAY
WHERE STAT_DATE = TRUNC(SYSDATE) - 1
ORDER BY SUCC_RATE ASC;   -- 成功率低的排前，便于治理
```

## 12. 运维作业工单（IEOM.A_IEOM_WKST_POOL_INFO）

```sql
-- 待处理运维作业（采集异常类）
SELECT WORK_ORDER_NO, ORDER_TYPE, ORDER_CHILD_TYPE,
       ORDER_LV, ORDER_TIME, ORDER_STATUS
FROM IEOM.A_IEOM_WKST_POOL_INFO
WHERE ORDER_TYPE = '01'                -- 01采集异常
  AND ORDER_STATUS = '01'              -- 待派工
ORDER BY ORDER_LV, ORDER_TIME;         -- 优先级高的先处理
```

## 13. 发电户档案（SGAMI_ARCH.A_ARCH_GPC_FULL_INFO）

```sql
-- 光伏发电户装机容量汇总（分布式电源场景）
SELECT MGT_ORG_CODE,
       COUNT(*) AS GEN_CUST_CNT,
       SUM(INST_CAP) AS TOTAL_INST_CAP_KW,  -- 装机容量(kW)
       SUM(ACCS_CAP) AS TOTAL_ACCS_CAP       -- 接入容量
FROM SGAMI_ARCH.A_ARCH_GPC_FULL_INFO
WHERE GEN_MODE = '01'                -- 01光伏发电
  AND EC_DATE >= TRUNC(SYSDATE, 'YYYY')
GROUP BY MGT_ORG_CODE;
```

## 14. 工单查询（SG_MIS.WK_ORDER）

```sql
-- 按工单状态统计
SELECT WK_ORDER_TYPE,
       WK_ORDER_STAT,
       COUNT(*) AS CNT
FROM SG_MIS.WK_ORDER
WHERE WK_ORDER_STAT = '01'           -- 01运行
GROUP BY WK_ORDER_TYPE, WK_ORDER_STAT;
```

## 15. 通用分页模板（Oracle 12c+）

```sql
SELECT * FROM (
    SELECT t.*, ROWNUM rn FROM (
        SELECT ... ORDER BY ...        -- 排序子查询
    ) t WHERE ROWNUM <= :end_row
) WHERE rn > :start_row;
-- 或 12c 简洁写法：
SELECT ... OFFSET :start_row ROWS FETCH NEXT :page_size ROWS ONLY;
```

## 编写注意

1. **schema 前缀**：正式环境表多带 schema（SG_MIS./SGAMI_ARCH./SGAMI_PROV./SGAMI_STAT./SGAMI_SUPPORT./IEOM.），业务库默认用户下可直接用表名
2. **日期存储**：日统计表 DATA_DATE/STAT_DATE 一般存 DATE 或 VARCHAR；月统计表存月份（YYYYMM），过滤用 `TO_CHAR(TRUNC(SYSDATE,'MM')-1,'YYYYMM')`
3. **费率字段**：确认目标表是 6 费率还是 4 费率（尖峰平谷），用错字段会拿不到数
4. **综合倍率**：示值类数据换算电量需乘 COMP_RTO；E 类电量字段已是倍率后结果，直接用
5. **NULL 处理**：电量汇总前用 `NVL(col,0)`，避免 SUM 结果为 NULL；除法加 `NULLIF(deno,0)`
6. **时间趋势**：做趋势分析用 `GROUP BY TO_CHAR(DATA_DATE,'YYYY-MM-DD')` 或按月分组
