# 🐍 Python 调接口完全指南：从零开始学 HTTP 请求传参

> **面向人群：** Python 新手 / 没有接口调用经验的同学  
> **后端环境：** C#（ASP.NET Core Web API）  
> **核心目标：** 搞懂各种情况下如何把参数正确地传给接口

---

## 📖 目录

1. [准备工作](#0-准备工作)
2. [基础：发一个最简单的请求](#1-基础发一个最简单的请求)
3. [情景A：参数全是简单参数](#2-情景a参数全是简单参数)
4. [情景B：参数是复杂对象](#3-情景b参数是复杂对象json)
5. [情景C：混合参数（简单+复杂）](#4-情景c混合参数简单复杂)
6. [情景D：上传文件](#5-情景d上传文件)
7. [请求头 Headers 详解](#6-必须知道的请求头-headers)
8. [常见错误与解决方法](#7-常见错误与解决方法)
9. [实战综合例子](#8-实战综合例子)
10. [参考资料](#参考资料)

---

## 0. 准备工作

### 安装 requests 库

打开终端（命令行），运行：

```bash
pip install requests
```

安装成功后，在 Python 代码里这样引入：

```python
import requests
```

### 几个必须理解的概念

| 概念 | 大白话解释 |
|------|-----------|
| **HTTP 请求** | 你的程序给服务器"打电话"，说"我要XXX" |
| **接口/API** | 服务器开放的"服务窗口"，每个窗口负责一件事 |
| **GET 请求** | 查询数据（不改变服务器数据） |
| **POST 请求** | 提交/新建数据（会改变服务器数据） |
| **参数** | 你请求时带过去的"附加信息" |
| **响应** | 服务器回给你的"答复" |
| **状态码** | 服务器告诉你请求成不成功的数字（200=成功，400=你传错了，500=服务器自己崩了） |

### 理解"参数放在哪里"

HTTP 请求传参有三个地方可以放：

```
┌─────────────────────────────────────────────────────┐
│  URL 地址栏                                          │
│  https://api.example.com/users?name=张三&age=25     │
│                                ↑这部分叫 Query 参数  │
├─────────────────────────────────────────────────────┤
│  请求头 Headers                                      │
│  Content-Type: application/json                      │
│  Authorization: Bearer xxxxx                         │
├─────────────────────────────────────────────────────┤
│  请求体 Body（只有 POST/PUT 才有）                    │
│  {"name": "张三", "age": 25}   ← JSON 格式           │
│  name=张三&age=25              ← 表单格式             │
└─────────────────────────────────────────────────────┘
```

> 💡 **关键认知：** C# 接口用 `[FromQuery]` 从 URL 取参，用 `[FromBody]` 从 Body 取参。你传错地方，接口就收不到！

---

## 1. 基础：发一个最简单的请求

### 发 GET 请求

```python
import requests

# 发一个最简单的 GET 请求
response = requests.get("https://jsonplaceholder.typicode.com/posts/1")

# 查看状态码（200 = 成功）
print("状态码：", response.status_code)

# 查看返回的内容（文本格式）
print("返回内容：", response.text)

# 如果返回的是 JSON，直接转成 Python 字典
data = response.json()
print("标题：", data["title"])
```

**输出结果：**
```
状态码：200
返回内容：{"userId":1,"id":1,"title":"sunt aut facere...","body":"quia et suscipit..."}
标题：sunt aut facere repellat provident occaecati excepturi optio reprehenderit
```

### response 对象常用属性速查

```python
response = requests.get("https://example.com/api")

response.status_code   # 状态码，200 表示成功
response.text          # 响应内容（字符串）
response.json()        # 响应内容（自动转成 Python 字典）
response.headers       # 响应头
response.content       # 响应内容（字节，用于下载文件）
```

---

## 2. 情景A：参数全是简单参数

> **对应 C# 接口写法：** `[FromQuery]` 或 GET 接口不写任何属性

### 场景举例

C# 接口代码：
```csharp
// 查询用户列表接口
[HttpGet("users")]
public IActionResult GetUsers([FromQuery] string name, [FromQuery] int age)
{
    // name 和 age 都从 URL 的 ?name=xxx&age=xxx 里取
    return Ok(new { name, age });
}
```

**这种接口，参数在 URL 里，你需要用 `params=` 传参。**

---

### ✅ 方法一：直接把参数拼进 URL（不推荐，容易出错）

```python
import requests

# 手动拼参数（不推荐，特殊字符会出错）
url = "http://localhost:5000/api/users?name=张三&age=25"
response = requests.get(url)
print(response.json())
```

---

### ✅ 方法二：用 params 字典传参（推荐！）

```python
import requests

url = "http://localhost:5000/api/users"

# 把参数放到字典里
params = {
    "name": "张三",
    "age": 25
}

# 传入 params 参数，requests 会自动拼成 ?name=张三&age=25
response = requests.get(url, params=params)

print("实际请求的 URL：", response.url)
# 输出：http://localhost:5000/api/users?name=%E5%BC%A0%E4%B8%89&age=25
# （中文会自动编码，不用担心）

print("返回结果：", response.json())
```

> 🎯 **为什么推荐 params=？** 
> 因为 requests 会自动处理中文、特殊字符的编码，你不用手动处理 `urllib.quote()`。

---

### ✅ 多个同名参数（传数组）

有时候接口要接收多个相同的参数名，比如 `?ids=1&ids=2&ids=3`：

```python
import requests

url = "http://localhost:5000/api/users/batch"

# 方法：传一个列表
params = {
    "ids": [1, 2, 3]
}

response = requests.get(url, params=params)
print("实际 URL：", response.url)
# 输出：http://localhost:5000/api/users/batch?ids=1&ids=2&ids=3
```

---

### POST 表单传参（Form Data）

如果是 POST 接口，但参数仍然是简单的键值对（表单格式）：

C# 接口：
```csharp
[HttpPost("login")]
public IActionResult Login([FromForm] string username, [FromForm] string password)
{
    return Ok("登录成功");
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/login"

# 用 data= 传表单参数（Content-Type 会自动设为 application/x-www-form-urlencoded）
data = {
    "username": "admin",
    "password": "123456"
}

response = requests.post(url, data=data)
print(response.text)
```

> ⚠️ **注意：** `data=` 是表单格式，`json=` 是 JSON 格式，两者不能混用！

---

## 3. 情景B：参数是复杂对象（JSON）

> **对应 C# 接口写法：** `[FromBody]`，接收一个对象/模型

这是最常见也是最容易出错的场景！

### 场景举例

C# 接口代码：
```csharp
// 创建订单接口
public class CreateOrderRequest
{
    public string ProductName { get; set; }
    public int Quantity { get; set; }
    public decimal Price { get; set; }
}

[HttpPost("orders")]
public IActionResult CreateOrder([FromBody] CreateOrderRequest request)
{
    // request 从请求的 Body 里读，格式是 JSON
    return Ok(request);
}
```

**这种接口，你需要把参数放在请求 Body 里，格式是 JSON。**

---

### ✅ 方法：用 json= 传参（推荐！）

```python
import requests

url = "http://localhost:5000/api/orders"

# 把对象写成 Python 字典
body = {
    "productName": "苹果手机",  # 注意：C# 通常用驼峰命名，Python 传参要匹配！
    "quantity": 2,
    "price": 9999.00
}

# 用 json= 传参，requests 自动：
# 1. 把字典转成 JSON 字符串
# 2. 设置 Content-Type: application/json
response = requests.post(url, json=body)

print("状态码：", response.status_code)
print("返回结果：", response.json())
```

> 🎯 **`json=` 的魔法：** 
> - 自动把 Python 字典 → JSON 字符串
> - 自动添加请求头 `Content-Type: application/json`
> - **C# 的 `[FromBody]` 需要这个请求头才能正确解析！**

---

### ✅ 嵌套对象（对象里有对象）

C# 接口：
```csharp
public class Address
{
    public string City { get; set; }
    public string Street { get; set; }
}

public class CreateUserRequest
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address HomeAddress { get; set; }  // 嵌套对象
}

[HttpPost("users")]
public IActionResult CreateUser([FromBody] CreateUserRequest request)
{
    return Ok(request);
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/users"

# 嵌套对象 → 嵌套字典
body = {
    "name": "李四",
    "age": 30,
    "homeAddress": {           # 嵌套字典对应嵌套对象
        "city": "深圳",
        "street": "科技园路 88 号"
    }
}

response = requests.post(url, json=body)
print(response.json())
```

---

### ✅ 列表参数（对象里有数组）

C# 接口：
```csharp
public class CreateOrderRequest
{
    public string CustomerName { get; set; }
    public List<OrderItem> Items { get; set; }  // 列表
}

public class OrderItem
{
    public string ProductName { get; set; }
    public int Quantity { get; set; }
}

[HttpPost("orders")]
public IActionResult CreateOrder([FromBody] CreateOrderRequest request) { ... }
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/orders"

body = {
    "customerName": "王五",
    "items": [                    # 列表 → Python 的 list
        {
            "productName": "键盘",
            "quantity": 1
        },
        {
            "productName": "鼠标",
            "quantity": 2
        },
        {
            "productName": "显示器",
            "quantity": 1
        }
    ]
}

response = requests.post(url, json=body)
print(response.json())
```

---

### ✅ 直接传一个列表（Body 就是数组）

C# 接口：
```csharp
[HttpPost("users/batch")]
public IActionResult BatchCreateUsers([FromBody] List<string> usernames)
{
    return Ok(usernames);
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/users/batch"

# Body 直接是一个列表
body = ["张三", "李四", "王五"]

response = requests.post(url, json=body)
print(response.json())
```

> ⚠️ **常见误区：** 很多新手会把列表塞进字典里，比如 `{"data": ["张三","李四"]}` 传过去，但接口期望的是直接的数组 `["张三","李四"]`，就会报错！要看接口定义决定怎么传。

---

## 4. 情景C：混合参数（简单+复杂）

这是最"复杂"的场景，接口同时需要 URL 里的简单参数 **和** Body 里的复杂对象。

### 场景一：URL 参数 + JSON Body

C# 接口：
```csharp
// 给指定用户创建订单
[HttpPost("users/{userId}/orders")]
public IActionResult CreateOrder(
    int userId,              // 来自路由
    [FromQuery] string source,   // 来自 URL 查询参数
    [FromBody] CreateOrderRequest request  // 来自 Body
)
{
    return Ok(new { userId, source, request });
}
```

Python 调用：
```python
import requests

# URL 里包含路由参数和查询参数
url = "http://localhost:5000/api/users/123/orders"

# URL 查询参数
params = {
    "source": "mobile"  # 拼成 ?source=mobile
}

# Body（JSON 格式）
body = {
    "productName": "耳机",
    "quantity": 1,
    "price": 299.00
}

# params 和 json 同时传！
response = requests.post(url, params=params, json=body)

print("实际 URL：", response.url)
# http://localhost:5000/api/users/123/orders?source=mobile
print("返回结果：", response.json())
```

> 🎯 **关键点：** `params=` 和 `json=` 可以同时使用，互不干扰！
> - `params=` → 拼到 URL 后面
> - `json=` → 放到 Body 里

---

### 场景二：URL 参数 + 表单 Body

C# 接口：
```csharp
[HttpPost("upload/info")]
public IActionResult UploadInfo(
    [FromQuery] string category,
    [FromForm] string title,
    [FromForm] string description
)
{
    return Ok(new { category, title, description });
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/upload/info"

# URL 查询参数
params = {
    "category": "document"
}

# 表单参数
form_data = {
    "title": "季度报告",
    "description": "2026年第一季度财务报告"
}

# params 和 data 同时传
response = requests.post(url, params=params, data=form_data)
print(response.json())
```

---

### 场景三：复杂对象嵌套复杂对象（深度嵌套）

```python
import requests

url = "http://localhost:5000/api/company"

# 深度嵌套，只要保持字典结构对应 C# 类结构即可
body = {
    "companyName": "科技有限公司",
    "address": {
        "province": "广东省",
        "city": "佛山市",
        "detail": "禅城区某某路 1 号"
    },
    "contacts": [
        {
            "name": "张总",
            "phone": "13800138000",
            "role": "CEO",
            "socialMedia": {
                "wechat": "zhangzong_666",
                "linkedin": "zhang-ceo"
            }
        },
        {
            "name": "李经理",
            "phone": "13900139000",
            "role": "CTO",
            "socialMedia": {
                "wechat": "li_cto"
            }
        }
    ],
    "establishedYear": 2020,
    "isPublicListed": False  # Python 的 False → JSON 的 false（自动转换）
}

response = requests.post(url, json=body)
print(response.json())
```

> 💡 **Python ↔ JSON 类型对照：**
> 
> | Python | JSON |
> |--------|------|
> | `dict` | `{}` 对象 |
> | `list` | `[]` 数组 |
> | `str` | `"字符串"` |
> | `int/float` | 数字 |
> | `True` | `true` |
> | `False` | `false` |
> | `None` | `null` |

---

## 5. 情景D：上传文件

### 场景一：只上传文件

C# 接口：
```csharp
[HttpPost("files/upload")]
public IActionResult Upload(IFormFile file)
{
    var fileName = file.FileName;
    // 处理文件...
    return Ok(new { fileName });
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/files/upload"

# 打开文件，以二进制模式读取
with open("report.pdf", "rb") as f:
    files = {
        # 格式：{"接口里的参数名": (文件名, 文件内容, 文件类型)}
        "file": ("report.pdf", f, "application/pdf")
    }
    response = requests.post(url, files=files)

print(response.json())
```

> ⚠️ **注意：** 用了 `files=` 后，不要再手动设置 `Content-Type`！
> requests 会自动设置 `Content-Type: multipart/form-data`，手动设置反而会报错。

---

### 场景二：上传文件 + 同时传其他参数

C# 接口：
```csharp
[HttpPost("files/upload")]
public IActionResult Upload(
    IFormFile file,
    [FromForm] string description,
    [FromForm] string category
)
{
    return Ok(new { file.FileName, description, category });
}
```

Python 调用：
```python
import requests

url = "http://localhost:5000/api/files/upload"

with open("report.pdf", "rb") as f:
    # 文件放 files=
    files = {
        "file": ("report.pdf", f, "application/pdf")
    }
    # 其他表单参数放 data=
    data = {
        "description": "2026年Q1财务报告",
        "category": "finance"
    }
    # 同时传 files 和 data
    response = requests.post(url, files=files, data=data)

print(response.json())
```

---

### 场景三：上传多个文件

```python
import requests

url = "http://localhost:5000/api/files/batch-upload"

with open("file1.jpg", "rb") as f1, open("file2.jpg", "rb") as f2:
    files = [
        ("files", ("file1.jpg", f1, "image/jpeg")),
        ("files", ("file2.jpg", f2, "image/jpeg")),
    ]
    response = requests.post(url, files=files)

print(response.json())
```

---

## 6. 必须知道的：请求头 Headers

### Content-Type 是什么？

Content-Type 告诉服务器："我发过来的数据是什么格式的"。
如果你告诉服务器格式 A，但实际发的是格式 B，服务器就会懵——这就是为什么会报 **415 错误**！

### 常见 Content-Type 对照表

| 场景 | Content-Type | requests 写法 |
|------|-------------|--------------|
| 发 JSON 数据 | `application/json` | `requests.post(url, json=data)` 自动设置 |
| 发表单数据 | `application/x-www-form-urlencoded` | `requests.post(url, data=data)` 自动设置 |
| 上传文件 | `multipart/form-data` | `requests.post(url, files=files)` 自动设置 |

> 🎯 **黄金原则：** 用 `json=`、`data=`、`files=` 参数时，**不要手动设置 Content-Type**，requests 会自动处理！

### 什么时候需要手动设置 Headers？

1. **需要身份验证（Token）**
2. **接口要求特定的 Accept 头**
3. **需要传 Cookie**

```python
import requests

url = "http://localhost:5000/api/protected/data"

# 手动设置请求头
headers = {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsIn...",  # Token 认证
    "Accept": "application/json",
    "X-Custom-Header": "my-value"  # 自定义头
}

body = {"query": "test"}

# headers 和 json 可以同时传
response = requests.post(url, headers=headers, json=body)
print(response.json())
```

---

## 7. 常见错误与解决方法

### ❌ 400 Bad Request（请求格式错误）

**原因：** 参数格式不对，或者缺少必填参数

```python
# ❌ 错误：用 data= 传 JSON Body 给 [FromBody] 接口
response = requests.post(url, data={"name": "张三"})

# ✅ 正确：用 json= 传 JSON Body
response = requests.post(url, json={"name": "张三"})
```

**排查步骤：**
1. 检查接口文档，确认参数名是否拼写正确
2. 确认是 `[FromBody]` 还是 `[FromQuery]`，选对传参方式
3. 打印请求内容检查：

```python
import requests

response = requests.post(url, json=body)

# 调试：看实际发出的请求
print("请求 URL：", response.request.url)
print("请求头：", response.request.headers)
print("请求体：", response.request.body)
print("响应：", response.text)
```

---

### ❌ 415 Unsupported Media Type（不支持的媒体类型）

**原因：** Content-Type 不对，最常见是用了 `data=` 但 C# 接口期望的是 JSON

```python
# ❌ 错误：data= 的 Content-Type 是 form-urlencoded，C# [FromBody] 不认识
response = requests.post(url, data={"name": "张三"})

# ✅ 正确：json= 的 Content-Type 是 application/json，C# [FromBody] 才能识别
response = requests.post(url, json={"name": "张三"})
```

**速查表：**

| 报错 | 最可能原因 | 解决方法 |
|------|-----------|---------|
| 415 | 用了 `data=` 但接口要 `[FromBody]` | 改成 `json=` |
| 415 | 手动设了错误的 Content-Type | 删掉手动设置的 Content-Type |
| 400 | 字段名拼错 | 对照 C# 类的属性名，注意大小写 |
| 400 | 缺少必填字段 | 检查接口文档 |

---

### ❌ 接口收到的值是 null（C# 端为空）

**原因：** 数据传到了错误的位置

```python
# 假设 C# 接口是：[FromBody] CreateUserRequest request

# ❌ 错误：把 JSON 放到了 URL 里
response = requests.get(url, params={"name": "张三"})

# ✅ 正确：JSON 应该放在 Body 里
response = requests.post(url, json={"name": "张三"})
```

---

### ❌ 连接超时

```python
import requests

try:
    # 设置超时时间（秒）：connect_timeout=5, read_timeout=30
    response = requests.post(url, json=body, timeout=(5, 30))
    print(response.json())
except requests.exceptions.ConnectTimeout:
    print("连接超时，请检查服务器地址是否正确")
except requests.exceptions.ReadTimeout:
    print("读取超时，服务器响应太慢")
except requests.exceptions.ConnectionError:
    print("连接失败，请检查网络或服务器是否在运行")
```

---

## 8. 实战综合例子

### 例子：调用一个完整的"员工管理"系统接口

场景描述：
- 接口A：查询员工列表（简单参数）
- 接口B：新建员工（复杂对象）
- 接口C：给员工上传头像（文件 + 简单参数）
- 接口D：更新员工信息（URL 参数 + JSON Body）

```python
import requests

BASE_URL = "http://localhost:5000/api"

# 公共请求头（如果需要 Token 认证）
HEADERS = {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiJ9..."
}


# ==================== 接口A：查询员工列表 ====================
def get_employees(department=None, page=1, page_size=10):
    """
    C# 接口：GET /employees?department=xxx&page=1&pageSize=10
    所有参数都是 [FromQuery]，用 params= 传
    """
    params = {
        "page": page,
        "pageSize": page_size
    }
    if department:
        params["department"] = department

    response = requests.get(
        f"{BASE_URL}/employees",
        params=params,
        headers=HEADERS
    )

    if response.status_code == 200:
        return response.json()
    else:
        print(f"查询失败：{response.status_code} - {response.text}")
        return None


# ==================== 接口B：新建员工 ====================
def create_employee(name, age, department, position, salary, skills):
    """
    C# 接口：POST /employees
    参数是 [FromBody] CreateEmployeeRequest
    包含嵌套对象和列表
    """
    body = {
        "name": name,
        "age": age,
        "department": department,
        "position": position,
        "salary": salary,
        "skills": skills,           # 列表
        "address": {                # 嵌套对象
            "city": "佛山",
            "detail": "禅城区"
        }
    }

    response = requests.post(
        f"{BASE_URL}/employees",
        json=body,          # [FromBody] 必须用 json=
        headers=HEADERS
    )

    if response.status_code == 201:
        print("创建成功！")
        return response.json()
    else:
        print(f"创建失败：{response.status_code} - {response.text}")
        return None


# ==================== 接口C：上传员工头像 ====================
def upload_avatar(employee_id, avatar_path):
    """
    C# 接口：POST /employees/{id}/avatar?type=profile
    文件用 multipart，URL 有查询参数
    """
    params = {"type": "profile"}  # URL 查询参数

    with open(avatar_path, "rb") as f:
        files = {
            "avatar": (avatar_path, f, "image/jpeg")
        }
        # 注意：上传文件时不要在 headers 里加 Content-Type！
        upload_headers = {k: v for k, v in HEADERS.items()}

        response = requests.post(
            f"{BASE_URL}/employees/{employee_id}/avatar",
            params=params,
            files=files,
            headers=upload_headers
        )

    if response.status_code == 200:
        print("头像上传成功！")
        return response.json()
    else:
        print(f"上传失败：{response.status_code} - {response.text}")
        return None


# ==================== 接口D：更新员工信息 ====================
def update_employee(employee_id, department, update_data):
    """
    C# 接口：PUT /employees/{id}?department=xxx
    URL 里有路由参数 + 查询参数，Body 里有 JSON 对象
    """
    params = {"department": department}  # URL 查询参数

    response = requests.put(
        f"{BASE_URL}/employees/{employee_id}",
        params=params,
        json=update_data,   # Body JSON
        headers=HEADERS
    )

    if response.status_code == 200:
        return response.json()
    else:
        print(f"更新失败：{response.status_code} - {response.text}")
        return None


# ==================== 主程序 ====================
if __name__ == "__main__":
    # 1. 查询研发部员工
    print("=== 查询员工列表 ===")
    employees = get_employees(department="研发部", page=1)
    print(employees)

    # 2. 新建一个员工
    print("\n=== 新建员工 ===")
    new_employee = create_employee(
        name="赵六",
        age=28,
        department="研发部",
        position="后端工程师",
        salary=20000,
        skills=["Python", "C#", "Java", "Docker"]
    )
    print(new_employee)

    # 3. 上传头像（假设新建员工 ID 是 101）
    print("\n=== 上传头像 ===")
    upload_avatar(employee_id=101, avatar_path="avatar.jpg")

    # 4. 更新员工信息
    print("\n=== 更新信息 ===")
    result = update_employee(
        employee_id=101,
        department="研发部",
        update_data={
            "position": "高级后端工程师",
            "salary": 25000
        }
    )
    print(result)
```

---

## 🗺️ 传参方式速查总结

```
你收到了一个接口，要判断怎么传参？按这个流程走：

1. 是 GET 请求？
   → 参数一定在 URL 里 → 用 params=

2. 是 POST/PUT 请求，参数是简单键值对（文本/数字）？
   → C# 是 [FromForm] → 用 data=
   → C# 是 [FromQuery] → 用 params=

3. 是 POST/PUT 请求，参数是一个对象（有很多字段/嵌套）？
   → C# 是 [FromBody] → 用 json=

4. 需要上传文件？
   → 用 files=
   → 如果同时有其他参数：files= + data=（不能 files= + json=！）

5. 又有 URL 参数，又有 Body 参数？
   → params= + json= 同时用（完全兼容）
```

---

## 参考资料

> 本教程参考以下 30 篇资料整理而成，均为可信来源：

**英文资料：**
1. [Python's Requests Library (Guide) – Real Python](https://realpython.com/python-requests/)
2. [GET and POST Requests Using Python – GeeksforGeeks](https://www.geeksforgeeks.org/python/get-post-requests-using-python/)
3. [Getting Started with Python HTTP Requests for REST APIs – DataCamp](https://www.datacamp.com/tutorial/making-http-requests-in-python)
4. [Python Requests Tutorial – Edureka/Medium](https://medium.com/edureka/python-requests-tutorial-30edabfa6a1c)
5. [Python Requests post Method – W3Schools](https://www.w3schools.com/python/ref_requests_post.asp)
6. [How to POST JSON data with Python Requests – Stack Overflow](https://stackoverflow.com/questions/9733638/how-to-post-json-data-with-python-requests)
7. [Post Nested Data Structure to the Server Using Requests – jdhao](https://jdhao.github.io/2021/04/08/send_complex_data_in_python_requests/)
8. [Python requests POST with headers and body – GeeksforGeeks](https://www.geeksforgeeks.org/python/python-requests-post-request-with-headers-and-body/)
9. [How do I post JSON using Python Requests – ReqBin](https://reqbin.com/code/python/m2g4va4a/python-requests-post-json-example)
10. [Parameter Binding in ASP.NET Web API – TutorialsTeacher](https://www.tutorialsteacher.com/webapi/parameter-binding-in-web-api)
11. [Get Request Body, Parameters & Headers in C# Controller](https://jd-bots.com/2023/01/28/get-request-body-parameters-headers-in-c-controller-for-incoming-http-requests/)
12. [ASP.NET Core Web API Model Binding Attributes – Medium](https://medium.com/@beyzaerdogmus/asp-net-core-web-api-model-binding-attributes-c7c4a5b85afc)
13. [Model Binding Using FromBody in ASP.NET Core – Dot Net Tutorials](https://dotnettutorials.net/lesson/frombody-inasp-net-core-web-api/)
14. [FromBody and FromQuery Example in ASP.NET Core – RoundTheCode](https://www.roundthecode.com/dotnet-code-examples/a-frombody-and-fromquery-example-in-asp-net-core-web-api)
15. [Why is ASP.NET Core FromBody not working or returning null?](https://www.roundthecode.com/dotnet-tutorials/why-asp-net-core-frombody-not-working-returning-null)
16. [How to send multipart/form-data with requests in Python – Stack Overflow](https://stackoverflow.com/questions/12385179/how-to-send-a-multipart-form-data-with-requests-in-python)
17. [Sending Multipart Form Data with Python Requests – ProxiesAPI](https://proxiesapi.com/articles/sending-multipart-form-data-with-python-requests)
18. [How to Send multipart/form-data with Requests – StackAbuse](https://stackabuse.com/bytes/how-to-send-multipart-form-data-with-requests-in-python/)
19. [Python request gives 415 error – Stack Overflow](https://stackoverflow.com/questions/52216808/python-request-gives-415-error-while-post-data)
20. [How to fix 415 Unsupported Media Type – Stack Overflow](https://stackoverflow.com/questions/55375001/how-to-fix-415-unsupported-media-type-error-in-python-using-requests)
21. [What is HTTP 415 Error – Scrapfly](https://scrapfly.io/blog/posts/what-is-http-415-error-unsupported-media-type)
22. [APIs: Params= vs json= – Reddit/learnpython](https://www.reddit.com/r/learnpython/comments/r2phi2/apis_params_vs_json_whats_the_difference/)
23. [Query Parameters and REST APIs in Python – CodeSignal](https://codesignal.com/learn/courses/basic-python-and-web-requests/lessons/mastering-data-retrieval-query-parameters-and-rest-apis-in-python)
24. [Difference between data and json in Python Requests – Stack Overflow](https://stackoverflow.com/questions/26685248/difference-between-data-and-json-parameters-in-python-requests-package)
25. [Post value always null between Python and ASP.NET Core – Stack Overflow](https://stackoverflow.com/questions/54595010/post-value-always-null-between-python-request-and-asp-net-core-api)
26. [How to send an array using requests.post – Stack Overflow](https://stackoverflow.com/questions/31168819/how-to-send-an-array-using-requests-post-python-value-error-too-many-values)
27. [Python File Upload: multipart/Form-Data – W3Resource](https://www.w3resource.com/python-exercises/urllib3/python-urllib3-exercise-19.php)

**中文资料：**
28. [Python-Requests.post 中 data 与 json 的区别 – CSDN](https://blog.csdn.net/grace666/article/details/90481970)
29. [requests.post 中 data 与 json 参数区别详解 – 腾讯云开发者社区](https://cloud.tencent.com/developer/article/1738491)
30. [requests 爬虫库使用简介 – 盖若](https://gairuo.com/p/python-requests)

---

*本教程由 AI 助手小亮 🐈‍⬛ 整理生成，如有疑问欢迎提问！*
