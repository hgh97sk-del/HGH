# 数据库运维与性能优化

慢查询排查、EXPLAIN 分析、索引优化与常见运维操作。覆盖 MySQL / PostgreSQL / Oracle。

## 一、慢查询定位

### 1. 开启慢查询日志

```sql
-- MySQL
SET GLOBAL slow_query_log = ON;
SET GLOBAL long_query_time = 1;          -- 超过 1 秒记录
SHOW VARIABLES LIKE 'slow%';

-- PostgreSQL
-- 修改 postgresql.conf：log_min_duration_statement = 1000

-- Oracle
-- 使用 AWR 报告 / v$sql 视图：
SELECT sql_id, elapsed_time/1000000 AS seconds, sql_text
FROM v$sql WHERE elapsed_time > 1000000 ORDER BY elapsed_time DESC;
```

### 2. 排查路径

```
SQL 慢 → EXPLAIN 看执行计划 → 找全表扫描/大行数节点
      → 确认索引是否生效 → 改写 SQL / 加索引 → 复测
```

## 二、EXPLAIN 执行计划解读

### MySQL（EXPLAIN 关键列）

| 列 | 好 | 差（需优化） |
|----|----|-------------|
| type | `const`/`eq_ref`/`ref`/`range` | `ALL`（全表扫描）、`index` 全索引扫描 |
| key | 实际使用索引非 NULL | NULL（未用索引） |
| rows | 估算行数小 | 行数巨大 |
| Extra | `Using index`（覆盖索引） | `Using filesort`、`Using temporary`（需优化） |

```sql
EXPLAIN SELECT * FROM work_order WHERE org_id = 100 AND status = 2;
-- 看到 type=ALL 且 rows 很大 → org_id 无索引，需加索引
```

### PostgreSQL（EXPLAIN ANALYZE）

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM work_order WHERE org_id = 100;
-- 关注：Seq Scan（顺序扫描，大表要避免）/ Index Scan / rows 估算偏差
```

### Oracle（EXPLAIN PLAN + DBMS_XPLAN）

```sql
EXPLAIN PLAN FOR SELECT * FROM work_order WHERE org_id = 100;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
-- 关注：TABLE ACCESS FULL（全表扫描）、NESTED LOOPS vs HASH JOIN
```

## 三、索引优化指南

### 1. 加索引优先级

```
高频 WHERE 等值条件 > JOIN 关联字段 > 高频 ORDER BY 字段 > 覆盖查询列
```

### 2. 复合索引设计

```sql
-- 原则：等值列在前，范围列在后；遵循最左前缀
-- 好：idx (org_id, status, created_at)  支持 WHERE org_id=? AND status=? AND created_at BETWEEN ...
-- 差：idx (created_at, status, org_id)  查询条件常不匹配最左前缀
```

### 3. 索引失效常见场景

| 写法 | 问题 | 正确写法 |
|------|------|---------|
| `WHERE YEAR(created_at)=2026` | 函数导致索引失效 | `WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'` |
| `WHERE name LIKE '%配变%'` | 前置通配符失效 | `WHERE name LIKE '配变%'`（或全文索引） |
| `WHERE org_id + 1 = 100` | 列参与运算失效 | `WHERE org_id = 99` |
| `WHERE org_id = 100 OR status = 2` | OR 导致失效 | 拆 UNION / 用索引合并 |
| 隐式类型转换 | `WHERE phone = 13800000000`（字段为 VARCHAR） | `WHERE phone = '13800000000'` |
| 区分度低 | 性别、布尔列建索引无意义 | 组合到复合索引 |

### 4. 覆盖索引

```sql
-- 查询列全部在索引内 → Extra: Using index，避免回表
CREATE INDEX idx_wo_org_status ON work_order (org_id, status);
SELECT org_id, status FROM work_order WHERE org_id = 100;
```

## 四、SQL 改写优化

| 问题模式 | 优化手段 |
|---------|---------|
| `SELECT *` | 只取所需列 |
| 大表深分页 `OFFSET 100000` | 改游标/键集分页 `WHERE id > ?` |
| 循环内逐条查询（N+1） | 一次 IN 查询后应用层组装 |
| 大表 COUNT(*) | 近似计数（PG `pg_class.reltuples`、MySQL 用缓存/统计表） |
| 无用 ORDER BY / DISTINCT | 确认业务是否真需要 |
| 子查询嵌套深 | 改 CTE 或 JOIN |
| 函数包裹索引列 | 改写范围条件 |

## 五、常见运维操作

### 备份与恢复

```bash
# MySQL
mysqldump -u root -p power_db > backup.sql          # 逻辑备份
mysql -u root -p power_db < backup.sql             # 恢复

# PostgreSQL
pg_dump -U postgres power_db > backup.sql
psql -U postgres power_db < backup.sql

# Oracle
expdp system/password schemas=POWER_DIR dumpfile=power.dmp
impdp system/password schemas=POWER_DIR dumpfile=power.dmp
```

### 表维护

```sql
-- MySQL：分析表更新统计信息、整理碎片
ANALYZE TABLE work_order;
OPTIMIZE TABLE work_order;

-- PostgreSQL：更新统计信息 + 回收空间（类似 VACUUM FULL）
ANALYZE work_order;
VACUUM (ANALYZE, VERBOSE) work_order;

-- Oracle：收集统计信息
EXEC DBMS_STATS.GATHER_TABLE_STATS('POWER', 'WORK_ORDER');
```

### 权限管理

```sql
-- MySQL
CREATE USER 'app'@'%' IDENTIFIED BY 'pwd';
GRANT SELECT, INSERT, UPDATE, DELETE ON power_db.* TO 'app'@'%';
-- 生产环境最小权限原则：应用账号不授予 DROP/ALTER

-- PostgreSQL
CREATE ROLE app LOGIN PASSWORD 'pwd';
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app;

-- Oracle
CREATE USER app IDENTIFIED BY pwd;
GRANT CONNECT, RESOURCE TO app;
```

## 六、优化流程（标准套路）

```
1. 拿到慢 SQL → EXPLAIN 分析执行计划
2. 判断：全表扫描？行数估算过大？临时表/文件排序？
3. 加索引或改写 SQL（先小表验证）
4. 复测：EXPLAIN 确认 type 提升、rows 下降
5. 观察实际查询耗时（避免仅看估算）
6. 若仍慢 → 检查数据分布（数据倾斜）、考虑分区表/物化视图/缓存
```

## 七、健康巡检清单

- [ ] 慢查询日志是否开启？阈值是否合理？
- [ ] 是否存在无索引的常用查询？（查 information_schema / pg_stat_user_indexes 的 idx_scan）
- [ ] 是否存在冗余索引（重复前缀）？
- [ ] 表碎片是否过大？统计信息是否过期？
- [ ] 磁盘空间、连接数、内存使用是否告警？
- [ ] 备份策略是否有效？（定期演练恢复）
- [ ] 应用账号权限是否符合最小权限原则？
