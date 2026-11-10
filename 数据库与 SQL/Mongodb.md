# 🌸 MongoDB 学习笔记 —— 最懂 JSON 的文档数据库

> 嘿嘿～MySQL 管账、Redis 提速、ES 搜索，这次轮到 MongoDB 登场！它把数据当成一份份 JSON 文档来存，字段想加就加、灵活到飞起，是 NoSQL 文档家族的当家花旦 🍃✨

---

## 📖 目录

1. [初识 MongoDB：它到底是啥？](#一初识-mongodb它到底是啥)
2. [核心概念：三分钟看懂文档模型](#二核心概念三分钟看懂文档模型)
3. [安装与连接](#三安装与连接)
4. [数据库与集合操作](#四数据库与集合操作)
5. [文档的增删改（CUD）](#五文档的增删改cud)
6. [文档查询（R）：重头戏！](#六文档查询r重头戏)
7. [聚合管道：MongoDB 的"流水线"](#七聚合管道mongodb-的流水线)
8. [索引：提速老三样](#八索引提速老三样)
9. [副本集：高可用三兄弟](#九副本集高可用三兄弟)
10. [分片集群：装不下的就分家](#十分片集群装不下的就分家)
11. [事务：MongoDB 也有事务！](#十一事务mongodb-也有事务)
12. [四方对决：MongoDB vs 三位前辈](#十二四方对决mongodb-vs-三位前辈)
13. [经典应用场景与避坑](#十三经典应用场景与避坑)
14. [总结与面试高频题](#十四总结与面试高频题)

---

## 一、初识 MongoDB：它到底是啥？

### 1.1 用大白话说 MongoDB

MySQL 存数据前要**先画好表格子**（定好列，插入必须对齐）📐

而 MongoDB 像一个**活页夹 📒**：

- 每一页是一个"文档"（就是一份 JSON）
- 每页想写几个字段、什么格式，**自己说了算**！
- 一批活页夹在一起 = 一个"集合"

```json
// 同一个集合里，这两条数据可以共存，MySQL 直呼离谱！
{ "name": "小花", "age": 18 }
{ "name": "小王", "age": 20, "hobby": ["爬山", "摄影"], "address": { "city": "成都" } }
```

### 1.2 MongoDB 的特点

- 📄 **文档模型**：数据长 JSON 样（内部是 BSON 二进制），程序里的对象直接存，不用拼 SQL
- 🌿 **Schema 灵活**：字段不固定，需求天天改也不怕
- 📈 **天生分布式**：分片、副本集开箱即用
- 💪 **查询够强**：条件、排序、聚合管道全都有，不像某些 NoSQL 只能按 key 取

### 1.3 什么时候用它？

| 需求 | 用 MongoDB 吗？ |
| --- | --- |
| 字段结构多变（商品不同类目属性不同）| ✅ 主场！ |
| 存日志、埋点、评论、Feed 流 | ✅ 很合适 |
| 嵌套数据（订单里套商品列表）| ✅ 主场！ |
| 强事务记账（转账、扣库存）| ⚠️ 能做但不是强项，首选 MySQL |
| 复杂多表 JOIN 报表 | ⚠️ 弱项，关系密集请回 MySQL |

---

## 二、核心概念：三分钟看懂文档模型

老规矩，对照 MySQL 一张表通关：

| MongoDB 概念 | MySQL 类比 | 说明 |
| --- | --- | --- |
| Database 数据库 | Database | 一栋楼 🏢 |
| Collection 集合 | Table 表 | 楼里的一间房 |
| Document 文档 | Row 行 | 一条数据（BSON/JSON）📄 |
| Field 字段 | Column 列 | 文档里的属性 |
| `_id` 字段 | 主键 | 自动生成，12 字节的 ObjectId |
| 嵌套文档/数组 | JOIN 才能实现 | 原生支持，一页纸写完！ |

```mermaid
flowchart TD
    DB[Database: shop 🏢] --> C1[Collection: users]
    DB --> C2[Collection: orders]
    C1 --> D1["📄 { name:'小花', age:18 }"]
    C1 --> D2["📄 { name:'小王', hobby:[...] }"]
    C2 --> D3["📄 { items:[ {goods:'奶茶', num:2} ] }"]
```

**ObjectId 长啥样？**

```
66 f a3 b2 c  0 0 0 1  2 3 a b  c d e f 12
└─时间戳─┘ └─机器码─┘ └进程号┘ └─随机数┘
```

> 🎀 前 4 字节是时间戳，所以 `_id` 天生**大致按时间递增**——当排序依据也挺香～

---

## 三、安装与连接

### 3.1 安装

```bash
# Docker 一行流
docker run -d --name mongo -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=123456 mongo:7
```

| 组件 | 端口 | 作用 |
| --- | --- | --- |
| mongod | 27017 | 服务本体 |
| mongosh | - | 官方命令行（shell）|
| Compass | - | 官方图形化界面（真香 ✅）|

### 3.2 连接

```bash
# 命令行连接
mongosh "mongodb://root:123456@localhost:27017"

# 应用里常见两种连接串
mongodb://host:27017/mydb           # 单机
mongodb://host1:27017,host2:27017/?replicaSet=rs0   # 副本集
```

```javascript
// mongosh 里的基本操作
show dbs          // 看所有库
use shop          // 切换/创建库（插入数据后才真正存在）
show collections  // 看集合
```

> 💡 MongoDB 很贴心：库和集合都是**插入数据时自动创建**，不用先 `CREATE`！

---

## 四、数据库与集合操作

```javascript
use shop                                   // 进入/创建库
db.createCollection("users")               // 显式建集合（也可以不建，插数据自动建）
db.users.drop()                            // 删除集合
db.dropDatabase()                          // 删除当前库（跑路警告 🚨）
db.users.renameCollection("members")       // 集合改名
```

| 操作 | 语法 | 备注 |
| --- | --- | --- |
| 建集合 | `db.createCollection("名")` | 可省略，自动建 |
| 看集合 | `show collections` | |
| 删集合 | `db.集合.drop()` | |
| 删库 | `db.dropDatabase()` | 先 use 再删 |

---

## 五、文档的增删改（CUD）

### 5.1 增：insertOne / insertMany

```javascript
// 插一条（不写 _id 会自动生成）
db.users.insertOne({
  name: "小花",
  age: 18,
  hobbies: ["爬山", "拍照"],           // 数组直接存！
  address: { city: "成都", street: "春熙路" }  // 嵌套对象直接存！
})

// 批量插（日常开发必备）
db.users.insertMany([
  { name: "小王", age: 20 },
  { name: "小刚", age: 22, tags: ["vip"] }
])
```

### 5.2 改：updateOne / updateMany / replaceOne

```javascript
// 只改第一条匹配的：把小花的年龄改成 19
db.users.updateOne(
  { name: "小花" },                    // 条件（相当于 WHERE）
  { $set: { age: 19 } }               // 修改内容（$set = 设置字段）
)

// 全量改：所有 18 岁的都标记为成年
db.users.updateMany(
  { age: 18 },
  { $set: { adult: true } }
)

// 原子自增：年龄 +1（用 $inc，不是 $set！）
db.users.updateOne({ name: "小花" }, { $inc: { age: 1 } })

// 整篇替换（_id 不变，其他全换）
db.users.replaceOne({ name: "小花" }, { name: "小花", age: 20 })
```

**常用更新操作符速查：**

| 操作符 | 作用 | 示例 |
| --- | --- | --- |
| `$set` | 设置字段值 | `{ $set: { age: 19 } }` |
| `$unset` | 删除字段 | `{ $unset: { age: "" } }` |
| `$inc` | 自增/自减 | `{ $inc: { age: 1 } }` |
| `$push` | 数组尾部追加 | `{ $push: { hobbies: "游泳" } }` |
| `$pull` | 数组删除某元素 | `{ $pull: { hobbies: "爬山" } }` |
| `$rename` | 字段改名 | `{ $rename: { age: "nianling" } }` |

> ⚠️ 不带 `$set` 直接写 `{ age: 19 }` 会把**整个文档替换**成 `{ age: 19 }`——其他字段全没了！新手经典翻车点 😱

### 5.3 删：deleteOne / deleteMany

```javascript
db.users.deleteOne({ name: "小刚" })     // 删第一条匹配
db.users.deleteMany({ age: { $lt: 18 } }) // 删所有匹配
db.users.deleteMany({})                   // 清空集合（慎用！）
```

---

## 六、文档查询（R）：重头戏！

### 6.1 基础查询

```javascript
// 查全部（相当于 SELECT *）
db.users.find()

// 格式化输出（mongosh 里看着舒服）
db.users.find().pretty()

// 条件查询：SELECT * FROM users WHERE name = '小花'
db.users.find({ name: "小花" })

// 多条件 AND：WHERE name = '小花' AND age = 18
db.users.find({ name: "小花", age: 18 })

// 投影：只要 name 和 age 字段（1要 / 0不要）
db.users.find({}, { name: 1, age: 1, _id: 0 })
```

### 6.2 比较操作符

| 操作符 | 含义 | SQL 类比 | 示例 |
| --- | --- | --- | --- |
| `$eq` / `$ne` | 等于 / 不等于 | = / != | `{ age: { $ne: 18 } }` |
| `$gt` / `$gte` | 大于 / 大于等于 | > / >= | `{ age: { $gt: 18 } }` |
| `$lt` / `$lte` | 小于 / 小于等于 | < / <= | `{ age: { $lte: 20 } }` |
| `$in` | 在集合中 | IN | `{ age: { $in: [18, 20] } }` |
| `$nin` | 不在集合中 | NOT IN | `{ age: { $nin: [18] } }` |
| 范围组合 | 区间 | BETWEEN | `{ age: { $gte: 18, $lt: 30 } }` |

```javascript
// 18 ≤ age < 30
db.users.find({ age: { $gte: 18, $lt: 30 } })
```

### 6.3 逻辑操作符

```javascript
// $and：年龄>18 且 城市=成都
db.users.find({ $and: [ { age: { $gt: 18 } }, { "address.city": "成都" } ] })

// $or：名字是小花 或者 年龄小于 20
db.users.find({ $or: [ { name: "小花" }, { age: { $lt: 20 } } ] })

// $not：年龄不大于 18
db.users.find({ age: { $not: { $gt: 18 } } })
```

> 💡 注意嵌套字段用**点语法**访问：`"address.city"`、`"items.0.goods"`（数组第 0 个元素）～

### 6.4 数组查询（文档库的杀手锏！）

```javascript
// hobbies 数组里包含"爬山"的文档（全包含一个也算！）
db.users.find({ hobbies: "爬山" })

// 数组必须**正好等于** [爬山, 拍照]（顺序、个数都对）
db.users.find({ hobbies: ["爬山", "拍照"] })

// $all：包含这些元素（不管顺序）
db.users.find({ hobbies: { $all: ["爬山", "拍照"] } })

// $size：数组长度等于 2
db.users.find({ hobbies: { $size: 2 } })

// $elemMatch：数组里有一个元素同时满足多个条件
db.orders.find({ items: { $elemMatch: { goods: "奶茶", num: { $gte: 2 } } } })
```

### 6.5 模糊查询（正则）

```javascript
// name 以"小"开头（相当于 LIKE '小%'）
db.users.find({ name: /^小/ })

// name 包含"花"
db.users.find({ name: /花/ })
```

### 6.6 排序、分页、统计

```javascript
// 排序：1 升序，-1 降序；先按年龄降序再按名字升序
db.users.find().sort({ age: -1, name: 1 })

// 分页：skip 跳过条数 + limit 取条数（第2页，每页10条）
db.users.find().skip(10).limit(10)

// 统计条数
db.users.countDocuments({ age: { $gte: 18 } })

// 去重
db.users.distinct("address.city")
```

### 6.7 查询 vs SQL 速查表

| SQL | MongoDB |
| --- | --- |
| `SELECT * FROM users` | `db.users.find()` |
| `WHERE name='小花'` | `find({ name: "小花" })` |
| `WHERE age > 18` | `find({ age: { $gt: 18 } })` |
| `WHERE a=1 AND b=2` | `find({ a: 1, b: 2 })` |
| `WHERE a=1 OR b=2` | `find({ $or: [{a:1},{b:2}] })` |
| `SELECT name, age` | `find({}, { name:1, age:1 })` |
| `ORDER BY age DESC` | `.sort({ age: -1 })` |
| `LIMIT 10` | `.limit(10)` |
| `COUNT(*)` | `.countDocuments()` |

---

## 七、聚合管道：MongoDB 的"流水线"

`find` 查不动复杂统计时，请出**聚合管道（aggregate）**——数据像流水线一样，一站一站被加工 🏭

```mermaid
flowchart LR
    A[(文档集)] --> B[$match<br>过滤] --> C[$group<br>分组] --> D[$sort<br>排序] --> E[$limit<br>截取] --> F[结果 📊]
```

**例子：统计成都各区用户的平均年龄和人数，取前 3 名**

```javascript
db.users.aggregate([
  // 相当于 WHERE city = '成都'
  { $match: { "address.city": "成都" } },

  // 相当于 GROUP BY city，顺便算平均、计数
  { $group: {
      _id: "$address.city",          // 分组键（$xxx 表示取字段值）
      avgAge: { $avg: "$age" },      // 平均年龄
      count: { $sum: 1 }             // 人数
  }},

  // 相当于 ORDER BY count DESC
  { $sort: { count: -1 } },

  // 相当于 LIMIT 3
  { $limit: 3 }
])
```

**常用管道阶段速查：**

| 阶段 | 作用 | SQL 类比 |
| --- | --- | --- |
| `$match` | 过滤数据 | WHERE |
| `$group` | 分组统计 | GROUP BY |
| `$sort` | 排序 | ORDER BY |
| `$limit` / `$skip` | 截取 / 跳过 | LIMIT / OFFSET |
| `$project` | 挑字段、改名、计算 | SELECT |
| `$unwind` | 数组拆成多行（一行变N行） | 无（JOIN 反操作的感觉）|
| `$lookup` | 关联其他集合 | LEFT JOIN ✨ |
| `$count` | 计数 | COUNT |

**$unwind + $group 经典组合：统计订单里每种商品卖了几个**

```javascript
db.orders.aggregate([
  { $unwind: "$items" },                          // 每个商品拆成单独一行
  { $group: { _id: "$items.goods", total: { $sum: "$items.num" } } },
  { $sort: { total: -1 } }
])
```

---

## 八、索引：提速老三样

### 8.1 和 MySQL 一样的道理

没索引 → 全集合扫描（COLLSCAN）🐌；有索引 → B 树查找（IXSCAN）🚀

（对，MongoDB 用的是传统 **B 树**，不像 ES 的 B+ 树叶子串链表，也不像 Redis 的跳表哈希那一套～）

```javascript
// 创建索引（1 升序 / -1 降序）
db.users.createIndex({ name: 1 })

// 联合索引
db.users.createIndex({ city: 1, age: -1 })

// 唯一索引（防重复，比如手机号）
db.users.createIndex({ phone: 1 }, { unique: true })

// 查看索引（_id 自带一个）
db.users.getIndexes()

// 删除索引
db.users.dropIndex("name_1")
db.users.dropIndexes()   // 删掉所有自定义索引
```

### 8.2 执行计划 explain

```javascript
db.users.find({ name: "小花" }).explain("executionStats")
```

| 看什么 | 好的样子 | 坏的样子 |
| --- | --- | --- |
| stage | `IXSCAN`（走索引）| `COLLSCAN`（全表扫）⚠️ |
| totalDocsExamined | 越小越好 | 等于总数就糟糕 |
| indexUsed | 有名字 | 空的 |

### 8.3 索引类型一览

| 类型 | 说明 |
| --- | --- |
| 单字段索引 | 一个字段建索引 |
| 联合索引 | 多字段，遵守**最左前缀**（和 MySQL 一样哦）|
| 唯一索引 | 值不许重复 |
| TTL 索引 | 到期自动删文档（存验证码、日志超好用！）|
| 文本索引 | 简单全文搜索（专业搜索还是 ES）|
| 地理索引 | 2dsphere，附近的人 📍 |

```javascript
// TTL 索引：60秒后自动删除，验证码场景神器
db.codes.createIndex({ createdAt: 1 }, { expireAfterSeconds: 60 })
```

---

## 九、副本集：高可用三兄弟

单机挂了数据就没了 → MongoDB 的方案叫**副本集（Replica Set）**，思路和 Redis 主从哨兵神似！

```mermaid
flowchart TD
    C[客户端写入] --> P[Primary 主节点<br>唯一可写]
    P -- 自动复制 --> S1[Secondary 从节点1<br>只读]
    P -- 自动复制 --> S2[Secondary 从节点2<br>只读]
    P --挂了😱--> E[心跳选举<br>几秒内选出新 Primary 🎉]
    S1 -.投票.-> E
    S2 -.投票.-> E
```

| 角色 | 职责 |
| --- | --- |
| Primary 主节点 | 处理所有写操作 |
| Secondary 从节点 | 同步数据、分担读 |
| 选举机制 | 主节点失联 → 从节点们投票选新主（自动故障转移！）|

> 💡 和 Redis 对比记忆：Redis 是"主从复制 + 单独的哨兵盯着"；MongoDB 是**节点们自己开会投票**，不需要额外的哨兵进程，更简洁～

**写关注（Write Concern）：** 写入几个节点确认才算成功

```javascript
db.orders.insertOne(doc, { writeConcern: { w: "majority" } })  // 多数节点确认，更安全
```

---

## 十、分片集群：装不下的就分家

数据量大到一台机器存不下 → **分片（Sharding）** 把集合数据水平拆到多台机器（思路同 ES 分片、同 Redis Cluster）～

```mermaid
flowchart TD
    A[客户端] --> R[路由 mongos<br>查路由表知道数据在哪]
    R --> C[Config Server<br>存元数据：哪些块在哪台]
    R --> S1[分片1<br>副本集A]
    R --> S2[分片2<br>副本集B]
    R --> S3[分片3<br>副本集C]
```

| 组件 | 作用 |
| --- | --- |
| mongos 路由 | 请求入口，指挥数据去哪个分片 |
| Config Server | 记录分片元数据（数据分布地图）|
| Shard 分片 | 真正存数据的副本集 |

**分片键怎么选？（很重要！）**

| 策略 | 特点 |
| --- | --- |
| 范围分片（如按时间） | 范围查询友好，但容易热点（新数据全挤一个分片）|
| 哈希分片 | 分布均匀，但范围查询要扫所有分片 |

> ⚠️ 分片键**建集合后几乎不能改**，选之前想清楚！一般选**基数高、查询常带、写入均匀**的字段（如 userId 哈希）～

---

## 十一、MongoDB 也有事务！

4.0 起支持**副本集多文档事务**，4.2 起分片集群也支持～

```javascript
const session = db.getMongo().startSession();
session.startTransaction();

try {
  session.getDatabase("shop").getCollection("account")
    .updateOne({ name: "小明" }, { $inc: { money: -100 } }, { session });
  session.getDatabase("shop").getCollection("account")
    .updateOne({ name: "小红" }, { $inc: { money: 100 } }, { session });
  session.commitTransaction();     // 提交 ✅
} catch (e) {
  session.abortTransaction();      // 回滚 ↩️
} finally {
  session.endSession();
}
```

| 对比项 | MongoDB 事务 | MySQL 事务 |
| --- | --- | --- |
| 支持 | ✅ 4.0+ | ✅ 老牌强项 |
| 隔离级别 | 快照隔离为主 | 四级齐全 |
| 性能开销 | 较大 | 优化成熟 |
| 建议 | 能不用就不用（单文档操作本身原子）| 强事务首选 |

> 🎀 MongoDB 设计哲学：**单文档操作天生原子**——把一次业务变更塞进一个文档里（嵌套设计），根本不需要跨文档事务！这是它和 MySQL 最大的思路差异～

---

## 十二、四方对决：MongoDB vs 三位前辈

学完四位，终极对决表镇楼！🏆

| 对比项 | MySQL 📊 | Redis ⚡ | ES 🔍 | MongoDB 🍃 |
| --- | --- | --- | --- | --- |
| 定位 | 关系型数据库 | 内存 KV 缓存 | 搜索引擎 | 文档数据库 |
| 数据模型 | 表（行列固定）| 键值对 | JSON 文档 | BSON 文档 |
| Schema | 严格 | 无 | Mapping | **灵活** ✨ |
| 强项 | 事务、JOIN、一致性 | 速度、过期、锁 | 全文检索、聚合 | 灵活结构、嵌套数据 |
| 事务 | ✅ ACID | 弱 | ❌ | ✅（4.0+）|
| JOIN | ✅ 强 | ❌ | 弱 | $lookup（够用）|
| 典型场景 | 核心业务账 | 缓存/排行榜/锁 | 搜索/日志分析 | 商品/评论/Feed/日志 |

**一句话选型口诀：**

> 要**强一致**找 MySQL，要**快**找 Redis，要**搜**找 ES，要**灵活**找 MongoDB！🎀

---

## 十三、经典应用场景与避坑

### 13.1 哪些场景特别香？

| 场景 | 为什么香 |
| --- | --- |
| 🛒 电商商品 | 不同类目属性不同（手机有内存、衣服有尺码），灵活 Schema 完美适配 |
| 💬 评论/弹幕 | 结构简单、写入量大、嵌套点赞列表方便 |
| 📰 内容管理 CMS | 文章字段五花八门，随加随存 |
| 📋 用户行为/埋点日志 | 海量写入 + 结构多变 |
| 📍 位置服务 | 地理索引，附近的人 |
| ⏱️ 验证码/临时数据 | TTL 索引自动过期，不用手动清理 |

### 13.2 常见坑 ⚠️

| 坑 | 后果 | 正确姿势 |
| --- | --- | --- |
| update 忘写 `$set` | 整个文档被替换，字段丢失 😱 | 改局部必须 `$set` |
| `_id` 当业务主键乱改 | 不可变，改不了 | 业务键另建唯一索引 |
| 集合完全不加索引 | 全表扫，越查越慢 | 常用查询字段建索引 |
| 16MB 文档塞爆 | 单文档上限 16MB | 数组无节制增长要拆分/用 Bucket 设计 |
| 无限增长的数组字段 | 文档越来越大，性能崩 | $push 配 $slice 限长 |
| skip 深分页 | 越翻越慢 | 游标分页（记上一页最后 _id）|
| 强事务场景硬上 Mongo | 事务开销大 | 转账/扣库存回 MySQL |
| 分片键随手选 | 数据倾斜、改不了 | 选高基数+查询高频字段 |

---

## 十四、总结与面试高频题

### 14.1 知识地图

```mermaid
mindmap
  root((MongoDB))
    基础
      文档模型 BSON
      库 集合 文档
      灵活Schema
    CRUD
      insertOne Many
      updateOne $set $inc
      find 操作符
      数组查询
    进阶
      聚合管道 aggregate
      索引 B树 TTL
      explain
    架构
      副本集 选举
      分片集群 mongos
      事务
```

### 14.2 面试官最爱问 Top 题 🎯

| 问题 | 答题要点 |
| --- | --- |
| MongoDB 和 MySQL 区别？ | 文档 vs 表；灵活 vs 固定 Schema；弱 JOIN vs 强 JOIN；场景差异 |
| 什么时候选 MongoDB？ | 字段多变、嵌套数据、高写入、无强事务要求 |
| `$set` 不写会怎样？ | 整个文档被替换，等于 replace |
| find 和 aggregate 区别？ | find 简单查询；aggregate 流水线多阶段加工（分组统计） |
| MongoDB 索引什么结构？ | B 树；联合索引最左前缀；TTL 索引自动过期 |
| 副本集原理？ | 一主多从自动复制；主挂自动选举；写关注可调 |
| 分片集群组成？ | mongos 路由 + Config Server + 多个分片（每片是副本集） |
| MongoDB 支持事务吗？ | 4.0+ 支持多文档事务；单文档操作天生原子 |
| 文档上限多少？ | 单文档 16MB；超了要拆分设计 |
| ObjectId 组成？ | 时间戳+机器+进程+随机数，天然按时间递增 |

### 14.3 学习建议路线 🗺️

1. **第一站**：Docker 起 MongoDB + Compass，CRUD 全敲一遍
2. **第二站**：玩转查询操作符 + 数组查询（文档库的灵魂）
3. **第三站**：聚合管道，$match/$group/$unwind 组合拳
4. **第四站**：索引 + explain，看懂 COLLSCAN 和 IXSCAN
5. **终点站**：副本集/分片搭一遍 + 事务实践，架构视野拉满 🌟

---

> 🌈 MongoDB 通关！现在你的数据库全家桶集齐四位成员：MySQL、Redis、ES、MongoDB——存储界的复仇者联盟正式成立！记牢口诀"强一致、快、搜、灵活"，选型再也不纠结～继续冲鸭！٩( 'ω' )و 💪✨
