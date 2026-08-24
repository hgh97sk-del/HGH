# SQL 查询与数据分析指南

通用取数与分析模式，适用于 MySQL / PostgreSQL / Oracle。示例以 PostgreSQL 为主，方言差异见各库文档。

## 一、多表关联

### JOIN 类型速查

| JOIN 类型 | 语义 | 典型场景 |
|-----------|------|---------|
| INNER JOIN | 只取两表匹配行 | 有工单的供电所 |
| LEFT JOIN | 左表全保留，右表无匹配为 NULL | 所有供电所及其工单数（含 0） |
| RIGHT JOIN | 右表全保留 | 不常用，可改写为 LEFT JOIN 颠倒表序 |
| FULL OUTER JOIN | 两表全保留 | 对比两表数据差异 |

### 关联陷阱

- **一对多放大问题**：`org LEFT JOIN work_order` 会放大 org 行数，聚合时用 `COUNT(DISTINCT ...)` 或先子查询聚合再关联
- **关联条件缺失**：忘记 `ON` 条件产生笛卡尔积，先 SELECT 前 100 行确认行数量级
- **NULL 陷阱**：`NULL = NULL` 不成立，用 `IS NULL` / `IS NOT NULL` 判断

### 正确写法示例

```sql
-- 先聚合再关联，避免行放大
SELECT o.org_name, COALESCE(w.cnt, 0) AS 工单数
FROM org o
LEFT JOIN (
    SELECT org_id, COUNT(*) AS cnt
    FROM work_order
    WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY org_id
) w ON o.org_id = w.org_id
ORDER BY 工单数 DESC;
```

## 二、聚合与分组

### 常用聚合函数

```sql
COUNT(*)          -- 行数（含 NULL 行）
COUNT(column)     -- 非 NULL 值数量
SUM / AVG         -- 求和/平均（自动忽略 NULL）
MIN / MAX         -- 最值
GROUP_CONCAT()    -- MySQL 字符串聚合
STRING_AGG()      -- PostgreSQL 字符串聚合
LISTAGG()         -- Oracle 字符串聚合
```

### 分组后过滤：HAVING

```sql
SELECT org_id, COUNT(*) AS cnt
FROM work_order
WHERE status = 'OVERDUE'
GROUP BY org_id
HAVING COUNT(*) > 10      -- 分组后条件用 HAVING，分组前用 WHERE
ORDER BY cnt DESC;
```

### 关键原则

- WHERE 在分组前过滤 → 减少计算量
- HAVING 在分组后过滤 → 只能引用聚合结果或分组键
- 聚合结果列给别名，便于阅读与排序

## 三、窗口函数（分析函数）

用于排名、同比环比、累计等场景，不改变行数。

```sql
-- 排名
ROW_NUMBER() OVER (PARTITION BY org_id ORDER BY due_date)  -- 1,2,3 不并列
RANK()         OVER (PARTITION BY org_id ORDER BY due_date) -- 并列跳号 1,1,3
DENSE_RANK()   OVER (PARTITION BY org_id ORDER BY due_date) -- 并列不跳号 1,1,2

-- 同组内取前 N
SELECT * FROM (
    SELECT t.*,
           ROW_NUMBER() OVER (PARTITION BY org_id ORDER BY amount DESC) AS rn
    FROM work_order t
) x WHERE rn <= 3;

-- 累计求和 / 移动平均
SUM(amount) OVER (PARTITION BY org_id ORDER BY created_at) AS 累计
AVG(amount) OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS 7日均

-- 同比环比（LAG 取上一期）
SELECT month,
       amount,
       LAG(amount) OVER (ORDER BY month) AS 上月,
       ROUND((amount - LAG(amount) OVER (ORDER BY month))
           / NULLIF(LAG(amount) OVER (ORDER BY month), 0) * 100, 2) AS 环比
FROM monthly_stat;
```

## 四、子查询与 CTE

### CTE（推荐，可读性好）

```sql
WITH overdue AS (
    SELECT org_id, COUNT(*) AS cnt
    FROM work_order WHERE status = 'OVERDUE'
    GROUP BY org_id
),
total AS (
    SELECT org_id, COUNT(*) AS cnt
    FROM work_order GROUP BY org_id
)
SELECT o.org_name, t.cnt AS 总数, od.cnt AS 逾期, ROUND(od.cnt::numeric / t.cnt * 100, 2) AS 逾期率
FROM total t
JOIN overdue od USING (org_id)
JOIN org o ON o.org_id = t.org_id
ORDER BY 逾期率 DESC;
```

### 相关子查询（谨慎使用）

```sql
-- 每行取"本供电所最新一条记录"
SELECT * FROM work_order w
WHERE w.created_at = (
    SELECT MAX(created_at) FROM work_order w2 WHERE w2.org_id = w.org_id
);
-- 大数据量下性能差，优先用窗口函数 ROW_NUMBER()
```

## 五、CASE 逻辑

```sql
SELECT
    CASE
        WHEN due_date < CURRENT_DATE THEN '已逾期'
        WHEN due_date = CURRENT_DATE THEN '今日到期'
        ELSE '未到期'
    END AS 状态标签,
    COUNT(*) AS 数量
FROM work_order
GROUP BY 状态标签;
```

## 六、去重与数据清洗

```sql
-- 精确去重
SELECT DISTINCT org_name FROM org;

-- 按关键字段去重取最新（ROW_NUMBER）
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY device_code ORDER BY update_time DESC) AS rn
    FROM device_master
)
SELECT * FROM ranked WHERE rn = 1;

-- 处理空值
COALESCE(amount, 0)         -- NULL 转 0
NULLIF(amount, 0)           -- 0 转 NULL（防除零）
TRIM(name)                  -- 去首尾空格
REGEXP_REPLACE(phone, '[^0-9]', '', 'g')  -- 清洗非数字
```

## 七、分页与性能边界

```sql
-- MySQL / PG
SELECT * FROM work_order ORDER BY id LIMIT 50 OFFSET 100;

-- Oracle 12c+
SELECT * FROM work_order ORDER BY id OFFSET 100 ROWS FETCH NEXT 50 ROWS ONLY;

-- 深分页优化：WHERE 游标代替 OFFSET（数据量大时）
SELECT * FROM work_order WHERE id > 10050 ORDER BY id LIMIT 50;
```

## 八、分析输出准则

1. 用户要"分析"时，SQL 结果配合文字解读：趋势、异常点、TOP 排名
2. 涉及百分比给出基数（`COUNT(*)`），避免"50% 上升"无上下文
3. 时间维度统一：明确口径（自然日/工作日/自然月），在注释中写明
4. 结果多时给 TOP 10/20 并说明"完整结果可按需导出"
