# Laravel Headless CMS 详细设计文档

> 基于《Laravel Headless CMS 项目规划书》展开的详细设计，覆盖架构、数据模型、接口、流程、权限与运维等核心内容。

## 1. 目标与范围

**目标**：细化 Headless CMS 的系统设计，使开发、测试与交付有清晰的实现路径。  
**范围**：
- 业务对象（Page/Layout/Component/Block/Slot）定义
- API 设计（REST/可扩展 GraphQL）
- 权限模型与工作流
- 版本管理与发布机制
- 数据库设计（MySQL/PostgreSQL）
- 系统模块拆分与目录结构建议
- 非功能性需求（性能、安全、可维护性）

## 2. 架构设计

### 2.1 总体架构

```
┌─────────────────────────────────────────────┐
│                Client Apps                  │
│  React / Vue / Flutter / Blade / Others      │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│             Headless CMS API                │
│  REST / GraphQL / Auth / Workflow / Render  │
└─────────────────────────────────────────────┘
        │                     │
        ▼                     ▼
┌───────────────────┐  ┌─────────────────────┐
│  Database (SQL)   │  │   Cache / Queue     │
│  MySQL/PostgreSQL │  │   Redis / Queue     │
└───────────────────┘  └─────────────────────┘
```

### 2.2 分层结构（Laravel）

- **Presentation**：Controllers / Requests / Resources
- **Application**：Services / UseCases / Workflow
- **Domain**：Entities / Value Objects / Policies
- **Infrastructure**：Repositories / DB / Cache / Queue

目录建议（简化）：

```
app/
  Http/Controllers/
  Http/Requests/
  Http/Resources/
  Models/
  Services/
  Policies/
  Enums/
  DTO/
  Repositories/
  Workflows/
  Render/
```

## 3. 核心领域模型

### 3.1 内容模型

| 实体 | 描述 | 主要字段 |
| --- | --- | --- |
| Page | 页面实体 | id, name, slug, layout_id, status, locale, site_id, seo_json |
| Layout | 页面布局模板 | id, name, slots_json |
| Component | 组件定义 | id, name, template_ref, schema_json |
| Block | 可复用内容块 | id, name, type, data_json |
| Slot | 布局中的容器 | name, allowed_components, max_items |

### 3.2 关系设计

- **Page** 关联 **Layout**
- **Layout** 定义多个 **Slot**
- **Slot** 绑定多个 **Component/Block**
- **Component** 可引用 **Block** 数据

关系示例：

```
Page (home)
 └─ Layout (default)
     ├─ Slot: header → Component: Navbar
     ├─ Slot: main   → Component: Hero, Block: Promo
     └─ Slot: footer → Component: Footer
```

### 3.3 JSON Schema 设计

**Component Schema** (schema_json)：

```json
{
  "fields": [
    {"name": "title", "type": "string", "required": true},
    {"name": "image", "type": "asset", "required": false},
    {"name": "cta", "type": "object", "schema": {
      "label": {"type": "string"},
      "url": {"type": "string"}
    }}
  ]
}
```

**Page Slot Data** (content_json)：

```json
{
  "slots": {
    "header": [{"ref": "component:navbar", "props": {}}],
    "main": [{"ref": "component:hero", "props": {"title": "Hello"}}],
    "footer": [{"ref": "component:footer", "props": {}}]
  }
}
```

## 4. 数据库设计（初稿）

### 4.1 表结构概览

- pages
- layouts
- components
- blocks
- page_versions
- assets
- users / roles / permissions
- audit_logs

### 4.2 示例建表字段

**pages**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint | 主键 |
| name | string | 页面名称 |
| slug | string | 路由标识 |
| layout_id | bigint | 关联 layout |
| content_json | json | slot 数据 |
| status | enum | draft/published/archived |
| locale | string | 语言 |
| site_id | bigint | 站点 |
| seo_json | json | SEO 配置 |
| created_by | bigint | 创建人 |
| updated_by | bigint | 更新人 |
| created_at | timestamp | 创建时间 |
| updated_at | timestamp | 更新时间 |

**page_versions**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint | 主键 |
| page_id | bigint | 页面 ID |
| version | int | 版本号 |
| content_json | json | 版本数据 |
| status | enum | draft/published |
| created_by | bigint | 创建人 |
| created_at | timestamp | 创建时间 |

## 5. API 详细设计

### 5.1 内容管理 API

#### Page
- `GET /api/pages`：分页查询
- `POST /api/pages`：创建页面
- `GET /api/pages/{id}`：获取详情
- `PATCH /api/pages/{id}`：更新页面
- `DELETE /api/pages/{id}`：删除页面

#### Layout
- `GET /api/layouts`
- `POST /api/layouts`
- `PATCH /api/layouts/{id}`

#### Component
- `GET /api/components`
- `POST /api/components`

#### Render Tree
- `GET /api/pages/{id}/render-tree`

返回示例：

```json
{
  "id": "home",
  "kind": "page",
  "layout_ref": "default",
  "slots": {
    "header": [
      {"id": "navbar", "kind": "component", "template_ref": "Navbar", "props": {}}
    ],
    "main": [
      {"id": "hero1", "kind": "component", "template_ref": "Hero", "props": {"title": "Welcome"}}
    ]
  }
}
```

### 5.2 Auth / 权限 API

- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/users`
- `GET /api/roles`
- `POST /api/roles`

### 5.3 Workflow / 发布 API

- `POST /api/pages/{id}/publish`
- `POST /api/pages/{id}/unpublish`
- `GET /api/pages/drafts`

## 6. 权限与工作流设计

### 6.1 角色模型

- Admin：全权限
- Editor：内容 CRUD、草稿、发布申请
- Viewer：只读

### 6.2 权限矩阵（示例）

| 操作 | Admin | Editor | Viewer |
| --- | --- | --- | --- |
| 创建页面 | ✅ | ✅ | ❌ |
| 编辑页面 | ✅ | ✅ | ❌ |
| 发布页面 | ✅ | ❌ | ❌ |
| 删除页面 | ✅ | ❌ | ❌ |
| 查看页面 | ✅ | ✅ | ✅ |

### 6.3 工作流状态

- Draft（草稿）
- Review（审核）
- Published（已发布）
- Archived（归档）

## 7. Render Tree 生成逻辑

1. 查询 Page → Layout → Slots
2. 解析 content_json
3. 组装 Render Tree JSON
4. 可选：缓存 Render Tree（按 page_id + locale + version）

## 8. 非功能性设计

### 8.1 性能

- Render Tree 缓存（Redis）
- Page/Component 数据缓存
- 静态发布快照（可选）

### 8.2 安全

- API Token 认证（Sanctum/Passport）
- 权限中间件
- SQL 注入与 XSS 防护（Eloquent + Validator）

### 8.3 扩展性

- 多语言：locale 字段 + 语言包
- 多站点：site_id 维度隔离
- 插件机制：Hook/事件系统

## 9. 实施计划（建议）

1. Starter：基础 Laravel 项目 + API 框架
2. Auth：角色/权限 + 用户管理
3. CMS 核心：Page/Layout/Component/Block + Render Tree
4. 扩展：多语言、多站点、插件

## 10. 交付物清单

- 详细设计文档（本文件）
- 数据库 schema 设计
- API 说明与 Mock 示例
- 权限矩阵与工作流说明

---

**版本记录**

| 版本 | 日期 | 说明 |
| --- | --- | --- |
| v1.0 | 2025-02-09 | 初稿 |
