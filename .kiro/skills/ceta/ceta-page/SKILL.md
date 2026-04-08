---
name: ceta-page
description: >
  CETA 页面配置管理。FormEntityPage 定义列表页、详情页、仪表盘等页面的 UI 结构。
  同时支持从 HTML 或自然语言描述生成列表页 schemaJson。
  页面通过 schemaJson 中的 fields 数组定义 UI 组件树。
  列表页核心组件是 Table，需要包含 ACTION_COLUMN 定义操作按钮。
license: MIT
metadata:
  openclaw:
    requires:
      env: ["CETA_API_BASE", "CETA_API_TOKEN"]
    version: "1.0.0"
    author: "CETA Team"
    tags: ["page", "ceta", "ui", "table", "schema", "conversion"]
    source: "builtin"
    dependencies:
      - "ceta/ceta-api"
      - "ceta/ceta-pbc"
      - "ceta-basic"
---

# CETA 页面管理

## 概述

FormEntityPage 定义 CETA 平台上的页面配置，包括列表页、详情页、仪表盘等。
页面通过 schemaJson 描述 UI 结构，通过 schemaId 在 PBC 的 routesJson 中引用。

本 SKILL 同时负责：
1. 通过 MCP API 创建/修改/删除页面资源
2. 从 HTML 或自然语言描述生成列表页 schemaJson

## 数据模型

```
FormEntityPage
├── id: number
├── schemaId: string        # 页面标识符（PBC 内唯一）
├── pbcId: number           # 所属 PBC ID
├── name: string            # 页面名称
├── description: string
├── schemaJson: string      # 页面 UI 配置（JSON 字符串）
└── contentVersion: string
```

## MCP 工具

- `form__form_entity_page__create` — 创建页面
- `form__form_entity_page__get` — 获取页面详情
- `form__form_entity_page__list` — 列出页面
- `form__form_entity_page__list_by_pbc_id` — 按 PBC 列出页面
- `form__form_entity_page__update` — 更新页面
- `form__form_entity_page__delete` — 删除页面

## HTML / 自然语言 → schemaJson 转换

当输入是 HTML 或自然语言描述时，本 SKILL 负责生成：

**FormEntityPage schemaJson** → 写入 `ceta-workspace/{业务名称}/{业务名称}-page.json`

### 需要读取的参考文档

| 什么时候读     | 读什么                                          |
| -------------- | ----------------------------------------------- |
| 每次都读       | `skills/ceta/references/schema-rules.md`        |
| 输入是 HTML 时 | `skills/ceta/references/html-mapping-rules.md`  |
| 输入含样式时   | `skills/ceta/references/style-mapping-rules.md` |
| Table 结构详情 | `skills/ceta/references/components/display/Table.md` |

### 转换流程

#### 1. 提取列表信息

从输入中提取：
- 页面标题
- 表格列（列名、字段 token、数据类型）
- 操作按钮（编辑、查看、删除）
- 筛选字段
- 关联的 FormEntity token

#### 2. 构造 columnDefs

根据每列的数据类型选择列类型：

| 数据类型 | columnDef type                           |
| -------- | ---------------------------------------- |
| 文本     | TEXT_COLUMN                              |
| 日期     | DATE_COLUMN                              |
| 数字     | NUMBER_COLUMN                            |
| 头像     | AVATAR_COLUMN                            |
| 关联引用 | ACL_COLUMN                               |
| 开关     | SWITCH_COLUMN                            |
| 状态枚举 | STATUS_COLUMN                            |
| 操作     | ACTION_COLUMN（固定最后，pinned: right） |

#### 3. 组装 schemaJson

列表页的固定结构：`{ form: { title }, fields: [Card > Table] }`

Card 和 Table 都需要 `style: { "height": "100%" }`。


## 列表页 schemaJson 模板（重要）

列表页的核心是 Table 组件。必须包含在 Card 组件内，且需要 ACTION_COLUMN 定义操作按钮。

