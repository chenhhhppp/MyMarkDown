# 🗄️ SQLx 学习笔记

> 一句话介绍：**SQLx = 异步 SQL 工具包天花板**——纯 Rust、无 ORM 绑架（SQL 自己写），杀手锏是 **`query!` 宏在编译期连数据库校验 SQL**！写错列名？编译直接报错，根本等不到上线 💥
>
> 📌 前置知识：[Axum.md](./Axum.md)（State 共享 + CRUD 实战）+ [Serde.md](./Serde.md)（序列化）+ [Rust并发编程.md](./Rust并发编程.md)——本篇是 Rust Web 技术栈的最后一块拼图！本文以 **PostgreSQL** 为例（MySQL 用法几乎一样）

---

## 📖 开篇：学习路线总览

```mermaid
flowchart LR
    A["① 认识<br/>SQLx"] --> B["② 连接池<br/>查询基础"] --> C["③ query!宏⭐<br/>编译期校验"] --> D["④ 事务与<br/>迁移"] --> E["⑤ Axum+DB<br/>完整实战"]
```

| 章节 | 内容 | 难度 |
|------|------|------|
| 一 | SQLx 定位、vs ORM 对比 | ⭐ |
| 二 | 连接池 + fetch 三兄弟 | ⭐⭐ |
| 三 | **query! 宏：编译期校验 SQL（灵魂！）** | ⭐⭐⭐ |
| 四 | 事务 + 数据库迁移 | ⭐⭐⭐ |
| 五 | Axum + SQLx 完整 CRUD | ⭐⭐⭐⭐ |

---

## 一、认识 SQLx 🗄️

### 1.1 三种数据库访问姿势

| 姿势 | 代表 | 特点 |
|------|------|------|
| 原生 SQL（SQLx）⭐ | SQLx | SQL 自己写、异步、**编译期校验** |
| ORM | SeaORM、Diesel | 对象映射、免写 SQL、学习曲线陡 |
| 手拼驱动 | tokio-postgres | 最原始，啥都要自己来 |

> 🎀 SQLx 的哲学："**SQL 就是最好的 API**"——不藏 SQL、不造 DSL，直接写原生 SQL，但用类型系统把安全补满！对会 SQL 的同学是零成本上手 ✨

### 1.2 四大卖点

| 卖点 | 说明 |
|------|------|
| **编译期校验** ⭐ | `query!` 宏编译时真连数据库，SQL 错误当场报 |
| 异步原生 | 基于 tokio，和 Axum 完美契合 |
| 多数据库 | Postgres / MySQL / SQLite 通用 API |
| 内置工具 | sqlx-cli：迁移、建库、查询一条龙 |

---

## 二、连接池与查询基础 🌊

### 2.1 依赖与建池

```toml
[dependencies]
sqlx = { version = "0.7", features = ["runtime-tokio", "tls-rustls", "postgres"] }
```

```rust
use sqlx::postgres::PgPoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // 连接池：数据库操作的"共享单车库" 🚲
    let pool = PgPoolOptions::new()
        .max_connections(5)                    // 最多 5 个连接
        .connect("postgres://user:pass@localhost/mydb").await?;

    println!("池子建好了！当前连接数: {}", pool.size());
    Ok(())
}
```

```mermaid
flowchart LR
    H1["处理器A"] --> P["PgPool 连接池<br/>(max=5)"]
    H2["处理器B"] --> P
    H3["处理器C"] --> P
    P --> C1["连接1"]
    P --> C2["连接2"]
    P --> C3["..."]
    C1 --> DB[("PostgreSQL")]
    C2 --> DB
```

> 🎀 为什么用连接池？——建连接（TCP 握手 + 认证）很贵！池子预先建好反复借还，**每个请求借一下就还**，高并发下性能天差地别～（Axum 篇的 `Arc<Mutex<Vec>>` 内存"数据库"今天光荣退役 🎖️）

### 2.2 fetch 三兄弟：按需求取结果 🥣

