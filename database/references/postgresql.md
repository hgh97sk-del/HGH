# PostgreSQL 方言速查与最佳实践

## 一、基础语法速查

| 操作 | 语法 |
|------|------|
| 分页 | `LIMIT n OFFSET m`（或标准 `OFFSET m ROWS FETCH NEXT n ROWS ONLY`） |
| 字符串拼接 | `a \|\| b`；多行聚合 `STRING_AGG(col, ',')` |
| 当前时间 | `CURRENT_TIMESTAMP` / `now()` / `CURRENT_DATE` |
| 日期加减 | `now() + INTERVAL '7 days'` / `CURRENT_DATE - 7` |
| 取年月 | `EXTRACT(YEAR FROM d)` / `TO_CHAR(d, 'YYYY-MM')` |
| 空值处理 | `COALESCE(x, 0)` / `NULLIF(a, b)` |
| 数组 | `ARRAY[1,2,3]`、`array_agg(col)` |
| JSON 操作 | `data->>'key'`、`data @> '{"a":1}'`、`jsonb_path_query` |
| 正则 | `col ~ '^[0-9]+$'` / `col ~* 'pattern'`（忽略大小写） |
| 类型转换 | `x::numeric` / `CAST(x AS numeric)` |
| 随机取 N | `ORDER BY random() LIMIT n`（慎用大表） |
| 生成序列 | `generate_series(1, 10)` |

## 二、DDL 关键点

```sql
-- 自增主键（两种写法）
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- 标准，推荐
id BIGSERIAL PRIMARY KEY,                              -- 传统简写

-- 约束与注释
CONSTRAINT uk_t_code UNIQUE (code),
CHECK (status IN (0,1,2,3)),
COMMENT ON TABLE t IS '表注释';
COMMENT ON COLUMN t.status IS '状态字段';

-- 修改表
ALTER TABLE t ADD COLUMN new_col INT NOT NULL DEFAULT 0;
ALTER TABLE t ADD CONSTRAINT fk_t_org FOREIGN KEY (org_id) REFERENCES org(id);
```

- 主键 IDENTITY 是标准做法；`BIGSERIAL` 兼容旧代码
- 所有表建议 `created_at TIMESTAMPTZ NOT NULL DEFAULT now()`

## 三、独特能力（相比 MySQL/Oracle）

### 1. JSONB（文档型能力）

```sql
CREATE TABLE device_meta (
    id BIGINT PRIMARY KEY,
    data JSONB NOT NULL DEFAULT '{}'
);
CREATE INDEX idx_device_meta ON device_meta USING GIN (data);

SELECT * FROM device_meta WHERE data @> '{"type":"光伏逆变器"}';
SELECT data->>'model' AS 型号, COUNT(*) FROM device_meta GROUP BY 1;
```

### 2. CTE 可写（WITH ... INSERT/UPDATE/DELETE）

```sql
WITH deleted AS (
    DELETE FROM work_order WHERE status = 3 RETURNING *
)
SELECT count(*) FROM deleted;   -- 删除并统计，一条语句
```

### 3. 物化视图

```sql
CREATE MATERIALIZED VIEW mv_org_stat AS
SELECT org_id, COUNT(*) FROM work_order GROUP BY org_id;
REFRESH MATERIALIZED VIEW mv_org_stat;   -- 手动刷新（可建索引）
```

### 4. 分区表（声明式分区，10+）

```sql
CREATE TABLE work_order_2026 PARTITION OF work_order
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
```

## 四、性能优化要点

```sql
-- 查看执行计划（真实执行）
EXPLAIN (ANALYZE, BUFFERS, TIMING) SELECT ...;

-- 统计信息
ANALYZE work_order;
-- 查看表实际行数（比 COUNT 快）
SELECT reltuples::bigint FROM pg_class WHERE relname='work_order';
-- 索引使用情况
SELECT relname, idx_scan, seq_scan FROM pg_stat_user_tables ORDER BY seq_scan DESC;
```

- **索引类型丰富**：B-tree（默认）、GIN（JSON/全文）、BRIN（大表有序列）、GiST（空间/范围）
- **VACUUM 机制**：MVCC 产生死元组，需定期 `VACUUM (ANALYZE)`；autovacuum 默认开启，勿关闭
- **NULL 排序**：默认 NULL 最大（升序排最后），需要控制用 `NULLS FIRST/LAST`
- **事务隔离**：默认 Read Committed；需要可重复读用 `REPEATABLE READ`

## 五、运维命令

```sql
-- 当前活动查询
SELECT pid, state, now()-query_start AS duration, query
FROM pg_stat_activity WHERE state='active' ORDER BY duration DESC;

-- 终止查询（仅限自己/超管）
SELECT pg_cancel_backend(<pid>);      -- 取消查询
SELECT pg_terminate_backend(<pid>);   -- 终止连接

-- 锁等待排查
SELECT * FROM pg_locks WHERE NOT granted;
```

```bash
# 备份（库级/表级）
pg_dump -U postgres -d power_db -F c -f power_db.dump
pg_restore -U postgres -d power_db power_db.dump
```

## 六、最佳实践清单

- [ ] 时间字段用 `TIMESTAMPTZ`（带时区），不用 `TIMESTAMP`
- [ ] 主键用 `IDENTITY`，不用 UUID 主键（可用 `gen_random_uuid()` 作业务键）
- [ ] JSON 数据用 `JSONB` 而非 `JSON`（可索引）
- [ ] 大查询先 `EXPLAIN (ANALYZE)` 验证
- [ ] 保持 autovacuum 开启；高频更新表定期 `VACUUM (ANALYZE)`
- [ ] 避免大事务长时间占用（导致 vacuum 无法回收）
- [ ] 应用连接池（如 pgbouncer）控制连接数