```json
{
  "form": {
    "title": "数据列表"
  },
  "fields": [
    {
      "component": "Card",
      "style": { "height": "100%" },
      "fields": [
        {
          "component": "Table",
          "id": "data-list-table",
          "style": { "height": "100%" },
          "componentProps": {
            "dataSource": {
              "token": "form-entity-token-here"
            },
            "columnDefs": [
              {
                "type": "TEXT_COLUMN",
                "colId": "fieldToken1",
                "field": "fieldToken1",
                "headerName": "字段名1"
              },
              {
                "type": "SELECT_COLUMN",
                "colId": "fieldToken2",
                "field": "fieldToken2",
                "headerName": "字段名2"
              },
              {
                "type": "TIME_COLUMN",
                "colId": "dateField",
                "field": "dateField",
                "headerName": "日期字段"
              },
              {
                "type": "NUMBER_COLUMN",
                "colId": "numberField",
                "field": "numberField",
                "headerName": "数字字段"
              },
              {
                "type": "ACTION_COLUMN",
                "width": 240,
                "pinned": "right",
                "cellRendererParams": {
                  "displayLikeButton": true,
                  "showIcon": true,
                  "actions": [
                    {
                      "type": "VIEW",
                      "href": "/form/:id/view"
                    },
                    {
                      "type": "EDIT",
                      "href": "/form/form-entity-token-here/layout/edit/:id"
                    },
                    {
                      "type": "DELETE"
                    }
                  ]
                }
              }
            ],
            "addButtonHref": "/form/form-entity-token-here/layout/new",
            "pinnedFilter": ["fieldToken1", "fieldToken2"],
            "pagination": true,
            "withCard": true
          }
        }
      ]
    }
  ]
}
```

### 列类型对照表

| 表单字段 type | Table columnDef type | 说明 |
|--------------|---------------------|------|
| TEXT_BOX | TEXT_COLUMN | 文本列 |
| TEXTAREA | TEXTAREA_COLUMN | 多行文本列 |
| NUMBER | NUMBER_COLUMN | 数字列 |
| DATE / TIME | TIME_COLUMN | 日期时间列 |
| SELECT | SELECT_COLUMN | 下拉选择列 |
| SWITCH | TEXT_COLUMN | 开关（显示为文本） |
| ACL | ACL_COLUMN | 关联引用列 |
| UPLOAD | UPLOAD_COLUMN | 上传文件列 |
| — | ACTION_COLUMN | 操作列（VIEW/EDIT/DELETE/APPROVAL） |

### ACTION_COLUMN 操作类型

| type | 说明 | href 模板 |
|------|------|-----------|
| VIEW | 查看详情 | `/form/:id/view` |
| EDIT | 编辑 | `/form/{formEntityToken}/layout/{editLayoutToken}/:id` |
| DELETE | 删除 | 无需 href |
| APPROVAL | 审批 | `/flow/:flowInstanceId/approval` |

### 关键配置项

- `addButtonHref` — 新增按钮的跳转地址，格式：`/form/{formEntityToken}/layout/{newLayoutToken}`
- `pinnedFilter` — 默认显示的筛选字段列表
- `pagination` — 是否分页（建议开启）
- `withCard` — 表格是否带卡片样式
- `dataSource.token` — 数据源表单的 token

### 完整列表页示例

```json
{
  "form": {
    "title": "用户列表"
  },
  "fields": [
    {
      "component": "Card",
      "style": { "height": "100%" },
      "fields": [
        {
          "component": "Table",
          "id": "user-form-table",
          "style": { "height": "100%" },
          "componentProps": {
            "dataSource": { "token": "user-form" },
            "columnDefs": [
              {
                "type": "AVATAR_COLUMN",
                "colId": "avatar",
                "field": "avatar",
                "headerName": "头像",
                "cellRendererParams": {
                  "componentProps": { "listType": "picture-card", "maxCount": 1 },
                  "avatarStyle": "avatar-label"
                },
                "width": 80,
                "minWidth": 60
              },
              { "type": "TEXT_COLUMN", "colId": "username", "field": "username", "headerName": "用户名" },
              { "type": "TEXT_COLUMN", "colId": "email", "field": "email", "headerName": "邮箱" },
              { "type": "ACL_COLUMN", "colId": "role", "field": "role", "headerName": "角色" },
              {
                "type": "ACTION_COLUMN",
                "width": 240,
                "headerName": "操作",
                "cellRendererParams": {
                  "displayLikeButton": true,
                  "showIcon": true,
                  "actions": [
                    { "type": "EDIT", "href": "/form/user-form/layout/edit/:id" },
                    { "type": "VIEW", "href": "/form/:id/view" },
                    { "type": "DELETE" }
                  ]
                },
                "pinned": "right"
              }
            ],
            "addButtonHref": "/form/user-form/layout/new",
            "pinnedFilter": ["username", "email", "role"],
            "pagination": true
          }
        }
      ]
    }
  ]
}
```

