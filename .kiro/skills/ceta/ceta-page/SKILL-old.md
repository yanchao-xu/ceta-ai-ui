---
name: ceta-page
description: >
  CETA 列表页 schemaJson 生成。将 HTML 或自然语言描述转换为 FormEntityPage 的 schemaJson。
  不负责调用 API 创建资源，只负责生成 JSON 结构。
inclusion: manual
metadata:
  version: "1.0.0"
  tags: ["page", "table", "list", "schema", "conversion"]
  layer: "conversion"
  dependencies:
    - "ceta-basic"
---

# CETA 列表页 schemaJson 生成

## 概述

FormEntityPage 定义 CETA 平台上的页面配置，包括列表页、详情页、仪表盘等。
页面通过 schemaJson 描述 UI 结构，通过 schemaId 在 PBC 的 routesJson 中引用。

## 数据模型

```
FormEntityPage
├── schemaId: string        # 页面标识符（PBC 内唯一，在 routesJson 中引用）
├── pbcId: number           # 所属 PBC ID
├── name: string            # 页面名称
├── description: string
└── schemaJson: string      # 页面 UI 配置（本 SKILL 生成的内容）
```

## 输出

**FormEntityPage schemaJson** → 写入 `output/{业务名称}/{业务名称}-page.json`

## 需要读取的参考文档

| 什么时候读     | 读什么                                     |
| -------------- | ------------------------------------------ |
| 每次都读       | `skills/references/schema-rules.md`        |
| 输入是 HTML 时 | `skills/references/html-mapping-rules.md`  |
| 输入含样式时   | `skills/references/style-mapping-rules.md` |
| Table 结构详情 | `skills/components/display/Table.md`       |

## 转换流程

### 1. 提取列表信息

从输入中提取：

- 页面标题
- 表格列（列名、字段 token、数据类型）
- 操作按钮（编辑、查看、删除）
- 筛选字段
- 关联的 FormEntity token

### 2. 构造 columnDefs

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

### 3. 组装 schemaJson

列表页的固定结构：

```
{ form: { title }, fields: [Card > Table] }
```

Card 和 Table 都需要 `style: { "height": "100%" }`。

## 完整示例

```json
{
  "form": {
    "title": "用户列表"
  },
  "fields": [
    {
      "component": "Card",
      "fields": [
        {
          "component": "Table",
          "id": "user-form-table",
          "componentProps": {
            "dataSource": { "token": "user-form" },
            "columnDefs": [
              {
                "type": "AVATAR_COLUMN",
                "colId": "avatar",
                "field": "avatar",
                "headerName": "头像",
                "cellRendererParams": {
                  "componentProps": {
                    "listType": "picture-card",
                    "maxCount": 1
                  },
                  "avatarStyle": "avatar-label"
                },
                "width": 80,
                "minWidth": 60
              },
              {
                "type": "TEXT_COLUMN",
                "colId": "username",
                "field": "username",
                "headerName": "用户名"
              },
              {
                "type": "TEXT_COLUMN",
                "colId": "email",
                "field": "email",
                "headerName": "邮箱"
              },
              {
                "type": "ACL_COLUMN",
                "colId": "role",
                "field": "role",
                "headerName": "角色"
              },
              {
                "type": "ACTION_COLUMN",
                "width": 240,
                "headerName": "操作",
                "cellRendererParams": {
                  "displayLikeButton": true,
                  "showIcon": true,
                  "actions": [
                    {
                      "type": "EDIT",
                      "href": "/form/user-form/layout/edit/:id"
                    },
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
          },
          "style": { "height": "100%" }
        }
      ],
      "style": { "height": "100%" }
    }
  ]
}
```

## href 路径规则

| 场景 | href 格式                                      |
| ---- | ---------------------------------------------- |
| 新增 | `/form/{entityToken}/layout/{layoutToken}`     |
| 编辑 | `/form/{entityToken}/layout/{layoutToken}/:id` |
| 查看 | `/form/:id/view`                               |
