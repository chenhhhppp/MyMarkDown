# 🌐 Requests 库详解

> 哈喽呀～今天的主角是 Python 最流行的 HTTP 库 requests！它被官方称为"给人类用的 HTTP 库"（HTTP for Humans）——让你发请求就像说人话一样简单！不管是爬虫、调接口还是做自动化测试，它都是必备神器！冲鸭！(≧∇≦)ﾉ

***

## 🗺️ 知识全景图

```mermaid
mindmap
  root((Requests 库))
    基础请求
      GET 获取数据
      POST 提交数据
      PUT 更新
      DELETE 删除
      HEAD OPTIONS
    参数传递
      params 查询参数
      data 表单数据
      json JSON 数据
      files 文件上传
    请求头与Cookie
      headers 伪装
      cookies 携带
      Session 会话保持
    响应处理
      text 文本
      content 二进制
      json 方法
      encoding 编码
      status_code 状态码
    高级用法
      timeout 超时
      proxies 代理
      verify SSL 验证
      auth 认证
      重试机制
    流式与钩子
      stream 流式下载
      hooks 钩子
      PreparedRequest 预备请求
    底层原理
      urllib3 连接池
      Session 复用连接
      适配器 Adapter
```

***

## 📑 目录

- [一、Requests 是什么](#一requests-是什么)
- [二、安装与初体验](#二安装与初体验)
- [三、HTTP 方法全家桶](#三http-方法全家桶)
- [四、请求参数详解](#四请求参数详解)
- [五、响应对象 Response 全解](#五响应对象-response-全解)
- [六、请求头与 Cookie](#六请求头与-cookie)
- [七、Session：会话保持神器](#七session会话保持神器)
- [八、超时与异常处理](#八超时与异常处理)
- [九、代理与 SSL](#九代理与-ssl)
- [十、认证与文件上传](#十认证与文件上传)
- [十一、高级玩法](#十一高级玩法)
- [十二、底层原理小探](#十二底层原理小探)
- [十三、最佳实践清单](#十三最佳实践清单)

***

## 一、Requests 是什么

### 1.1 一句话解释

**Requests = 一个优雅简单的 Python HTTP 请求库** 💌

它把复杂的 HTTP 协议包装成简洁的 Python 接口。用 urllib 发请求要写 5 行，requests 只要 1 行！

### 1.2 对比见真章

```python
# 😫 urllib 的写法（又臭又长）
from urllib.request import urlopen, Request
from urllib.parse import urlencode

req = Request("https://httpbin.org/get?" + urlencode({"page": 1}),
              headers={"User-Agent": "Mozilla/5.0"})
resp = urlopen(req, timeout=5)
text = resp.read().decode("utf-8")

# 🥰 requests 的写法（优雅！）
import requests
resp = requests.get("https://httpbin.org/get",
                    params={"page": 1},
                    headers={"User-Agent": "Mozilla/5.0"},
                    timeout=5)
text = resp.text
```

### 1.3 一次请求的旅程

```mermaid
flowchart LR
    A[你的代码<br>requests.get] --> B[构建 Request 对象]
    B --> C[urllib3 连接池<br>建立连接]
    C -->|HTTP 报文| D[🌐 服务器]
    D -->|响应报文| C
    C --> E[构建 Response 对象]
    E --> F[你的代码拿到结果<br>resp.text]
```

***

## 二、安装与初体验

### 2.1 安装

```bash
pip install requests

# 查看版本
pip show requests
```

### 2.2 第一个请求

```python
import requests

resp = requests.get("https://httpbin.org/get")
print(resp.status_code)   # 200
print(resp.text)          # 响应内容
```

> 💡 推荐用 `httpbin.org` 练手，它会把你发的请求"原样弹回来"，方便观察请求长啥样！

### 2.3 requests 能干什么

| 用途        | 场景举例 🌰          |
| --------- | ---------------- |
| 爬虫        | 抓取网页数据           |
| 接口测试      | 测试后端 API         |
| 调用第三方 API | 微信机器人、AI 接口、天气查询 |
| 文件下载      | 下载图片、视频、安装包      |
| 自动化办公     | 批量提交表单、打卡        |

***

## 三、HTTP 方法全家桶 🚗

### 3.1 七个方法一览

```python
import requests

requests.get("https://httpbin.org/get")      # 获取数据（最常用！）
requests.post("https://httpbin.org/post")    # 提交数据（第二常用）
requests.put("https://httpbin.org/put")      # 全量更新
requests.patch("https://httpbin.org/patch")  # 部分更新
requests.delete("https://httpbin.org/delete")# 删除资源
requests.head("https://httpbin.org/get")     # 只要响应头
requests.options("https://httpbin.org/get")  # 查询支持的方法
```

### 3.2 GET vs POST 使用场景

```mermaid
flowchart TD
    A[要发请求] --> B{要给服务器提交数据吗?}
    B -->|不用 只拿数据| C[GET<br>参数放 URL 里]
    B -->|要提交<br>登录 注册 上传| D[POST<br>数据放请求体里]
    C --> E[翻页 搜索 详情页]
    D --> F[登录 发帖 上传文件]
```

### 3.3 通用参数模板

所有方法都支持这些关键字参数，一通百通：

```python
resp = requests.get(
    url,                      # 目标网址（必填）
    params=None,              # URL 查询参数（字典）
    headers=None,             # 请求头（字典）
    cookies=None,             # Cookie（字典或 Jar）
    timeout=None,             # 超时秒数（强烈建议写！）
    proxies=None,             # 代理 IP
    verify=True,              # 是否验证 SSL 证书
    allow_redirects=True,     # 是否跟随重定向
    stream=False,             # 是否流式下载
)
```

***

## 四、请求参数详解 📦

### 4.1 params：URL 查询参数

```python
# 这三种写法效果一样！
requests.get("https://httpbin.org/get?page=1&size=20")

requests.get("https://httpbin.org/get", params={"page": 1, "size": 20})

requests.get("https://httpbin.org/get?page=1", params={"size": 20})  # 会合并

# 列表参数（重复 key）
requests.get("https://httpbin.org/get", params={"tag": ["python", "爬虫"]})
# 实际 URL: ?tag=python&tag=爬虫
```

> 💡 强烈推荐用 `params` 传参！requests 会自动做 URL 编码，中文和特殊字符都不会出问题～

### 4.2 data：提交表单数据

```python
# 表单格式（Content-Type: application/x-www-form-urlencoded）
resp = requests.post("https://httpbin.org/post",
                     data={"username": "小美", "password": "123456"})

# 还可以传元组列表（同 key 多值）
resp = requests.post("https://httpbin.org/post",
                     data=[("key1", "value1"), ("key1", "value2")])

# 传原始字符串（极少用）
resp = requests.post("https://httpbin.org/post", data="这是原始数据")
```

### 4.3 json：提交 JSON 数据（现代 API 首选！）

```python
# 方式一：json 参数（推荐！自动序列化 + 自动设置 Content-Type）
resp = requests.post("https://httpbin.org/post",
                     json={"title": "笔记", "tags": ["a", "b"]})

# 方式二：手动 dumps（效果一样，但要自己设置请求头）
import json
resp = requests.post("https://httpbin.org/post",
                     data=json.dumps({"title": "笔记"}),
                     headers={"Content-Type": "application/json"})
```

### 4.4 data vs json 怎么选

| 参数     | Content-Type     | 数据格式            | 什么时候用         |
| ------ | ---------------- | --------------- | ------------- |
| `data` | form-urlencoded  | `a=1&b=2`       | 传统网站表单、老接口    |
| `json` | application/json | `{"a":1,"b":2}` | 现代 REST API ⭐ |

> 💡 拿不准就 F12 抓包看请求的 Content-Type，服务器要哪种就给哪种！

### 4.5 files：上传文件

```python
# 上传单个文件
files = {"file": open("report.pdf", "rb")}
resp = requests.post("https://httpbin.org/post", files=files)

# 指定文件名和类型（更规范）
files = {
    "file": (
        "报告.pdf",                              # 文件名
        open("report.pdf", "rb"),                # 文件对象
        "application/pdf",                       # MIME 类型
    )
}
resp = requests.post("https://httpbin.org/post", files=files)

# 上传多个文件
files = [
    ("file1", open("a.png", "rb")),
    ("file2", open("b.png", "rb")),
]
```

***

## 五、响应对象 Response 全解 📬

> 每次请求都会返回一个 Response 对象，它就是服务器给你的"回信"，里面全是宝贝！

### 5.1 常用属性速查表

| 属性/方法         | 类型        | 说明         | 示例                             |
| ------------- | --------- | ---------- | ------------------------------ |
| `status_code` | int       | HTTP 状态码   | `200`                          |
| `text`        | str       | 响应文本（自动解码） | HTML 源码                        |
| `content`     | bytes     | 二进制内容      | `b'\x89PNG...'`                |
| `json()`      | dict/list | 解析 JSON 响应 | `{"code": 0}`                  |
| `headers`     | dict      | 响应头        | `resp.headers["Content-Type"]` |
| `cookies`     | Jar       | 响应 Cookie  | `resp.cookies["key"]`          |
| `url`         | str       | 最终请求的 URL  | 重定向后的地址                        |
| `encoding`    | str       | 文本编码       | `utf-8`                        |
| `history`     | list      | 重定向历史      | `[301, 302...]`                |
| `ok`          | bool      | 状态码是否 2xx  | `True`                         |
| `elapsed`     | timedelta | 请求耗时       | 性能测试用                          |
| `reason`      | str       | 状态描述文字     | `OK` / `Not Found`             |

### 5.2 三种取内容的方式

```python
resp = requests.get("https://httpbin.org/get")

# 1. 文本内容（网页、普通文本）
print(resp.text)

# 2. 二进制内容（图片、视频、压缩包）
with open("img.png", "wb") as f:
    f.write(resp.content)

# 3. JSON 内容（调 API 必备！）
resp = requests.get("https://httpbin.org/get")
data = resp.json()          # 直接得到字典
print(data["headers"]["User-Agent"])
```

```mermaid
flowchart TD
    A[拿到 Response] --> B{内容是什么类型?}
    B -->|网页 HTML| C[resp.text]
    B -->|图片 视频 文件| D[resp.content]
    B -->|API 返回 JSON| E[resp.json]
    B -->|只要看信息| F[status_code headers cookies]
```

### 5.3 状态码判断

```python
resp = requests.get("https://httpbin.org/get")

# 方式一：直接比较
if resp.status_code == 200:
    print("成功！")

# 方式二：内置常量（可读性更好）
if resp.status_code == requests.codes.ok:
    print("成功！")

# 方式三：ok 属性（2xx 都是 True）
if resp.ok:
    print("成功！")

# 方式四：raise_for_status（异常处理最爱）
resp.raise_for_status()   # 非 2xx 直接抛 HTTPError 异常
```

### 5.4 中文乱码问题

```python
resp = requests.get("https://example.com")
print(resp.text)   # 乱码？😱

# 原理：requests 根据 HTTP 头猜编码，猜错了就乱码
# 解决：手动指定编码，重新访问 text
resp.encoding = "gbk"        # 或 "utf-8"，看网页 meta charset
print(resp.text)             # 正常了！✨

# 万能方案：让 requests 用 headers 里声明的编码（Python 3.9+ 支持更好）
resp.encoding = resp.apparent_encoding   # 用 chardet 检测真实编码
```

### 5.5 响应头与 Cookie

```python
# 响应头（大小写不敏感的字典）
print(resp.headers["Content-Type"])    # application/json
print(resp.headers.get("Server"))      # 不存在返回 None 不报错

# 响应里的 Cookie
print(resp.cookies["session_id"])      # 像 dict 一样取
print(dict(resp.cookies))              # 一次全拿
```

***

## 六、请求头与 Cookie 🪪

### 6.1 伪装请求头（爬虫必修课）

```python
headers = {
    # 浏览器身份（最关键！不带就像"裸奔"）
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    # 从哪个页面来的（防盗链）
    "Referer": "https://www.example.com/",
    # 接受的内容类型
    "Accept": "application/json, text/plain, */*",
    # 接受的语言
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
}
resp = requests.get("https://httpbin.org/get", headers=headers)
```

> 💡 实用技巧：打开浏览器 F12 → Network → 找到请求 → 右侧 Request Headers 全选复制，直接贴到代码里，服务器还以为你就是真浏览器！

### 6.2 携带 Cookie 的三种方式

```python
# 方式一：放进 headers（简单粗暴）
headers = {"Cookie": "session_id=abc123; token=xyz789"}
requests.get(url, headers=headers)

# 方式二：用 cookies 参数（更清晰）
cookies = {"session_id": "abc123", "token": "xyz789"}
requests.get(url, cookies=cookies)

# 方式三：从浏览器复制完整 Cookie 字符串
cookie_str = "复制来的长串"
headers = {"Cookie": cookie_str}
```

### 6.3 Cookie 的过期问题

```mermaid
flowchart LR
    A[登录成功] --> B[服务器发 Cookie<br>有效期几小时到几天]
    B --> C[Cookie 过期]
    C --> D{怎么处理?}
    D -->|手动| E[重新从浏览器复制]
    D -->|自动| F[程序模拟登录接口<br>拿新 Cookie]
    F --> G[存下来循环使用 ⭐]
```

***

## 七、Session：会话保持神器 🔑

### 7.1 为什么需要 Session

普通请求每次都是"新面孔"——即使服务器发了 Cookie，下次请求也不会自动带上！

```mermaid
flowchart TD
    subgraph S1["普通请求 😵"]
        A1["请求1 登录"] --> B1["服务器发 Cookie"]
        B1 --> C1["请求2 查数据<br>没带 Cookie 被拒!"]
    end
    subgraph S2["Session 会话 😍"]
        A2["session.post 登录"] --> B2["Cookie 自动存进 session"]
        B2 --> C2["session.get 查数据<br>自动带上 Cookie 成功!"]
    end
```

### 7.2 基本用法

```python
import requests

session = requests.Session()

# 登录：服务器返回的 Set-Cookie 会自动保存
session.post("https://example.com/login",
             data={"username": "xiaomei", "password": "123456"})

# 后续请求自动携带 Cookie，就像已登录状态
profile = session.get("https://example.com/profile")
orders = session.get("https://example.com/orders")
```

### 7.3 Session 的隐藏福利：连接复用

```mermaid
flowchart LR
    subgraph S1["requests.get x 100 😢"]
        A1["建连接"] --> B1["断开"] --> C1["建连接"] --> D1["断开"] --> E1["..."]
    end
    subgraph S2["session.get x 100 🚀"]
        A2["建连接一次"] --> B2["复用"] --> C2["复用"] --> D2["复用..."]
    end
```

> 💡 大量请求同一网站时，Session 能**复用 TCP 连接**，速度快一倍不止！爬虫、压测必备！

### 7.4 Session 进阶配置

```python
session = requests.Session()

# 全局默认值：之后所有请求都生效
session.headers.update({
    "User-Agent": "Mozilla/5.0 ...",
    "Authorization": "Bearer my_token",
})
session.verify = False          # 全局跳过 SSL 验证（慎用）

# 手动往 session 里塞 Cookie
session.cookies.update({"token": "abc123"})

# 更新或删除默认头
session.headers["X-Custom"] = "custom"
del session.headers["X-Custom"]

# 用完记得关（释放连接池）
session.close()

# 推荐用 with 上下文，自动关闭
with requests.Session() as s:
    s.get("https://httpbin.org/get")
```

### 7.5 requests.get vs Session 对比

| 对比项       | requests.get | session.get |
| --------- | ------------ | ----------- |
| Cookie 保持 | ❌ 每次丢失       | ✅ 自动保持      |
| 连接复用      | ❌ 每次新建       | ✅ 复用连接池     |
| 默认配置      | 每次都要传        | 设置一次全局生效    |
| 性能（大量请求）  | 慢            | 快 🚀        |
| 使用建议      | 一次性请求        | 项目中优先用它 ⭐   |

***

## 八、超时与异常处理 ⏰

### 8.1 timeout 详解（必须掌握！）

```python
# 不写 timeout 的后果：服务器没响应，程序永远卡死！☠️

# 写法一：连接和读取共用一个超时
resp = requests.get(url, timeout=5)

# 写法二：分别指定（元组 = (连接超时, 读取超时)）
resp = requests.get(url, timeout=(3, 10))  # 连接3秒 读取10秒
```

```mermaid
flowchart LR
    A[发起请求] --> B{timeout 范围内收到响应?}
    B -->|是| C[✅ 正常返回]
    B -->|连接阶段超时| D[ConnectionTimeout<br>服务器找不到/不响应]
    B -->|读取阶段超时| E[ReadTimeout<br>服务器太慢 数据没传完]
```

### 8.2 异常家族一览

```mermaid
flowchart TD
    A[requests.exceptions.RequestException<br>所有异常的爹]
    A --> B[ConnectionError<br>连接失败 DNS错误 拒绝连接]
    B --> C[ConnectTimeout<br>连接阶段超时]
    B --> D[ReadTimeout<br>读取阶段超时]
    A --> E[HTTPError<br>状态码 4xx 5xx<br>需手动 raise_for_status 触发]
    A --> F[TooManyRedirects<br>重定向太多]
    A --> G[JSONDecodeError<br>resp.json 解析失败]
    A --> H[InvalidURL / MissingSchema<br>URL 格式错误]
```

### 8.3 标准异常处理模板

```python
import requests
from requests.exceptions import (
    Timeout, ConnectionError, HTTPError, RequestException
)

def safe_get(url, try_times=3):
    """带重试的安全请求"""
    for i in range(try_times):
        try:
            resp = requests.get(url, headers=HEADERS, timeout=5)
            resp.raise_for_status()          # 4xx/5xx 抛 HTTPError
            return resp
        except Timeout:
            print(f"第{i+1}次：超时了，重试...")
        except ConnectionError:
            print(f"第{i+1}次：连不上，重试...")
        except HTTPError as e:
            print(f"状态码错误：{e}")
            return None                       # 状态码错误重试也没用
        except RequestException as e:
            print(f"其他异常：{e}")
            return None
    return None                               # 重试用尽
```

### 8.4 自动重试（HTTPAdapter 方案）

```python
from requests.adapters import HTTPAdapter

session = requests.Session()
adapter = HTTPAdapter(max_retries=3)      # 失败自动重试 3 次
session.mount("http://", adapter)
session.mount("https://", adapter)

resp = session.get("https://httpbin.org/get", timeout=5)
```

***

## 九、代理与 SSL 🕵️

### 9.1 使用代理 IP

```python
proxies = {
    "http": "http://127.0.0.1:7890",
    "https": "http://127.0.0.1:7890",
}
resp = requests.get("https://httpbin.org/get",
                    headers=HEADERS, proxies=proxies, timeout=5)

# 带用户名密码的代理
proxies = {
    "http": "http://用户名:密码@127.0.0.1:7890",
}

# socks5 代理（需要 pip install requests[socks]）
proxies = {"http": "socks5://127.0.0.1:1080", "https": "socks5://127.0.0.1:1080"}
```

### 9.2 SSL 证书验证

```python
# 默认 verify=True：校验证书，自签名证书会报 SSLError

# 场景一：公司内网自签名证书 → 跳过验证（会警告）
resp = requests.get("https://internal.example.com", verify=False)

# 场景二：指定自制的 CA 证书（更安全）
resp = requests.get("https://internal.example.com",
                    verify="/path/to/cert.pem")

# 客户端证书（双向认证）
resp = requests.get("https://secure.example.com",
                    cert=("/path/client.cert", "/path/client.key"))
```

> ⚠️ `verify=False` 只是测试时的权宜之计，生产环境请用 `verify=证书路径`，不然数据传输不安全哦！

### 9.3 控制重定向

```python
# 默认 allow_redirects=True：301/302 会自动跳转
# 关闭后可以看到重定向的过程
resp = requests.get("http://github.com", allow_redirects=False)
print(resp.status_code)          # 301
print(resp.headers["Location"])  # https://github.com/

# 查看重定向轨迹
resp = requests.get("http://github.com")
for r in resp.history:
    print(r.status_code, r.url)  # 301 http://github.com/
print(resp.url)                  # https://github.com/
```

***

## 十、认证与文件上传 🔐

### 10.1 基础认证（Basic Auth）

```python
from requests.auth import HTTPBasicAuth

# 方式一：auth 参数
resp = requests.get("https://httpbin.org/basic-auth/user/passwd",
                    auth=HTTPBasicAuth("user", "passwd"))

# 方式二：简写（直接传元组）
resp = requests.get("https://httpbin.org/basic-auth/user/passwd",
                    auth=("user", "passwd"))

# 方式三：塞进 URL（不推荐，密码裸奔在 URL 里）
resp = requests.get("https://user:passwd@httpbin.org/basic-auth/user/passwd")
```

### 10.2 Token / Bearer 认证（现代 API 标配）

```python
# 大部分现代 API 用这种：请求头里放 Token
headers = {"Authorization": "Bearer eyJhbGciOiJIUzI1NiIs..."}
resp = requests.get("https://api.example.com/data", headers=headers)

# 自己封装一个带认证的 session（推荐做法）
session = requests.Session()
session.headers.update({"Authorization": f"Bearer {my_token}"})
```

### 10.3 流式下载大文件

```python
# 😱 错误示范：大文件直接 content，内存直接爆！
# resp = requests.get(big_file_url)
# data = resp.content

# ✅ 正确姿势：stream=True 流式下载，一块一块写盘
with requests.get(big_file_url, stream=True, timeout=10) as resp:
    resp.raise_for_status()
    with open("big_video.mp4", "wb") as f:
        for chunk in resp.iter_content(chunk_size=8192):  # 每次 8KB
            f.write(chunk)

# 带进度条的下载（ tqdm：pip install tqdm ）
from tqdm import tqdm
total = int(resp.headers.get("Content-Length", 0))
with open("file.zip", "wb") as f, tqdm(total=total, unit="B", unit_scale=True) as bar:
    for chunk in resp.iter_content(8192):
        f.write(chunk)
        bar.update(len(chunk))
```

***

## 十一、高级玩法 🎮

### 11.1 准备请求 PreparedRequest（先造后发）

```python
from requests import Request, Session

session = Session()

# 先构建请求（此时还没发送）
req = Request("GET", "https://httpbin.org/get", params={"page": 1})
prepped = req.prepare()

# 可以先检查/修改
print(prepped.url)        # 看看 URL 拼对了没
prepped.headers["X-Custom"] = "hello"

# 再发送
resp = session.send(prepped, timeout=5)
```

### 11.2 hooks：请求事件钩子

```python
# 每次响应都自动执行一个函数
def print_status(resp, *args, **kwargs):
    print(f"[{resp.status_code}] {resp.url}")

resp = requests.get("https://httpbin.org/get",
                    hooks={"response": print_status})
# 输出: [200] https://httpbin.org/get
```

### 11.3 事件钩子实战：自动记录日志

```python
import logging
import requests

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s %(message)s")
logger = logging.getLogger("spider")

def log_response(resp, *args, **kwargs):
    cost = resp.elapsed.total_seconds()
    logger.info(f"{resp.status_code} {resp.url} 耗时{cost:.2f}s")
    return resp

session = requests.Session()
session.hooks["response"] = [log_response]
```

### 11.4 环境变量与配置

```python
# requests 会自动读取这些环境变量（无需在代码里写代理）
# set HTTP_PROXY=http://127.0.0.1:7890
# set HTTPS_PROXY=http://127.0.0.1:7890

# 代码里强制不用环境代理
resp = requests.get(url, proxies={"http": None, "https": None})
# 或 session.trust_env = False
```

### 11.5 漂亮地打印响应（调试神器）

```python
import requests

resp = requests.get("https://httpbin.org/get")

# 打印请求信息（发给服务器的东西）
print(resp.request.headers)     # 请求头
print(resp.request.body)        # 请求体
print(resp.request.url)         # 请求 URL

# 打印响应信息（服务器返回的东西）
print(resp.status_code)
print(resp.headers)
print(resp.text[:500])          # 前 500 字符
```

### 11.6 简易压测小工具

```python
import time
import requests
from concurrent.futures import ThreadPoolExecutor

URL = "https://httpbin.org/get"
HEADERS = {"User-Agent": "Mozilla/5.0"}

def one_request(i):
    start = time.time()
    resp = requests.get(URL, headers=HEADERS, timeout=5)
    return resp.status_code, time.time() - start

with ThreadPoolExecutor(max_workers=20) as pool:
    results = list(pool.map(one_request, range(100)))

ok = sum(1 for code, _ in results if code == 200)
avg = sum(cost for _, cost in results) / len(results)
print(f"成功 {ok}/100，平均耗时 {avg:.3f}s")
```

***

## 十二、底层原理小探 🔬

### 12.1 requests 的架构

```mermaid
flowchart TB
    A[你的代码<br>requests.get] --> B[requests.api<br>便捷函数层]
    B --> C[Session<br>会话管理 连接池]
    C --> D[Adapter 适配器<br>HTTPAdapter]
    D --> E[urllib3<br>真正的网络层]
    E --> F[socket<br>操作系统网络接口]
    F --> G[🌐 互联网]
```

### 12.2 为什么 requests.get 也很快

```python
# 真相：requests.get 内部其实也是创建一次性 Session！
def get(url, **kwargs):
    with sessions.Session() as session:
        return session.request("GET", url, **kwargs)
```

所以大量请求时，自己创建一个长命的 Session 复用，比反复调 `requests.get` 快得多！

### 12.3 请求-响应完整流程

```mermaid
sequenceDiagram
    participant U as 你的代码
    participant S as Session
    participant A as Adapter
    participant N as urllib3
    participant W as 服务器
    U->>S: requests.get url params headers
    S->>S: 合并默认配置<br>组装 Request
    S->>A: send 请求
    A->>N: 从连接池取/建连接
    N->>W: 发送 HTTP 报文
    W-->>N: 返回响应报文
    N-->>A: 原始响应
    A-->>S: Response 对象
    S-->>U: 返回 Response<br>text json status_code
```

***

## 十三、最佳实践清单 ✅

### 13.1 生产级代码模板

```python
import time
import random
import requests
from requests.adapters import HTTPAdapter

def make_session(retries=3):
    """创建一个配置齐全的 Session"""
    session = requests.Session()
    # 1. 全局请求头
    session.headers.update({
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0",
        "Accept-Language": "zh-CN,zh;q=0.9",
    })
    # 2. 自动重试
    adapter = HTTPAdapter(max_retries=retries)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    return session

def fetch(session, url):
    """带超时、异常处理、限速的请求函数"""
    try:
        resp = session.get(url, timeout=(3, 10))
        resp.raise_for_status()
        return resp
    except requests.RequestException as e:
        print(f"请求失败 {url}: {e}")
        return None

if __name__ == "__main__":
    session = make_session()
    for page in range(1, 6):
        resp = fetch(session, f"https://httpbin.org/get?page={page}")
        if resp:
            print(page, resp.json()["args"])
        time.sleep(random.uniform(1, 2))   # 礼貌限速
    session.close()
```

### 13.2 检查清单

| 检查项                  | 说明                | 重要度   |
| -------------------- | ----------------- | ----- |
| 写了 `timeout`         | 防止永久卡死            | ⭐⭐⭐⭐⭐ |
| 用了 `Session`         | 保持 Cookie + 复用连接  | ⭐⭐⭐⭐⭐ |
| 带了 `User-Agent`      | 不被服务器秒拒           | ⭐⭐⭐⭐⭐ |
| `raise_for_status()` | 状态码异常能及时发现        | ⭐⭐⭐⭐  |
| try/except 兜底        | 网络抖动不崩程序          | ⭐⭐⭐⭐  |
| 请求间加了延时              | 对服务器温柔点           | ⭐⭐⭐⭐  |
| 大文件用 `stream`        | 不撑爆内存             | ⭐⭐⭐   |
| JSON 用 `json=` 参数    | 少踩 Content-Type 坑 | ⭐⭐⭐   |
| 日志记录请求               | 出问题能排查            | ⭐⭐⭐   |

### 13.3 常见坑与避坑指南

| 坑 🕳️      | 现象         | 解决方案 💊                                      |
| ---------- | ---------- | -------------------------------------------- |
| 忘写 timeout | 程序卡死不动     | 永远带上 timeout=5                               |
| 中文乱码       | text 输出火星文 | resp.encoding = "utf-8" 或 apparent\_encoding |
| Cookie 不生效 | 登录后仍被拒     | 用 Session 而不是 requests.get                   |
| 提交 JSON 失败 | 服务器说参数错误   | 用 json= 参数替代 data=                           |
| SSLError   | 证书报错       | verify=False（测试）或指定证书                        |
| 429 太多请求   | 被限流        | 加随机延时 + 降低并发                                 |
| 内存爆炸       | 下大文件电脑卡死   | stream=True 流式写盘                             |

***

## 🎓 总结

> 恭喜解锁 requests 全技能！(๑•̀ㅂ•́)و✧ 修炼路线回顾：

```mermaid
flowchart LR
    A[基础请求<br>GET POST] --> B[参数<br>params data json files]
    B --> C[响应处理<br>text content json]
    C --> D[会话<br>Session Cookie]
    D --> E[健壮性<br>timeout 重试 异常]
    E --> F[进阶<br>代理 SSL 流式]
    F --> G[原理<br>Adapter urllib3]
    G --> H[🎉 生产级<br>最佳实践]
```

**核心心法** 📝

- 项目里**默认用 Session**，便捷函数留给小脚本
- **timeout 必写**，异常必抓，重试机制备好
- 接口优先 `json=` 参数，网页注意 `encoding`
- 大文件记得 `stream=True`，并发记得限速

requests 学透了，爬虫和接口自动化就成功一大半啦！下一站可以去探索 httpx（异步版 requests）或 aiohttp，速度更飞快哦～ 🚀💖
