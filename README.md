# PAT - API 测试框架

## 概述

PAT 是一个基于 Python 的 API 测试框架，提供了简洁的语法和丰富的功能来编写和执行 HTTP API 测试。该框架支持所有主要的 HTTP 方法（GET、POST、PUT、PATCH、DELETE、OPTIONS），并内置了响应数据提取、断言验证和美观的结果展示功能。

## 功能特性

- 支持完整的 HTTP 方法：GET、POST、PUT、PATCH、DELETE、OPTIONS
- 链式 API 调用，支持响应数据提取并在后续测试中使用
- 深路径数据提取，支持嵌套 JSON 字段访问
- 自动化的成功/失败断言
- 丰富的终端输出，使用 Rich 库提供美观的格式化显示
- 自定义请求头支持
- 错误处理和异常捕获
- 统一的错误显示格式，支持 JSON 美化显示
- 测试结果自动收集和汇总显示
- 简洁的 API 设计，易于编写和维护测试用例

## 安装

推荐使用 uv 包管理器进行安装：

> 下载uv: https://docs.astral.sh/uv/getting-started/installation

```bash
git clone https://github.com/zyoung11/API-TEST.git

# 创建项目目录
mkdir API-TEST
cd API-TEST

# 创建虚拟环境并安装依赖
uv sync
```

## 基本用法

### 1. 基础 GET + 多字段提取

```python
# ---------- 1. 基础 GET + 多字段提取 ----------
uid, uname, city = run_test(
    "1. 获取 1 号用户",
    get("https://jsonplaceholder.typicode.com/users/1"),
    "id",
    "name",
    "address.city"
)
```

### 2. 字符串插值 URL

```python
# ---------- 2. 字符串插值 URL ----------
run_test(
    "2. 用 uid 查询该用户帖子列表",
    get(f"https://jsonplaceholder.typicode.com/users/{uid}/posts")
)
```

### 3. 获取单个列表元素字段

```python
# ---------- 3. 获取单个列表元素字段 ----------
body, title = run_test(
    "3. 用 uid 查询该用户帖子列表",
    get(f"https://jsonplaceholder.typicode.com/users/{uid}/posts"),
    "0.body",
    "0.title"
)
```

### 4. POST 创建资源 + 提取值

```python
# ---------- 4. POST 创建资源 + 提取值 ----------
new_post = run_test(
    "4. 新建帖子",
    post(
        "https://jsonplaceholder.typicode.com/posts",
        body={
            "title":"帖子",
            "body":"由脚本创建",
            "userId":uid
        }
    ),
    "id"
)
```

### 5. PUT 修改

```python
# ---------------- 5. PUT 修改----------------
run_test(
    "5. 修改刚才的帖子",
    put(
        "https://jsonplaceholder.typicode.com/posts/1",
        body={
            "id":new_post,
            "title":"已更新",
            "body":"新内容",
            "userId":uid
        }
    )
)
```

### 6. PATCH 修改

```python
# ---------------- 6. PATCH 修改----------------
run_test(
    "6. 修改刚才的帖子",
    patch(
        "https://jsonplaceholder.typicode.com/posts/1",
        body={
            "id":new_post,
            "title":"已更新",
            "body":"新内容",
            "userId":uid
        }
    )
)
```

### 7. 自定义头

```python
# ---------------- 7. 自定义头 ----------------
run_test(
    "7. 带自定义头查询帖子详情",
    get(
        "https://jsonplaceholder.typicode.com/posts/1",
        headers={"X-Source": "APITEST-demo"}
    )
)
```

### 8. OPTION

```python
# ---------------- 8. OPTION ----------------
run_test(
    "8. 带自定义头查询帖子详情",
    option(
        "https://jsonplaceholder.typicode.com/posts/1"
    )
)
```

### 9. DELETE 删除

```python
# ---------- 9. DELETE 删除 ----------
run_test(
    "9. 删除帖子",
    delete(f"https://jsonplaceholder.typicode.com/posts/{new_post}")
)
```

### 10. 预期 404：资源不存在

```python
# ---------- 10. 预期 404：资源不存在 ----------
run_test(
    "10. 再次查询应返回 404（预期失败）",
    get(f"https://jsonplaceholder.typicode.com/posts/{new_post}", should_fail=True)
)
```

