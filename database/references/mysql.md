# MySQL 方言速查与最佳实践

## 一、基础语法速查

| 操作 | 语法 |
|------|------|
| 分页 | `LIMIT n OFFSET m` |
| 字符串拼接 | `CONCAT(a, b)`；多行聚合 `GROUP_CONCAT(col)` |
| 当前时间 | `NOW()` / `CURRENT_TIMESTAMP` / `CURDATE()` |
| 日期加减 | `DATE_ADD(now(), INTERVAL 7 DAY)` |
| 取年份/月份 | `YEAR(d)` / `MONTH(d)` / `DATE_FORMAT(d, '%Y-%m')` |
| 空值处理 | `IFNULL(x, 0)` / `COALESCE(x, 0)` |
| 条件 | `IF(cond, a, b)` / `CASE WHEN ... END` |
| 类型转换 | `CAST(x AS SIGNED)` / `CONVERT(x, DECIMAL(10,2))` |
| 正则 | `REGEXP '[0-9]+'` |
| 随机取 N 行 | `ORDER BY RAND() LIMIT n`（慎用，大表极慢） |
| 去重计数 | `COUNT(DISTINCT col)` |

## 二、DDL 关键点

```sql
CREATE TABLE t (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
    ...
    PRIMARY KEY (id),
    UNIQUE KEY uk_t_code (code),
    KEY idx_t_org (org_id, status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='表注释';

ALTER TABLE t ADD COLUMN new_col INT NOT NULL DEFAULT 0 COMMENT '新字段';
ALTER TABLE t ADD INDEX idx_t_status (status);
ALTER TABLE t DROP COLUMN old_col;
```

- 字符集统一 `utf8mb4`（支持 emoji 与生僻字），不用 `utf8`
- 生产引擎用 `InnoDB`（事务、行锁、崩溃恢复）
- 字段变更用 `ALTER TABLE`，大表 DDL 需评估锁表时间（8.0 支持 INSTANT 算法部分 DDL）

## 三、SQL 模式差异与陷阱

1. **分组模式**：MySQL 默认 `ONLY_FULL_GROUP_BY` 关闭时，`SELECT 非分组列` 不报错但结果不确定。标准写法：
   ```sql
   -- 需列出全部非聚合列或改用 ANY_VALUE()
   SELECT org_id, ANY_VALUE(org_name) FROM work_order GROUP BY org_id;
   ```
2. **隐式转换**：VARCHAR 字段与数字比较会转数值，可能索引失效且结果意外。保持类型一致。
3. **大小写**：取决于表 collation（`utf8mb4_0900_ai_ci` 不区分大小写）。查询字符串比较同样受 collation 影响。
4. **字符串比较默认不区分大小写**：需要区分时用 `BINARY col = 'A'` 或 `COLLATE utf8mb4_bin`。
5. **`DELETE` 无 `LIMIT` 之外**：注意 `DELETE` 与 `UPDATE` 不带 WHERE 全表操作风险，先 SELECT 确认。

## 四、索引与性能要点

- `EXPLAIN` 看 `type`：`ALL`（全表扫描）需要优化；`range`/`ref`/`eq_ref` 正常
- `SHOW INDEX FROM t` 查看索引；`SHOW CREATE TABLE t` 看完整定义
- 深分页优化：
  ```sql
  -- 慢：SELECT * FROM t ORDER BY id LIMIT 1000000, 20;
  -- 快：SELECT * FROM t WHERE id > 1000000 ORDER BY id LIMIT 20;
  ```
- `EXPLAIN ANALYZE`（8.0.18+）给出真实执行耗时，比 `EXPLAIN` 估算更准
- 查询缓存 8.0 已移除，不依赖；用 `innodb_buffer_pool_size` 调内存

## 五、常用运维命令

```sql
SHOW PROCESSLIST;                -- 查看当前连接/慢查询
KILL <thread_id>;                -- 终止卡死连接
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW TABLE STATUS WHERE Name='work_order';   -- 表行数/碎片
OPTIMIZE TABLE work_order;       -- 整理碎片（在线DDL）
```

```bash
# 备份/恢复
mysqldump -u root -p --single-transaction power_db > backup.sql
mysql -u root -p power_db < backup.sql
```

## 六、最佳实践清单

- [ ] 所有表 InnoDB + utf8mb4
- [ ] 主键用 BIGINT 自增或雪花 ID，不用 UUID 做主键（随机插入碎片化）
- [ ] 金额 DECIMAL(18,2)，禁止 FLOAT
- [ ] DATETIME 加毫秒 `DATETIME(3)`，TIMESTAMP 有 2038 上限
- [ ] 每个查询先 EXPLAIN，杜绝无索引全表扫描
- [ ] 批量操作用事务包裹，设置合理 batch 大小
- [ ] 应用账号最小权限，不用 root
