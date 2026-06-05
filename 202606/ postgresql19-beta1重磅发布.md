# postgresql 19 beta1 重磅发布：一文速览全部新特性

> 2026 年 6 月，postgresql 19 beta1 正式发布。作为全球最先进的开源关系型数据库，pg19 带来了超过 60 项新特性与改进，涵盖图查询、性能优化、逻辑复制、安全认证等多个领域。本文为你全面梳理。

----

>  [开源征程，邀你同行｜IvorySQL 2026 布道者招募启动，快来报名！](https://mp.weixin.qq.com/s/fpV-I0OwHtwdD7nbqJJE3w)

#### 报名方式
扫描下方二维码填写报名表单：

![](https://fastly.jsdelivr.net/gh/bucketio/img0@main/2026/06/05/1780638832342-62eef1fc-e221-463a-a626-d7c5c2363e96.png)

提交后，运营同学会邀请你加入布道者专属微信群。

----

## 一、sql 与查询能力大升级

### 1. sql 属性图查询（sql/pgq）

pg19 正式引入 **sql/pgq 标准**，可以在关系型数据上直接执行图查询，无需额外的图数据库。

```sql
-- 建图定义
create property graph social_network
  node tables (
    users  label person,
    cities label city
  )
  edge tables (
    follows   source key (follower_id) references users(id)
              destination key (followee_id) references users(id)
              label follows,
    lives_in  source key (user_id) references users(id)
              destination key (city_id) references cities(id)
              label lives_in
  );

-- 图查询：查找"我关注的人住在哪些城市"
select city_name
from graph_table social_network
  match (p:person where p.name = 'alice')
        -[f:follows]-> (friend:person)
        -[l:lives_in]-> (c:city)
  columns (c.name as city_name);
```

**真正的"一库搞定关系型+图"。**

### 2. update/delete for portion of

支持对 **时间范围（temporal range）** 进行局部更新或删除。

```sql
-- 创建时态表
create table employee_salary (
    id         int primary key,
    name       text,
    salary     numeric,
    valid_from date not null,
    valid_to   date not null,
    period for validity (valid_from, valid_to)
);

insert into employee_salary values
  (1, 'alice', 10000, '2025-01-01', '2025-12-31');

-- 只更新 3~6 月的薪资（其余时间段自动拆分保留）
update employee_salary
for portion of validity from '2025-03-01' to '2025-07-01'
set salary = 12000
where id = 1;

-- 结果：自动拆分为 3 段
-- 2025-01-01 ~ 2025-03-01  salary=10000
-- 2025-03-01 ~ 2025-07-01  salary=12000
-- 2025-07-01 ~ 2025-12-31  salary=10000

-- 同理可以局部删除
delete from employee_salary
for portion of validity from '2025-06-01' to '2025-09-01'
where id = 1;
```


### 3. group by all

告别手动罗列 group by 字段，自动对所有非聚合列分组：

```sql
-- 以前需要这样写
select region, product, channel, sum(amount), count(*)
from sales
group by region, product, channel;

-- pg19 简化为
select region, product, channel, sum(amount), count(*)
from sales
group by all;
```


### 4. 窗口函数 ignore nulls

`lead`、`lag`、`first_value`、`last_value`、`nth_value` 新增 `ignore nulls` 选项：

```sql
-- 场景：股票价格有缺失值，取前一个有效价格
create table stock_prices (ts timestamptz, price numeric);

insert into stock_prices values
  ('2026-06-01', 100),
  ('2026-06-02', null),
  ('2026-06-03', null),
  ('2026-06-04', 105),
  ('2026-06-05', null);

-- pg18 写法（结果包含 null）
select ts, price,
       lag(price) over (order by ts) as prev_price_old
from stock_prices;

-- pg19 写法（自动跳过 null）
select ts, price,
       lag(price) ignore nulls over (order by ts) as prev_valid_price
from stock_prices;

-- 结果：
-- 2026-06-01 | 100 | (null)
-- 2026-06-02 | (null) | 100 
-- 2026-06-03 | (null) | 100
-- 2026-06-04 | 105  | 100
-- 2026-06-05 | (null) | 105
```


### 5. insert ... on conflict do select

冲突时不再只能"更新"或"忽略"，还能 **直接返回冲突行** ：

```sql
create table users (
    id    serial primary key,
    email text unique not null,
    name  text
);

insert into users (email, name) values ('alice@test.com', 'alice');

-- 插入冲突时，返回已有的冲突行
insert into users (email, name) values ('alice@test.com', 'alice new')
on conflict (email) do select
returning id, email, name;

-- 返回: 1 | alice@test.com | alice
-- 不修改数据，但能拿到冲突行的内容，方便上层逻辑判断
```


### 6. repack 命令

替代 `vacuum full` 和 `cluster` 的统一命令，支持在线操作：

```sql
-- 以前：vacuum full 会排他锁表，业务必须停
vacuum full orders;

-- pg19：repack 支持 concurrently，不阻塞读写
repack table orders;

-- 在线重组（不排他锁）
repack table orders concurrently;

-- 按指定列重新聚簇
repack table orders concurrently using idx_orders_created_at;

-- 控制并行度
set max_repack_replication_slots = 4;
repack table big_table concurrently;
```


### 7. 分区合并与拆分

```sql
-- 创建分区表
create table orders (
    id         serial,
    created_at date not null,
    amount     numeric
) partition by range (created_at);

create table orders_2025_q1 partition of orders
  for values from ('2025-01-01') to ('2025-04-01');
create table orders_2025_q2 partition of orders
  for values from ('2025-04-01') to ('2025-07-01');

-- 合并两个分区
alter table orders
  merge partitions (orders_2025_q1, orders_2025_q2)
  into orders_2025_h1;

-- 拆分分区（按月拆分上半年的分区）
alter table orders
  split partition orders_2025_h1 into
    (partition orders_2025_jan for values from ('2025-01-01') to ('2025-02-01'),
     partition orders_2025_feb for values from ('2025-02-01') to ('2025-03-01'),
     partition orders_2025_mar for values from ('2025-03-01') to ('2025-04-01'),
     partition orders_2025_apr for values from ('2025-04-01') to ('2025-05-01'),
     partition orders_2025_may for values from ('2025-05-01') to ('2025-06-01'),
     partition orders_2025_jun for values from ('2025-06-01') to ('2025-07-01'));
```


### 8. copy 增强

```sql
-- (a) copy to json：导出为 json 格式
copy (select id, name, email from users limit 3)
to '/tmp/users.json' (format json, force_array);

-- 输出结果：
-- [{"id":1,"name":"alice","email":"alice@test.com"},
--  {"id":2,"name":"bob","email":"bob@test.com"},
--  {"id":3,"name":"carol","email":"carol@test.com"}]

-- (b) copy from 多行头跳过：csv 文件有 3 行说明文字
copy sensor_data from '/tmp/sensor.csv'
  with (format csv, header 3);  -- 跳过前 3 行

-- (c) on_error set_null：无效值自动置为 null
copy products (id, name, price) from '/tmp/products.csv'
  with (format csv, header, on_error set_null);

-- 遇到 price='n/a' 等非数字 → 自动存为 null，不再报错中断
```


## 二、性能与优化器：更快、更智能

### 1. jit 默认禁用

```sql
-- pg18：jit 默认开启
show jit;  -- on

-- pg19：jit 默认关闭
show jit;  -- off

-- 分析型查询需手动开启
set jit = on;
set jit_above_cost = 100000;
select count(*), avg(amount) from huge_table group by category;
```


### 2. anti-join 优化

```sql
-- 场景：查找没有下过订单的用户
-- pg18 可能走 hash left join + filter
-- pg19 自动优化为 anti-join，性能更好

explain analyze
select u.id, u.name
from users u
where u.id not in (select user_id from orders);

-- pg19 执行计划：
hash anti join  (cost=... rows=...)
 hash cond: (u.id = orders.user_id)
 ->  seq scan on users u
 ->  hash  (cost=... rows=...)
        ->  seq scan on orders

-- not in 含 null 值也能正确转 anti-join
-- memoize 节点也支持 anti-join 缓存，重复子查询更快
```


### 3. 并行 autovacuum

```sql
-- 全局设置：最多 4 个并行 vacuum worker
alter system set autovacuum_max_parallel_workers = 4;
select pg_reload_conf();

-- 表级设置：大表分配更多并行 worker
alter table huge_orders
  set (autovacuum_vacuum_parallel_workers = 4);

-- 查看 vacuum 执行情况
select schemaname, relname, last_autovacuum, last_autoanalyze
from pg_stat_user_tables
order by last_autovacuum desc nulls last
limit 10;
```


### 4. simd 加速 copy

```sql
-- 准备 1000 万行测试数据
create table sensor_data (
    ts     timestamptz,
    device text,
    value  float8
);

-- 生成 csv
copy (select
  '2025-01-01'::timestamptz + (i || ' seconds')::interval,
  'device_' || (i % 100),
  random() * 1000
  from generate_series(1, 10000000) i
) to '/tmp/sensor.csv' csv;

-- pg19 导入速度显著提升（自动使用 simd 指令解析文本/csv）
copy sensor_data from '/tmp/sensor.csv' csv;
-- 在支持 avx2/avx-512 的 cpu 上提速明显
```


### 5. lz4 成为默认 toast 压缩算法

```sql
-- 查看当前默认压缩算法
show default_toast_compression;
-- pg18: pglz
-- pg19: lz4

-- 新建表自动使用 lz4
create table documents (
    id      serial primary key,
    content text  -- 大文本自动 toast 压缩
);

-- 已有表可以手动切换
alter table documents
  alter column content set compression lz4;

-- 验证压缩效果
select pg_column_size(content) as compressed_size,
       length(content) as raw_length,
       round(1.0 * pg_column_size(content) / length(content), 2) as ratio
from documents
where length(content) > 10000
limit 5;
```


### 6. 其他性能改进

```sql
-- (a) radix sort：整数排序自动使用基数排序
explain analyze
select * from big_table order by id;
-- 执行计划中出现 "sort method: radix sort" 即表示生效

-- (b) tid range scan 并行化
set max_parallel_workers_per_gather = 4;
explain analyze
select * from huge_table where ctid between '(0,0)' and '(10000,0)';
-- 执行计划中出现 parallel tid range scan

-- (c) 异步 i/o 预读
alter system set io_min_workers = 2;
alter system set io_max_workers = 8;
select pg_reload_conf();

-- (d) 外键检查性能提升（无显式 demo，dml 自动受益）
-- 大量外键约束的 insert/update 场景性能提升明显

-- (e) 流式读取：gin vacuum 使用流式 i/o
-- vacuum 操作在 gin 索引表上自动使用流式读取
vacuum gin_indexed_table;
```


## 三、逻辑复制与高可用

### 1. 序列同步

```sql
-- 发布端（primary）
create publication pub_all for all tables;

-- 订阅端（subscriber）
create subscription sub_all
  connection 'host=primary dbname=mydb'
  publication pub_all;

-- 同步序列值
alter subscription sub_all refresh sequences;

-- 检查序列同步状态
select * from pg_get_sequence_data('my_table_id_seq');
-- 返回当前序列的 last_value、is_called 等信息

-- 确认订阅端序列值与发布端一致
select last_value from my_table_id_seq;
```


### 2. 发布排除表

```sql
-- 发布所有表，但排除日志表和临时表
create publication pub_main
  for all tables
  except (audit_log, temp_cache, session_data);

-- 修改已有发布，增加排除
alter publication pub_main
  set all tables except (audit_log, temp_cache, session_data, debug_log);

-- 查看发布包含哪些表
select * from pg_publication_tables where pubname = 'pub_main';
```


### 3. standby 等待 lsn

```sql
-- 场景：应用写入主库后，需要确认备库已回放
-- 主库写入并获取 lsn
insert into orders (customer, amount) values ('alice', 99.9)
  returning pg_current_wal_lsn() as write_lsn;
-- 返回: 0/1a2b3c4d

-- 在备库等待该 lsn 已回放（超时 5 秒）
select pg_wal_wait_for_lsn('0/1a2b3c4d', 'replay', 5000);
-- true  = 已回放
-- false = 超时未回放

-- 也可以等待 flush（刷盘）
select pg_wal_wait_for_lsn('0/1a2b3c4d', 'flush', 5000);
```


### 4. 其他复制改进

```sql
-- (a) 冲突信息保留
alter subscription sub_all set (retain_conflict_info = true);
-- 冲突解决时保留完整信息，便于排查

-- (b) 死元组保留限制
alter subscription sub_all
  set (retain_dead_tuples = true,
       max_retention_duration = '24 hours');
-- 保留死元组用于冲突检测，但限制最多 24 小时防止膨胀

-- (c) wal sender 关闭超时
alter system set wal_sender_shutdown_timeout = '30s';
select pg_reload_conf();
-- 关闭时最多等 30 秒让副本同步

-- (d) 每订阅 wal receiver 超时
-- 为不同订阅设置不同超时
create subscription sub_fast
  connection 'host=fast_replica dbname=mydb'
  publication pub_all
  with (wal_receiver_timeout = '10s');

create subscription sub_slow
  connection 'host=slow_replica dbname=mydb'
  publication pub_all
  with (wal_receiver_timeout = '60s');
```


## 四、监控、统计与可观测性

```sql
-- (a) pg_stat_lock：查看锁统计
select * from pg_stat_lock;
-- 按锁类型展示获取次数、等待次数、等待时间等

-- 配合函数使用
select * from pg_stat_get_lock('relation');

-- (b) pg_stat_recovery：查看备库恢复进度
select * from pg_stat_recovery;
-- 展示恢复的 wal 位置、延迟、回放速率等

-- (c) autovacuum 评分系统
-- 通过权重参数精细控制 vacuum 优先级
alter system set autovacuum_vacuum_score_weight = 2.0;
alter system set autovacuum_freeze_score_weight = 5.0;
alter system set autovacuum_analyze_score_weight = 1.5;
select pg_reload_conf();

-- 表级权重
alter table critical_table
  set (autovacuum_vacuum_score_weight = 10.0);

-- (d) explain analyze io
explain (analyze, io)
select * from huge_table where created_at > '2025-01-01';
-- 输出新增 io 相关信息：异步读取次数、预读命中数等

-- auto_explain 也支持
alter system set auto_explain.log_io = on;

-- (e) wal 全页写统计
select fpw_bytes from pg_stat_wal;
-- 查看全页写（full page image）的字节数

-- explain 中也可以看到
explain (analyze, wal) insert into orders select * from temp_orders;
-- 输出包含 "wal: records=n bytes=n fpw_bytes=n"

-- (f) 进程级日志级别
-- 只为 autovacuum 设置 debug 级别，其他保持 warning
alter system set log_min_messages = 'warning,autovacuum:debug1';
select pg_reload_conf();

-- 也可以为 wal writer 单独设置
-- 'warning,wal_writer:notice,wal_receiver:debug2'

-- (g) autoanalyze 独立日志
alter system set log_autoanalyze_min_duration = '5s';
select pg_reload_conf();
-- 超过 5 秒的 autoanalyze 会单独记录到日志

-- (h) xid 回卷预警
-- 当 oldestxid 距 wraparound 不足 1 亿时告警（原 4000 万）
select datname, age(datfrozenxid) as xid_age,
       2000000000 - age(datfrozenxid) as remaining
from pg_database
order by age(datfrozenxid) desc;
```


## 五、安全与认证

### 1. 移除 radius 认证

```
# pg18 的 pg_hba.conf 中可以配置：
# host all all 0.0.0.0/0 radius radiusserver=10.0.0.1

# pg19：radius 认证已完全移除
# 上述配置在 pg19 中不再支持，启动会报错
# 建议迁移到 ldap 或 scram-sha-256
```


### 2. md5 密码警告

```sql
-- 使用 md5 密码登录时会收到警告
-- warning:  md5 password authentication is deprecated
--           and will be removed in a future release.

-- 禁用警告（不推荐）
alter system set md5_password_warnings = off;
select pg_reload_conf();

-- 推荐：迁移到 scram-sha-256
alter system set password_encryption = 'scram-sha-256';
select pg_reload_conf();

-- 重新设置密码（自动使用 scram-sha-256）
alter user alice password 'new_secure_password';
```


### 3. 在线数据校验和

```sql
-- 在线启用数据校验和（无需停机！）
-- pg18 及之前：必须用 pg_checksums 工具离线操作

-- pg19：直接在线启用
alter database mydb set data_checksums = on;
-- 后台自动为所有数据页计算校验和

-- 查看校验和状态
select datname, data_checksums
from pg_database;

-- 在线禁用
alter database mydb set data_checksums = off;
```


### 4. oauth 增强

```sql
-- libpq 连接时指定 oauth token 和 ca 证书
-- psql "host=db.example.com dbname=mydb
--       oauth_bearer_token=eyjhbgci...
--       oauth_ca_file=/etc/ssl/certs/ca-bundle.crt"

-- 自定义 token 验证 hook（扩展开发）
-- 注册 pqauthdata_oauth_bearer_token_v2 hook
create extension my_oauth_validator;

-- 在 pg_hba.conf 中配置
-- host all all 0.0.0.0/0 scram-sha-256 oauth
```


### 5. 对象名称安全检查

```sql
-- pg18 及之前：允许创建包含换行符的名称（存在注入风险）
-- create role "admin\n--drop table users";  -- 可能成功

-- pg19：自动拦截
create role $$evil
name$$;
-- error:  role name must not contain newline characters

-- pg_upgrade 也会检查旧集群
-- 如果发现含回车/换行符的数据库/角色/表空间名
-- pg_upgrade 会中断并提示先清理
```

## 六、数据类型与函数

```sql
-- (a) oid8：64 位无符号 oid
select oid8 '18446744073709551615';
-- 支持超大 oid 值，扩展系统目录空间

-- (b) jsonpath 字符串方法
select jsonb_path_query(
  '{"name": "  hello world  "}',
  '$.name.trim()'
);
-- 返回: "hello world"

select jsonb_path_query(
  '{"email": "user@example.com"}',
  '$.email.lower()'
);
-- 返回: "user@example.com"

select jsonb_path_query(
  '{"path": "a/b/c"}',
  '$.path.split_part("/", 2)'
);
-- 返回: "b"

select jsonb_path_query(
  '{"title": "hello world"}',
  '$.title.replace("hello", "hello")'
);
-- 返回: "hello world"

-- (c) bytea ↔ uuid 转换
select 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11'::uuid::bytea;
-- 返回: \xa0eebc999c0b4ef8bb6d6bb9bd380a11

select '\xa0eebc999c0b4ef8bb6d6bb9bd380a11'::bytea::uuid;
-- 返回: a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11

-- (d) random(min, max) 支持时间类型
select random('2025-01-01'::date, '2025-12-31'::date);
-- 返回: 2025-07-14（随机日期）

select random('2025-01-01 00:00:00'::timestamp,
              '2025-01-01 23:59:59'::timestamp);
-- 返回: 2025-01-01 14:32:17（随机时间戳）

-- (e) base64url / base32hex 编码
select encode('hello, pg19!'::bytea, 'base64url');
-- 返回: sgvsbg8sifbhmtkh

select decode('sgvsbg8sifbhmtkh', 'base64url');
-- 返回: \x48656c6c6f2c205047313921

select encode('hello'::bytea, 'base32hex');
-- 返回: 91imor3f（保留字典序）

-- (f) ddl 生成函数
select pg_get_role_ddl('alice');
-- 返回: create role alice login password '...' ...

select pg_get_tablespace_ddl('pg_default');
-- 返回: create tablespace pg_default location '...'

select pg_get_database_ddl('mydb');
-- 返回: create database mydb with ...

-- (g) 全文搜索：波兰语/世界语
select to_tsvector('polish', 'programowanie jest fajne');
-- 波兰语词干提取

select to_tsvector('esperanto', 'programado estas amuza');
-- 世界语词干提取
```

## 七、升级须知：breaking changes

升级前务必关注以下不兼容变更：

```sql
-- (a) jit 默认关闭
-- 依赖 jit 的分析负载需手动开启
set jit = on;

-- (b) max_locks_per_transaction 默认值翻倍
show max_locks_per_transaction;
-- pg18: 64 → pg19: 128（实际锁容量不变）

-- (c) inet/cidr 索引 opclass 变更
-- 升级前删除旧的 btree_gist inet 索引
drop index idx_inet_btree_gist;
-- pg19 默认使用 gist，重建即可
create index idx_inet_gist on connections using gist(client_ip);

-- (d) standard_conforming_strings 强制开启
-- 以下写法在 pg19 中报错
set standard_conforming_strings = off;
-- error:  cannot change standard_conforming_strings

-- escape_string_warning 变量已被移除
show escape_string_warning;
-- error:  unrecognized configuration parameter

-- (e) mule_internal 编码移除
-- 如果旧库使用了 mule_internal 编码：
-- 1. 在旧版本上 pg_dump
-- 2. 创建新编码的库
-- 3. pg_restore
create database new_db encoding 'utf8';
pg_restore -d new_db old_db.dump

-- (f) json_array() 行为变更
select json_array(select 1 where false);
-- pg18: null
-- pg19: []

-- 如需兼容旧行为
select case when count(*) = 0 then null
       else json_array(select 1 where false) end
from (select 1 where false) t;

-- (g) postgres_fdw 只读事务传递
begin read only;
-- pg19：远程会话也被强制只读
select * from remote_table;      -- ok
update remote_table set x = 1;   -- error: cannot execute update in a read-only transaction
commit;

-- (h) c 语言标准升级
# 源码编译需要 c11 支持的编译器
./configure cc=gcc  # gcc 4.8+ 支持 c11
# windows 需要 vs2019 或更高版本
```

## 八、其他亮点

```bash
# (a) pg_dumpall 支持非文本格式
pg_dumpall --format=custom -f backup.custom
pg_dumpall --format=directory -f backup_dir/
pg_dumpall --format=tar -f backup.tar

# (b) pg_upgrade 大对象优化
# pg16+ 升级时自动优化，无需额外操作
pg_upgrade -b /usr/lib/postgresql/18/bin \
           -b /usr/lib/postgresql/19/bin \
           -d /var/lib/postgresql/18/data \
           -d /var/lib/postgresql/19/data
# 含大量大对象的库升级速度显著提升

# (c) unicode 17.0.0
select * from pg_unicode_properties where property = 'age' and value = '17.0';
-- 支持最新 unicode 字符集

# (d) pg_plan_advice 模块
# postgresql.conf 中加载
# shared_preload_libraries = 'pg_plan_advice'
create extension pg_plan_advice;

-- 强制特定查询使用索引扫描
insert into plan_advice (query_pattern, directives) values
  ('select * from orders where%', '{"enable_seqscan": false}');
```

## 总结

postgresql 19 是一次**全面而深入的版本更新**：

- 🔍 **sql/pgq 图查询**让 pg 成为"关系型 + 图"双引擎
- ⚡ **simd 加速、并行 autovacuum、lz4 默认压缩**带来全方位性能提升
- 🔄 **逻辑复制增强**（序列同步、排除表、冲突保留）让分布式场景更可靠
- 🔒 **在线数据校验和、oauth 增强、md5 警告**持续加固安全基线
- 📊 **pg_stat_lock、io 报告、进程级日志**让运维可观测性再升级

目前 pg19 处于 beta1 阶段，预计将于 2026 年 9-10 月正式发布。
