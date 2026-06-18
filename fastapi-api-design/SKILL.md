---
name: fastapi-api-design
description: FastAPI 接口设计规范：RESTful 请求方式、动作词路径、按 HTTP 方法区分 query/body 传参、Pydantic schema 复用、前端迁移文档同步。
---

# FastAPI API Design

用于在 FastAPI 项目中设计、重构或评审接口时，保持接口风格一致，特别适用于本项目这类需要”动作词路径 + RESTful 请求方式 + 按 HTTP 方法选择 query/body 传参”的后端接口。

## 何时使用

- 新增 FastAPI 接口。
- 重构已有接口路径、请求方式或请求体。
- 将路径参数迁移到 query 或 body。
- GET / DELETE 使用 query 传参，POST / PUT / PATCH 使用 body 传参。
- 为前端输出接口迁移文档。
- 检查接口是否符合当前项目约定。

## 核心规范

### 1. 请求方式必须符合 RESTful 语义

不能为了统一处理而把所有接口都改成 `POST`。请求方法按操作语义选择：

| 操作 | HTTP 方法 | 示例 |
|------|-----------|------|
| 创建资源 | `POST` | `POST /experts/create` |
| 获取列表 | `GET` | `GET /experts/list` |
| 获取详情 | `GET` | `GET /experts/get` |
| 更新资源 | `PUT` | `PUT /experts/update` |
| 删除资源 | `DELETE` | `DELETE /experts/delete` |
| 非 CRUD 动作 | `POST` | `POST /experts/reset-password` |

判断原则：

- 获取信息用 `GET`。
- 删除信息用 `DELETE`。
- 更新信息用 `PUT`。
- 创建信息用 `POST`。
- 重置密码、登录、刷新 token、上传文件等动作型接口一般用 `POST`。

### 2. 路径可以保留动作词，但方法不能失去语义

当前项目允许在接口路径末尾追加动作词，用来显式区分接口：

```http
POST   /experts/create
GET    /experts/list
GET    /experts/get
PUT    /experts/update
DELETE /experts/delete
POST   /experts/reset-password
```

注意：动作词路径不代表所有请求都使用 `POST`。路径负责表达业务动作，HTTP 方法负责表达操作语义。

### 3. 移除路径参数

不要使用：

```http
GET /experts/{expert_id}
PUT /experts/{expert_id}
DELETE /experts/{expert_id}
```

改为：

```http
GET    /experts/get?expert_id=expert_2064000000000000001
PUT    /experts/update
DELETE /experts/delete?expert_id=expert_2064000000000000001
```

FastAPI 写法示例：

```python
from fastapi import Query

@router.get("/get")
async def get_expert(expert_id: str = Query(...)):
    ...

@router.delete("/delete")
async def delete_expert(expert_id: str = Query(...)):
    ...
```

### 4. 按 HTTP 方法区分 query / body 传参

遵循 HTTP 规范，避免 nginx 和标准 HTTP 实现无法识别 `GET` / `DELETE` 请求 body 的问题。

| HTTP 方法 | 传参方式 | 说明 |
|-----------|----------|------|
| `GET` | query | 获取信息，参数放在 URL 中 |
| `DELETE` | query | 删除信息，参数放在 URL 中 |
| `POST` | body | 创建资源或动作型接口 |
| `PUT` | body | 全量更新资源 |
| `PATCH` | body | 部分更新资源 |

#### GET / DELETE 示例（query 传参）

```http
GET /experts/list?page_no=1&page_size=20
DELETE /experts/delete?expert_id=expert_2064000000000000001
```

FastAPI 写法：

```python
from fastapi import Query

@router.get("/list")
async def list_experts(
    page_no: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
):
    ...

@router.delete("/delete")
async def delete_expert(expert_id: str = Query(...)):
    ...
```

#### POST / PUT / PATCH 示例（body 传参）

```http
POST /experts/create
Content-Type: application/json

{
  "name": "张三",
  "title": "教授"
}
```

FastAPI 写法：

```python
from fastapi import Body

@router.post("/create")
async def create_expert(data: ExpertCreate):
    ...

@router.put("/update")
async def update_expert(data: ExpertUpdate):
    ...
```

### 5. 不要使用裸 `dict` 接收请求体

避免：

```python
async def update_expert(data: dict):
    ...
```

原因：

