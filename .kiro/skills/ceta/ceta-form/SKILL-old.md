---
name: ceta-form
description: >
  CETA 表单布局 schemaJson 生成。将 HTML 或自然语言描述转换为 FormEntity 的 Layout schemaJson。
  同时输出字段定义列表，供创建 FormEntity Field 使用。
  不负责调用 API 创建资源，只负责生成 JSON 结构。
inclusion: manual
metadata:
  version: "1.0.0"
  tags: ["form", "layout", "schema", "conversion"]
  layer: "conversion"
  dependencies:
    - "ceta-basic"
---

# CETA 表单布局 schemaJson 生成

## 概述

FormEntity 是 CETA 的数据模型，包含字段定义（Fields）和表单布局（Layouts）。
本 SKILL 负责生成 Layout 的 schemaJson，描述新建/编辑/查看表单的 UI 结构。

## 数据模型

### FormEntity

```
FormEntity
├── name: string            # 表单名称
├── token: string           # 表单标识符（PBC 内唯一，kebab-case）
├── fields: Field[]         # 字段列表
└── layouts: Layout[]       # 布局列表
```

### Field

```
Field
├── name: string            # 字段显示名
├── token: string           # 字段标识符（表单内唯一，camelCase）
├── type: string            # 字段类型（TEXT_BOX, SELECT, DATE 等）
├── maxLength: number       # 最大长度
├── multiple: boolean       # 是否多选
└── uniqueField: boolean    # 是否唯一字段
```

### Layout

```
Layout
├── name: string            # 布局名称（如 "新建用户", "编辑用户"）
├── token: string           # 布局标识符（如 "new", "edit", "view"）
├── defaultLayout: boolean  # 是否默认布局
└── schemaJson: string      # 布局 JSON Schema（本 SKILL 生成的内容）
```

一个表单通常需要多个 Layout：new（新建）、edit（编辑）、view（查看）。

## 输出

1. **Layout schemaJson** → 写入 `output/{业务名称}/{业务名称}-form.json`
2. **字段定义列表** → 写入 `output/{业务名称}/{业务名称}-fields.json`

## 需要读取的参考文档

| 什么时候读     | 读什么                                                                          |
| -------------- | ------------------------------------------------------------------------------- |
| 每次都读       | `skills/references/schema-rules.md`                                             |
| 输入是 HTML 时 | `skills/references/html-mapping-rules.md`                                       |
| 输入含样式时   | `skills/references/style-mapping-rules.md`                                      |
| 遇到具体组件时 | `skills/components/input/{组件名}.md` 或 `skills/components/layout/{组件名}.md` |

## 转换流程

### 1. 确定布局模式

| 输入特征               | 布局模式      | JSON 结构                                                       |
| ---------------------- | ------------- | --------------------------------------------------------------- |
| 字段少（≤6个），无分组 | 简单平铺      | `{ form, fields: [输入组件...] }`                               |
| 字段有明确分组标题     | Card 分组     | `{ form, fields: [Card > Grid > 输入组件] }`                    |
| 字段多且有多个业务分组 | Collapse 折叠 | `{ form, fields: [Card > Collapse > items > Grid > 输入组件] }` |

### 2. form 对象

表单页的 form 通常包含：

```json
{
  "form": {
    "defaultSubmitButton": true,
    "defaultCancelButton": true,
    "browserTitle": { "notShowPbcName": true }
  }
}
```

### 3. 组装字段

- 从输入中提取每个字段的 id（camelCase）、title、组件类型
- 根据组件类型读取对应的组件文档确认 JSON 结构
- 多列布局用 Grid（通常 `colNumber: 2, blankChildPlace: false`）

### 4. 输出字段定义列表

```json
[
  { "name": "用户名", "token": "username", "type": "TEXT_BOX" },
  { "name": "邮箱", "token": "email", "type": "TEXT_BOX" },
  { "name": "状态", "token": "status", "type": "SWITCH" }
]
```

type 值参照 `ceta-basic` 中的"字段类型映射"表。

## 三种布局模式完整示例

### 模式 A：简单平铺

```json
{
  "form": {
    "title": "布局",
    "defaultSubmitButton": true,
    "defaultCancelButton": true
  },
  "fields": [
    {
      "component": "Input",
      "id": "name",
      "title": "姓名",
      "validation": { "maxLength": 200 }
    },
    {
      "component": "Textarea",
      "id": "note",
      "title": "备注",
      "componentProps": { "rows": 4 }
    },
    {
      "component": "DatePicker",
      "id": "date",
      "title": "日期",
      "componentProps": { "showTime": true }
    },
    {
      "component": "Select",
      "id": "status",
      "title": "状态",
      "componentProps": {
        "options": [
          { "label": "成功", "value": "success" },
          { "label": "失败", "value": "failure" }
        ]
      }
    }
  ]
}
```

### 模式 B：Card + Grid 分组

```json
{
  "form": {
    "defaultSubmitButton": true,
    "defaultCancelButton": true
  },
  "fields": [
    {
      "component": "Card",
      "componentProps": { "title": "基本信息" },
      "fields": [
        {
          "component": "Grid",
          "componentProps": { "colNumber": 2, "blankChildPlace": false },
          "fields": [
            { "id": "name", "title": "姓名", "component": "Input" },
            { "id": "phone", "title": "电话", "component": "Input" }
          ]
        }
      ]
    }
  ]
}
```

### 模式 C：Card + Collapse + Grid 折叠分组

```json
{
  "form": {
    "browserTitle": { "notShowPbcName": true },
    "defaultSubmitButton": true,
    "defaultCancelButton": true
  },
  "fields": [
    {
      "component": "Card",
      "componentProps": { "style": { "margin": "0 0 20px" } },
      "fields": [
        {
          "component": "Collapse",
          "componentProps": {
            "defaultActiveKey": ["key1", "key2"],
            "itemSplit": true,
            "items": [
              {
                "key": "key1",
                "label": "基本信息",
                "fields": [
                  {
                    "component": "Grid",
                    "componentProps": {
                      "blankChildPlace": false,
                      "colNumber": 2
                    },
                    "fields": [
                      { "id": "name", "title": "姓名", "component": "Input" },
                      { "id": "phone", "title": "电话", "component": "Input" }
                    ]
                  }
                ]
              },
              {
                "key": "key2",
                "label": "详细信息",
                "fields": [
                  {
                    "component": "Grid",
                    "componentProps": {
                      "blankChildPlace": false,
                      "colNumber": 2
                    },
                    "fields": [
                      {
                        "id": "address",
                        "title": "地址",
                        "component": "Input"
                      },
                      {
                        "id": "birthday",
                        "title": "生日",
                        "component": "DatePicker"
                      }
                    ]
                  }
                ]
              }
            ]
          }
        }
      ]
    }
  ]
}
```