## 页面与路由/菜单的关系

创建页面后，需要配置三层才能在前端完整访问：

### 第 1 层：PBC routesJson（页面路由映射）

```json
{
  "desktop": {
    "/": {
      "pageTitle": "请假管理",
      "schemaId": "leave-list-schema"
    },
    "/form/leave-request-form/layout/new": {
      "pageTitle": "新建请假",
      "schemaId": "leave-list-schema",
      "parent": "/",
      "formType": "ENTITY",
      "formEntityToken": "leave-request-form",
      "formEntityLayoutToken": "new"
    },
    "/form/:id/view": {
      "pageTitle": "请假详情",
      "schemaId": "leave-list-schema",
      "parent": "/",
      "formType": "ENTITY",
      "formEntityToken": "leave-request-form",
      "formEntityLayoutToken": "view"
    }
  }
}
```

### 第 2 层：PBC config（PBC 菜单元数据）

```json
{
  "menu": [
    {
      "label": "请假管理",
      "to": "/leave-management/page/leave-list-schema",
      "icon": "material:event-note"
    }
  ]
}
```

### 第 3 层：项目 frontEndConfig（全局菜单 — 必须配置）

**仅设置 PBC 的 routesJson 和 config 不会让菜单出现在前端。**
必须在项目的 frontEndConfig 的 `templateConfig.navbar.items` 中注册菜单项。

详见 `skills/ceta/ceta-project/SKILL.md` 的 frontEndConfig 章节和 `references/frontend-config-template.json`。

## href 路径规则

| 场景 | href 格式                                      |
| ---- | ---------------------------------------------- |
| 新增 | `/form/{entityToken}/layout/{layoutToken}`     |
| 编辑 | `/form/{entityToken}/layout/{layoutToken}/:id` |
| 查看 | `/form/:id/view`                               |

## 注意事项
- schemaId 在 PBC 内唯一
- schemaJson 是 JSON 字符串
- 列表页必须用 Table 组件（包在 Card 内），不能只用顶层 columns
- Table 的 columnDefs 必须包含 ACTION_COLUMN 才有操作按钮
- **创建页面后必须完成三步才能在前端看到：1) 更新 PBC routesJson 2) 更新 PBC config 3) 更新项目 frontEndConfig 的 navbar**
- Table 和 Card 都需要 `style: { "height": "100%" }` 才能正确撑满页面
- 遇到无法映射的 HTML 结构，用 `Text` 组件兜底并告知用户
- 输出文件使用格式化 JSON（2 空格缩进）

### 输出文件的用途（重要）

生成的 `-page.json` 是**中间产物**，不要直接通过 MCP API 创建到平台。

正确流程：
1. 本 SKILL 生成 `{page}-page.json` 到 `ceta-workspace/` 目录
2. 由 `ceta_sync.sh assemble-pbc` 将这些文件拼装为 `pbc-seed-data.json`
3. 由 `ceta_sync.sh import-pbc` 统一导入到 CETA 平台

**禁止**对包含复杂 schemaJson 的页面直接调用 `form__form_entity_page__create` 创建。
MCP API 参数有大小限制，复杂页面的 schemaJson（通常几千字节）会被截断，导致页面渲染异常。

只有在更新已有页面的简单属性（如 name）时，才使用 MCP API 逐个操作。

## 参考文档
- 组件详细文档（按需读取） → `skills/ceta/references/components/input/`、`skills/ceta/references/components/layout/`、`skills/ceta/references/components/display/`
- Table 列类型和 ACTION_COLUMN → 读取 `references/table-column-types.md`
- 列表页模板 → 读取 `references/list-page-template.json`
- schemaJson 通用结构规范 → 读取 `skills/ceta/references/schema-rules.md`
