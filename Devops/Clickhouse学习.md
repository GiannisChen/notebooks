# Clickhouse 学习笔记



## 官网连接整理



### Clickhouse介绍等

- **架构概述** https://clickhouse.tech/docs/zh/development/architecture/ ；
- **数据库特性** https://clickhouse.tech/docs/zh/introduction/distinctive-features/ ；
- **数据库性能** https://clickhouse.tech/docs/zh/introduction/performance/ ；
- **数据库安装** [Docker-Server] https://hub.docker.com/r/yandex/clickhouse-server/ ；



## Clickhouse

### 杂项

#### 存储

- `data` 文件，存储数据内容
- `metadata` 文件，存储数据结构（`.sql` 文件）



### 数据类型

- https://clickhouse.tech/docs/zh/sql-reference/data-types



### 表引擎

#### MergeTree系列

##### MergeTree

- **SQL**

  ```mysql
  CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
  (
      name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
      name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
      ...
      INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
      INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
  ) ENGINE = MergeTree()
  ORDER BY expr
  [PARTITION BY expr]
  [PRIMARY KEY expr]
  [SAMPLE BY expr]
  [TTL expr
      [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx' [, ...] ]
      [WHERE conditions]
      [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ] ]
  [SETTINGS name=value, ...]
  ```

- **example**

  ```mysql
  create table test.t_order_mt ( 
      id UInt32, 
      sku_id String, 
      total_amount Decimal(16,2), 
      create_time Datetime
  ) engine=MergeTree 
  partition by toYYYYMMDD(create_time) 
  primary key(id) 
  order by (id,sku_id);
  ```

- **参数**

  - `engine` — 引擎名称，`MergeTree` 无参；

  - `order by` — 用来排序的`KEY` ；

  - `partition by` — 用来分割数据块，避免全表扫描，一个线程一个分区；

  - `primary key` — 主键，一般用于 `where` 部分过滤；

  - `index_granularity` — index粒度，稀疏索引，两个索引之间的间隔；如果该列中重复值多，可以修改，不建议自定义（`default:8192`）；

  - `INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2` — 二级索引，一般记录最大最小值，需要指定粒度；

  - `TTL` — `Time To Live` 数据表或者数据列存活时间（生命周期）；建完表也能修改：

    ```mysql
    ALTER TABLE table.name 
    	MODIFY COLUMN
    	c String TTL d + INTERVAL 1 DAY;
    ```

    

##### ReplacingMergeTree

- 有主键去重功能；去重只发生在合并的过程中，合并时间未知，但可以用 `optimize` 控制合并；

- 最终一致性；（新版本插入时可能就对同批数据去重）

- 不跨分区合并；

  

- **SQL**

  ```mysql
  CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
  (
      name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
      name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
      ...
  ) ENGINE = ReplacingMergeTree([ver])
  [PARTITION BY expr]
  [ORDER BY expr]
  [PRIMARY KEY expr]
  [SAMPLE BY expr]
  [SETTINGS name=value, ...]
  ```

  

##### SummingMergeTree

- 以维度进行聚合的场景（预聚合）；
- 分区内聚合，合并时聚合；



- **SQL**

  ```mysql
  CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
  (
      name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
      name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
      ...
  ) ENGINE = SummingMergeTree([columns])
  [PARTITION BY expr]
  [ORDER BY expr]
  [SAMPLE BY expr]
  [SETTINGS name=value, ...]
  ```

- **参数**

  - `columns` — 用于计算 `sum` 的列；

- 需要注意的是，当计算和发生时，时间往往会归并至最早时间，如果是 `Map` 类型，则会变为 `Map` 类型的数组，详细内容见： https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/summingmergetree/ ；



##### AggregatingMergeTree

- **SQL**

  ```mysql
  CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
  (
      name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
      name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
      ...
  ) ENGINE = AggregatingMergeTree()
  [PARTITION BY expr]
  [ORDER BY expr]
  [SAMPLE BY expr]
  [TTL expr]
  [SETTINGS name=value, ...]
  ```

- 示例给出的是基于某一张表的`视图`，似乎对聚合查询有优化，具体案例再看吧...



##### CollapsingMergeTree



##### VersionedCollapsingMergeTree



##### GraphiteMergeTree



## Clickhouse高级（优化）



### 基本语法

```mysql
EXPLAIN [AST | SYNTAX | PLAN | PIPELINE] [setting = value, ...] SELECT ... [FORMAT ...]
```

- `PLAN` 查看执行计划；
- `AST`用于查看语法树 ；
- `SYNTAX` 用于优化语法；



### 建表优化

- 时间日期类型用特殊的时间类型：`Date` `Datetime` ；
- 采用自定义的空值表示，例如 `-1` 在生产环境中表示 `NULL` ；
- 必须指定索引列，ClickHouse 中的索引列即排序列，通过 `order by` 指定，一般在查询条件中经常被用来充当筛选条件的属性被纳入进来；可以是单一维度，也可以是组合维度的索引；通常需要满足高级列在前、查询频率大的在前原则；还有基数特别大的不适合做索引列， 如用户表的 userid 字段；通常筛选后的数据满足在**百万以内**为最佳，userid 可以用 `intHash32(userid)` 来做 `order by` ；



### 写入和删除优化

- 尽量执行大批量插入（2 ~ 3次每秒，2w ~ 5w条数据）



### 单表查询

1. `Prewhere` 替代 `where`

   只支持 `MergeTree` 家族，读取指定列，先做过滤，再取其他对应行的列；



### 多表查询

- 使用 `in` 代替 `join`；
- 使用字典表；
- 提前过滤，少用虚表虚列；

 

### 数据一致性（重要）

1. `ReplacingMergeTree` 

- `optimize` 手动去重，但会产生大量IO；

- `group by` 去重，示例：

  ```mysql
  SELECT
       user_id ,
       argMax(score, create_time) AS score,
       argMax(deleted, create_time) AS deleted,
       max(create_time) AS ctime
  FROM test_a
  GROUP BY user_id
  HAVING deleted = 0;
  ```

  其中，`argMax(a, b)` 表示按 `b` 的最大值取 `a` 的值； 

- 通过 `final` 查询，新版的ck中 `final` **支持多线程**；



### 物化视图



## Clickhouse常用操作备忘



#### clickhouse settings

```sql
SET xxxx_xxxx = 0;
```

