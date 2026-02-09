# Laravel Headless CMS 项目规划书

## 1. 项目概述

**项目名称**：Laravel Headless CMS

**项目类型**：内容管理系统（Headless）

**项目目标**：

* 基于 Laravel 构建一个可扩展、组件化的 Headless CMS
* 提供内容模型管理、权限控制、页面装配和组件化渲染
* 提供标准化 REST / GraphQL API，支持多前端技术栈（React / Vue / Blade / Flutter 等）
* 支持版本管理、多语言、多站点及插件扩展

**核心价值**：

* 高复用性、组件化的内容与渲染体系
* 可扩展的权限与发布机制
* 前后端解耦，支持任意前端消费

---

## 2. 核心模块设计

### 2.1 内容模块（Content）

**职责**：管理所有 CMS 内容数据

* 数据模型：
  * Page / Layout / Component / Block
  * 数据字段类型：文本、图片、JSON、数组、引用
  * 关系映射：Page → Layout → Slots → Components / Blocks
* 功能：
  * CRUD API
  * 版本管理（Draft / Published / Archived）
  * 数据验证（Laravel FormRequest / Validator）

### 2.2 视图模块（View）

**职责**：管理组件、布局、页面渲染逻辑

* 核心概念：
  * Block → Component → Layout → Page → Slot
  * Render Tree 构建逻辑
* 功能：
  * Blade 模板 / Component 注册
  * Layout slot 配置
  * Page 装配与 Render Tree JSON 输出
* 输出：
  * 前端或移动端可直接解析的 JSON / GraphQL

### 2.3 权限与控制模块（Auth / Control）

**职责**：用户管理、角色权限与安全控制

* 用户管理：
  * Laravel Auth / Sanctum / Passport 支持
  * 用户注册 / 登录 / 登出
* 角色管理：
  * Admin / Editor / Viewer
  * CRUD 权限控制、发布权限
* 中间件：
  * API 权限校验（Laravel Middleware）
  * 日志 / 审计 / 异常处理

---

## 3. 开发阶段规划

| 阶段             | 核心目标                | 产出物                                                                | 备注             |
| -------------- | ------------------- | ------------------------------------------------------------------ | -------------- |
| 0. Starter 框架  | Laravel 项目骨架 + 基础配置 | Laravel 项目结构、数据库迁移、API 框架、测试环境                                     | 可运行基础服务        |
| 1. Auth & 权限控制 | 用户和角色权限管理           | Laravel Auth / Sanctum 或 Passport 集成、用户系统、权限中间件、审计基础               | 为 CMS 提供安全基础   |
| 2. CMS 核心功能    | 内容与渲染管理             | Page/Layout/Component/Block CRUD API、Render Tree JSON 输出、版本管理、发布流程 | 前端可通过 API 构建页面 |
| 3. 扩展功能        | 多语言、多站点、插件机制        | Laravel Localization、多站点配置、插件 API                                  | 可在阶段 2 后逐步实现   |

---

## 4. 技术选型（Laravel 栈）

| 层级      | 技术 / 框架                         | 说明                                            |
| ------- | ------------------------------- | --------------------------------------------- |
| 后端      | Laravel 10                      | REST / GraphQL API 服务、Eloquent ORM、Artisan 工具 |
| 数据库     | MySQL / PostgreSQL              | 支持关系映射，使用 Eloquent 模型                         |
| 认证      | Laravel Sanctum 或 Passport      | 用户认证与 API Token / OAuth2 支持                   |
| 前端消费    | Vue 3 / React / Blade / Flutter | 消费 Render Tree API 渲染页面                       |
| 模板      | Laravel Blade                   | Block / Component / Layout 的模板渲染              |
| 队列 / 缓存 | Laravel Queue / Redis           | 异步任务、缓存内容或 Render Tree                        |
| 测试      | PHPUnit / Pest                  | 单元测试 / 集成测试                                   |
| 文档      | Swagger / Scribe                | API 文档生成                                      |

---

## 5. API 核心设计

### 5.1 内容 API

* Page / Layout / Component / Block CRUD
* Slot 配置与组件填充
* Render Tree JSON 输出

示例 Render Tree：

```json
{
  "id": "home",
  "kind": "page",
  "layout_ref": "html-layout-default",
  "slots": {
    "header": [
      {"id": "navbar", "kind": "component", "template_ref": "NavbarComponent", "props": {}}
    ],
    "main": [
      {"id": "hero1", "kind": "component", "template_ref": "HeroComponent", "props": {"title": "Welcome"}}
    ],
    "footer": [
      {"id": "footer1", "kind": "component", "template_ref": "FooterComponent"}
    ]
  },
  "free_components": [],
  "page_capabilities": {"route": "/", "publish_state": "published", "seo": {"title": "Home"}}
}
```

### 5.2 用户与权限 API

* POST /auth/login
* POST /auth/logout
* GET /users
* GET /roles / POST /roles / PATCH /roles
* 权限中间件（Laravel Middleware）

### 5.3 发布与 Workflow API

* POST /pages/publish
* POST /pages/unpublish
* GET /pages/drafts
* 审批 / 审核状态接口

---

## 6. 项目交付物

1. Laravel 项目代码仓库
2. 数据库 migration / seed 脚本
3. API 文档（Swagger / Scribe）
4. Mock 数据和 Render Tree 样例
5. 用户权限管理系统
6. 开发测试指南与 CI/CD 配置

---

## 7. 风险与注意事项

* **权限安全**：阶段 1 必须稳定，否则 CMS 功能不可上线
* **Render Tree 复杂度**：深层嵌套组件可能影响前端解析性能
* **模板语言限制**：Blade 对 slot 支持有限，需用 Component / Include 组合实现
* **扩展性**：多语言、多站点、插件接口需在初期设计时标准化

---

## 8. 项目时间与里程碑（示例）

| 阶段            | 时间    | 主要里程碑                                               |
| ------------- | ----- | --------------------------------------------------- |
| 0. Starter 框架 | 2 周   | Laravel 项目骨架、数据库、API 框架可运行                          |
| 1. Auth & 权限  | 2-3 周 | 用户注册、登录、角色权限可用，中间件生效                                |
| 2. CMS 核心功能   | 4-6 周 | Page/Layout/Component/Block API、Render Tree 输出、版本管理 |
| 3. 扩展功能       | 2-4 周 | 国际化、多站点、插件机制、前端消费端完成集成                              |

---

✅ **总结**

Laravel 版本的 Headless CMS 遵循以下原则：

1. **阶段开发**：Starter → Auth → CMS → 扩展
2. **组件化 + Render Tree**：Page / Layout / Component / Block → JSON 输出
3. **权限先行**：确保 CMS 安全可靠
4. **前后端解耦**：前端可用任意框架解析 Render Tree
