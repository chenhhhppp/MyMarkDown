# 🌸 PostgreSQL 学习笔记 —— 号称世界最强开源数据库的六边形战士

> 嘿嘿～学完 MySQL 这个国民管家，这节课来见见它的"高冷学霸"对手——PostgreSQL（爱称 PG）！功能多到溢出、标准 SQL 模范生，近几年的新项目都爱用它 🐘✨

---

## 📖 目录

1. [初识 PostgreSQL：它到底是啥？](#一初识-postgresql它到底是啥)
2. [PG vs MySQL：学霸和国民管家的对决](#二pg-vs-mysql学霸和国民管家的对决)
3. [安装与连接](#三安装与连接)
4. [核心概念：多一层 Schema！](#四核心概念多一层-schema)
5. [数据类型：PG 的十八般武器](#五数据类型pg-的十八般武器)
6. [表操作与约束（DDL）](#六表操作与约束ddl)
7. [数据增删改（DML）：RETURNING 与 UPSERT 真香](#七数据增删改dmlreturning-与-upsert-真香)
8. [查询进阶（DQL）：这些 MySQL 都做不到！](#八查询进阶dql这些-mysql-都做不到)
9. [JSONB：数据库里的文档库](#九jsonb数据库里的文档库)
10. [窗口函数与 CTE：SQL 天花板写法](#十窗口函数与-ctesql-天花板写法)
11. [索引全家桶：B-tree 只是冰山一角](#十一索引全家桶b-tree-只是冰山一角)
12. [事务与 MVCC：PG 的看家本领](#十二事务与-mvccpg-的看家本领)
13. [高可用与分区](#十三高可用与分区)
14. [总结与面试高频题](#十四总结与面试高频题)

---

## 一、初识 PostgreSQL：它到底是啥？

### 1.1 用大白话说 PG

如果说 MySQL 是"够用就好的国民工具车"🚗，那 PostgreSQL 就是"配置拉满的六边形战士"🏎️：

- **关系型数据库**（和 MySQL 同门派），但它**功能更全、更贴 SQL 标准**
- **开源免费**，协议超宽松（想拿去改造成商用产品都行）
- 圈内绰号："**世界上最先进的开源关系数据库**"（官方自己说的，但大家基本服气 😆）

### 1.2 PG 的性格特点

- 🎓 **SQL 标准模范生**：SQL 特性支持最全，窗口函数、CTE、递归查询样样精通
- 🧰 **类型系统超丰富**：数组、JSONB、UUID、枚举、地理信息、自定义类型……
- 🧬 **MVCC 干净利落**：读写互不阻塞（后面细讲，这是看家本领）
- 🧩 **可扩展性拉满**：能自己写函数、自定义索引、装插件（PostGIS 直接变身 GIS 专家）
- 🍃 近几年 AI/地理/新创公司项目的心头好

### 1.3 什么时候用它？

| 需求 | 用 PG 吗？ |
| --- | --- |
| 复杂查询、数据分析、报表 | ✅ 强项！ |
| 要存 JSON 又想查询 | ✅ JSONB 完美 |
| 地理位置计算（GIS） | ✅ PostGIS 无敌 |
| 全文搜索（不想引入 ES） | ✅ 内置就够用 |
| 极简单的读多写少小站 | MySQL/SQLite 也行，PG 依然能打 |

---

## 二、PG vs MySQL：学霸和国民管家的对决

学过 MySQL 的你，最快的路径就是对比着学！⭐

| 对比项 | MySQL 🚗 | PostgreSQL 🏎️ |
| --- | --- | --- |
| 定位 | 简单可靠、Web 时代国民库 | 功能全面、标准至上 |
| SQL 标准符合度 | 一般 | 非常高 |
| 数据类型 | 基础款 | 超丰富（数组/JSONB/枚举/UUID/网络地址…）|
| 复杂查询（窗口/CTE/递归） | 8.0 后支持（稍弱）| 原生强大 |
| JSON 支持 | JSON 类型（函数较简）| **JSONB 二进制存储 + GIN 索引，可查可索引** ✨ |
| 并发控制 | InnoDB MVCC（回滚段）| **多版本元组，读写互不阻塞** ✨ |
| 全文搜索 | 一般 | 内置（tsvector）|
| 复制 | 主从 + 组复制 | 流复制 + 逻辑复制 |
| 生态 | 超大（老项目巨多）| 快速增长（新项目偏爱）|
| 适合场景 | 简单 Web 业务、读密集 | 复杂业务、分析、GIS、JSON 混合 |

> 🎀 怎么选：老项目、团队熟、简单读写 → MySQL 稳；新项目、查询复杂、类型多样、想要 JSON + 关系混合 → PG 真香！

**语法差异速记（学过 MySQL 的重点看这里）：**

| 事项 | MySQL | PostgreSQL |
| --- | --- | --- |
| 字符串引号 | 单双引号都行 | **只能单引号**，双引号是标识符（表名/列名）|
| 自增主键 | `AUTO_INCREMENT` | `SERIAL` / `GENERATED ALWAYS AS IDENTITY` |
| 查看表结构 | `DESC 表名` | `\d 表名`（psql 命令）|
| 大小写敏感 | 不敏感 | 标识符不写引号会**转小写**；字符串比较大小写敏感 |
| 模糊匹配 | `LIKE` | `LIKE` + **`ILIKE`（忽略大小写）** ✨ |
| 当前库切换 | `USE db` | `\c db`（psql）|
| 注释 | `--` / `#` | `--` / `/* */` |

---

## 三、安装与连接

### 3.1 安装

```bash
# Docker 一行流
docker run -d --name pg -p 5432:5432 -e POSTGRES_PASSWORD=123456 postgres:16
```

| 组件 | 端口 | 作用 |
| --- | --- | --- |
| postgres | 5432 | 服务本体 |
| psql | - | 官方命令行（\ 开头的元命令超好用）|
| pgAdmin / DBeaver | - | 图形化界面 ✅ |

### 3.2 连接与常用元命令

```bash
# 命令行连接
psql -U postgres -h localhost -p 5432
```

```sql
-- psql 里的元命令（反斜杠开头，仅命令行有效）
\l          -- 列出所有数据库（like SHOW DATABASES）
\c shop     -- 切换数据库（like USE shop）
\dt         -- 列出当前库的表
\d users    -- 查看 users 表结构（like DESC users）
\di         -- 查看索引
\q          -- 退出
\timing on  -- 显示查询耗时
```

> 💡 应用代码连接串长这样：`postgresql://postgres:123456@localhost:5432/shop`

---

## 四、核心概念：多一层 Schema！

PG 的层级比 MySQL **多一层 Schema**：

```mermaid
flowchart TD
    S[(PostgreSQL 服务器<br>实例:5432)] --> D1[Database: shop]
    S --> D2[Database: blog]
    D1 --> SC1[schema: public<br>默认的，不写就是它]
    D1 --> SC2[schema: analytics]
    SC1 --> T1[table: users]
    SC1 --> T2[table: orders]
    SC2 --> T3[table: reports]
```

| PG 概念 | MySQL 类比 | 说明 |
| --- | --- | --- |
| Database | Database | 一个实例里多个库，**跨库查不了**（要用 dblink/FDW）|
| Schema | （无，MySQL 没这层）| 库内的"命名空间"，默认 public |
| Table | Table | 表 |
| Row / Column | 行 / 列 | 同 MySQL |

> 🎀 记法：PG 的层级是 **库 → 模式 → 表**；平时不写 schema 就默认 `public`，所以新手可以假装它不存在，团队大了再用来分模块～

---

## 五、数据类型：PG 的十八般武器

### 5.1 基础款（和 MySQL 大同小异）

| 类别 | 类型 | 备注 |
| --- | --- | --- |
| 数值 | smallint / integer / bigint | 对应 MySQL 的 TINY 档到 BIG |
| 精确小数 | numeric / decimal | 钱就用它 💰 |
| 浮点 | real / double precision | 有误差 |
| 自增 | serial / identity | PG 的 AUTO_INCREMENT |
| 文本 | varchar(n) / char(n) / **text** | text 无限长，日常无脑 text |
| 布尔 | **boolean** | 真 true/false（MySQL 是 tinyint 模拟）|
| 时间 | timestamp / timestamptz / date | timestamptz 带时区，推荐 |
| 二进制 | bytea | 对应 MySQL 的 BLOB |

### 5.2 特色款（MySQL 流口水的部分）✨

| 类型 | 说明 | 示例 |
| --- | --- | --- |
| **数组** | 字段直接存数组！ | `text[]` → `{a,b,c}` |
| **JSONB** | 二进制 JSON，可索引可查询 | `{ "tags": ["vip"] }` |
| **uuid** | 原生 UUID 类型 | 自动生成全局唯一 id |
| **枚举 enum** | 自定义枚举类型 | `CREATE TYPE mood AS ENUM (...)` |
| **json** | 文本 JSON（快存慢查）| 和 JSONB 对比见下章 |
| 网络地址 | inet / cidr / macaddr | 存 IP 很方便 |
| 几何/地理 | point / polygon / PostGIS | 地理计算 |
| 区间 | int4range / daterange | 存范围（如时间段）|

```sql
-- 数组类型：一张表搞定"多标签"，不用再拆关联表！
CREATE TABLE articles (
    id      serial PRIMARY KEY,
    title   text,
    tags    text[]                    -- 数组字段！
);

INSERT INTO articles (title, tags) VALUES ('PG 入门', ARRAY['数据库', '教程']);
SELECT * FROM articles WHERE '数据库' = ANY(tags);   -- 包含某标签
SELECT * FROM articles WHERE tags @> ARRAY['教程'];  -- 包含全部这些标签
```

> 💡 数组查询符号：`@>` 包含、`<@` 被包含、`&&` 有交集、`ANY()` 任一等于～

---

## 六、表操作与约束（DDL）

### 6.1 建库建表

```sql
-- 建库（PG 用模板机制，字符集默认就是 UTF8，很省心）
CREATE DATABASE shop;

-- 建表：注意自增是 serial 或 identity
CREATE TABLE users (
    id          serial PRIMARY KEY,           -- 方式1：serial 自增
    username    varchar(50) NOT NULL UNIQUE,
    email       varchar(100) NOT NULL UNIQUE,
    age         integer CHECK (age >= 0 AND age <= 150),  -- PG 的 CHECK 真生效！
    status      text DEFAULT 'active',
    created_at  timestamptz DEFAULT now(),
    extra       jsonb DEFAULT '{}'::jsonb
);

-- 方式2：标准写法（更现代，推荐）
CREATE TABLE orders (
    id      bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id integer NOT NULL REFERENCES users(id),   -- 外键
    amount  numeric(10,2) NOT NULL
);
```

### 6.2 约束清单

| 约束 | 写法 | 备注 |
| --- | --- | --- |
| 主键 | PRIMARY KEY | |
| 自增 | serial / IDENTITY | 两套写法 |
| 唯一 | UNIQUE | |
| 非空 | NOT NULL | |
| 默认 | DEFAULT | |
| 检查 | CHECK | **严格生效**（MySQL 8.0.16 前是摆设）|
| 外键 | REFERENCES 表(列) | |
| 排他 | EXCLUDE | PG 独有，如"时间区间不许重叠" |

### 6.3 改表

```sql
ALTER TABLE users ADD COLUMN phone varchar(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users RENAME COLUMN age TO user_age;
ALTER TABLE users ALTER COLUMN username TYPE varchar(100);   -- 改类型
ALTER TABLE users ADD CONSTRAINT fk FOREIGN KEY (id) REFERENCES t(id);
```

> 💡 PG 加列几乎瞬间完成（不锁表重写），加默认值也快——线上加字段比某些库淡定多了～

---

## 七、数据增删改（DML）：RETURNING 与 UPSERT 真香

### 7.1 增

```sql
INSERT INTO users (username, email, age) VALUES ('xiaohua', 'xh@test.com', 18);

-- 批量
INSERT INTO users (username, email) VALUES
('xiaowang', 'xw@test.com'),
('xiaogang', 'xg@test.com');
```

### 7.2 ⭐ RETURNING：操作完直接把数据还给你

MySQL 插入后想拿自增 id 得再查一次；PG 一条搞定！

```sql
-- 插入并直接返回生成的 id
INSERT INTO users (username, email) VALUES ('xiaohong', 'xhr@test.com')
RETURNING id, username;

-- 更新并返回更新后的行
UPDATE users SET age = 19 WHERE id = 1 RETURNING *;

-- 删除前先把数据带出来留档
DELETE FROM users WHERE id = 2 RETURNING *;
```

### 7.3 ⭐ UPSERT：存在就更新，不存在就插入

```sql
-- ON CONFLICT：冲突字段/约束出现时的处理
INSERT INTO users (username, email, age)
VALUES ('xiaohua', 'xh@test.com', 20)
ON CONFLICT (username)              -- 用户名冲突了
DO UPDATE SET age = EXCLUDED.age;   -- 那就改成新值（EXCLUDED 指本次想插入的行）

-- 冲突就啥也不干
INSERT INTO users (username, email) VALUES ('xiaohua', 'xh@test.com')
ON CONFLICT (username) DO NOTHING;
```

| 对比 | MySQL | PostgreSQL |
| --- | --- | --- |
| 插入拿 id | `LAST_INSERT_ID()` 二次查询 | `RETURNING id` 一步到位 ✨ |
| 存在即更新 | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT DO UPDATE`（还能指定冲突键）✨ |
| 跳过冲突 | `INSERT IGNORE` | `ON CONFLICT DO NOTHING` |

### 7.4 删改注意

```sql
UPDATE users SET age = age + 1 WHERE id = 1;
DELETE FROM users WHERE id = 3;   -- 老规矩：不带 WHERE 全表遭殃 🚨
```

---

## 八、查询进阶（DQL）：这些 MySQL 都做不到！

### 8.1 ILIKE：忽略大小写的模糊查询

```sql
SELECT * FROM users WHERE username LIKE 'xiao%';   -- 区分大小写
SELECT * FROM users WHERE username ILIKE 'XIAO%';  -- 忽略大小写 ✨
```

### 8.2 DISTINCT ON：每组只留一条（去重取代表）

```sql
-- 每个用户最新的一条订单（MySQL 要写子查询窗口绕圈圈）
SELECT DISTINCT ON (user_id) user_id, id, amount, created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

### 8.3 FILTER：聚合函数的条件过滤

```sql
SELECT
    count(*)                                   AS 总数,
    count(*) FILTER (WHERE age >= 18)          AS 成年数,
    avg(age)  FILTER (WHERE status = 'active') AS 活跃用户平均年龄
FROM users;
-- MySQL 得用 SUM(CASE WHEN ... THEN 1 ELSE 0 END) 绕着写 😩
```

### 8.4 INTERSECT / EXCEPT：交差集

```sql
SELECT id FROM a INTERSECT SELECT id FROM b;   -- 交集（两表都有）
SELECT id FROM a EXCEPT    SELECT id FROM b;   -- 差集（a 有 b 没有）
```

### 8.5 常规查询照样全

```sql
SELECT username, age FROM users
WHERE age BETWEEN 18 AND 30 AND status = 'active'
ORDER BY age DESC
LIMIT 10 OFFSET 20;    -- 分页（PG 还支持 FETCH FIRST 10 ROWS ONLY 标准写法）
```

---

## 九、JSONB：数据库里的文档库

PG 的招牌特性：**关系型 + 文档型合体**！不用 MongoDB 也能爽存 JSON～

### 9.1 json vs jsonb

| 对比 | json | jsonb |
| --- | --- | --- |
| 存储 | 原样文本 | 解析后的二进制 |
| 写入速度 | 快 | 稍慢（要解析）|
| 查询性能 | 慢（每次现解析）| **快** ⭐ |
| 支持索引 | ❌ | ✅ GIN 索引 |
| 保留格式/重复键 | 保留 | 不保留（规范化）|

> 🎀 结论：**99% 场景直接用 JSONB**！

### 9.2 JSONB 玩法

```sql
-- 插入 JSONB 数据
INSERT INTO users (username, extra) VALUES
('xiaohua', '{"vip": true, "tags": ["靓女", "成都"], "level": 3}');

-- 查询字段：-> 返回json，->> 返回文本
SELECT extra->'level'    FROM users;   -- 3（json 数字）
SELECT extra->>'level'   FROM users;   -- '3'（文本）
SELECT extra->'tags'->>0 FROM users;   -- 第一个标签 '靓女'

-- 当条件查
SELECT * FROM users WHERE extra->>'vip' = 'true';
SELECT * FROM users WHERE extra @> '{"level": 3}';   -- 包含此结构（走GIN索引！）

-- 修改 JSONB
UPDATE users SET extra = extra || '{"level": 4}'::jsonb WHERE id = 1;  -- 合并更新

-- GIN 索引：让 JSON 查询飞起来
CREATE INDEX idx_extra ON users USING gin(extra);
```

### 9.3 什么时候 JSONB，什么时候拆表？

| 情况 | 建议 |
| --- | --- |
| 字段多变、查询少、就是存着展示 | ✅ JSONB |
| 要按内部字段高频查/统计/建索引 | 拆成正式列更好 |
| 混合：常用列 + 扩展属性 | 正常列 + 一个 extra JSONB（真香组合）⭐ |

---

## 十、窗口函数与 CTE：SQL 天花板写法

### 10.1 窗口函数：分组但不合并行

聚合函数把多行压成一行；**窗口函数每行都保留，还能看到"同组的统计值"** 🪟

```sql
-- 每个订单 + 所在用户的订单总额（行不减少！）
SELECT id, user_id, amount,
       SUM(amount) OVER (PARTITION BY user_id)          AS 用户总额,
       AVG(amount) OVER (PARTITION BY user_id)          AS 用户均值,
       ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS 组内排名
FROM orders;
```

| 函数 | 作用 |
| --- | --- |
| ROW_NUMBER() | 组内行号 1,2,3 |
| RANK() | 排名（并列同名次，跳号）|
| DENSE_RANK() | 排名（并列不跳号）|
| LAG(col) / LEAD(col) | 取上一行 / 下一行的值（算环比超好用！）|
| FIRST_VALUE / LAST_VALUE | 组内第一个 / 最后一个 |

```sql
-- 经典：每组 TopN（每用户金额最大的 2 笔订单）
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rn
    FROM orders
) t WHERE rn <= 2;
```

### 10.2 CTE：WITH 子句，SQL 也能"定义变量"

```sql
-- 先定义"临时结果集"，主查询引用它，可读性爆棚
WITH vip_users AS (
    SELECT id, username FROM users WHERE extra->>'vip' = 'true'
),
vip_orders AS (
    SELECT o.* FROM orders o JOIN vip_users v ON o.user_id = v.id
)
SELECT username, SUM(amount) AS total
FROM vip_orders JOIN vip_users USING (id)
GROUP BY username;
```

### 10.3 递归 CTE：SQL 玩转树结构

```sql
-- 查组织架构树：从大老板往下找所有下属（员工表有 manager_id）
WITH RECURSIVE sub AS (
    SELECT id, name, manager_id FROM emp WHERE id = 1      -- 起点：大老板
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM emp e JOIN sub s ON e.manager_id = s.id            -- 一层层往下挖
)
SELECT * FROM sub;
```

```mermaid
flowchart LR
    A[WITH 定义临时集] --> B[主查询引用] --> C[结果]
    D[WITH RECURSIVE<br>自己查自己] --> E[树/图遍历<br>组织架构/菜单树 🌳]
```

> 🎀 窗口函数 + CTE + 递归，是分析类 SQL 的三板斧——PG 里全都原生丝滑支持！

---

## 十一、索引全家桶：B-tree 只是冰山一角

```sql
-- 常规 B-tree（默认）
CREATE INDEX idx_username ON users(username);

-- 唯一索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- 部分索引：只索引满足条件的行（省空间！）
CREATE INDEX idx_active ON users(status) WHERE status = 'active';

-- 表达式索引：对表达式结果建索引
CREATE INDEX idx_lower_email ON users(lower(email));

-- GIN 索引：JSONB / 数组 / 全文搜索专用
CREATE INDEX idx_extra ON users USING gin(extra);

-- BRIN 索引：超大数据按块存范围（日志表神器）
CREATE INDEX idx_created ON logs USING brin(created_at);
```

| 索引类型 | 适用场景 |
| --- | --- |
| B-tree（默认）| 等值、范围、排序，万金油 ⭐ |
| Hash | 纯等值（PG 10+ 持久化）|
| GIN | **JSONB、数组、全文检索** ✨ |
| GiST | 地理空间、范围类型 |
| BRIN | 超大表 + 物理顺序相关列（时间日志）|
| SP-GiST | 前缀/四叉树类数据 |

```sql
-- 查看执行计划（相当于 MySQL 的 EXPLAIN，信息更细）
EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'xiaohua';
```

> 💡 `EXPLAIN ANALYZE` 会真的执行并给出每步实际耗时，调优神器！注意别对 UPDATE/DELETE 乱用 ANALYZE（它真执行）～

---

## 十二、事务与 MVCC：PG 的看家本领

### 12.1 事务基本操作

```sql
BEGIN;
UPDATE account SET money = money - 100 WHERE name = '小明';
UPDATE account SET money = money + 100 WHERE name = '小红';
COMMIT;      -- 出问题就 ROLLBACK;
```

PG 默认隔离级别是 **READ COMMITTED**（和 MySQL 默认的 RR 不同哦）。

### 12.2 MVCC：读写互不阻塞的秘密

PG 的 MVCC 比 MySQL 更"干净"：

- 每行数据带 `xmin`（创建它的事务id）/ `xmax`（删除它的事务id）标记
- 更新 = **新写一行新版本 + 旧行标记 xmax**，不是原地改！
- 每个事务按快照决定"哪些行对我可见"→ **读不挡写，写不挡读** 🕊️

```mermaid
flowchart LR
    A[UPDATE age=18→19] --> B[旧行 age=18<br>xmax=事务100<br>对老快照仍可见]
    A --> C[新行 age=19<br>xmin=事务100<br>新事务可见]
    D[读事务<br>各自按快照看版本] --> B
    E[写事务] -.不阻塞.-> D
```

**代价：死元组（旧版本垃圾）需要清理 → `VACUUM` 登场**

| 概念 | 说明 |
| --- | --- |
| 死元组 dead tuple | 更新/删除留下的旧版本行 |
| VACUUM | 回收死元组空间（自动进程 autovacuum 常驻）|
| 事务 ID 回卷 | 事务号会用完，autovacuum 负责防回卷 |

> 🎀 类比：PG 更新像"盖新楼留旧楼"，老住户（进行中的事务）还能住旧楼，搬完再拆（VACUUM）；MySQL InnoDB 则用回滚段记旧版本。路子不同，目的相同～

### 12.3 隔离级别

| 隔离级别 | PG 实现要点 |
| --- | --- |
| READ COMMITTED（默认）| 每条语句拿新快照，无脏读 |
| REPEATABLE READ | 事务开始定快照，基本无幻读 |
| SERIALIZABLE | 可串行化快照隔离 SSI，最严格 |

---

## 十三、高可用与分区

### 13.1 流复制（主从）

```mermaid
flowchart LR
    P[Primary 主库<br>可写] --|WAL 日志流| S1[Standby 从库1<br>只读]
    P --|WAL 日志流| S2[Standby 从库2<br>只读]
    P --主库挂了--> E[Promote 提升某个从库为新主<br>通常配合 Patroni 自动化]
```

| 机制 | 说明 |
| --- | --- |
| WAL | 预写日志（类似 MySQL binlog+redo 合体思路）|
| 流复制 | 物理复制，整库同步，延迟低 |
| 逻辑复制 | 按表订阅发布，可跨版本、选择性同步 |
| Patroni + etcd | 自动故障转移的高可用套装（生产标配）|

### 13.2 表分区：大表切片管理

```sql
-- 按 range 分区（时间最常用）
CREATE TABLE logs (
    id      bigint,
    msg     text,
    created_at timestamptz
) PARTITION BY RANGE (created_at);

-- 建分区
CREATE TABLE logs_2026_10 PARTITION OF logs
    FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');
CREATE TABLE logs_2026_11 PARTITION OF logs
    FOR VALUES FROM ('2026-11-01') TO ('2026-12-01');

-- 也可以按 list（如按地区）或 hash（均匀打散）
```

| 分区方式 | 适用 |
| --- | --- |
| RANGE | 时间序列（日志、订单）⭐ |
| LIST | 枚举类（地区、类别）|
| HASH | 均匀打散 |

> 💡 查询带分区键时自动"分区裁剪"，只扫相关分区——大表查询和维护都轻松～

---

## 十四、总结与面试高频题

### 14.1 知识地图

```mermaid
mindmap
  root((PostgreSQL))
    基础
      库 Schema 表
      数据类型特色
      serial identity
    增删改查
      RETURNING
      ON CONFLICT UPSERT
      ILIKE FILTER
    高级特性
      JSONB + GIN
      数组类型
      窗口函数 CTE
      递归查询
    内核机制
      MVCC 多版本
      VACUUM
      WAL 流复制
    架构
      主从复制
      Patroni 高可用
      表分区
```

### 14.2 面试官最爱问 Top 题 🎯

| 问题 | 答题要点 |
| --- | --- |
| PG 和 MySQL 怎么选？ | 简单 Web/老项目 MySQL；复杂查询、类型丰富、JSONB、GIS、标准 SQL → PG |
| MVCC 实现区别？ | PG 多版本元组（xmin/xmax）+ VACUUM；MySQL InnoDB 回滚段 undo |
| json 和 jsonb 区别？ | 文本存 vs 二进制存；jsonb 可索引（GIN）查询快，日常用 jsonb |
| RETURNING 是啥？ | DML 语句直接返回受影响行，免二次查询拿自增 id |
| UPSERT 怎么写？ | INSERT ... ON CONFLICT(键) DO UPDATE / DO NOTHING |
| PG 有哪些特色索引？ | GIN（JSONB/全文）、GiST（地理）、BRIN（大表）、部分索引、表达式索引 |
| 窗口函数和 GROUP BY 区别？ | 分组统计但不合并行，每行都能看到组内统计值 |
| PG 默认隔离级别？ | READ COMMITTED（MySQL 默认 RR）|
| 什么是 VACUUM？ | 清理 MVCC 留下的死元组，autovacuum 自动跑 |
| PG 支持分区吗？ | 原生声明式分区：RANGE/LIST/HASH，查询自动裁剪 |

### 14.3 学习建议路线 🗺️

1. **第一站**：Docker 起 PG，psql 元命令 + 基础 CRUD 敲熟（对照 MySQL 差异表）
2. **第二站**：玩特色货——SERIAL/RETURNING/UPSERT/ILIKE
3. **第三站**：JSONB + 数组类型 + GIN 索引，体会"一库两用"
4. **第四站**：窗口函数 + CTE + 递归，SQL 功力质变
5. **终点站**：MVCC/VACUUM/流复制/分区，内核与架构视野拉满 🌟

---

> 🌈 PG 通关！现在你的数据库图鉴已有五位成员：MySQL、Redis、ES、MongoDB、PostgreSQL——从国民管家到六边形学霸全都拿下！下次别人问"PG 凭啥先进"，你就把 JSONB、RETURNING、窗口函数、MVCC 一套甩他脸上 😎 继续冲鸭！(๑•̀ㅂ•́)و✧💪
