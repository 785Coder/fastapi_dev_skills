---
name: fastapi-doc-generator
description: 从 FastAPI 控制器代码自动生成面向前端的 Markdown 接口文档。解析路由、参数、响应格式，输出标准化接口文档。
---

# FastAPI 接口文档生成器

自动读取 FastAPI 控制器（router）代码，提取接口定义，生成面向前端的 Markdown 接口文档。

## 使用方式

直接调用 `/fastapi-doc-generator` 或描述需求如 "生成接口文档"。

## 工作流程

### 1. 扫描目标文件

读取用户指定的 FastAPI 控制器文件（如 `src/controllers/qa_database.py`）。

### 2. 解析接口定义

逐个提取每个路由函数的关键信息：

| 字段 | 提取来源 |
|------|----------|
| **Method** | `@router.get/post/put/delete` 装饰器 |
| **Path** | 装饰器 `path` 参数 |
| **描述** | 装饰器 `description` 参数或函数 docstring |
| **Auth** | 是否有 `Depends(get_user_by_token)` 等依赖 |
| **Query 参数** | 函数签名中无 `Body()` 注解的参数 |
| **Body 参数** | 函数签名中有 `Body(...)` 注解的参数 |
| **响应格式** | 返回 `ReadyResponse` 的字段推断，或 `StreamingResponse` 标记 |

### 3. 推断字段类型和必填性

- 类型从函数签名注解提取（`str`、`int`、`list[dict]` 等）
- 有 `= Body(...)` 或 `= Body(..., embed=True)` 视为必填
- 有默认值（如 `= Body("")`）视为可选

### 4. 整理通用错误响应

如果控制器中使用统一的错误返回格式（如 `ReadyResponse`），在文档末尾附加通用错误响应说明。

### 5. 生成 Markdown 文档

输出格式要求：

- 顶部包含 `# 模块接口文档` 和更新时间
- 先写认证方式说明（如有）
- 每个接口用二级标题（`##`）分节
- 接口表格包含：Method、Path、Auth
- 参数表格包含：参数名、类型、必填、说明
- 提供标准 Response JSON 示例
- SSE 流接口需说明 `tag` 枚举和流格式
- 二进制流接口（如 Word/Excel 导出）需说明 `Content-Type` 和 `Content-Disposition`

### 6. 保存文件

保存到 `docs/api/` 目录下，文件名格式：

```
yyymmdd-<module_name>.md
```

例如：`20260610-qa_database.md`

如果目录不存在则自动创建。

## 输出示例

```markdown
# 数据库问答模块接口文档

> 更新时间：2026-06-10

## 认证方式

Authorization: Bearer <token>

## 1. 发起对话

| 项目 | 内容 |
|------|------|
| **Method** | `POST` |
| **Path** | `/qa_database/chat` |
| **Auth** | 需要 |

**Body 参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `history_id` | `string` | 否 | 已有历史记录 ID |
| `content` | `string` | 是 | 用户输入内容 |

**Response**
```json
{
  "detail": "success",
  "status_code": 200,
  "data": "history-xxx"
}
```
```

## 注意事项

- 只提取对外暴露的接口（有路由装饰器的函数）
- 忽略内部辅助函数和未装饰的函数
- 如果返回格式不明确，基于代码中的 `return` 语句推断
- 文件命名必须遵循 `yyymmdd-name.md` 格式