### 11. 深路径 + 多字段同时提取

```python
# ---------- 11. 深路径 + 多字段同时提取 ----------
lat, lng = run_test(
    "11. 提取用户地址坐标",
    get("https://jsonplaceholder.typicode.com/users/1"),
    "address.geo.lat", "address.geo.lng"
)
```

### 12. 输出自定义键值对信息

```python
# ---------- 12. 输出自定义键值对信息 ----------
print_info(
    "输出键值对信息",
    {
        "用户 ID": uid,
        "用户姓名": uname,
        "所在城市": city,
        "帖子 ID": new_post,
        "用户的文章标题": title,
        "用户的文章内容": body,
        "纬度": lat,
        "经度": lng
    }
)
```

### 13. 显示测试结果汇总

```python
# ---------- 13. 显示测试结果汇总 ----------
show_result()

# 可以自定义标题
show_result("API测试结果汇总")
```

## 函数参考

### HTTP 方法函数

所有 HTTP 方法函数都支持以下参数：

- •`url`: 请求的 URL
- •`body`: 请求体（对于 POST、PUT、PATCH）
- •`headers`: 自定义请求头
- •`should_fail`: 布尔值，表示是否期望请求失败

### run_test 函数

```python
run_test(description, response, *extract_paths)
```

- •`description`: 测试描述，显示在输出中
- •`response`: HTTP 方法函数返回的响应元组
- •`extract_paths`: 可变参数，指定要从响应中提取的字段路径

### print_info 函数

```python
print_info(title, info_dict)
```

- •`title`: 信息面板的标题
- •`info_dict`: 要显示的键值对字典

### show_result 函数

```python
show_result(title="测试结果汇总")
```

- •`title`: 汇总面板的标题（可选，默认为"测试结果汇总"）
- 自动收集所有 `run_test` 函数的测试结果
- 以表格形式显示测试描述和结果（成功/失败）
- 成功显示为绿色✅，失败显示为红色❌
- 自动统计成功和失败的数量
- 调用后自动清空测试结果记录

### clear_test_results 函数

```python
clear_test_results()
```

- 手动清空测试结果记录
- 用于分组测试或重置测试状态

## 响应数据提取语法

使用点号表示法访问嵌套的 JSON 字段：

- •`"id"` - 提取顶层的 id 字段
- •`"address.city"` - 提取 address 对象中的 city 字段
- •`"address.geo.lat"` - 提取嵌套的经纬度信息
- `"0.address.geo.lat"` - 提取单个列表元素的嵌套的经纬度信息

## 错误处理

框架会自动处理以下情况：

- •非 2xx 状态码（除非使用 `should_fail=True`）
- •JSON 解析错误
- •网络连接异常
- •数据提取路径不存在

### 错误显示格式

所有错误情况都使用统一的显示格式：

```json
{
  "error": "错误描述",
  "details": "错误详情或响应内容"
}
```

- 当 `should_fail=True` 但请求成功时，显示为失败并包含完整的响应内容
- 错误详情会自动解析 JSON 并美化显示
- 异常情况也会使用相同的格式

## 运行测试

保存测试文件后，直接运行：

```bash
uv run my_test.py
```

框架会自动执行所有测试步骤，并在终端中显示格式化的结果。

## 最佳实践

1. **清晰的描述**: 为每个测试步骤提供有意义的描述，便于在结果汇总中识别
2. **错误处理**: 合理使用 `should_fail` 参数来验证错误场景
3. **结果验证**: 使用 `print_info` 输出关键测试数据用于验证
4. **结果汇总**: 使用 `show_result` 函数查看所有测试的执行结果
5. **循环测试**: 在循环中使用动态描述确保每个测试有唯一标识
6. **分组测试**: 使用 `clear_test_results` 进行测试分组和结果管理

## 循环测试示例

```python
# 批量测试多个用户
user_ids = [1, 2, 3, 999]
for user_id in user_ids:
    should_fail = (user_id == 999)
    run_test(
        f"测试用户 {user_id}",
        get(f"https://api.example.com/users/{user_id}", should_fail=should_fail)
    )

show_result("用户API批量测试结果")
```

## 导入语句

```python
from PAT import (
    get, post, put, patch, delete, option,  # HTTP方法
    run_test, print_info, show_result, clear_test_results  # 测试函数
)
```