```rust
use sqlx::Row;

// ① execute：不取数据（INSERT/UPDATE/DELETE）
let result = sqlx::query("INSERT INTO users (name) VALUES ($1)")
    .bind("小花")                            // $1 占位符绑定参数（防注入！）
    .execute(&pool).await?;
println!("影响 {} 行", result.rows_affected());

// ② fetch_all：取全部行 → Vec
let rows = sqlx::query("SELECT id, name FROM users")
    .fetch_all(&pool).await?;
for row in rows {
    let id: i32 = row.get("id");
    let name: String = row.get("name");
    println!("{}: {}", id, name);
}

// ③ fetch_one：必须恰好一行（0 行或 2 行都 Err）
let row = sqlx::query("SELECT name FROM users WHERE id = $1")
    .bind(1)
    .fetch_one(&pool).await?;

// ④ fetch_optional：可能没有（→ None 而不是报错）
let maybe = sqlx::query("SELECT name FROM users WHERE id = $1")
    .bind(999)
    .fetch_optional(&pool).await?;            // Ok(None) → 用户不存在
```

| 方法 | 返回 | 适合 |
|------|------|------|
| `execute` | 影响行数 | 增删改 |
| `fetch_all` | `Vec<Row>` | 查列表 |
| `fetch_one` | 单行 | 按主键查（坚信存在）|
| `fetch_optional` | `Option<Row>` | 查单个（可能没有）→ 404 |
| `fetch` | Stream 流 | 大结果逐行处理 |

### 2.3 query_as：直接映射到结构体 ⭐

```rust
use sqlx::FromRow;

#[derive(sqlx::FromRow, serde::Serialize)]   // FromRow：行 → 结构体自动转换
struct User {
    id: i32,
    name: String,
}

// 一行代码：SQL → Vec<User>
let users: Vec<User> = sqlx::query_as::<_, User>(
        "SELECT id, name FROM users ORDER BY id"
    )
    .fetch_all(&pool).await?;
```

> 🎀 `query_as` + `FromRow` = 手写 `row.get()` 的解放宣言！字段名对上就自动填装，配合 Serde 直接喂给 Axum 的 `Json<T>`，一条龙拉通 🚀

---

## 三、query! 宏：编译期校验（灵魂！）✨

### 3.1 普通 SQL 错误什么时候发现？

```rust
// 普通方式：运行时才炸 💥
let users = sqlx::query_as::<_, User>("SELECT id, namee FROM users")  // namee 拼错了！
    .fetch_all(&pool).await?;   // → 500 报错，用户已经看到了
```

### 3.2 query! 宏：编译期就报错！

```rust
// 编译时真的连数据库执行 EXPLAIN 校验！
let users = sqlx::query!("SELECT id, name FROM users WHERE age > $1", 18)
    .fetch_all(&pool).await?;
// users[i].id、users[i].name 直接用——类型都是编译期定好的！

// ❌ 写错列名：直接编译失败！
// sqlx::query!("SELECT id, namee FROM users")
// error: column "namee" does not exist

// query_as! 版：直接映射结构体（连 FromRow 都不用派生！）
let users = sqlx::query_as!(User, "SELECT id, name FROM users")
    .fetch_all(&pool).await?;
```

| 方式 | SQL 错误发现时机 | 结果类型 |
|------|------------------|----------|
| `query_as::<_, T>` | **运行时** | 手动 `FromRow` |
| `query!` / `query_as!` | **编译期** ⭐ | 自动推断（宏生成）|

### 3.3 编译期校验的配置

```bash
# 方式1：环境变量指路（开发时最方便）
export DATABASE_URL="postgres://user:pass@localhost/mydb"
cargo build    # 编译时连库校验

# 方式2：离线模式（CI/服务器上没有数据库！）
cargo install sqlx-cli
sqlx prepare          # 生成 .sqlx/ 查询缓存文件（提交到 git！）
cargo build           # 编译时读缓存校验，不需要真数据库 ✅
```

| 场景 | 方案 |
|------|------|
| 本地开发 | `DATABASE_URL` 环境变量 |
| CI / 生产构建 | `sqlx prepare` 离线缓存（**.sqlx 目录要进 git**）|

> ⚠️ 可空列坑：`query!` 对可空列推断出 `Option<T>`，若 SQL 用 `WHERE ... IS NOT NULL` 仍不确定，可用 `query_as!` + 显式类型或 `COALESCE` 兜底～

---

## 四、事务与迁移 🏦

### 4.1 事务：要么全成功，要么全滚回

```rust
// 经典场景：转账（A 扣钱 + B 加钱，必须同生共死）
async fn transfer(pool: &PgPool, from: i32, to: i32, amount: i64) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;              // 开事务

    sqlx::query("UPDATE accounts SET balance = balance - $1 WHERE id = $2")
        .bind(amount).bind(from)
        .execute(&mut *tx).await?;                 // ⚠️ 注意 &mut *tx

    sqlx::query("UPDATE accounts SET balance = balance + $1 WHERE id = $2")
        .bind(amount).bind(to)
        .execute(&mut *tx).await?;

    tx.commit().await?;                            // 提交 ✅（中途出错自动回滚）
    Ok(())
}
```

