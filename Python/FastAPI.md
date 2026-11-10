# ⚡ FastAPI 框架详解

> 哈喽呀～欢迎来到 FastAPI 的专属课堂！🌟 这是目前 Python 界最"潮"的框架——类型注解驱动、原生异步、自动生成 API 文档，性能直逼 Go 和 Node.js！冲鸭！(๑•̀ㅂ•́)و✧

***

## 🗺️ FastAPI 知识全景图

```mermaid
mindmap
  root(("FastAPI 框架"))
    基础入门
      环境搭建
      第一个应用
      自动文档
      路径操作
    参数处理
      路径参数
      查询参数
      请求体
      Header / Cookie
    Pydantic
      数据模型定义
      数据验证
      嵌套模型
      响应模型
    核心机制
      依赖注入 Depends
      异步 async await
      后台任务
      WebSocket
    安全认证
      OAuth2 密码模式
      JWT Token
      API Key
      权限控制
    数据库
      SQLAlchemy 2.0
      异步数据库
      连接管理
    工程化
      项目结构
      中间件 CORS
      异常处理
      配置管理
    部署
      Uvicorn
      Gunicorn 多进程
      Nginx
      Docker
```

***

## 📑 目录

- [一、FastAPI 简介：现代 API 框架之王](#一fastapi-简介现代-api-框架之王)
- [二、环境搭建与第一个应用](#二环境搭建与第一个应用)
- [三、路径操作与参数详解](#三路径操作与参数详解)
- [四、Pydantic：数据验证神器](#四pydantic数据验证神器)
- [五、请求体与响应模型](#五请求体与响应模型)
- [六、依赖注入系统](#六依赖注入系统)
- [七、安全认证：OAuth2 + JWT](#七安全认证oauth2--jwt)
- [八、数据库集成](#八数据库集成)
- [九、异步编程](#九异步编程)
- [十、后台任务与 WebSocket](#十后台任务与-websocket)
- [十一、中间件与 CORS](#十一中间件与-cors)
- [十二、异常处理](#十二异常处理)
- [十三、项目结构与配置](#十三项目结构与配置)
- [十四、实战：待办事项 API](#十四实战待办事项-api)
- [十五、部署上线](#十五部署上线)
- [十六、知识总结](#十六知识总结)

***

## 一、FastAPI 简介：现代 API 框架之王 👑

### 1.1 FastAPI 是什么？

FastAPI 由 Sebastián Ramírez（tiangolo）于 2018 年发布，是基于 **Starlette**（Web 部分）+ **Pydantic**（数据验证部分）构建的现代框架。

> 🎀 一句话理解：Flask 的简洁 + 类型注解的严谨 + 异步的性能 + 自动文档的省心 = FastAPI！

### 1.2 四大核心特性

```mermaid
graph TD
    A["FastAPI 四大杀器"] --> B["⚡ 快<br/>性能比肩 NodeJS/Go"]
    A --> C["✍️ 简单<br/>代码量少，直觉化开发"]
    A --> D["🔧 类型安全<br/>自动验证数据"]
    A --> E["📖 自动文档<br/>免写 API 文档"]

    E --> E1["/docs Swagger UI"]
    E --> E2["/redoc ReDoc"]

    style A fill:#e1bee7
    style E fill:#c8e6c9
```

| 特性        | 说明                              |
| :-------- | :------------------------------ |
| **高性能**   | 与 NodeJS、Go 相当（得益于 ASGI + 异步）   |
| **开发快**   | 类型注解自动完成大量工作，开发效率提升约 200%\~300% |
| **Bug 少** | 编辑器智能提示 + 自动数据验证，人为错误大幅减少       |
| **自动文档**  | 交互式 API 文档自动生成，可直接测试接口          |

### 1.3 适合场景

| 场景              |         是否适合         |
| :-------------- | :------------------: |
| 前后端分离 API       |        ✅ 最佳选择        |
| 微服务             |        ✅ 轻量高性能       |
| 机器学习模型服务        |      ✅（社区主流选择！）      |
| 实时应用（WebSocket） |        ✅ 原生支持        |
| 需要后台管理界面        | ❌（无 Admin，考虑 Django） |
| 服务端渲染传统网页       |       ⚠️ 可以但不擅长      |

### 1.4 与 Flask/Django 对比

| 对比项   | FastAPI ⚡     | Flask 🌸 | Django 🏗️ |
| :---- | :------------ | :------- | :--------- |
| 诞生年份  | 2018          | 2010     | 2005       |
| 类型系统  | ✅ Pydantic 强制 | ❌ 可选     | ⚠️ 可选 Form |
| 异步支持  | ✅ 原生          | ⚠️ 部分    | ⚠️ 部分      |
| 自动文档  | ✅ 自带          | ❌ 扩展     | ❌ 扩展       |
| 性能    | ⭐⭐⭐⭐⭐         | ⭐⭐⭐      | ⭐⭐⭐        |
| 学习曲线  | ⭐⭐⭐           | ⭐⭐       | ⭐⭐⭐⭐       |
| 生态成熟度 | ⭐⭐⭐⭐          | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⭐      |

***

## 二、环境搭建与第一个应用 🚀

### 2.1 安装

```bash
# 安装 FastAPI + ASGI 服务器（推荐带 standard 全家桶）
pip install "fastapi[standard]"

# 或最小安装
pip install fastapi
pip install "uvicorn[standard]"
```

### 2.2 第一个应用

```python
# main.py
from fastapi import FastAPI

# 创建应用实例
app = FastAPI()

# 定义路由：装饰器 + 类型注解
@app.get("/")
def read_root():
    return {"Hello": "FastAPI"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

```bash
# 启动服务器（文件名:应用对象）
uvicorn main:app --reload
#                    ↑ 改代码自动重启（开发必备）

# 完整参数
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### 2.3 两大自动文档（惊喜！）

启动后浏览器打开：

| 地址                            | 说明                             |
| :---------------------------- | :----------------------------- |
| `http://127.0.0.1:8000/docs`  | **Swagger UI**：交互式文档，可直接测试接口 ⭐ |
| `http://127.0.0.1:8000/redoc` | **ReDoc**：阅读型文档，适合分享           |

> 💡 不用写一行文档代码！FastAPI 根据你的**类型注解**自动生成文档，还带"Try it out"按钮直接测试～

### 2.4 类型注解的魔法

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

仅仅加了 `: int`，FastAPI 就自动帮你做了：

```mermaid
graph LR
    A["请求 /items/5"] --> B["item_id: int<br/>自动转成 5 ✅"]
    A2["请求 /items/abc"] --> B2["自动验证失败<br/>返回 422 错误 ❌"]

    B --> C["进入函数处理"]
    B2 --> D["detail: value is not<br/>a valid integer"]

    style B fill:#c8e6c9
    style B2 fill:#ffcdd2
```

```json
// /items/abc 的响应（自动的，零代码！）
{
    "detail": [
        {
            "type": "int_parsing",
            "loc": ["path", "item_id"],
            "msg": "Input should be a valid integer, unable to parse string as an integer",
            "input": "abc"
        }
    ]
}
```

***

## 三、路径操作与参数详解 🛣️

### 3.1 HTTP 方法装饰器

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items")          # 查列表
@app.post("/items")         # 创建
@app.put("/items/{id}")     # 全量更新
@app.patch("/items/{id}")   # 部分更新
@app.delete("/items/{id}")  # 删除
@app.options("/items")      # OPTIONS 预检请求
@app.head("/items")         # HEAD 只取响应头
@app.api_route("/items", methods=["GET", "POST"])   # 多方法共存
```

### 3.2 三种参数一览

```python
from fastapi import FastAPI

app = FastAPI()

@app.put("/items/{item_id}")
def update_item(item_id: int, q: str | None = None, body: dict = None):
    #      item_id: int          → 路径参数（URL 里的 {item_id}）
    #      q: str | None = None  → 查询参数（?q=xxx）
    #      body: dict            → 请求体（JSON）……但推荐用 Pydantic！
    return {"item_id": item_id, "q": q}
```

```mermaid
graph TD
    A["函数参数"] --> B{"参数名在<br/>路径里吗？"}
    B -->|"是（如 {item_id}）"| C["📍 路径参数"]
    B -->|"否，是简单类型"| D["❓ 查询参数"]
    B -->|"否，是 Pydantic 模型"| E["📦 请求体"]

    C --> F["/items/5"]
    D --> G["/items?q=keyword<br/>&page=2"]
    E --> H["POST JSON 数据"]

    style C fill:#bbdefb
    style D fill:#fff9c4
    style E fill:#c8e6c9
```

### 3.3 路径参数详解

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):          # 类型注解决定验证和文档
    return {"user_id": user_id}

# —— 枚举限制 ——
from enum import Enum

class ModelName(str, Enum):                # 继承 str + Enum
    alexnet = "alexnet"
    resnet = "resnet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model": model_name}
    return {"model": model_name}           # 传入枚举外的值 → 422

# —— 路径参数校验 ——
from fastapi import Path

@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(
        ...,                    # ... 表示必填
        title="商品ID",
        ge=1,                   # >= 1
        le=10000,               # <= 10000
        description="商品的唯一标识",
    )
):
    return {"item_id": item_id}
```

### 3.4 查询参数详解

```python
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

@app.get("/items/")
async def list_items(
    q: str | None = Query(None, min_length=2, max_length=50),   # 可选 + 长度限制
    page: int = Query(1, ge=1),                                  # 默认 1，>= 1
    size: int = Query(10, ge=1, le=100),                         # 1~100
    tags: list[str] = Query([]),                                 # 列表参数 ?tags=a&tags=b
):
    return {"q": q, "page": page, "size": size, "tags": tags}

# Python 3.9+ 推荐用 Annotated（官方新写法）
@app.get("/search/")
async def search(
    keyword: Annotated[str | None, Query(min_length=1)] = None,
):
    return {"keyword": keyword}
```

| 校验参数                        | 适用类型 | 说明        |
| :-------------------------- | :--- | :-------- |
| `gt` / `ge`                 | 数值   | 大于 / 大于等于 |
| `lt` / `le`                 | 数值   | 小于 / 小于等于 |
| `min_length` / `max_length` | 字符串  | 长度限制      |
| `pattern`                   | 字符串  | 正则匹配      |
| `title` / `description`     | 全部   | 文档显示用     |

### 3.5 Header 与 Cookie

```python
from fastapi import FastAPI, Header, Cookie

app = FastAPI()

@app.get("/headers/")
async def read_headers(
    user_agent: str | None = Header(None),           # 自动转下划线 ↔ 连字符
    x_token: str | None = Header(None),              # X-Token 请求头
):
    return {"User-Agent": user_agent, "X-Token": x_token}

@app.get("/cookies/")
async def read_cookies(session_id: str | None = Cookie(None)):
    return {"session_id": session_id}
```

### 3.6 表单与文件上传

```bash
pip install python-multipart    # 表单和文件需要
```

```python
from fastapi import FastAPI, Form, File, UploadFile

app = FastAPI()

# —— 表单数据 ——
@app.post("/login/")
async def login(
    username: str = Form(...),
    password: str = Form(...),
):
    return {"username": username}

# —— 文件上传（推荐 UploadFile）——
@app.post("/upload/")
async def upload_file(file: UploadFile):
    contents = await file.read()              # 异步读取内容
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents),
    }

# —— 保存到磁盘 ——
import shutil
from pathlib import Path

@app.post("/upload-save/")
async def upload_save(file: UploadFile):
    path = Path("uploads") / file.filename
    with path.open("wb") as f:
        shutil.copyfileobj(file.file, f)
    return {"saved": str(path)}

# —— 多文件上传 ——
@app.post("/uploads/")
async def upload_multi(files: list[UploadFile]):
    return {"filenames": [f.filename for f in files]}
```

***

## 四、Pydantic：数据验证神器 🛡️

> Pydantic 是 FastAPI 的"灵魂搭档"——用类定义数据结构，自动验证、转换、清洗数据！

### 4.1 定义数据模型

```python
from pydantic import BaseModel, Field
from typing import Optional

class UserBase(BaseModel):
    username: str = Field(..., min_length=2, max_length=20, description="用户名")
    age: int = Field(..., ge=0, le=150, description="年龄")
    email: str                                       # EmailStr 更严格（需 pip install email-validator）
    bio: str | None = None                           # 可选字段
    is_vip: bool = False                             # 带默认值

class UserCreate(UserBase):
    """创建用户：继承 + 增加密码字段"""
    password: str = Field(..., min_length=6)

class UserOut(UserBase):
    """输出用户：不要密码！"""
    id: int
```

| Field 参数                | 说明         | 示例                                |
| :---------------------- | :--------- | :-------------------------------- |
| `...`                   | 第一个参数，必填标记 | `Field(...)`                      |
| `default`               | 默认值        | `Field(default=0)`                |
| `gt/ge/lt/le`           | 数值范围       | `Field(ge=1, le=100)`             |
| `min_length/max_length` | 字符串长度      | `Field(min_length=2)`             |
| `pattern`               | 正则校验       | `Field(pattern=r"^\d{11}$")`      |
| `title/description`     | 文档显示       | `Field(title="手机号")`              |
| `examples`              | 文档示例       | `Field(examples=["13800138000"])` |

### 4.2 数据验证演示

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(..., min_length=1)
    price: float = Field(..., gt=0)

@app.post("/items/")
async def create_item(item: Item):
    return item
```

**提交合法数据**：

```json
// POST /items/  {"name": "苹果", "price": 9.9}
// 响应 200：
{"name": "苹果", "price": 9.9}
```

**提交非法数据（price 为负数）**：

```json
// 响应 422：
{
    "detail": [
        {
            "type": "greater_than",
            "loc": ["body", "price"],
            "msg": "Input should be greater than 0",
            "input": -5
        }
    ]
}
```

> 💡 一行校验代码都没写！Pydantic 自动验证并返回规范的错误信息～

### 4.3 嵌套模型

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    address: Address                    # 嵌套单个
    tags: list[str] = []                # 列表
    scores: dict[str, int] = {}         # 字典

# 请求体示例：
# {
#     "name": "小明",
#     "address": {"city": "北京", "street": "长安街"},
#     "tags": ["vip", "active"],
#     "scores": {"math": 90}
# }
```

### 4.4 自定义验证器

```python
from pydantic import BaseModel, field_validator, model_validator

class SignupData(BaseModel):
    username: str
    password: str
    password_confirm: str

    @field_validator("username")
    @classmethod
    def username_alpha(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError("用户名只能是字母数字")
        return v

    @model_validator(mode="after")      # 多字段联合校验
    def check_passwords(self):
        if self.password != self.password_confirm:
            raise ValueError("两次密码不一致")
        return self
```

### 4.5 Pydantic v2 常用功能速查

| 功能         | 写法                                          |
| :--------- | :------------------------------------------ |
| 模型转字典      | `model.model_dump()`                        |
| 转 JSON 字符串 | `model.model_dump_json()`                   |
| 从字典创建      | `Model.model_validate(dict)`                |
| 从 JSON 创建  | `Model.model_validate_json(json_str)`       |
| 字段别名       | `Field(alias="userName")`                   |
| 忽略多余字段     | `model_config = ConfigDict(extra="ignore")` |
| 禁止多余字段     | `model_config = ConfigDict(extra="forbid")` |

***

## 五、请求体与响应模型 📦

### 5.1 请求体的标准姿势

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class ItemCreate(BaseModel):
    """创建商品的请求体模型"""
    name: str = Field(..., examples=["苹果"])
    price: float = Field(..., gt=0, examples=[9.9])
    tags: list[str] = []

@app.post("/items/", status_code=201)       # 成功返回 201
async def create_item(item: ItemCreate):
    # 走到这里，数据 100% 合法！
    return {"message": f"创建 {item.name} 成功", "price": item.price}
```

### 5.2 响应模型：response\_model

> **response\_model 是 FastAPI 的安全卫士**——过滤掉不该返回的字段（比如密码）！

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# —— 数据库用户（含密码哈希）——
fake_db = {
    1: {
        "id": 1, "username": "小明",
        "email": "xm@test.com",
        "password_hash": "$2b$12$xxxxx",    # 绝不能泄露！
    }
}

class UserOut(BaseModel):
    """响应模型：白名单机制"""
    id: int
    username: str
    email: str
    # 没有 password_hash → 自动被过滤！

@app.get("/users/{user_id}", response_model=UserOut)
async def get_user(user_id: int):
    user = fake_db[user_id]
    return user
    # 虽然返回了完整 dict，但响应只有 id/username/email！

# 响应：{"id": 1, "username": "小明", "email": "xm@test.com"}
# password_hash 被自动剔除 ✅
```

```mermaid
graph LR
    A["视图函数返回<br/>完整数据"] --> B["response_model 过滤"]
    B --> C["只保留模型定义的字段<br/>+ 类型转换"]
    C --> D["安全的 JSON 响应 ✅"]

    E["password_hash ❌"] -.->|被过滤| B
    F["datetime ✅"] -.->|转 ISO 字符串| C

    style D fill:#c8e6c9
    style E fill:#ffcdd2
```

### 5.3 请求模型 vs 响应模型（最佳实践）

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from datetime import datetime

app = FastAPI()

# —— 三层模型设计（企业规范）——
class UserBase(BaseModel):
    """公共字段"""
    username: str = Field(..., min_length=2)
    email: str

class UserCreate(UserBase):
    """创建：Base + 密码"""
    password: str = Field(..., min_length=6)

class UserUpdate(BaseModel):
    """更新：全部可选（PATCH 语义）"""
    username: str | None = None
    email: str | None = None

class UserOut(UserBase):
    """输出：Base + id + 时间（无密码）"""
    id: int
    created_at: datetime

@app.post("/users/", response_model=UserOut, status_code=201)
async def create_user(user: UserCreate):
    # 请求用 UserCreate 验证，响应用 UserOut 过滤
    return {"id": 1, "username": user.username,
            "email": user.email, "created_at": datetime.now()}

@app.patch("/users/{user_id}", response_model=UserOut)
async def update_user(user_id: int, user: UserUpdate):
    return {"id": user_id, "username": user.username or "默认",
            "email": user.email or "a@b.c", "created_at": datetime.now()}
```

| 模型           | 用途                 | 特点        |
| :----------- | :----------------- | :-------- |
| `UserCreate` | POST 请求体验证         | 必填字段 + 密码 |
| `UserUpdate` | PATCH 请求体验证        | 全部可选      |
| `UserOut`    | response\_model 响应 | 白名单，无敏感字段 |

### 5.4 响应状态码与响应头

```python
from fastapi import FastAPI, status
from fastapi.responses import JSONResponse

app = FastAPI()

# —— 方式 1：装饰器参数 ——
@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item():
    return {"ok": True}

# —— 方式 2：Response 参数（动态设置）——
from fastapi import Response

@app.get("/set-cookie/")
async def set_cookie(response: Response):
    response.status_code = 202
    response.headers["X-Custom"] = "hello"
    response.set_cookie(key="token", value="abc123")
    return {"message": "done"}

# —— 方式 3：JSONResponse 完全自定义 ——
@app.get("/custom/")
async def custom_response():
    return JSONResponse(
        status_code=200,
        content={"code": 0, "msg": "success"},
        headers={"X-Powered-By": "FastAPI"},
    )
```

### 5.5 表单模型 ModelForm（进阶）

```python
# 官方推荐用 pydantic 模型处理表单（FastAPI 0.115+ 支持模型直接接收表单）
from fastapi import FastAPI, Form
from pydantic import BaseModel

app = FastAPI()

class LoginForm(BaseModel):
    username: str
    password: str

@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

### 5.6 其他响应类型

```python
from fastapi import FastAPI
from fastapi.responses import (
    HTMLResponse, PlainTextResponse,
    RedirectResponse, StreamingResponse, FileResponse,
)

app = FastAPI()

@app.get("/html/", response_class=HTMLResponse)
async def get_html():
    return "<h1>你好 HTML</h1>"

@app.get("/text/", response_class=PlainTextResponse)
async def get_text():
    return "纯文本"

@app.get("/redirect/")
async def redirect():
    return RedirectResponse(url="/html/")

@app.get("/file/")
async def download():
    return FileResponse("report.pdf", filename="报表.pdf")

@app.get("/stream/")
async def stream():
    def generate():
        for i in range(10):
            yield f"第 {i} 行\n"
    return StreamingResponse(generate(), media_type="text/plain")
```

***

## 六、依赖注入系统 💉

> 依赖注入（Dependency Injection）是 FastAPI 的**核心精髓**——把公共逻辑（认证、数据库连接、分页）抽成"依赖"，哪里需要哪里注入！

### 6.1 什么是依赖注入？

```mermaid
graph TD
    A["Depends(get_db)"] --> B["视图函数 A"]
    A --> C["视图函数 B"]
    A --> D["视图函数 C"]
    E["Depends(get_current_user)"] --> F["视图函数 D"]
    E --> G["视图函数 E"]

    A --> H["统一管理连接/认证<br/>避免重复代码 ✅"]

    style A fill:#c8e6c9
    style E fill:#bbdefb
    style H fill:#fff9c4
```

> 🎀 通俗理解：依赖注入就像"外卖配送"——你不用自己买菜做饭（写重复代码），说一声要什么（声明依赖），平台自动送上门（FastAPI 执行）！

### 6.2 基本用法

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# —— 依赖就是一个函数 ——
async def common_parameters(q: str | None = None, page: int = 1, size: int = 10):
    return {"q": q, "page": page, "size": size}

@app.get("/items/")
async def list_items(commons: dict = Depends(common_parameters)):
    # commons 就是 common_parameters 的返回值！
    return commons

@app.get("/users/")
async def list_users(commons: dict = Depends(common_parameters)):
    return commons     # 复用同一个依赖，零重复代码
```

### 6.3 依赖嵌套（依赖套依赖）

```python
from fastapi import Depends, FastAPI, HTTPException, Header

async def verify_token(x_token: str = Header()):
    """验证 Token"""
    if x_token != "secret-token":
        raise HTTPException(status_code=401, detail="无效 Token")
    return x_token

async def verify_key(x_key: str = Header()):
    if x_key != "secret-key":
        raise HTTPException(status_code=400, detail="无效 Key")
    return x_key

async def verify_all(token: str = Depends(verify_token), key: str = Depends(verify_key)):
    """组合依赖：两个都通过才行"""
    return {"token": token, "key": key}

@app.get("/protected/")
async def protected(creds: dict = Depends(verify_all)):
    return {"message": "你通过了所有验证！", "creds": creds}
```

### 6.4 类作为依赖

```python
class CommonQueryParams:
    def __init__(self, q: str | None = None, page: int = 1, size: int = 10):
        self.q = q
        self.page = page
        self.size = size

@app.get("/items/")
async def list_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    return {"q": commons.q, "page": commons.page}
```

### 6.5 全局依赖 & 路由组依赖

```python
from fastapi import FastAPI, Depends, APIRouter

# —— 应用级全局依赖（所有路由都要过）——
app = FastAPI(dependencies=[Depends(verify_token)])

# —— 路由器级依赖（一组路由共享）——
router = APIRouter(
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(verify_key)],      # 这个组都要验证 key
)

@router.get("/stats/")
async def admin_stats():                     # 自动带上所有依赖
    return {"visits": 9999}

app.include_router(router)
```

### 6.6 yield 依赖：善后处理

```python
async def get_db():
    """数据库连接依赖：用完自动清理"""
    db = DatabaseSession()
    try:
        yield db               # 视图函数执行期间，db 可用
    finally:
        db.close()             # 视图结束后自动执行（即使出错）！

@app.get("/users/")
async def list_users(db=Depends(get_db)):
    return db.query_all()
```

```mermaid
sequenceDiagram
    participant R as 请求
    participant D as get_db 依赖
    participant V as 视图函数

    R->>D: 进入依赖
    D->>D: 创建连接
    D-->>V: yield db（交出控制权）
    V->>V: 使用 db 查询
    V-->>D: 视图结束
    D->>D: finally 关闭连接 ✅
```

### 6.7 Annotated 写法（官方推荐）

```python
from typing import Annotated
from fastapi import Depends, FastAPI

app = FastAPI()

async def common_parameters(q: str | None = None, page: int = 1):
    return {"q": q, "page": page}

# 新写法：类型和默认值分离，更清晰
@app.get("/items/")
async def list_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons

# 甚至可以统一声明一次，到处复用
CommonsDep = Annotated[dict, Depends(common_parameters)]

@app.get("/users/")
async def list_users(commons: CommonsDep):      # 超简洁！
    return commons
```

***

## 七、安全认证：OAuth2 + JWT 🔐

### 7.1 认证方案概览

| 方案                    | 原理                     | 适用场景          |
| :-------------------- | :--------------------- | :------------ |
| HTTPBasic             | 请求头带明文账号密码（Base64）     | 内部工具          |
| **OAuth2 密码模式 + JWT** | 登录换 Token，之后带 Token 访问 | **前后端分离主流** ⭐ |
| API Key               | URL/Header 带 key       | 开放 API、爬虫     |
| OAuth2 授权码            | 跳转第三方授权                | 微信/GitHub 登录  |

### 7.2 JWT 是什么？

> JWT（JSON Web Token）= 一段加密的"电子门票"，里面装着用户信息，服务器不用存 Session！

```mermaid
graph LR
    A["用户登录"] --> B["服务器验证成功"]
    B --> C["签发 JWT Token"]
    C --> D["客户端保存 Token"]
    D --> E["每次请求带 Token<br/>Authorization: Bearer xxx"]
    E --> F["服务器验证签名<br/>解析出用户身份 ✅"]

    style C fill:#fff9c4
    style F fill:#c8e6c9
```

**JWT 结构**（三段，用 `.` 分隔）：

```text
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiIxMjMifQ . SflKxwRJSMeKKF2QT4f
└──────┬──────┘      └──────┬──────┘   └──────┬──────┘
     Header              Payload           Signature
   算法类型          用户数据（可解码）      防篡改签名
```

### 7.3 完整实现：OAuth2 密码模式

```bash
pip install "python-jose[cryptography]" "passlib[bcrypt]"
```

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# —— 配置 ——
SECRET_KEY = "change-me-to-a-long-random-string"   # 生产用环境变量！
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")   # 告诉文档去哪登录

app = FastAPI()

# —— 密码工具 ——
def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

# —— 模拟用户库（真实项目查数据库）——
fake_users_db = {
    "xiaoming": {
        "username": "xiaoming",
        "hashed_password": hash_password("123456"),
        "disabled": False,
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class UserOut(BaseModel):
    username: str
    disabled: bool

# —— 创建 Token ——
def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# —— 当前用户依赖（核心！）——
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]) -> UserOut:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无效的认证凭证",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = fake_users_db.get(username)
    if user is None:
        raise credentials_exception
    return UserOut(**user)

# —— 登录接口（换 Token）——
@app.post("/token", response_model=Token)
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user = fake_users_db.get(form_data.username)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        raise HTTPException(status_code=401, detail="用户名或密码错误")

    token = create_access_token(data={"sub": user["username"]})
    return {"access_token": token, "token_type": "bearer"}

# —— 受保护的接口 ——
@app.get("/users/me", response_model=UserOut)
async def read_me(current_user: Annotated[UserOut, Depends(get_current_user)]):
    return current_user
```

### 7.4 使用流程（在 /docs 里体验）

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as FastAPI

    C->>S: POST /token（username + password）
    S-->>C: {"access_token": "eyJ...", "token_type": "bearer"}
    Note over C: 保存 Token
    C->>S: GET /users/me<br/>Authorization: Bearer eyJ...
    Note over S: 验证签名 → 解出 username
    S-->>C: 200 用户信息
```

> 💡 **/docs 页面右上角有个 Authorize 按钮**！输入用户名密码后，所有接口测试都自动带 Token，超方便～

### 7.5 API Key 认证（简单场景）

```python
from fastapi import FastAPI, Header, HTTPException

app = FastAPI()

async def verify_api_key(x_api_key: str = Header()):
    if x_api_key != "my-secret-key":
        raise HTTPException(status_code=403, detail="无效 API Key")

@app.get("/open-api/", dependencies=[Depends(verify_api_key)])
async def open_api():
    return {"data": "受 API Key 保护的数据"}
```

***

## 八、数据库集成 🗄️

### 8.1 SQLAlchemy 2.0 + FastAPI（同步版）

```bash
pip install sqlalchemy
```

**database.py**：

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

DATABASE_URL = "sqlite:///./app.db"
# MySQL: "mysql+pymysql://user:pass@localhost:3306/mydb"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},   # SQLite 专用
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass
```

**models.py**：

```python
from datetime import datetime
from sqlalchemy import String, Integer, Boolean, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from database import Base

class Todo(Base):
    __tablename__ = "todos"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    done: Mapped[bool] = mapped_column(Boolean, default=False)
    created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())
```

**依赖注入数据库会话（标准模板）**：

```python
from sqlalchemy.orm import Session
from database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db            # 视图函数使用 db
    finally:
        db.close()          # 自动关闭（即使出错）
```

**视图中使用**：

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import Annotated

app = FastAPI()
DbDep = Annotated[Session, Depends(get_db)]     # 统一别名

@app.get("/todos/")
async def list_todos(db: DbDep, skip: int = 0, limit: int = 100):
    return db.query(Todo).offset(skip).limit(limit).all()

@app.post("/todos/", status_code=201)
async def create_todo(title: str, db: DbDep):
    todo = Todo(title=title)
    db.add(todo)
    db.commit()
    db.refresh(todo)         # 重新加载数据（拿到 id）
    return todo

@app.patch("/todos/{todo_id}")
async def toggle_todo(todo_id: int, db: DbDep):
    todo = db.query(Todo).filter(Todo.id == todo_id).first()
    if not todo:
        raise HTTPException(status_code=404, detail="待办不存在")
    todo.done = not todo.done
    db.commit()
    db.refresh(todo)
    return todo
```

### 8.2 启动时建表

```python
import database, models

@app.on_event("startup")           # 应用启动时执行（新版本推荐 lifespan）
def startup():
    models.Base.metadata.create_all(bind=database.engine)
```

**lifespan 写法（官方新推荐）**：

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动前执行
    models.Base.metadata.create_all(bind=engine)
    yield
    # 关闭后执行（清理资源）
    engine.dispose()

app = FastAPI(lifespan=lifespan)
```

### 8.3 异步数据库（性能更高）

```bash
pip install sqlalchemy asyncpg    # PostgreSQL 用 asyncpg
# pip install aiosqlite           # SQLite 用这个
```

```python
# async_database.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/mydb"
# SQLite 异步："sqlite+aiosqlite:///./app.db"

async_engine = create_async_engine(DATABASE_URL)
AsyncSessionLocal = async_sessionmaker(async_engine, expire_on_commit=False)

async def get_async_db():
    async with AsyncSessionLocal() as session:
        yield session
```

```python
# 视图（全异步链路）
from sqlalchemy import select

@app.get("/todos/")
async def list_todos(db: AsyncSessionDep):
    result = await db.execute(select(Todo))
    return result.scalars().all()
```

```mermaid
graph TD
    A["同步链路"] --> B["def 视图<br/>+ 同步 SQLAlchemy"]
    B --> C["跑在线程池<br/>简单但占资源"]

    D["异步链路（推荐）"] --> E["async def 视图<br/>+ asyncpg"]
    E --> F["单线程高并发<br/>资源占用低 ⚡"]

    style C fill:#fff9c4
    style F fill:#c8e6c9
```

***

## 九、异步编程 ⚡

### 9.1 async def vs def

```python
import asyncio
from fastapi import FastAPI

app = FastAPI()

# —— async def：跑在事件循环（要用 await / 异步库）——
@app.get("/async-route/")
async def async_route():
    await asyncio.sleep(1)            # 异步等待（不阻塞）
    return {"message": "async"}

# —— def：自动放到线程池（同步库用这个）——
@app.get("/sync-route/")
def sync_route():
    import time
    time.sleep(1)                     # 阻塞等待（线程池里无所谓）
    return {"message": "sync"}
```

> ⚠️ **黄金法则**：
>
> - 用 `async def` + **异步库**（httpx、asyncpg）
> - 用 `def` + **同步库**（requests、pymysql）
> - ❌ 最差：`async def` 里面调**同步阻塞库**——阻塞整个事件循环，并发全毁！

```mermaid
graph TD
    A["选择视图类型"] --> B{"用的库是<br/>异步的吗？"}
    B -->|"是（httpx 等）"| C["async def + await"]
    B -->|"否（requests 等）"| D["普通 def<br/>（自动进线程池）"]

    C --> E["✅ 最佳性能"]
    D --> F["✅ 安全正确"]

    G["❌ async def 里<br/>调 requests"] --> H["💥 阻塞事件循环<br/>并发全毁！"]

    style E fill:#c8e6c9
    style F fill:#c8e6c9
    style H fill:#ffcdd2
```

### 9.2 并发执行多个任务

```python
import asyncio
import httpx

async def fetch_url(client, url):
    resp = await client.get(url)
    return {"url": url, "status": resp.status_code}

@app.get("/aggregate/")
async def aggregate():
    urls = ["https://httpbin.org/delay/1"] * 5

    # —— asyncio.gather：并发执行（总耗时 ≈ 最长的那个）——
    async with httpx.AsyncClient() as client:
        results = await asyncio.gather(*[fetch_url(client, u) for u in urls])

    return {"results": results, "note": "5 个请求只花 ~1 秒"}
```

***

## 十、后台任务与 WebSocket 📨

### 10.1 BackgroundTasks：请求后做杂事

> 发邮件、写日志、推送通知……响应先返回，杂事后台做，用户体验好！

```python
from fastapi import BackgroundTasks, FastAPI
import time

app = FastAPI()

def send_email(to: str, subject: str):
    """模拟发邮件（耗时 3 秒）"""
    time.sleep(3)
    print(f"邮件已发送给 {to}：{subject}")

def write_log(message: str):
    with open("app.log", "a", encoding="utf-8") as f:
        f.write(message + "\n")

@app.post("/register/")
async def register(email: str, background_tasks: BackgroundTasks):
    # —— 注册逻辑（秒回）——
    write_log(f"新用户注册：{email}")

    # —— 后台任务：响应返回后再执行 ——
    background_tasks.add_task(send_email, email, "欢迎注册")

    return {"message": "注册成功！欢迎邮件稍后送达～"}   # 立刻响应！
```

```mermaid
sequenceDiagram
    participant C as 客户端
    participant F as FastAPI
    participant B as 后台任务

    C->>F: POST /register
    F->>F: 写日志
    F->>B: 注册后台任务 send_email
    F-->>C: 立刻返回"注册成功"（不等邮件）
    Note over B: 后台慢慢发邮件（3秒）
```

### 10.2 WebSocket 实时通信

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

class ConnectionManager:
    """管理所有 WebSocket 连接"""
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def broadcast(self, message: str):
        """群发消息"""
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{username}")
async def chat(websocket: WebSocket, username: str):
    await manager.connect(websocket)
    await manager.broadcast(f"📢 {username} 加入了聊天室")
    try:
        while True:
            data = await websocket.receive_text()          # 等客户端消息
            await manager.broadcast(f"💬 {username}: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"👋 {username} 离开了")
```

**前端测试代码**：

```html
<script>
    const ws = new WebSocket("ws://127.0.0.1:8000/ws/小明");
    ws.onmessage = (e) => console.log("收到:", e.data);
    ws.onopen = () => ws.send("大家好！");
</script>
```

***

## 十一、中间件与 CORS 🧅

### 11.1 自定义中间件

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    # —— 请求前 ——
    start = time.time()

    response = await call_next(request)       # 执行后续处理

    # —— 响应后 ——
    duration = time.time() - start
    response.headers["X-Process-Time"] = f"{duration:.4f}"
    return response
```

### 11.2 CORS：跨域资源共享（前后端联调必配！）

> 前端 localhost:3000 请求后端 localhost:8000，浏览器会拦截——这就是跨域问题！

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",      # Vue/React 开发服务器
        "https://myfrontend.com",     # 生产前端地址
    ],
    allow_credentials=True,           # 允许带 Cookie
    allow_methods=["*"],              # 允许所有方法
    allow_headers=["*"],              # 允许所有请求头
)
```

```mermaid
graph LR
    A["前端 :3000"] -->|"请求 API"| B["后端 :8000"]
    B -->|"无 CORS 配置"| C["❌ 浏览器拦截<br/>CORS 错误"]
    B -->|"有 CORS 配置"| D["✅ 响应带<br/>Access-Control-Allow-Origin"]

    style C fill:#ffcdd2
    style D fill:#c8e6c9
```

### 11.3 常用第三方中间件

```python
from fastapi.middleware.gzip import GZipMiddleware

# Gzip 压缩（响应体积变小，加载更快）
app.add_middleware(GZipMiddleware, minimum_size=1000)

# HTTPS 重定向
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
app.add_middleware(HTTPSRedirectMiddleware)

# TrustedHost（防 Host 头攻击）
from fastapi.middleware.trustedhost import TrustedHostMiddleware
app.add_middleware(TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"])
```

***

## 十二、异常处理 🚨

### 12.1 HTTPException：主动抛错

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()
fake_db = {1: "苹果"}

@app.get("/items/{item_id}")
async def get_item(item_id: int):
    if item_id not in fake_db:
        raise HTTPException(
            status_code=404,
            detail="商品不存在",          # 会放进响应的 detail 字段
            headers={"X-Error": "NotFound"},   # 可选：自定义头
        )
    return {"item": fake_db[item_id]}
```

### 12.2 自定义异常 + 全局处理器

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# —— 自定义业务异常 ——
class BusinessError(Exception):
    def __init__(self, code: int, message: str):
        self.code = code
        self.message = message

# —— 全局异常处理器 ——
@app.exception_handler(BusinessError)
async def business_error_handler(request: Request, exc: BusinessError):
    return JSONResponse(
        status_code=400,
        content={"code": exc.code, "msg": exc.message, "data": None},
    )

# —— 兜底：捕获所有未处理异常 ——
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"code": 500, "msg": "服务器开小差了，请稍后再试～"},
    )

# —— 使用 ——
@app.post("/orders/")
async def create_order(amount: float):
    if amount <= 0:
        raise BusinessError(code=40001, message="订单金额必须大于 0")
    return {"order_id": "ORD-001"}
```

```mermaid
graph TD
    A["视图抛异常"] --> B{"什么类型？"}
    B -->|"HTTPException"| C["FastAPI 内置处理<br/>返回 {detail: ...}"]
    B -->|"BusinessError"| D["自定义处理器<br/>返回 {code, msg, data}"]
    B -->|"其他 Exception"| E["全局兜底<br/>返回友好提示"]

    style C fill:#fff9c4
    style D fill:#c8e6c9
    style E fill:#ffcdd2
```

### 12.3 统一响应格式（企业实践）

```python
from pydantic import BaseModel
from typing import Any, Generic, TypeVar

T = TypeVar("T")

class ApiResponse(BaseModel, Generic[T]):
    """统一响应包装"""
    code: int = 0
    msg: str = "success"
    data: T | None = None

@app.get("/users/", response_model=ApiResponse[list[str]])
async def list_users():
    return ApiResponse(data=["小明", "小红"])

# 响应：{"code": 0, "msg": "success", "data": ["小明", "小红"]}
```

***

## 十三、项目结构与配置 🏗️

### 13.1 标准项目结构（APIRouter 分模块）

```text
my_project/
├── app/
│   ├── __init__.py
│   ├── main.py                ← 应用入口（创建 app、挂路由）
│   ├── config.py              ← 配置（Pydantic Settings）
│   ├── database.py            ← 数据库连接
│   ├── models.py              ← SQLAlchemy 模型
│   ├── schemas.py             ← Pydantic 模型
│   ├── deps.py                ← 公共依赖（get_db 等）
│   └── routers/               ← 路由模块
│       ├── __init__.py
│       ├── users.py
│       └── items.py
├── .env                       ← 环境变量（不进 Git！）
├── requirements.txt
└── Dockerfile
```

### 13.2 main.py：组装应用

```python
# app/main.py
from fastapi import FastAPI
from app.routers import users, items
from app.database import engine
from app import models

app = FastAPI(
    title="我的商城 API",
    description="FastAPI 最佳实践项目",
    version="1.0.0",
)

# 挂载路由（自动分组 + 统一前缀）
app.include_router(users.router)
app.include_router(items.router)

@app.get("/", tags=["默认"])
async def root():
    return {"app": "我的商城 API", "docs": "/docs"}
```

### 13.3 routers/users.py：模块化路由

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["用户管理"],           # 文档分组
)

@router.get("/")                 # 实际路径 /users/
async def list_users():
    return [{"id": 1, "name": "小明"}]

@router.get("/{user_id}")        # 实际路径 /users/123
async def get_user(user_id: int):
    return {"id": user_id}
```

### 13.4 配置管理：pydantic-settings

```bash
pip install pydantic-settings
```

**.env**：

```text
DATABASE_URL=postgresql://user:pass@localhost/mydb
SECRET_KEY=super-secret-key-here
DEBUG=true
```

**config.py**：

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """自动从 .env 文件和环境变量读取配置"""
    database_url: str
    secret_key: str
    debug: bool = False

    model_config = {"env_file": ".env"}

settings = Settings()            # 全局唯一实例

# 使用：settings.database_url
```

> 💡 好处：类型安全 + 编辑器提示 + 环境隔离 + 密钥不进代码库！

### 13.5 应用元信息配置

```python
app = FastAPI(
    title="商城 API",
    description="""
    ## 商城 API 文档

    * 用户管理
    * 商品管理
    """,
    version="2.0.0",
    docs_url="/docs",            # 文档地址（设为 None 禁用）
    redoc_url="/redoc",
    openapi_url="/openapi.json",
    contact={"name": "API 支持", "email": "api@example.com"},
    license_info={"name": "MIT"},
)
```

***

## 十四、实战：待办事项 API ✅

> 综合运用：CRUD + Pydantic + 依赖注入 + 统一响应！完整可运行～

### 14.1 项目文件

```text
todo_app/
├── main.py          ← 全部代码（教学用单文件）
└── requirements.txt
```

```text
# requirements.txt
fastapi
uvicorn[standard]
sqlalchemy
```

### 14.2 完整代码

```python
# main.py
from contextlib import asynccontextmanager
from datetime import datetime
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, Query, status
from pydantic import BaseModel, Field
from sqlalchemy import (
    create_engine, String, Boolean, DateTime, func,
    select,
)
from sqlalchemy.orm import (
    DeclarativeBase, Mapped, mapped_column,
    Session, sessionmaker,
)

# ============ 数据库配置 ============
DATABASE_URL = "sqlite:///./todos.db"

engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(bind=engine, autoflush=False)

class Base(DeclarativeBase):
    pass

# ============ SQLAlchemy 模型 ============
class TodoModel(Base):
    __tablename__ = "todos"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100))
    done: Mapped[bool] = mapped_column(default=False)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

# ============ Pydantic 模型 ============
class TodoCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=100, examples=["买牛奶"])

class TodoUpdate(BaseModel):
    title: str | None = Field(None, min_length=1)
    done: bool | None = None

class TodoOut(BaseModel):
    id: int
    title: str
    done: bool
    created_at: datetime

    model_config = {"from_attributes": True}    # 支持 ORM 对象转换

# ============ 依赖 ============
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

DbDep = Annotated[Session, Depends(get_db)]

# ============ 生命周期 ============
@asynccontextmanager
async def lifespan(app: FastAPI):
    Base.metadata.create_all(bind=engine)       # 启动建表
    yield
    engine.dispose()                            # 关闭连接池

app = FastAPI(title="待办事项 API", lifespan=lifespan)

# ============ 接口 ============
@app.get("/todos/", response_model=list[TodoOut], tags=["待办"])
async def list_todos(
    db: DbDep,
    done: bool | None = Query(None, description="按状态筛选"),
    page: int = Query(1, ge=1),
    size: int = Query(10, ge=1, le=50),
):
    stmt = select(TodoModel)
    if done is not None:
        stmt = stmt.where(TodoModel.done == done)
    stmt = stmt.offset((page - 1) * size).limit(size)
    return db.scalars(stmt).all()

@app.get("/todos/{todo_id}", response_model=TodoOut, tags=["待办"])
async def get_todo(todo_id: int, db: DbDep):
    todo = db.get(TodoModel, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail="待办不存在")
    return todo

@app.post("/todos/", response_model=TodoOut,
          status_code=status.HTTP_201_CREATED, tags=["待办"])
async def create_todo(data: TodoCreate, db: DbDep):
    todo = TodoModel(title=data.title)
    db.add(todo)
    db.commit()
    db.refresh(todo)
    return todo

@app.patch("/todos/{todo_id}", response_model=TodoOut, tags=["待办"])
async def update_todo(todo_id: int, data: TodoUpdate, db: DbDep):
    todo = db.get(TodoModel, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail="待办不存在")

    if data.title is not None:                  # 只更新传了的字段
        todo.title = data.title
    if data.done is not None:
        todo.done = data.done

    db.commit()
    db.refresh(todo)
    return todo

@app.delete("/todos/{todo_id}", status_code=status.HTTP_204_NO_CONTENT,
            tags=["待办"])
async def delete_todo(todo_id: int, db: DbDep):
    todo = db.get(TodoModel, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail="待办不存在")
    db.delete(todo)
    db.commit()
    # 204 无内容返回
```

```bash
uvicorn main:app --reload
# 打开 http://127.0.0.1:8000/docs 开始测试！
```

### 14.3 API 一览

| 方法     | 路径            | 功能        | 状态码 |
| :----- | :------------ | :-------- | :-: |
| GET    | `/todos/`     | 列表（分页+筛选） | 200 |
| GET    | `/todos/{id}` | 详情        | 200 |
| POST   | `/todos/`     | 创建        | 201 |
| PATCH  | `/todos/{id}` | 部分更新      | 200 |
| DELETE | `/todos/{id}` | 删除        | 204 |

```mermaid
graph TD
    A["请求到达"] --> B["Pydantic 验证"]
    B -->|"❌ 不合法"| C["422 错误详情"]
    B -->|"✅ 合法"| D["Depends(get_db)"]
    D --> E["业务逻辑"]
    E -->|"成功"| F["response_model 过滤"]
    F --> G["JSON 响应 ✅"]
    E -->|"失败"| H["HTTPException"]
    H --> I["404/400 错误"]
    D --> J["finally 关闭连接"]

    style G fill:#c8e6c9
    style C fill:#ffcdd2
    style I fill:#ffcdd2
```

***

## 十五、部署上线 🚀

### 15.1 Uvicorn 单进程（开发）

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### 15.2 Uvicorn 多进程（生产）

```bash
# workers = CPU 核心数（或 2×核心+1）
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4

# 不加 --reload（生产禁用！）
```

### 15.3 Gunicorn + Uvicorn Worker（推荐）

```bash
pip install "gunicorn" "uvicorn[standard]"

gunicorn main:app \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \   # 关键：用 uvicorn worker！
    -b 0.0.0.0:8000 \
    --timeout 60
```

> 💡 Gunicorn 管理进程（崩溃自动重启、平滑升级），Uvicorn 处理异步请求——强强联合！

### 15.4 Nginx 反向代理

```nginx
server {
    listen 80;
    server_name api.example.com;

    client_max_body_size 10M;        # 上传文件大小限制

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

```mermaid
graph LR
    A["客户端"] -->|"https"| B["Nginx"]
    B --> C["Gunicorn 管理者"]
    C --> D["Uvicorn Worker 1"]
    C --> E["Uvicorn Worker 2"]
    C --> F["Uvicorn Worker 3"]
    C --> G["Uvicorn Worker 4"]
    D & E & F & G --> H["FastAPI 应用"]

    style B fill:#fff9c4
    style C fill:#bbdefb
```

### 15.5 Docker 部署

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# 非 root 用户
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

```bash
docker build -t myapi .
docker run -d -p 8000:8000 --name myapi-container myapi
```

### 15.6 部署检查清单

| 检查项                | 说明                                      |
| :----------------- | :-------------------------------------- |
| ✅ 去掉 --reload      | 生产环境绝不用                                 |
| ✅ SECRET\_KEY 环境变量 | 不能硬编码                                   |
| ✅ CORS 白名单         | 不要 allow\_origins=\["\*"] + credentials |
| ✅ HTTPS            | Nginx 配 SSL 证书                          |
| ✅ 数据库连接池           | asyncpg/psycopg 池化                      |
| ✅ 多 Worker         | 利用多核 CPU                                |
| ✅ 日志收集             | --log-config 或 sentry                   |

***

## 十六、知识总结 📌

### FastAPI 核心知识图

```mermaid
mindmap
  root(("FastAPI 核心"))
    参数处理
      路径参数 Path
      查询参数 Query
      请求体 Pydantic
      Header Cookie Form File
    数据验证
      Field 校验
      field_validator
      嵌套模型
      response_model 白名单
    依赖注入
      Depends 基础
      yield 善后
      嵌套依赖
      APIRouter 批量依赖
    安全认证
      OAuth2 密码模式
      JWT 签发验证
      get_current_user
      API Key
    异步编程
      async def vs def
      异步库选择
      asyncio gather
      WebSocket
    工程化
      APIRouter 模块化
      pydantic-settings
      统一异常处理
      CORS 中间件
    部署
      Uvicorn workers
      Gunicorn + Uvicorn
      Nginx
      Docker
```

### 高频写法速查表

| 需求     | 写法                                            |
| :----- | :-------------------------------------------- |
| 路径参数   | `def f(item_id: int)`                         |
| 可选查询参数 | `q: str \| None = None`                       |
| 请求体    | `def f(item: ItemModel)`                      |
| 文件上传   | `file: UploadFile`                            |
| 数据库会话  | `db: Annotated[Session, Depends(get_db)]`     |
| 响应过滤   | `@app.get(..., response_model=UserOut)`       |
| 状态码    | `@app.post(..., status_code=201)`             |
| 抛 404  | `raise HTTPException(404, detail="xx")`       |
| 后台任务   | `background_tasks.add_task(func, args)`       |
| 模块路由   | `APIRouter(prefix="/users", tags=["xx"])`     |
| 统一依赖   | `DbDep = Annotated[Session, Depends(get_db)]` |

### 三大框架终极对比

| 对比项  | FastAPI ⚡       | Flask 🌸      | Django 🏗️         |
| :--- | :-------------- | :------------ | :----------------- |
| 定位   | 现代 API 框架       | 微框架           | 全栈全家桶              |
| 核心优势 | 自动文档+类型+性能      | 简单灵活          | Admin+生态           |
| 数据验证 | Pydantic 内置     | 手动/扩展         | Forms              |
| 异步   | 原生              | 部分            | 部分                 |
| 部署   | Uvicorn/ASGI    | Gunicorn/WSGI | Gunicorn/WSGI+ASGI |
| 项目建议 | 前后端分离、微服务、ML 服务 | 小项目、原型        | 内容网站、企业应用          |

### 学习路线建议

```mermaid
graph LR
    A["① Hello + /docs<br/>体验自动文档"] --> B["② 三种参数<br/>+ Pydantic"]
    B --> C["③ response_model<br/>+ 状态码"]
    C --> D["④ 依赖注入<br/>Depends"]
    D --> E["⑤ JWT 认证<br/>登录系统"]
    E --> F["⑥ SQLAlchemy<br/>数据库集成"]
    F --> G["⑦ APIRouter<br/>模块化重构"]
    G --> H["⑧ Docker+Nginx<br/>生产部署"]

    style A fill:#c8e6c9
    style H fill:#fff9c4
```

> 🎀 呼～FastAPI 全部精髓都打包送给你啦！🎉 类型注解的魔法、依赖注入的优雅、JWT 认证的完整实现、还有实战 Todo API——你现在可以独立搭建生产级的现代 API 服务了！配合 [Flask.md](file:///d:/A-H2H/Python/Flask.md)、[Django.md](file:///d:/A-H2H/Python/Django.md) 和 [PythonWeb.md](file:///d:/A-H2H/Python/PythonWeb.md) 对比学习，框架理解会突飞猛进～快去 /docs 页面试试"Try it out"吧！(◍•ᴗ•◍)❤

