# MySQL2PG v3.4.0 正式发布：支持 MySQL 5.7+ 完整评估和迁移报告的数据库迁移工具

> **导读**：在生产环境中，数据库迁移一直是一项高风险、高成本的技术挑战。如何在不影响业务的前提下，将 MySQL 数据库平滑迁移到 PostgreSQL？今天为大家介绍一款专业的数据库迁移工具——MySQL2PG，它支持完整的迁移前评估、可视化 HTML 报告、42 个视图和 113 个函数 100% 转换，以及百万级数据的高性能同步。

---

## 一、为什么需要数据库迁移工具？

### 1.1 生产环境的迁移痛点

在实际的数据库迁移项目中，DBA 和运维团队通常面临以下挑战：

- **兼容性风险**：MySQL 和 PostgreSQL 在数据类型、函数语法、索引机制等方面存在显著差异，手动转换容易遗漏
- **数据一致性**：百万级甚至亿级数据量的同步，如何确保数据完整性和一致性？
- **迁移评估缺失**：迁移前无法准确评估工作量和风险，项目进度难以把控
- **回滚成本高**：迁移失败后缺乏有效的回滚机制，可能导致业务中断
- **性能瓶颈**：单线程迁移速度慢，影响业务窗口期

### 1.2 MySQL2PG 的工具定位

**MySQL2PG** 是一款用 Go 语言开发的专业级数据库转换工具，专注于将 MySQL 数据库无缝迁移到 PostgreSQL。它提供了**评估模式**和**迁移模式**两种工作方式，涵盖表结构、数据、视图、索引、函数、用户及权限的完整转换。

**核心优势**：
- ✅ **评估先行**：迁移前生成详细的兼容性评估报告，识别潜在风险
- ✅ **可视化报告**：深色终端风格的 HTML 报告，迁移进度和结果一目了然
- ✅ **广泛兼容**：支持 MySQL 5.7/8.0/9.0 → PostgreSQL 12/13/14/15/16/17/18
- ✅ **高性能**：并发转换引擎，速度比单线程提升 5-10 倍
- ✅ **数据校验**：同步后自动验证数据一致性，确保迁移完整性

### 1.3 v3.4.0 核心亮点

本次发布的 v3.4.0 版本带来了以下重大更新：

1. **新增 assess 评估模式**：迁移前兼容性评估，生成 HTML 评估报告
2. **完整版本兼容**：MySQL 5.7/8.0/9.0 → PostgreSQL 12-18 全版本支持
3. **MPP 数据库支持**：Greenplum、YugabyteDB 等分布式数据库
4. **42 个视图 100% 转换**：所有 MySQL 5.7+ 视图语法完整支持
5. **113 个函数核心语法覆盖**：复杂存储过程语法准确转换

**技术统计**：
- 40+ 种 MySQL 字段类型映射，准确率 99.9%
- 42 个视图 100% 可转换
- 113 个函数核心语法 100% 可转换
- 41+ 测试用例全部通过
- 代码覆盖率 88%+

---

## 二、支持的数据库版本矩阵

### 2.1 MySQL（源数据库）

| 版本 | 类型 | EOL | 支持状态 | 特性 |
|------|------|-----|---------|------|
| **5.7** | LTS | 2023-10 | ✅ 完全支持 | 基本函数、JSON 基础 |
| **8.0** | LTS | 2026-04 | ✅ 完全支持 | REGEXP_*、窗口函数、CTE |
| **8.4** | LTS | 2032-04 | ✅ 完全支持 | 同 8.0，性能优化 |
| **9.0** | Innovation | 2026-04 | ✅ 完全支持 | JSON_ARRAY_INSERT、增强 REGEXP_* |

### 2.2 PostgreSQL（目标数据库）

| 版本 | 发布日期 | EOL | 支持状态 | 特性 |
|------|---------|-----|---------|------|
| **12** | 2019-11 | 2024-11 | ✅ 完全支持 | 最低支持版本 |
| **13** | 2020-09 | 2025-11 | ✅ 完全支持 | 聚合函数增强 |
| **14** | 2021-09 | 2026-11 | ✅ 完全支持 | JSONB 路径查询，**推荐版本** |
| **15** | 2022-10 | 2027-11 | ✅ 完全支持 | 权限管理变更 |
| **16** | 2023-09 | 2028-11 | ✅ 完全支持 | JSONB 性能提升，**推荐版本** |
| **17** | 2024-09 | 2029-11 | ✅ 完全支持 | SQL/JSON 增强 |
| **18** | 2025-09 | 2030-11 | ✅ 完全支持 | 最新版本 |

---

## 三、核心功能详解

### 3.1 评估模式（assess 命令）⭐

**assess 模式** 是 v3.4.0 新增的核心功能，用于在正式迁移前进行兼容性评估，识别潜在风险并生成详细的 HTML 报告。

#### 3.1.1 使用方式

```bash
# 创建配置文件
cp config.example.yml config.yml

# 编辑配置文件，填写 MySQL 和 PostgreSQL 连接信息
vim config.yml

# 运行评估模式
./mysql2pg assess config.yml

# 生成评估报告（自动打开浏览器）
open assessment-*.html
```

#### 3.1.2 评估报告内容

评估报告采用**深色终端风格**设计，包含以下核心内容：

**（1）统计卡片**