```mermaid
flowchart TD
    S["pool.begin() 开事务"] --> A["操作1：A 扣钱"]
    A --> B["操作2：B 加钱"]
    B -->|"全部成功"| C["tx.commit() 提交 ✅"]
    B -->|"任何一步失败 💥"| D["tx 回滚<br/>（tx 被 drop 自动 rollback）"]
    D --> E["数据库像什么都没发生过"]
```

> 🎀 Rust 的贴心细节：事务对象 `tx` 被 drop 时**自动回滚**——中途 `?` 冒泡出去也不怕，不会有"半提交"状态！

### 4.2 迁移：数据库版本管理

```bash
cargo install sqlx-cli --no-default-features --features rustls,postgres

sqlx database create                            # 建库
sqlx migrate add create_users_table             # 生成迁移文件
# → migrations/20240101000000_create_users_table.sql
```

```sql
-- migrations/20240101000000_create_users_table.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);
```

```bash
sqlx migrate run      # 应用所有未执行的迁移 ✅
```

| 命令 | 作用 |
|------|------|
| `sqlx database create` | 建数据库 |
| `sqlx migrate add 名字` | 新建迁移文件 |
| `sqlx migrate run` | 执行迁移 |
| `sqlx migrate revert` | 回滚一步 |

---

## 五、实战：Axum + SQLx 完整 CRUD 🛠️

> 还记得 [Axum.md](./Axum.md) 的内存版用户管理吗？今天升级成**真数据库版**——整体结构几乎不变，只把 `Arc<Mutex<Vec>>` 换成 `PgPool`！

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    routing::get,
    Json, Router,
};
use sqlx::PgPool;

#[derive(sqlx::FromRow, serde::Serialize, serde::Deserialize)]
struct User {
    id: i32,
    name: String,
    email: Option<String>,
}

#[derive(serde::Deserialize)]
struct CreateUser {
    name: String,
    email: Option<String>,
}

