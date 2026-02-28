# 🐍 Python 调接口实战手册
### —— 专为实施同事打造，手把手教你用 Python 和 ABP 系统"对话"

> 🎯 **写给谁看的？** 软件公司实施同事，不需要编程基础，照着抄就能用  
> 🏗️ **后端是什么？** C# 开发的 ABP 框架 / ABP vNext 框架  
> ⏱️ **需要多久？** 从头读到尾约 30 分钟，之后查哪节用哪节

---

## 🌟 在开始之前：用一个生活比喻理解"接口调用"

想象你去餐厅吃饭：

```
你（Python 程序）
    ↓ 点菜（发请求）
服务员（HTTP 接口）
    ↓ 传单给厨房
厨房（C# ABP 后端）
    ↓ 做好菜
服务员（HTTP 接口）
    ↓ 端给你
你（Python 程序）拿到菜（返回数据）✅
```

- **点菜单** = 你发送的参数
- **点菜方式** = GET / POST 请求
- **菜** = 接口返回给你的数据

**参数放错地方，就像把点菜单塞进厨房窗口而不交给服务员 → 服务员不知道你要啥 → 出错！**

---

## 📖 目录

0. [准备工作（5分钟搞定）](#0-准备工作)
1. [先认识 GET 和 POST](#1-先认识-get-和-post)
2. [情景A：查数据（全是简单参数）](#2-情景a查数据全是简单参数)
3. [情景B：新建/修改数据（复杂对象参数）](#3-情景b新建修改数据复杂对象参数)
4. [情景C：又有简单参数又有复杂对象](#4-情景c又有简单参数又有复杂对象)
5. [情景D：上传文件](#5-情景d上传文件)
6. [⭐ ABP 框架专项指南](#6-abp-框架专项指南重点)
7. [常见报错急救手册](#7-常见报错急救手册)
8. [完整实战例子](#8-完整实战例子)
9. [参考资料](#参考资料)

---

## 0. 准备工作

### 第一步：安装 requests 库

打开「命令提示符」或「终端」，输入：

```bash
pip install requests
```

看到 `Successfully installed requests-x.x.x` 就成功了 🎉

### 第二步：在代码最开头加这一行

```python
import requests  # 以后每次写代码都要加这行！
```

### 第三步：认识三个关键词

| 词 | 意思 | 生活类比 |
|----|------|---------|
| `params=` | 参数放 URL 里 | 在餐厅门口的牌子上写"我要一份炒饭" |
| `json=` | 参数放请求体里（JSON格式） | 把详细订单装在信封里递给服务员 |
| `data=` | 参数放请求体里（表单格式） | 填写纸质表单交给服务员 |

---

## 1. 先认识 GET 和 POST

### GET：我想"拿"点什么

- 用于**查询数据**，不改变任何东西
- 参数直接跟在网址后面，比如：`/api/users?name=张三`
- 就像在图书馆查书目，你只是在看，不是在借

```python
import requests

response = requests.get("http://你的系统地址/api/app/user")

print(response.status_code)  # 200 = 成功，404 = 没找到，500 = 服务器出问题了
print(response.json())       # 打印返回的数据
```

### POST：我想"提交"点什么

- 用于**新建、修改、删除数据**，会改变数据库
- 参数放在"请求体"里（看不见的地方）
- 就像填完表单交给柜台，柜台去帮你办理业务

```python
import requests

response = requests.post("http://你的系统地址/api/app/user")

print(response.status_code)
print(response.json())
```

### 状态码速查表

| 状态码 | 意思 | 你该怎么办 |
|--------|------|-----------|
| **200** | ✅ 成功 | 完美，取数据用就行 |
| **201** | ✅ 创建成功 | 新建操作成功 |
| **400** | ❌ 你的请求有问题 | 检查参数名字/格式是否正确 |
| **401** | 🔒 没登录/Token失效 | 重新获取 Token |
| **403** | 🚫 没权限 | 联系管理员 |
| **404** | 🔍 接口地址不存在 | 检查URL是否拼对了 |
| **415** | 📄 格式不对 | 检查是用 `json=` 还是 `data=` |
| **500** | 💥 服务器崩了 | 联系开发同事 |

---

## 2. 情景A：查数据（全是简单参数）

> 🏷️ 特征：接口是 **GET 请求**，参数都是简单的文字/数字  
> 🔧 对应 ABP：`GetAsync` / `GetListAsync` 开头的方法

### 什么是"简单参数"？

简单参数就是一个名字对一个值，比如：
- `姓名=张三`
- `年龄=25`  
- `部门=研发部`

### 怎么传？用 `params=`

```python
import requests

# 🌰 例子1：查询用户列表
url = "http://localhost:5000/api/app/user"

# 把参数写成字典（大括号包起来的键值对）
params = {
    "filter": "张三",       # 搜索关键词
    "skipCount": 0,          # 从第几条开始（0表示第一条）
    "maxResultCount": 10     # 最多返回几条
}

response = requests.get(url, params=params)

# 查看实际发出的完整地址（用于调试）
print("完整URL：", response.url)
# 输出：http://localhost:5000/api/app/user?filter=张三&skipCount=0&maxResultCount=10

print("状态码：", response.status_code)
print("返回数据：", response.json())
```

```python
# 🌰 例子2：查询单个用户（通过ID）
user_id = "abc123"
response = requests.get(f"http://localhost:5000/api/app/user/{user_id}")
print(response.json())
```

```python
# 🌰 例子3：带时间范围的查询
params = {
    "startDate": "2026-01-01",
    "endDate": "2026-02-28",
    "status": 1,          # 数字也可以直接传
    "isActive": "true"    # 布尔值传字符串 "true" / "false"
}
response = requests.get(url, params=params)
```

> 💡 **小技巧：** 不确定参数名叫什么？让开发同事打开接口文档（Swagger），里面有所有参数的名字！

---

## 3. 情景B：新建/修改数据（复杂对象参数）

> 🏷️ 特征：接口是 **POST / PUT 请求**，参数是一个包含多个字段的"对象"  
> 🔧 对应 ABP：`CreateAsync` / `UpdateAsync` 开头的方法

### 什么是"复杂对象参数"？

不是单个值，而是一组相关的数据合在一起，比如创建一个用户需要：
- 姓名、邮箱、手机号、角色……

这些数据打包成一个"对象"传过去。

### 怎么传？用 `json=`（注意！不是 `data=`！）

```python
import requests

# 🌰 例子1：创建新用户
url = "http://localhost:5000/api/app/user"

# 把要新建的数据写成字典
body = {
    "userName": "zhangsan",
    "name": "张三",
    "email": "zhangsan@company.com",
    "phoneNumber": "13800138000",
    "password": "Abc@123456"
}

# ⚠️ 关键：用 json= 传，不是 data=！
response = requests.post(url, json=body)

print("状态码：", response.status_code)
print("返回数据：", response.json())
```

```python
# 🌰 例子2：修改已有数据（PUT 请求）
user_id = "abc-123-def"
url = f"http://localhost:5000/api/app/user/{user_id}"

body = {
    "name": "张三（已修改）",
    "email": "zhangsan_new@company.com"
}

response = requests.put(url, json=body)
print(response.json())
```

### 嵌套对象（对象里还有对象）

```python
# 🌰 例子3：创建客户（地址是一个嵌套的子对象）
body = {
    "companyName": "佛山科技公司",
    "contactName": "李经理",
    "phoneNumber": "075712345678",
    "address": {                    # ← 这里是嵌套对象，对应的是个子字典
        "country": "中国",
        "province": "广东",
        "city": "佛山",
        "street": "禅城区季华路 100 号"
    }
}

response = requests.post("http://localhost:5000/api/app/customer", json=body)
print(response.json())
```

### 列表参数（有多个条目的数据）

```python
# 🌰 例子4：创建订单（包含多个商品明细）
body = {
    "customerId": "cust-001",
    "remark": "请尽快发货",
    "items": [                      # ← 这是一个列表，用方括号
        {
            "productCode": "P001",
            "productName": "工业传感器",
            "quantity": 5,
            "unitPrice": 320.00
        },
        {
            "productCode": "P002",
            "productName": "控制器模块",
            "quantity": 2,
            "unitPrice": 1500.00
        }
    ]
}

response = requests.post("http://localhost:5000/api/app/order", json=body)
print(response.json())
```

> 🎓 **记忆口诀：**
> - 字典 `{}` → 对象（一组相关字段）
> - 列表 `[]` → 数组（多条同类数据）
> - 嵌套字典 → 对象里有对象
> - 字典里套列表 → 对象里有数组

---

## 4. 情景C：又有简单参数又有复杂对象

> 🏷️ 特征：一个接口里，**URL 需要简单参数，Body 里需要复杂对象**  
> 🔧 对应 ABP：带有路由ID 且需要传 DTO 对象的接口

### 怎么传？`params=` 和 `json=` **同时写**！

```python
import requests

# 🌰 例子1：给指定客户创建合同
# URL：/api/app/contract?source=mobile
# Body：合同详情对象

url = "http://localhost:5000/api/app/contract"

# 简单参数 → params=（会跑到 URL 里）
params = {
    "source": "mobile",      # 来源渠道
    "tenantId": "tenant-001" # 租户ID
}

# 复杂对象 → json=（会进入 Body）
body = {
    "customerId": "cust-001",
    "contractName": "2026年设备采购合同",
    "totalAmount": 58000.00,
    "startDate": "2026-03-01",
    "endDate": "2027-02-28"
}

# 两个参数同时写，互不干扰！
response = requests.post(url, params=params, json=body)

print("实际请求URL：", response.url)
# http://localhost:5000/api/app/contract?source=mobile&tenantId=tenant-001
print("返回结果：", response.json())
```

```python
# 🌰 例子2：更新某个记录（URL里有ID，Body里有修改内容）
record_id = "rec-20260301-001"
url = f"http://localhost:5000/api/app/workorder/{record_id}"

# URL 查询参数
params = {"notify": "true"}  # 是否发通知

# Body：要修改的字段
body = {
    "status": 2,
    "handlerName": "王师傅",
    "remark": "已完成维修，更换了三个零件"
}

response = requests.put(url, params=params, json=body)
print(response.json())
```

---

## 5. 情景D：上传文件

> 🏷️ 特征：需要上传图片、文档、Excel 等文件  
> ⚠️ **特别注意：用了 `files=` 就不能手动设置 Content-Type！**

### 只上传文件

```python
import requests

url = "http://localhost:5000/api/app/file/upload"

# 打开文件（"rb" 表示以二进制方式读取，上传文件必须这样）
with open("合同扫描件.pdf", "rb") as f:
    files = {
        # 格式：{"接口里的参数名": (文件名, 文件内容, 文件类型)}
        "file": ("合同扫描件.pdf", f, "application/pdf")
    }
    response = requests.post(url, files=files)

print(response.json())
```

### 上传文件 + 同时传其他参数

```python
import requests

url = "http://localhost:5000/api/app/attachment"

with open("产品图片.jpg", "rb") as f:
    files = {
        "file": ("产品图片.jpg", f, "image/jpeg")
    }
    # 其他文字参数用 data=（注意：有 files= 时，其他参数必须用 data= 不能用 json=）
    data = {
        "relatedId": "prod-001",    # 关联的产品ID
        "fileType": "product_image",
        "description": "主图-正面"
    }
    response = requests.post(url, files=files, data=data)

print(response.json())
```

> ⚠️ **记住这个规则：**
> - 有文件 → `files=` + `data=`（其他参数）
> - 没文件，传对象 → `json=`
> - 不能同时用 `files=` 和 `json=`！

---

## 6. ⭐ ABP 框架专项指南（重点）

> 这一节是专门针对你们公司 C# ABP 系统的内容，**跟普通接口有些不同！**

### 6.1 认识两个版本的 ABP

你可能接触到两个版本，它们**接口地址和返回格式都不一样**：

| 对比项 | 老版 ABP（ASP.NET Boilerplate） | 新版 ABP（ABP vNext / ABP Framework） |
|--------|-------------------------------|--------------------------------------|
| 官网 | aspnetboilerplate.com | abp.io |
| 接口地址前缀 | `/api/services/` | `/api/app/` |
| 返回数据 | 有 `result` 包装层 | **直接返回数据**，无包装 |
| 怎么判断 | 看接口文档或问开发 | 同左 |

---

### 6.2 新版 ABP（ABP Framework / vNext）

#### 接口地址规律

ABP vNext 会把 C# 的服务名自动变成接口路径，规律是：

```
/api/app/{服务名（小写去掉AppService）}/{方法名（小写去掉Async）}
```

举例：
- `UserAppService.GetListAsync()` → `GET /api/app/user`
- `UserAppService.GetAsync(id)` → `GET /api/app/user/{id}`
- `OrderAppService.CreateAsync()` → `POST /api/app/order`
- `OrderAppService.UpdateAsync(id)` → `PUT /api/app/order/{id}`
- `OrderAppService.DeleteAsync(id)` → `DELETE /api/app/order/{id}`

#### HTTP 方法自动映射规律

| C# 方法名开头 | 对应 HTTP 方法 | Python 用法 |
|--------------|--------------|------------|
| `Get` / `Find` / `List` | GET | `requests.get()` |
| `Create` / `Add` | POST | `requests.post()` |
| `Update` / `Edit` / `Set` | PUT | `requests.put()` |
| `Delete` / `Remove` | DELETE | `requests.delete()` |
| 其他（自定义方法） | POST | `requests.post()` |

#### 参数在哪里传（ABP vNext 的核心规则）

ABP vNext **不写 `[FromBody]` 等标注**，但它有内部规律：

```
GET / DELETE 请求：
  → 不管参数是什么类型，全部用 params= 传到 URL 里

POST / PUT 请求：
  → 简单类型（文字、数字、ID）→ 用 params= 传
  → 复杂类型（一个DTO对象）  → 用 json= 传到 Body 里
  → 如果既有简单又有复杂     → params= 和 json= 同时用！
```

#### 返回格式（ABP vNext）

ABP vNext 直接返回数据，**没有 `result` 包装层**：

```python
response = requests.get(url, params=params)
data = response.json()

# 直接取数据，不需要 data["result"]
print(data["items"])        # 列表数据
print(data["totalCount"])   # 总数
```

#### 完整代码示例（ABP vNext）

```python
import requests

BASE_URL = "http://localhost:5000"

# 如果需要认证，加上 Token
HEADERS = {
    "Authorization": "Bearer 你的Token"
}

# ── 查询列表 ──────────────────────────────────
response = requests.get(
    f"{BASE_URL}/api/app/user",
    params={
        "filter": "张",
        "skipCount": 0,
        "maxResultCount": 20
    },
    headers=HEADERS
)
result = response.json()
print(f"共 {result['totalCount']} 条，本页 {len(result['items'])} 条")

# ── 查询单条 ──────────────────────────────────
user_id = "3fa85f64-5717-4562-b3fc-2c963f66afa6"
response = requests.get(
    f"{BASE_URL}/api/app/user/{user_id}",
    headers=HEADERS
)
user = response.json()
print(f"用户姓名：{user['name']}")

# ── 新建 ──────────────────────────────────────
response = requests.post(
    f"{BASE_URL}/api/app/user",
    json={
        "userName": "lisi",
        "name": "李四",
        "email": "lisi@company.com",
        "password": "Admin@123"
    },
    headers=HEADERS
)
print(f"新建结果：{response.status_code}")

# ── 修改 ──────────────────────────────────────
response = requests.put(
    f"{BASE_URL}/api/app/user/{user_id}",
    json={
        "name": "李四（已修改）",
        "email": "lisi_new@company.com"
    },
    headers=HEADERS
)
print(f"修改结果：{response.status_code}")

# ── 删除 ──────────────────────────────────────
response = requests.delete(
    f"{BASE_URL}/api/app/user/{user_id}",
    headers=HEADERS
)
print(f"删除结果：{response.status_code}")
```

---

### 6.3 老版 ABP（ASP.NET Boilerplate）

#### 接口地址格式

```
/api/services/{命名空间}/{服务名}/{方法名（camelCase）}
```

例如：
- `TaskAppService.GetTasks()` → `GET /api/services/tasksystem/task/getTasks`
- `TaskAppService.CreateTask()` → `POST /api/services/tasksystem/task/createTask`

#### ⚠️ 老版 ABP 的返回格式有包装层！

老版 ABP 的返回 JSON 长这样：
```json
{
    "result": {
        "items": [...],
        "totalCount": 100
    },
    "targetUrl": null,
    "success": true,
    "error": null,
    "unAuthorizedRequest": false,
    "__abp": true
}
```

取数据时**必须先取 `result` 字段**：

```python
import requests

response = requests.get(
    "http://localhost:5000/api/services/myapp/task/getTasks",
    params={"status": 1}
)

data = response.json()

# ⚠️ 老版 ABP 必须先取 result！
if data["success"]:
    tasks = data["result"]["items"]
    print(f"获取到 {len(tasks)} 条任务")
else:
    print(f"出错了：{data['error']['message']}")
```

对比新版（ABP vNext）：
```python
# 新版 ABP vNext，直接取数据
response = requests.get("http://localhost:5000/api/app/task", params={"status": 1})
data = response.json()
tasks = data["items"]  # 直接取，没有 result 包装！
```

---

### 6.4 如何获取登录 Token（两个版本通用）

大多数接口都需要先登录获取 Token，再带着 Token 去调其他接口。

```python
import requests

# ── 第一步：登录获取 Token ──────────────────
login_response = requests.post(
    "http://localhost:5000/api/account/login",  # 登录接口地址（问开发同事确认）
    json={
        "userNameOrEmailAddress": "admin",
        "password": "Admin@123"
    }
)

login_data = login_response.json()

# ABP vNext 的登录返回
token = login_data.get("accessToken") or login_data.get("token")

# 老版 ABP 的登录返回（在 result 里）
# token = login_data["result"]["accessToken"]

print(f"Token 获取成功：{token[:30]}...")

# ── 第二步：带着 Token 调其他接口 ──────────
headers = {
    "Authorization": f"Bearer {token}"
}

response = requests.get(
    "http://localhost:5000/api/app/user",
    params={"maxResultCount": 10},
    headers=headers
)
print(response.json())
```

---

### 6.5 多租户系统（如果有 TenantId 的话）

如果你的 ABP 系统是多租户的，有些接口需要传租户信息：

```python
# 方法1：通过请求头传租户ID
headers = {
    "Authorization": "Bearer your_token",
    "__tenant": "租户名称或ID"  # ABP 多租户标识头
}

# 方法2：通过 URL 传
params = {
    "tenantId": "your-tenant-id"
}
```

---

## 7. 常见报错急救手册

### 🚨 报错：400 Bad Request（请求格式有问题）

**最常见原因和解决方法：**

```python
# ❌ 问题：该用 json= 的地方用了 data=
response = requests.post(url, data={"name": "张三"})
# ✅ 解决：改成 json=
response = requests.post(url, json={"name": "张三"})

# ❌ 问题：字段名写错了（C# 的属性名是大小写敏感的！）
json={"UserName": "zhangsan"}   # 错：大写 U 和 N
json={"userName": "zhangsan"}   # ✅ 对：小写 u 和大写 N
```

**调试技巧：打印请求内容看看发出了什么**

```python
response = requests.post(url, json=body)

print("=== 调试信息 ===")
print("请求URL：", response.request.url)
print("请求头：", dict(response.request.headers))
print("请求体：", response.request.body)
print("响应码：", response.status_code)
print("响应内容：", response.text)
print("================")
```

---

### 🚨 报错：415 Unsupported Media Type（格式不支持）

```
解决方法99%：把 data= 改成 json=

requests.post(url, data={...})   ← ❌ data= 默认是表单格式
requests.post(url, json={...})   ← ✅ json= 才是 JSON 格式
```

---

### 🚨 报错：401 Unauthorized（没权限/未登录）

```python
# 原因：没带 Token 或 Token 过期

# 解决：加上 Authorization 头
headers = {"Authorization": "Bearer 你的Token"}
response = requests.get(url, headers=headers)

# 如果 Token 过期，重新登录获取新 Token
```

---

### 🚨 接口返回 200 但数据是 null 或空

```python
# 原因：参数传对了位置但字段名不匹配

# 调试方法：先用 Postman/浏览器开发工具确认接口正常工作
# 再对比 Python 代码发送的内容是否一致

response = requests.post(url, json=body)
print("发出的内容：", response.request.body)  # 看这里
```

---

### 🚨 连接报错 / 超时

```python
import requests

try:
    response = requests.get(url, timeout=10)  # 设置10秒超时
    print(response.json())
except requests.exceptions.ConnectionError:
    print("❌ 连接失败！检查：1.系统是否在运行 2.IP/端口是否正确 3.网络是否通")
except requests.exceptions.Timeout:
    print("❌ 超时！系统响应太慢，或者网络有问题")
```

---

## 8. 完整实战例子

### 场景：用 Python 操作 ABP vNext 的"工单管理"系统

```python
import requests

# =================== 基础配置 ===================
BASE_URL = "http://192.168.1.100:8080"  # 换成你们系统的地址

def get_token(username, password):
    """登录并获取 Token"""
    response = requests.post(
        f"{BASE_URL}/api/account/login",
        json={
            "userNameOrEmailAddress": username,
            "password": password
        }
    )
    if response.status_code == 200:
        data = response.json()
        return data.get("accessToken") or data.get("token")
    else:
        print(f"登录失败：{response.text}")
        return None


def make_headers(token):
    """生成带 Token 的请求头"""
    return {"Authorization": f"Bearer {token}"}


# =================== 主程序 ===================
# 1. 先登录
TOKEN = get_token("admin", "Admin@123")
if not TOKEN:
    exit("登录失败，程序退出")

HEADERS = make_headers(TOKEN)
print("✅ 登录成功！")


# 2. 查询工单列表
print("\n📋 查询待处理工单...")
response = requests.get(
    f"{BASE_URL}/api/app/workOrder",
    params={
        "status": 0,           # 0=待处理
        "skipCount": 0,
        "maxResultCount": 50
    },
    headers=HEADERS
)
result = response.json()
work_orders = result.get("items", [])
print(f"找到 {len(work_orders)} 条待处理工单")

for wo in work_orders[:3]:  # 只打印前3条
    print(f"  - [{wo['id']}] {wo['title']} | 负责人：{wo.get('assigneeName', '未分配')}")


# 3. 新建一条工单
print("\n➕ 新建工单...")
response = requests.post(
    f"{BASE_URL}/api/app/workOrder",
    json={
        "title": "设备异常报警",
        "description": "3号车间的PLC控制器报警，需要立即检查",
        "priority": 1,              # 1=紧急
        "deviceId": "dev-003",
        "location": "3号车间",
        "contactPhone": "13900139001"
    },
    headers=HEADERS
)
if response.status_code in [200, 201]:
    new_wo = response.json()
    print(f"✅ 工单创建成功！ID：{new_wo['id']}")
    new_wo_id = new_wo['id']
else:
    print(f"❌ 创建失败：{response.text}")
    new_wo_id = None


# 4. 给工单分配负责人（URL参数+Body混合）
if new_wo_id:
    print("\n👤 分配负责人...")
    response = requests.put(
        f"{BASE_URL}/api/app/workOrder/{new_wo_id}",
        params={"notify": "true"},   # URL参数：是否发通知
        json={                        # Body：修改内容
            "assigneeId": "user-001",
            "expectedFinishTime": "2026-02-28T18:00:00",
            "remark": "已分配给王师傅处理"
        },
        headers=HEADERS
    )
    print(f"分配结果：{'✅ 成功' if response.status_code == 200 else '❌ 失败'}")


# 5. 上传现场照片
if new_wo_id:
    print("\n📷 上传现场照片...")
    try:
        with open("现场照片.jpg", "rb") as f:
            response = requests.post(
                f"{BASE_URL}/api/app/workOrder/{new_wo_id}/attachment",
                files={"file": ("现场照片.jpg", f, "image/jpeg")},
                data={"fileType": "photo", "description": "设备报警现场"},
                headers=HEADERS
            )
        print(f"上传结果：{'✅ 成功' if response.status_code == 200 else '❌ 失败'}")
    except FileNotFoundError:
        print("（跳过：文件不存在）")


print("\n🎉 全部操作完成！")
```

---

## 🗺️ 最终速查卡片（建议截图保存）

```
┌──────────────────────────────────────────────────────┐
│              Python 调接口传参速查                     │
├──────────────┬───────────────────────────────────────┤
│ 接口类型      │ 怎么传参                               │
├──────────────┼───────────────────────────────────────┤
│ GET 查询      │ requests.get(url, params={...})        │
│ POST 简单参数  │ requests.post(url, params={...})       │
│ POST 复杂对象  │ requests.post(url, json={...})         │
│ POST 混合     │ requests.post(url, params={}, json={}) │
│ PUT 更新      │ requests.put(url, json={...})          │
│ DELETE 删除   │ requests.delete(url)                   │
│ 上传文件      │ requests.post(url, files={}, data={})  │
├──────────────┼───────────────────────────────────────┤
│ 加 Token     │ headers={"Authorization":"Bearer xxx"} │
├──────────────┼───────────────────────────────────────┤
│ ABP vNext    │ 直接 response.json() 取数据             │
│ 老版 ABP     │ response.json()["result"] 取数据        │
└──────────────┴───────────────────────────────────────┘
```

---

## 参考资料

1. [Python's Requests Library (Guide) – Real Python](https://realpython.com/python-requests/)
2. [GET and POST Requests in Python – GeeksforGeeks](https://www.geeksforgeeks.org/python/get-post-requests-using-python/)
3. [HTTP Requests in Python – DataCamp](https://www.datacamp.com/tutorial/making-http-requests-in-python)
4. [Python Requests post Method – W3Schools](https://www.w3schools.com/python/ref_requests_post.asp)
5. [How to POST JSON with Python Requests – Stack Overflow](https://stackoverflow.com/questions/9733638/how-to-post-json-data-with-python-requests)
6. [Post Nested Data Structure with Requests – jdhao.github.io](https://jdhao.github.io/2021/04/08/send_complex_data_in_python_requests/)
7. [Python Requests POST with headers and body – GeeksforGeeks](https://www.geeksforgeeks.org/python/python-requests-post-request-with-headers-and-body/)
8. [Difference: data= vs json= in Requests – Stack Overflow](https://stackoverflow.com/questions/26685248/difference-between-data-and-json-parameters-in-python-requests-package)
9. [ABP Auto API Controllers 官方文档](https://abp.io/docs/latest/framework/api-development/auto-controllers)
10. [ABP vNext Auto API Controllers – docs.abp.io](https://docs.abp.io/en/abp/6.0/API/Auto-API-Controllers)
11. [ABP Framework Auto API Controller v2.7](https://abp.io/docs/2.7/API/Auto-API-Controllers)
12. [ASP.NET Boilerplate Dynamic Web API](https://aspnetboilerplate.com/Pages/Documents/Dynamic-Web-API)
13. [ABP Response Wrapper (__abp field) – GitHub Issue](https://github.com/aspnetboilerplate/aspnetboilerplate/issues/3749)
14. [ABP WrapResult vs DontWrapResult – Stack Overflow](https://stackoverflow.com/questions/49423775/is-there-an-asp-net-boilerplate-way-of-getting-json-data)
15. [Parameter Binding in ASP.NET Web API – Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/web-api/overview/formats-and-model-binding/parameter-binding-in-aspnet-web-api)
16. [ASP.NET Core Model Binding Attributes – Medium](https://medium.com/@beyzaerdogmus/asp-net-core-web-api-model-binding-attributes-c7c4a5b85afc)
17. [FromBody in ASP.NET Core Web API – Dot Net Tutorials](https://dotnettutorials.net/lesson/frombody-inasp-net-core-web-api/)
18. [Post value null between Python and ASP.NET Core – Stack Overflow](https://stackoverflow.com/questions/54595010/post-value-always-null-between-python-request-and-asp-net-core-api)
19. [How to fix 415 Unsupported Media Type – Stack Overflow](https://stackoverflow.com/questions/55375001/how-to-fix-415-unsupported-media-type-error-in-python-using-requests)
20. [Sending Multipart Form Data with Requests – ProxiesAPI](https://proxiesapi.com/articles/sending-multipart-form-data-with-python-requests)
21. [How to send multipart/form-data – Stack Overflow](https://stackoverflow.com/questions/12385179/how-to-send-a-multipart-form-data-with-requests-in-python)
22. [ABP vNext vs ABP Boilerplate Response Format – GitHub](https://github.com/abpframework/abp/issues/8805)
23. [Python request gives 415 error – Stack Overflow](https://stackoverflow.com/questions/52216808/python-request-gives-415-error-while-post-data)
24. [Query Parameters and REST APIs – CodeSignal](https://codesignal.com/learn/courses/basic-python-and-web-requests/lessons/mastering-data-retrieval-query-parameters-and-rest-apis-in-python)
25. [Python Requests Tutorial – Edureka/Medium](https://medium.com/edureka/python-requests-tutorial-30edabfa6a1c)

---

*本教程由 AI 助手小亮 🐈‍⬛ 精心整理，专为实施同事定制*  
*最后更新：2026-02-28*