- 无法清晰表达字段结构。
- OpenAPI 文档不友好。
- 校验弱，容易产生隐式约定。
- 前端对接不明确。

优先使用：

1. 已有 Pydantic schema。
2. 必要时在原 schema 上增加字段。
3. 单字段 body 使用 `Body(..., embed=True)`。

### 6. 不要新增没必要的 schema 类

如果只是为了把 `expert_id` 加到某个业务请求中，不要额外创建 `ExpertUpdateBody`、`PasswordResetBody` 这类包装类。

优先把新增字段合入原业务 schema：

```python
class ExpertUpdate(BaseModel):
    expert_id: str = Field(..., description="专家ID")
    phone: str | None = Field(None, description="手机号")
    title: str | None = Field(None, description="职称")
    role: str | None = Field(None, description="角色")
    organization: str | None = Field(None, description="所属单位")


class PasswordReset(BaseModel):
    expert_id: str = Field(..., description="专家ID")
    new_password: str = Field(..., description="新密码")
```

只有当请求体结构确实独立、可复用，或语义明显不同，才新增 schema。

### 7. Schema 命名保持业务简洁

避免在类型名里加入不必要的场景词，例如：

```python
ExpertAdminCreate
ExpertAdminUpdate
ExpertUpdateBody
PasswordResetBody
```

优先使用：

```python
ExpertCreate
ExpertUpdate
PasswordReset
```

如果某个 schema 只服务当前明确业务，也应优先保持名称简洁，避免过度拆分。

## 推荐 FastAPI 路由模板

```python
from fastapi import Query, Body

@router.post("/create", response_model=ResponseWrapper[ExpertProfile])
async def create_expert(data: ExpertCreate):
    ...


@router.get("/list", response_model=ResponseWrapper[PaginatedData[ExpertListItem]])
async def list_experts(
    page_no: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
):
    ...


@router.get("/get", response_model=ResponseWrapper[ExpertProfile])
async def get_expert(expert_id: str = Query(...)):
    ...


@router.put("/update", response_model=ResponseWrapper[ExpertProfile])
async def update_expert(data: ExpertUpdate):
    ...


@router.delete("/delete", response_model=ResponseWrapper[None])
async def delete_expert(expert_id: str = Query(...)):
    ...


@router.post("/reset-password", response_model=ResponseWrapper[None])
async def reset_password(data: PasswordReset):
    ...
```

## 前端迁移文档要求

接口变更后，必须同步输出或更新前端迁移文档，至少包含：

1. 接口清单。
2. 旧接口到新接口对照表。
3. 请求方法、路径、传参方式（query / body）。
4. TypeScript interface。
5. 前端封装示例。
6. 明确标注：不使用路径参数；`GET` / `DELETE` 使用 query，`POST` / `PUT` / `PATCH` 使用 body。

前端封装示例：

```ts
export async function listExperts(page_no = 1, page_size = 20) {
  return request.get('/experts/list', { params: { page_no, page_size } });
}

export async function getExpert(expert_id: string) {
  return request.get('/experts/get', { params: { expert_id } });
}

export async function updateExpert(data: ExpertUpdate) {
  return request.put('/experts/update', data);
}

export async function deleteExpert(expert_id: string) {
  return request.delete('/experts/delete', { params: { expert_id } });
}
```

## 检查清单

完成接口修改后逐项检查：

- [ ] 获取接口使用 `GET`，传参使用 query。
- [ ] 删除接口使用 `DELETE`，传参使用 query。
- [ ] 更新接口使用 `PUT`，传参使用 body。
- [ ] 创建和动作型接口使用 `POST`，传参使用 body。
- [ ] 没有路径参数，例如 `/{id}`、`/{expert_id}`。
- [ ] `GET` / `DELETE` 不使用 body 传参。
- [ ] `POST` / `PUT` / `PATCH` 不使用 query 传参。
- [ ] 没有使用裸 `dict` 接收请求体。
- [ ] 没有新增不必要的 `*Body` 包装 schema。
- [ ] 新增字段优先合入原业务 schema。
- [ ] 前端迁移文档已同步更新。
- [ ] 运行 Python 语法检查或相关测试。

## 常用验证命令

```bash
uv run --no-sync python -m py_compile app/controllers/<router>.py app/schemas/<schema>.py
```

如项目有对应测试，再运行相关测试。