![统计卡片](https://example.com/stats-cards.png)

- **总体评分**：0-100 分，分数越高风险越低
- **风险等级**：低/中/高，基于无法转换的对象数量
- **表数量**：待迁移的表总数
- **视图数量**：待迁移的视图总数
- **函数数量**：待迁移的函数总数
- **索引数量**：待迁移的索引总数
- **用户数量**：待迁移的用户数
- **权限数量**：待迁移的表权限数

**（2）表详细清单**

展示所有待迁移表的信息，包括：
- 表名
- 行数
- DDL 行数
- 风险等级（无风险/低风险/中风险/高风险）
- 风险描述和建议

**（3）视图详细清单**

展示所有待迁移视图的信息：
- 视图名
- DDL 行数（格式化后的真实行数）
- 风险等级
- 风险描述（如包含 MySQL 特有函数）

**（4）函数详细清单**

展示所有待迁移函数的信息：
- 函数名
- DDL 行数
- 返回类型
- 风险等级
- 风险描述（如包含不兼容的语法）

**（5）索引详细清单**

展示所有待迁移索引的信息：
- 索引名
- 所属表
- 索引类型（主键/唯一索引/普通索引/全文索引）
- 索引列

**（6）用户详细清单**

展示所有待迁移用户的信息：
- 用户名（仅显示用户名，不显示@host）
- 风险等级
- 风险描述（如密码策略不兼容）

**（7）权限详细清单**

展示所有待迁移表权限的信息：
- 用户名
- 表名
- 权限类型（SELECT/INSERT/UPDATE/DELETE 等）
- 风险等级

#### 3.1.3 评估报告截图示例

> **注**：实际使用时会生成真实的 HTML 报告文件，以下为报告结构示意图。

```
┌──────────────────────────────────────────────┐
│ > MySQL2PG 迁移前评估报告    2026-05-01     │
│   源：MySQL @ 192.168.1.1:3306             │
│   目标：PostgreSQL @ 192.168.1.1:5432      │
├──────────────────────────────────────────────┤
│  [总体评分：100/100]  [风险等级：低]        │
│  [表：189] [视图：14] [函数：11]            │
│  [索引：588] [用户：4] [权限：12]           │
├──────────────────────────────────────────────┤
│  📋 表详细清单                  [189 张表]   │
│  #  | 表名           | 行数 | DDL 行数 | 风险  │
│  1  | case_01_integers| 0   | 17      | 无风险│
│  2  | case_02_boolean | 0   | 10      | 无风险│
│  ...                                        │
├──────────────────────────────────────────────┤
│  👁 视图详细清单                [14 个视图]  │
│  #  | 视图名         | DDL 行数 | 风险       │
│  1  | view_case01   | 3        | 无风险     │
│  2  | view_case02   | 5        | 无风险     │
│  ...                                        │
├──────────────────────────────────────────────┤
│  ⚡ 函数详细清单                [11 个函数]  │
│  #  | 函数名         | DDL 行数 | 返回类型   │
│  1  | get_user_data | 15       | TABLE      │
│  2  | calc_total    | 8        | INTEGER    │
│  ...                                        │
└──────────────────────────────────────────────┘
```

#### 3.1.4 评估逻辑说明

assess 模式使用与正常迁移模式**相同的转换函数**来判断兼容性：

- **表结构**：调用 `postgres.ConvertTableDDL()` 尝试转换表 DDL
- **视图**：调用 `postgres.ConvertViewDDL()` 尝试转换视图定义
- **函数**：调用 `postgres.ConvertFunctionDDL()` 尝试转换函数定义
- **索引**：调用索引转换逻辑判断索引类型和兼容性
- **用户和权限**：调用用户和权限转换逻辑判断兼容性

如果转换函数抛出错误，则记录为**高风险**对象；如果转换成功但存在潜在问题（如使用 MySQL 特有函数），则记录为**中风险**或**低风险**对象。

---

### 3.2 迁移模式（正常命令）⭐

迁移模式是 MySQL2PG 的核心功能，执行完整的数据库迁移流程。

#### 3.2.1 8 步转换流程

```
开始
 │
 ├─▶ [Step 0] test_only 模式？
 │     ├─ 是 → 测试 MySQL & PostgreSQL 连接 → 显示版本 → 退出
 │     └─ 否 → 继续
 │
 ├─▶ [Step 1] 读取 MySQL 表定义
 │     ├─ 过滤表（白名单/黑名单如果配置）
 │     └─ 获取元数据：表、视图、索引、函数、用户、权限
 │
 ├─▶ [Step 2] 转换表结构 (tableddl: true)
 │     ├─ 解析 MySQL CREATE TABLE
 │     ├─ 字段类型智能映射（40+ 类型映射）
 │     ├─ 提取主键列作为 MPP 分布键
 │     └─ 在 PostgreSQL 中创建表（skip_existing_tables 控制是否跳过）
 │
 ├─▶ [Step 3] 转换视图 (view: true)
 │     ├─ 过滤视图（exclude_view_list 如果配置）
 │     ├─ MySQL 视图定义转换为 PostgreSQL 兼容语法
 │     └─ 执行 CREATE VIEW 语句
 │
 ├─▶ [Step 4] 同步数据 (data: true)
 │     ├─ 清空目标表（如果 truncate_before_sync=true）
 │     ├─ 分批读取 MySQL 数据（max_rows_per_batch，默认 50000）
 │     ├─ 批量插入 PostgreSQL（batch_insert_size，默认 50000）
 │     ├─ 并发线程数由 concurrency 控制
 │     └─ 自动禁用外键约束和索引提高性能
 │
 ├─▶ [Step 5] 转换索引 (indexes: true)
 │     ├─ 重建：主键、唯一索引、普通索引、全文索引
 │     └─ 批量处理（max_indexes_per_batch=20）
 │
 ├─▶ [Step 6] 转换函数 (functions: true)
 │     ├─ 过滤函数（exclude_function_list 如果配置）
 │     ├─ 50+ 函数映射（如 NOW()→CURRENT_TIMESTAMP）
 │     └─ 执行 CREATE FUNCTION 语句
 │
 ├─▶ [Step 7] 转换用户 (users: true)
 │     └─ MySQL 用户 → PostgreSQL 角色（保留密码哈希）
 │
 ├─▶ [Step 8] 转换表权限 (table_privileges: true)
 │     └─ GRANT 语句转换为 PostgreSQL 等效语法
 │
 └─▶ [Final Step] 数据校验与完成 (validate_data: true)
       ├─ 比较行数：MySQL vs PostgreSQL
       ├─ 重新启用外键约束和索引
       ├─ 记录不一致表（如果 truncate_before_sync=false，继续执行）
       └─ 输出转换统计报告和性能指标
```

#### 3.2.2 迁移报告内容

迁移完成后，使用以下命令生成 HTML 迁移报告：

```bash
# 从转换日志生成报告
./mysql2pg report -l conversion.log

# 包含错误日志
./mysql2pg report -l conversion.log -e errors.log

# 自定义输出路径
./mysql2pg report -l conversion.log -o my-report.html
```

迁移报告包含以下核心内容：

**（1）统计卡片**

![迁移报告统计卡片](https://example.com/migration-stats.png)

- **表数量**：已迁移的表总数
- **行数**：已同步的总行数
- **视图数量**：已迁移的视图总数
- **索引数量**：已迁移的索引总数
- **函数数量**：已迁移的函数总数
- **错误数量**：迁移过程中的错误数

**（2）性能柱状图**

展示各阶段的耗时情况：

```
┌──────────────────────────────────────────────┐
│  ⚡ 性能统计                                 │
│  转换表结构  [177]  ████████████   2.79s    │
│  同步表数据  [177]  ████████       1.19s    │
│  转换表视图  [14]   ████           1.20s    │
│  转换表索引  [588]  ██████████     2.15s    │
│  转换库函数  [11]   ██             0.25s    │
│  转换库用户  [4]    █              0.18s    │
│  转换表权限  [12]   ███            1.62s    │
├──────────────────────────────────────────────┤
│  总耗时：5.2s  |  193 rows/s                │
└──────────────────────────────────────────────┘
```

**（3）表详细清单**

展示所有已迁移表的详细信息：

| # | 表名 | 行数 | 状态 |
|---|------|------|------|
| 1 | case_01_integers | 0 | 已转换 |
| 2 | case_02_boolean | 0 | 已转换 |
| 3 | act_hi_comment | 10 | 数据一致 |
| 4 | sessions | - | 已存在 |

**状态说明**：
- **已转换**：表结构成功创建
- **已存在**：表已存在，跳过创建
- **数据一致**：数据同步完成且行数一致
- **数据不一致**：数据同步完成但行数不一致
- **空表**：表没有数据，跳过同步

**（4）数据不一致表统计**

如果 `truncate_before_sync=false`，会显示数据不一致的表：

```
┌──────────────────────────────────────────────┐
│  ⚠ 数据不一致表统计             [1 张表]     │
├───────────────┬─────────┬─────────┬─────────┤
│ 表名          │ MySQL   │ PG      │ 差异    │
├───────────────┼─────────┼─────────┼─────────┤
│ act_hi_comment│ 10      │ 30      │ -20     │
└───────────────┴─────────┴─────────┴─────────┘
```

**（5）错误和警告**

展示迁移过程中的错误和警告信息：

```
┌──────────────────────────────────────────────┐
│  ❌ 错误                        [2 个]       │
├──────────────────────────────────────────────┤
│ #1 插入表 sessions 数据失败：...             │
│ #2 转换索引 idx_email 失败：...              │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│  ⚡ 警告                        [1 个]       │
├──────────────────────────────────────────────┤
│ #1 表 sessions: 没有主键，可能导致同步效率低  │
└──────────────────────────────────────────────┘
```

#### 3.2.3 迁移报告截图示例

> **注**：实际使用时会生成真实的 HTML 报告文件，以下为报告结构示意图。

```
┌──────────────────────────────────────────────┐
│ > MySQL2PG 迁移报告              2026-05-01 │
│   源：MySQL 8.0.45                           │
│   目标：PostgreSQL 16.1                      │
│   进度：████████████ 100% (796/796) 完成     │
├──────────────────────────────────────────────┤
│  [表：177] [行：10] [视图：14]              │
│  [索引：588] [函数：11] [错误：0]           │
├──────────────────────────────────────────────┤
│  ⚡ 性能统计                  [各阶段耗时]   │
│  转换表结构  [177]  ████████       2.79s    │
│  同步表数据  [177]  ████           1.19s    │
│  Total: 5.2s  |  193 rows/s                 │
├──────────────────────────────────────────────┤
│  📋 表详细清单                  [177 张表]   │
│  #  | 表名              | 行数 | 状态       │
│  1  | case_01_integers  | 0    | [已转换]   │
│  2  | act_hi_comment    | 10   | [数据一致] │
│  ...                                        │
├──────────────────────────────────────────────┤
│  ⚠ 数据不一致表统计            [1 张表]     │
│  Table           | MySQL | PG   | Delta     │
│  act_hi_comment  | 10    | 30   | -20       │
├──────────────────────────────────────────────┤
│  ❌ 错误                        [0 个]       │
│  （无错误）                                  │
├──────────────────────────────────────────────┤
│  ⚡ 警告                        [0 个]       │
│  （无警告）                                  │
└──────────────────────────────────────────────┘
```

---

### 3.3 表结构类型映射

MySQL2PG 支持 40+ 种 MySQL 字段类型到 PostgreSQL 的精确转换，映射准确率达到 99.9%。

#### 3.3.1 整数类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `bigint`, `bigint(20)` | `BIGINT` | 所有 bigint 变体 |
| `int`, `int(11)`, `integer` | `INTEGER` | 所有 int 变体 |
| `mediumint`, `mediumint(9)` | `INTEGER` | mediumint 转换 |
| `smallint`, `smallint(6)` | `SMALLINT` | 所有 smallint 变体 |
| `tinyint(1)` | `BOOLEAN` | **特殊情况**：布尔值 |
| `tinyint`, `tinyint(4)` | `SMALLINT` | 其他 tinyint 变体 |
| `bigint AUTO_INCREMENT` | `BIGSERIAL` | 自增 bigint |
| `int AUTO_INCREMENT` | `SERIAL` | 自增 int |

#### 3.3.2 浮点类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `decimal`, `numeric` | `DECIMAL` | 保留精度 |
| `double`, `double precision` | `DOUBLE PRECISION` | 双精度浮点数 |
| `float` | `REAL` | 单精度浮点数 |

#### 3.3.3 字符串类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `char`, `char(1)` | `CHAR` | 保留长度 |
| `varchar`, `varchar(255)` | `VARCHAR` | 保留长度 |
| `text`, `longtext` | `TEXT` | 所有 text 变体 |
| `enum` | `VARCHAR(255)` | 枚举转字符串 |
| `set` | `VARCHAR(255)` | 集合转字符串 |

#### 3.3.4 二进制类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `blob`, `longblob` | `BYTEA` | 所有 binary 类型 |
| `binary`, `varbinary` | `BYTEA` | 所有 binary 类型 |

#### 3.3.5 时间类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `datetime`, `datetime(6)` | `TIMESTAMP` | 保留精度 |
| `timestamp`, `timestamp(6)` | `TIMESTAMP` | 保留精度 |
| `date` | `DATE` | 日期 |
| `time` | `TIME` | 时间，保留精度 |
| `year` | `INTEGER` | 年份转整数 |

#### 3.3.6 JSON 和特殊类型

| MySQL 类型 | PostgreSQL 类型 | 说明 |
|-----------|----------------|------|
| `json`, `json(1024)` | `JSON` | JSON 对象 |
| `jsonb` | `JSONB` | 二进制 JSON |
| `geometry`, `point` | 相同 | 空间类型保留 |

---

### 3.4 视图和函数转换

#### 3.4.1 视图转换（42 个视图 100% 转换）

MySQL2PG 支持所有 MySQL 5.7+ 视图语法的完整转换，包括：

**（1）标识符处理**
- 反引号 (`) → 双引号 (")

**（2）语法兼容调整**
- `LIMIT a,b` → `LIMIT b OFFSET a`
- 表连接条件优化，自动添加表别名

**（3）常用函数转换**

| MySQL 函数 | PostgreSQL 函数 | 说明 |
|-----------|----------------|------|
| `IFNULL(expr1, expr2)` | `COALESCE(expr1, expr2)` | 空值处理 |
| `IF(cond, then, else)` | `CASE WHEN cond THEN then ELSE else END` | 条件判断 |
| `GROUP_CONCAT(x)` | `STRING_AGG(CAST(x AS TEXT), ',')` | 分组拼接 |
| `CONCAT(a, b)` | `a \|\| b` | 字符串连接 |
| `DATE_FORMAT(dt, fmt)` | `TO_CHAR(dt, fmt)` | 日期格式化 |
| `JSON_EXTRACT(doc, path)` | `doc -> 'path'` | JSON 提取 |

#### 3.4.2 函数转换（113 个函数 100% 转换）

**（1）JSON 函数**

| MySQL 函数 | PostgreSQL 函数 | 说明 |
|-----------|----------------|------|
| `JSON_OBJECT()` | `json_build_object()` | 创建 JSON 对象 |
| `JSON_ARRAY()` | `json_build_array()` | 创建 JSON 数组 |
| `JSON_INSERT(doc, path, val)` | `jsonb_set(doc, path, to_jsonb(val), true)` | 插入 JSON |
| `JSON_REPLACE(doc, path, val)` | `jsonb_set(doc, path, to_jsonb(val), false)` | 替换 JSON |
| `JSON_SET(doc, path, val)` | `jsonb_set(doc, path, to_jsonb(val))` | 设置 JSON |
| `JSON_REMOVE(doc, path)` | `doc - 'key'` | 删除 JSON 键 |
| `JSON_MERGE_PATCH(doc1, doc2)` | `(doc1::jsonb \|\| doc2::jsonb)` | JSON 合并 |
| `JSON_KEYS(doc)` | `json_object_keys(doc)` | JSON 键名 |
| `JSON_LENGTH(doc)` | `jsonb_array_length(doc)` | JSON 长度 |

**（2）时间函数**

| MySQL 函数 | PostgreSQL 函数 | 说明 |
|-----------|----------------|------|
| `UNIX_TIMESTAMP()` | `extract(epoch from now())` | 时间戳 |
| `FROM_UNIXTIME(ts)` | `to_timestamp(ts)` | 时间戳转日期 |
| `DATE_FORMAT(dt, fmt)` | `to_char(dt, fmt)` | 日期格式化 |
| `STR_TO_DATE(str, fmt)` | `to_date(str, fmt)` | 字符串转日期 |
| `DATEDIFF(d1, d2)` | `date_part('day', d1 - d2)` | 日期差 |
| `DATE_ADD(dt, INTERVAL n unit)` | `dt + n::interval '1 unit'` | 日期加法 |
| `DATE_SUB(dt, INTERVAL n unit)` | `dt - n::interval '1 unit'` | 日期减法 |
| `YEARWEEK(dt)` | `(extract(year from dt) * 100 + extract(week from dt))` | 年周 |
| `DAYNAME(dt)` | `to_char(dt, 'Day')` | 星期名称 |
| `MONTHNAME(dt)` | `to_char(dt, 'Month')` | 月份名称 |
| `QUARTER(dt)` | `extract(quarter from dt)` | 季度 |
| `WEEK(dt)` | `extract(week from dt)` | 周数 |

**（3）字符串函数**

| MySQL 函数 | PostgreSQL 函数 | 说明 |
|-----------|----------------|------|
| `INSTR(str, substr)` | `STRPOS(str, substr)` | 子串位置 |
| `LOCATE(substr, str)` | `STRPOS(str, substr)` | 子串位置 |
| `RLIKE pattern` | `~ 'pattern'` | 正则匹配 |
| `REGEXP_LIKE(expr, pat)` | `expr ~ 'pat'` | 正则匹配 |
| `REGEXP_REPLACE(str, pat, repl)` | `regexp_replace(str, pat, repl)` | 正则替换 |
| `REGEXP_SUBSTR(str, pat)` | `SUBSTRING(str FROM pat)` | 正则子串 |

**（4）系统与加密函数**

| MySQL 函数 | PostgreSQL 函数 | 说明 |
|-----------|----------------|------|
| `LAST_INSERT_ID()` | `lastval()` | 最后插入 ID |
| `CONNECTION_ID()` | `pg_backend_pid()` | 连接 ID |
| `DATABASE()` | `current_database()` | 数据库名 |
| `USER()` | `current_user` | 用户名 |
| `MD5(str)` | `md5(str)` | MD5 加密 |
| `SHA1(str)` | `sha1(str)` | SHA1 加密 |
| `UUID()` | `uuid_generate_v4()` | 生成 UUID |

**（5）存储过程语法转换**

| MySQL | PostgreSQL | 说明 |
|-------|-----------|------|
| `LEAVE label` | `EXIT label` | 退出循环 |
| `ITERATE label` | `CONTINUE label` | 继续循环 |
| `WHILE cond DO ... END WHILE` | `WHILE cond LOOP ... END LOOP` | While 循环 |
| `REPEAT ... UNTIL cond` | `LOOP ... EXIT WHEN cond; END LOOP` | Repeat 循环 |
| `RETURNS DOUBLE` | `RETURNS DOUBLE PRECISION` | 返回类型 |
| `READS SQL DATA` | _移除_ | PostgreSQL 不检查 |
| `DETERMINISTIC` | `IMMUTABLE` | 确定性函数 |

---

### 3.5 数据同步

#### 3.5.1 批量处理优化

MySQL2PG 采用多项优化技术，实现高性能数据同步：

- **批量读取**：每批 50,000 行（可配置），减少网络往返
- **批量插入**：使用 `pgx.CopyFrom`，每批 50,000 行
- **并发控制**：默认 10 个并发线程（可配置）
- **连接池管理**：MySQL 最大 100 连接，PostgreSQL 最大 50 连接

#### 3.5.2 性能优化技术

| 优化项 | 优化前 | 优化后 | 效果 |
|-------|--------|--------|------|
| 批次大小 | 1,000 / 10,000 | 50,000 / 50,000 | -98% 网络往返 |
| Scan 目标 | `*interface{}` | 类型化（`*int64`, `*string`） | -99.998% 分配 |
| 行切片分配 | 每行 `make()` | `sync.Pool` 复用 | -99.9% 分配 |
| `[]byte`→`string` | 每行转换 | 直接传递 `[]byte` | -99.6% 分配 |
| **每批堆分配** | ~60MB | ~0.4MB | **-99.2%** |

**结果**：消除 GC 压力，**5-8 倍吞吐量提升**。

#### 3.5.3 数据校验机制

同步完成后，自动进行数据校验：

```yaml
conversion:
  options:
    validate_data: true         # 启用数据校验
    truncate_before_sync: true  # 同步前清空表数据
```

**校验逻辑**：
1. 查询 MySQL 和 PostgreSQL 表的行数
2. 比较行数是否一致
3. 如果不一致：
   - `truncate_before_sync=true`：中断执行并返回错误
   - `truncate_before_sync=false`：继续执行，记录不一致表

**不一致表统计**：
```
+------------------+----------------+------------------+
| 表名             | MySQL 数据量   | PostgreSQL 数据量 |
+------------------+----------------+------------------+
| user             | 327680         | 655360           |
| users_20251201   | 200002         | 600006           |
+------------------+----------------+------------------+
```

---

### 3.6 索引和权限转换

#### 3.6.1 索引转换

支持所有 MySQL 索引类型到 PostgreSQL 的转换：

| 索引类型 | MySQL | PostgreSQL | 说明 |
|---------|-------|-----------|------|
| 主键 | `PRIMARY KEY (id)` | `PRIMARY KEY (id)` | 主键保留 |
| 唯一索引 | `UNIQUE INDEX idx_name` | `CREATE UNIQUE INDEX` | 唯一索引 |
| 普通索引 | `INDEX idx_name` | `CREATE INDEX` | 普通索引 |
| 全文索引 | `FULLTEXT INDEX` | `CREATE INDEX ... USING GIN` | 全文索引 |
| 空间索引 | `SPATIAL INDEX` | `CREATE INDEX USING GIST` | 空间索引 |

**MPP 分布式数据库支持**：
- **Greenplum/YugabyteDB**：自动添加 `DISTRIBUTED BY` 子句
- **分布键选择**：默认使用主键列
- **UNIQUE INDEX 处理**：在 Greenplum 上跳过（分布键已确保唯一性）

```sql
-- MySQL 主键
PRIMARY KEY (id, user_id)

-- PostgreSQL with MPP
CREATE TABLE "users" ("id" BIGINT, "user_id" BIGINT, ...);
ALTER TABLE public.users SET DISTRIBUTED BY (id, user_id);
```

#### 3.6.2 用户和权限转换

**（1）用户转换**

MySQL 用户 → PostgreSQL 角色（保留密码哈希）：

```sql
-- MySQL
CREATE USER 'read_only'@'%' IDENTIFIED BY 'password';

-- PostgreSQL
CREATE ROLE read_only WITH LOGIN PASSWORD 'md5hash';
```

**（2）表权限转换**

MySQL GRANT 语句 → PostgreSQL 等效语法：

```sql
-- MySQL
GRANT SELECT, INSERT ON table TO 'user'@'%';

-- PostgreSQL
GRANT USAGE, SELECT, INSERT ON TABLE table TO user;
```

**支持的权限类型**：
- `SELECT` → `SELECT`
- `INSERT` → `INSERT`
- `UPDATE` → `UPDATE`
- `DELETE` → `DELETE`
- `ALL PRIVILEGES` → `ALL`

---

## 四、配置指南和最佳实践

### 4.1 配置文件详解

```yaml
# MySQL 连接配置
mysql:
  host: localhost
  port: 3306
  username: root
  password: password
  database: test_db
  test_only: false           # 仅测试连接，不执行转换
  max_open_conns: 100        # 最大连接数
  max_idle_conns: 50         # 最大空闲连接数
  conn_max_lifetime: 3600    # 连接最大生命周期（秒）
  connection_params: charset=utf8mb4&parseTime=false&interpolateParams=true

# PostgreSQL 连接配置
postgresql:
  host: localhost
  port: 5432
  username: postgres
  password: password
  database: test_db
  test_only: false
  max_conns: 50
  pg_connection_params: search_path=public connect_timeout=300 statement_timeout=0

# 转换配置
conversion:
  # 转换选项
  options:
    tableddl: true              # 转换表结构
    data: true                  # 同步数据
    view: true                  # 转换视图
    indexes: true               # 转换索引
    functions: true             # 转换函数
    users: true                 # 转换用户
    table_privileges: true      # 转换表权限
    lowercase_columns: true     # 字段名转小写
    skip_existing_tables: true  # 跳过已存在的表
    use_table_list: false       # 白名单模式
    table_list: [table1]        # 指定要同步的表
    exclude_use_table_list: false  # 黑名单模式
    exclude_table_list: [table1]   # 跳过的表
    validate_data: true         # 数据校验
    truncate_before_sync: false # 同步前不清空表
    
    # 视图排除
    exclude_use_view_list: false
    exclude_view_list: [view1, view2]
    
    # 函数排除
    exclude_use_function_list: false
    exclude_function_list: [func1, func2]

  # MPP 分布式数据库支持
  mpp:
    enabled: false              # 启用 MPP 模式
    database: auto              # greenplum/yugabyte/auto

  # 限制配置
  limits:
    concurrency: 10             # 并发数
    bandwidth_mbps: 100         # 带宽限制 (Mbps)
    max_ddl_per_batch: 10       # 每批 DDL 数量
    max_functions_per_batch: 5  # 每批函数数量
    max_indexes_per_batch: 20   # 每批索引数量
    max_users_per_batch: 10     # 每批用户数量
    max_rows_per_batch: 50000   # 每批行数
    batch_insert_size: 50000    # 批量插入大小

# 运行配置
run:
  show_progress: true           # 显示进度
  error_log_path: ./errors.log  # 错误日志路径
  enable_file_logging: true     # 启用文件日志
  log_file_path: ./conversion.log  # 日志文件路径
  show_console_logs: true       # 控制台显示日志
  show_log_in_console: false    # 控制台显示详细日志
```

### 4.2 核心参数说明

#### 4.2.1 test_only（连接测试）

```yaml
mysql:
  test_only: true
postgresql:
  test_only: true
```

**功能**：仅测试数据库连接，不执行任何转换操作
**响应时间**：< 1 秒
**使用场景**：快速验证连接配置

#### 4.2.2 validate_data（数据校验）

```yaml
conversion:
  options:
    validate_data: true
```

**功能**：同步后验证数据一致性
**校验方式**：比较 MySQL 和 PostgreSQL 表的行数
**处理逻辑**：
- 行数一致：继续执行
- 行数不一致：根据 `truncate_before_sync` 决定

#### 4.2.3 truncate_before_sync（同步策略）

```yaml
conversion:
  options:
    truncate_before_sync: true   # 同步前清空表数据
```

**两种模式**：
- `true`：全量同步，确保数据完全一致
- `false`：保留已有数据，新数据追加

#### 4.2.4 concurrency（并发数）

```yaml
conversion:
  limits:
    concurrency: 10   # 默认 10 并发
```

**推荐配置**：
- 2 核 2GB：`concurrency: 4`
- 4 核 8GB：`concurrency: 10`
- 8 核 16GB：`concurrency: 20`
- 16 核 32GB：`concurrency: 30`

#### 4.2.5 max_rows_per_batch 和 batch_insert_size

```yaml
conversion:
  limits:
    max_rows_per_batch: 50000    # 每批读取 50,000 行
    batch_insert_size: 50000     # 每批插入 50,000 行
```

**推荐配置**：
- 小数据量（< 100 万行）：`10000`
- 中等数据量（100 万 -1000 万行）：`50000`
- 大数据量（> 1000 万行）：`100000`

### 4.3 生产环境最佳实践

#### 4.3.1 生产环境配置

```yaml
conversion:
  options:
    validate_data: true         # 确保数据一致性
    truncate_before_sync: true  # 全量同步
    concurrency: 20             # 根据系统资源调整
    max_rows_per_batch: 5000    # 适中批次，避免内存过高
    batch_insert_size: 5000     # 适中批量，避免数据库压力

  limits:
    bandwidth_mbps: 200         # 带宽限制，避免影响业务
```

#### 4.3.2 保留已有数据同步

```yaml
conversion:
  options:
    validate_data: true
    truncate_before_sync: false  # 保留已有数据
    use_table_list: true         # 只同步指定表
    table_list: [users, orders]  # 指定表列表
```

#### 4.3.3 性能优化配置

```yaml
conversion:
  limits:
    concurrency: 30              # 高并发
    max_rows_per_batch: 100000   # 大批次
    batch_insert_size: 100000    # 大批量插入
    bandwidth_mbps: 500          # 高带宽限制
```

#### 4.3.4 表过滤配置

```yaml
conversion:
  options:
    # 白名单模式：只同步指定表
    use_table_list: true
    table_list: [users, orders, products]
    
    # 或者黑名单模式：跳过指定表
    exclude_use_table_list: true
    exclude_table_list: [temp_table, log_table]
```

#### 4.3.5 视图和函数排除

```yaml
conversion:
  options:
    # 跳过复杂视图
    exclude_use_view_list: true
    exclude_view_list: [complex_view, temp_view]
    
    # 跳过 MySQL 特有函数
    exclude_use_function_list: true
    exclude_function_list: [mysql_specific_func]
```

---

## 五、实战案例

### 5.1 完整迁移流程

#### 5.1.1 环境准备

```bash
# 1. 克隆仓库
git clone https://github.com/xfg0218/MySQL2PG.git
cd MySQL2PG

# 2. 编译
make build

# 3. 创建配置文件
cp config.example.yml config.yml
```

#### 5.1.2 编辑配置文件

```yaml
# config.yml
mysql:
  host: 192.168.1.1
  port: 3306
  username: root
  password: your_password
  database: test_db

postgresql:
  host: 192.168.1.1
  port: 5432
  username: postgres
  password: your_password
  database: test_db

conversion:
  options:
    tableddl: true
    data: true
    view: true
    indexes: true
    functions: true
    users: true
    table_privileges: true
    validate_data: true
    truncate_before_sync: true
  limits:
    concurrency: 10
    max_rows_per_batch: 50000
    batch_insert_size: 50000

run:
  show_progress: true
  enable_file_logging: true
  log_file_path: ./conversion.log
```

#### 5.1.3 运行评估模式

```bash
# 1. 先运行评估模式，识别潜在风险
./mysql2pg assess config.yml

# 2. 查看评估报告
open assessment-*.html
```

**评估报告输出示例**：

```
+-------------------------------------------------------------+
| MySQL2PG 迁移前评估报告                                     |
+-------------------------------------------------------------+
| 总体评分：100/10                                           |
| 风险等级：低                                                |
+-------------------------------------------------------------+
| 表：189  | 视图：14  | 函数：11  | 索引：588              |
| 用户：4  | 权限：12                                        |
+-------------------------------------------------------------+
| 高风险对象：0                                               |
| 中风险对象：0                                               |
| 低风险对象：0                                               |
+-------------------------------------------------------------+
```

#### 5.1.4 运行迁移模式

```bash
# 运行迁移
./mysql2pg config.yml
```

**控制台输出示例**：

```
+-------------------------------------------------------------+
| 数据库版本信息：                                            |
+--------------+----------------------------------------------+
| 数据库类型   | 版本信息                                     |
+--------------+----------------------------------------------+
| MySQL        | 8.0.45                                       |
| PostgreSQL   | PostgreSQL 16.1 on x86_64-pc-linux-gn...     |
+--------------+----------------------------------------------+

按照指定选项执行转换...

1. 开始转换表结构...
进度：0.43% (1/232) : 转换表 case_31_sys_utf8mb3 成功
进度：16.81% (39/232) : 转换表 case_35_enum_charset 成功

2. 同步表数据...
进度：16.81% (40/232) : 同步表 case_04_mb3_suffix 数据成功，共有 0 行数据，数据一致
进度：33.19% (78/232) : 同步表 case_23_weird_syntax 数据成功，共有 0 行数据，数据一致

3. 转换表视图...
进度：34.05% (79/232) : 转换表视图 view_case01_integers 成功
进度：37.93% (88/232) : 转换表视图 view_case10_defaults 成功

4. 转换表索引...
进度：38.36% (89/232) : [case_13_enum_set]转换索引 idx_case13_e1 成功
进度：95.69% (222/232) : [case_12_unsigned]转换索引 idx_case12_c3 成功

5. 开始转换函数...
进度：96.12% (223/232) : 转换库函数 get_combined_data 成功
进度：96.98% (225/232) : 转换库函数 get_joined_data 成功

6. 开始转换用户...
进度：97.41% (226/232) : 转换用户 mysql2pg@% 的权限成功
进度：98.28% (228/232) : 转换用户 test2@% 的权限成功

7. 转换表权限...
进度：99.14% (230/232) : 转换用户 test1 表权限成功
进度：100.00% (232/232) : 转换用户 test2 表权限成功

+-------------------------------------------------------------+
| 各阶段及耗时汇总如下：                                      |
+--------------------------+----------------+-----------------+
| 阶段                     | 对象数量       | 耗时 (秒)        |
+--------------------------+----------------+-----------------+
| 转换表结构               | 39             | 3.08            |
| 同步表数据               | 39             | 1.15            |
| 转换表视图               | 10             | 1.20            |
| 转换表索引               | 132            | 2.15            |
| 转换库函数               | 3              | 0.25            |
| 转换库用户               | 3              | 0.18            |
| 转换表权限               | 6              | 1.62            |
+--------------------------+----------------+-----------------+
| 总耗时                   |                | 9.63            |
+--------------------------+----------------+-----------------+

转换完成！
```

#### 5.1.5 生成迁移报告

```bash
# 生成 HTML 报告
./mysql2pg report -l conversion.log

# 查看报告
open report-*.html
```

### 5.2 性能测试案例

在 2 核 2GB 环境上，`concurrency=4`，`batch_insert_size=10000` 的情况下：

**测试表**：`case_01_integers`（1200 万行数据）

```sql
DROP TABLE IF EXISTS case_01_integers;
CREATE TABLE case_01_integers (
  col_tiny tinyint,
  col_small smallint,
  col_medium mediumint,
  col_int int,
  col_integer integer,
  col_big bigint,
  col_int_prec int(11),
  col_big_prec bigint(20)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
CREATE INDEX idx_case_01_col_tiny ON case_01_integers(col_tiny);
```

**同步速度**：

```
进度：0.00% (1/1) : 同步表 case_01_integers 完成，12000000 行数据，跳过验证

+--------------------------+----------------+-----------------+
| 阶段                     | 对象数量       | 耗时 (秒)        |
+--------------------------+----------------+-----------------+
| 同步表数据               | 1              | 7093.55         |
+--------------------------+----------------+-----------------+
| 总耗时                   |                | 7093.55         |
+--------------------------+----------------+-----------------+

real    118m13.675s
user    6m7.256s
sys     0m6.487s
```

**平均速度**：约 1691 行/秒

**优化建议**：
- 增加并发数：`concurrency: 10` → 速度提升 2-3 倍
- 增加批次大小：`batch_insert_size: 50000` → 速度提升 3-5 倍
- 使用 SSD 磁盘：I/O 性能提升 → 速度提升 2-3 倍

### 5.3 常见问题处理

#### 5.3.1 数据校验失败

**问题**：数据校验失败，行数不一致

**解决方法**：
1. 检查 `truncate_before_sync` 设置
2. 如果 `true`：检查 PostgreSQL 表是否有其他进程写入
3. 如果 `false`：工具会继续执行，查看 HTML 报告中的不一致表统计

#### 5.3.2 主键冲突

**错误信息**：
```
错误：插入表 users_20251201 数据失败：批量插入失败：
ERROR: duplicate key value violates unique constraint "users_20251201_pkey"
```

**原因**：目标表已有数据，主键冲突

**解决方法**：
1. 设置 `truncate_before_sync: true` 清空目标表
2. 或者设置 `skip_existing_tables: true` 跳过已有表

#### 5.3.3 连接超时

**错误信息**：
```
dial tcp [IP]:3306: i/o timeout
```

**解决方法**：
1. 检查数据库连接配置
2. 确保 MySQL 和 PostgreSQL 服务正常运行
3. 检查网络连接和防火墙设置
4. 增加连接超时：`connect_timeout=300`

#### 5.3.4 依赖下载超时

**错误信息**：
```
github.com/spf13/viper@v1.18.2: Get "https://proxy.golang.org/...": i/o timeout
```

**解决方法**：
```bash
# 配置 Go 代理
go env -w GOPROXY=https://goproxy.cn,direct

# 验证配置
go env GOPROXY

# 重新编译
make build
```

---

## 六、总结和资源

### 6.1 工具优势总结

MySQL2PG v3.4.0 相比其他数据库迁移工具，具有以下优势：

1. **评估先行**：迁移前生成详细的兼容性评估报告，识别潜在风险
2. **可视化报告**：深色终端风格的 HTML 报告，迁移进度和结果一目了然
3. **完整兼容**：支持 MySQL 5.7/8.0/9.0 → PostgreSQL 12-18 全版本
4. **高性能**：并发转换引擎，5-8 倍吞吐量提升
5. **数据校验**：自动验证数据一致性，确保迁移完整性
6. **MPP 支持**：Greenplum、YugabyteDB 等分布式数据库
7. **开源免费**：Apache-2.0 许可证，社区驱动

### 6.2 技术统计

| 指标 | 数值 |
|------|------|
| 支持的 MySQL 版本 | 5.7/8.0/8.4/9.0 |
| 支持的 PostgreSQL 版本 | 12/13/14/15/16/17/18 |
| 字段类型映射 | 40+ 种，准确率 99.9% |
| 视图转换 | 42 个，100% 可转换 |
| 函数转换 | 113 个，100% 可转换 |
| 测试用例 | 41+ 个，全部通过 |
| 代码覆盖率 | 88%+ |
| 并发性能 | 5-10 倍提速 |
| 数据同步速度 | 1691 行/秒（2 核 2GB） |

### 6.3 资源链接

- **GitHub 仓库**：https://github.com/xfg0218/MySQL2PG
- **完整文档**：https://github.com/xfg0218/MySQL2PG/blob/main/docs/MYSQL2PG.md
- **示例配置**：https://github.com/xfg0218/MySQL2PG/blob/main/config.example.yml
- **Issue 追踪**：https://github.com/xfg0218/MySQL2PG/issues
- **版本发布**：https://github.com/xfg0218/MySQL2PG/releases

### 6.4 社区支持

- **问题反馈**：通过 GitHub Issues 提交问题
- **功能建议**：欢迎提交 Feature Request
- **代码贡献**：欢迎提交 Pull Request
- **技术交流**：添加作者微信或发送邮件交流

---

**作者简介**：MySQL2PG 开源项目维护者，专注于数据库迁移和同步技术研发。

**版权声明**：本文采用 Apache-2.0 许可证，转载请注明出处。

---

**如果觉得本文有帮助，欢迎转发分享！** 🎉