#[tokio::main]
async fn main() {
    // ① 建连接池（替代内存数据库）
    let pool = PgPool::connect(&std::env::var("DATABASE_URL").unwrap())
        .await
        .expect("数据库连不上");

    // ② 注入 State（和内存版姿势一模一样！）
    let app = Router::new()
        .route("/users", get(list_users).post(create_user))
        .route("/users/:id", get(get_user))
        .with_state(pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 数据库版服务启动！");
    axum::serve(listener, app).await.unwrap();
}

// GET /users —— 列表
async fn list_users(State(pool): State<PgPool>) -> Result<Json<Vec<User>>, StatusCode> {
    let users = sqlx::query_as::<_, User>(
            "SELECT id, name, email FROM users ORDER BY id"
        )
        .fetch_all(&pool).await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    Ok(Json(users))
}

// POST /users —— 创建（RETURNING 把新行带回来！）
async fn create_user(
    State(pool): State<PgPool>,
    Json(payload): Json<CreateUser>,
) -> Result<(StatusCode, Json<User>), StatusCode> {
    let user = sqlx::query_as::<_, User>(
            "INSERT INTO users (name, email) VALUES ($1, $2)
             RETURNING id, name, email"
        )
        .bind(&payload.name)
        .bind(&payload.email)
        .fetch_one(&pool).await
        .map_err(|_| StatusCode::BAD_REQUEST)?;
    Ok((StatusCode::CREATED, Json(user)))
}

// GET /users/:id —— 单个（fetch_optional → 404）
async fn get_user(
    State(pool): State<PgPool>,
    Path(id): Path<i32>,
) -> Result<Json<User>, StatusCode> {
    sqlx::query_as::<_, User>(
            "SELECT id, name, email FROM users WHERE id = $1"
        )
        .bind(id)
        .fetch_optional(&pool).await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?
        .map(Json)
        .ok_or(StatusCode::NOT_FOUND)     // None → 404，和内存版神同步！✨
}
```

**与内存版的对比**（进步一目了然）：

| 维度 | Axum 内存版 | SQLx 版 |
|------|-------------|---------|
| 存储 | `Arc<Mutex<Vec>>`（重启就没）| PostgreSQL（持久化 ✅）|
| 并发 | 单线程锁 | 连接池多路并发 ✅ |
| 查询 | `iter().find()` 全表扫 | 数据库索引 ⚡ |
| ID 自增 | `len() + 1`（删完就重复 💣）| `SERIAL` 数据库保证 ✅ |
| handler 签名 | `State<Db>` | `State<PgPool>`（无缝平移）|

---

## 六、常见坑与排查 🚑

| 坑 | 现象 | 解法 |
|----|------|------|
| 编译时要连数据库 | `error: configuration: DATABASE_URL not set` | 设环境变量 或 `sqlx prepare` 离线模式 |
| execute 传池报错 | 类型不匹配提示 | 事务里传 `&mut *tx`，普通传 `&pool` |
| 可空列类型错 | `ColumnIsNullable` 编译错误 | 用 `Option<T>` 接可空列 |
| fetch_one 空 | `RowNotFound` 运行时错误 | 可能不存在的用 `fetch_optional` |
| 时间类型不匹配 | `chrono` 类型报错 | 开 feature `features = ["chrono"]` |
| SSL 报错 | `no authentication` 之类 | 用 `tls-rustls` feature + 连接串加 `?sslmode=disable`（本地）|
| 参数序号混 | MySQL 用 `?`，PG 用 `$1` | 占位符风格不同，别搞混～ |
| 迁移没生效 | 表不存在 | `sqlx migrate run`；检查文件名时间戳前缀 |

---

## 七、高频问题速查 🎯

| 问题 | 要点 |
|------|------|
| SQLx 是 ORM 吗？ | ❌ 不是！写原生 SQL 的异步工具包（ORM 用 SeaORM/Diesel）|
| 杀手锏是什么？ | `query!` 宏**编译期**连库校验 SQL，写错当场编译失败 ⭐ |
| query 和 query_as 区别？ | query 返回 Row 手动取列；query_as 直接映射结构体 |
| fetch_one vs fetch_optional？ | 必有→fetch_one（空则 Err）；可能没有→fetch_optional（None）|
| 连接池为什么快？ | 预建连接反复借还，免 TCP+认证开销 |
| $1 是什么？ | 参数占位符（PG 风格），bind() 绑定值，**天然防 SQL 注入** ✅ |
| RETURNING 干嘛的？ | INSERT/UPDATE 后把受影响的行返回（省一次查询）⭐ |
| 事务怎么保证原子性？ | begin→操作→commit；失败自动回滚（tx drop 时）|
| CI 没数据库怎么编译？ | `sqlx prepare` 生成 .sqlx 缓存提交 git，离线校验 |
| 迁移怎么管理？ | sqlx migrate add/run，文件带时间戳按序执行 |
| 和 Axum 怎么集成？ | `with_state(pool)` + `State<PgPool>` 提取（和内存版无缝切换）|
| 支持 MySQL 吗？ | ✅ 换 feature 和 Pool 类型，SQL 语法微调 |

---

## 八、结语 🌸

SQLx 的知识树收好啦：

```mermaid
flowchart TB
    S["🗄️ SQLx"] --> A["连接池：PgPool<br/>max_connections 借还机制"]
    S --> B["查询：execute/fetch_all<br/>fetch_one/fetch_optional"]
    S --> C["映射：query_as+FromRow<br/>query!宏编译期校验⭐"]
    S --> D["事务：begin→操作→commit<br/>drop 自动回滚"]
    S --> E["迁移：sqlx migrate<br/>版本化建表"]
    S --> F["实战：Axum+PgPool<br/>内存版无缝升级"]
```

> 🎀 学习建议：
> 1. **跑通第五章完整实战**——本地起个 PostgreSQL（Docker 一行命令），把 Axum 内存版升级成数据库版，你就拥有了一套**生产级 Rust Web 全栈骨架**！
> 2. 试试故意把列名拼错：普通 `query_as` 运行时才炸，`query_as!` 宏编译期就拦——**亲眼对比一次，永远记住这个杀手锏** 😆
> 3. 进阶方向：连接池参数调优、`sqlx::test` 写集成测试（自动建临时库）、再往上接 SeaORM 体验 ORM 风格！
>
> Rust Web 全栈拼图完成 🎉：[基础语法](./Rust基础语法.md) → [并发编程](./Rust并发编程.md) → [Axum](./Axum.md) → [Serde](./Serde.md) → **SQLx**——路由、序列化、数据库三大件齐活，可以撸真实项目啦！还想学什么尽管吩咐～(๑•̀ㅂ•́)و✧
