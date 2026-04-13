---
name: ceta-form
description: >
  CETA 表单设计与管理。用于创建、修改表单结构（FormEntity），
  配置字段（Field）和表单布局（Layout）。
  定义 Layout schemaJson 的结构规范和字段定义 JSON 的生成规则。
  当任务涉及数据模型和表单操作时使用。
license: MIT
metadata:
  openclaw:
    requires:
      env: ["CETA_API_BASE", "CETA_API_TOKEN"]
    version: "1.0.0"
    author: "CETA Team"
    tags: ["form", "ceta", "low-code", "schema"]
    source: "builtin"
    dependencies:
      - "ceta/ceta-api"
      - "ceta/ceta-pbc"
      - "ceta-basic"
---

# CETA 表单管理

## 概述

FormEntity 是 CETA 的数据模型，包含字段定义（Fields）和表单布局（Layouts）。
一个 FormEntity 属于一个 PBC，对应一张数据库表。

FormEntity 的 Layout schemaJson 可以包含任意组件：
- 输入控件（Input、Select 等）— `id` 映射为数据库字段
- 展示控件（Table、Text、Button、Image 等）— 不映射字段，纯 UI
- 布局控件（Card、Grid、Stack、Collapse 等）— 组织结构

本 SKILL 负责：
1. 定义 FormEntity 的数据模型、字段类型和 Layout schemaJson 结构规范
2. 通过 MCP API 创建/修改/删除表单资源
3. 生成 Layout schemaJson 和字段定义 JSON 文件（供 assemble → import 流程使用）

本 SKILL 不关心输入来源（HTML、图片、需求文档等），只关心最终的 CETA JSON 输出规范。
输入源的解析和字段提取由上游 SKILL（如 ceta-html-analyzer）完成后，将结构化数据传递给本 SKILL 生成 JSON。

## 数据模型

### FormEntity
```
FormEntity
├── id: number
├── name: string            # 表单名称
├── token: string           # 表单标识符（PBC 内唯一）
├── pbcId: number           # 所属 PBC ID
├── pbcToken: string        # 所属 PBC Token
├── fields: Field[]         # 字段列表
├── layouts: Layout[]       # 布局列表
├── standalone: number      # 是否独立表（0/1，int 不是 boolean）
├── standaloneTableName: string  # 独立表名
├── isInnerEntity: number   # 是否内部实体（0/1，int 不是 boolean）
├── isMasterData: number    # 是否主数据（0/1，int 不是 boolean）
├── isAutoLock: number      # 是否自动锁定（0/1，int 不是 boolean）
├── isDeletableWhenReferenced: number  # 被引用时是否可删除（0/1）
├── enableForbidUpdateSomeFieldsValueWhenReferenced: number  # （0/1）
└── description: string
```

### Field
```
Field
├── id: number
├── name: string            # 字段显示名
├── token: string           # 字段标识符（表单内唯一，camelCase）
├── type: string            # 字段类型（TEXT_BOX, SELECT, DATE 等）
├── formEntityId: number    # 所属表单 ID
├── storageField: string    # 存储字段名（value1, value2, ...）
├── maxLength: number       # 最大长度
├── multiple: boolean       # 是否多选
├── uniqueField: boolean    # 是否唯一字段
├── autoLock: boolean
└── forbidToUpdate: boolean
```

### Layout
```
Layout
├── id: number
├── name: string            # 布局名称（如 "New User", "Edit User"）
├── token: string           # 布局标识符（如 "new", "edit", "view"）
├── formEntityId: number
├── defaultLayout: boolean  # 是否默认布局
├── schemaJson: string      # 布局 JSON Schema（定义表单 UI 结构）
└── sortIndex: number
```

一个表单通常需要多个 Layout：new（新建）、edit（编辑）、view（查看）。

## 字段类型

| type | 说明 | 对应前端组件 |
|------|------|-------------|
| TEXT_BOX | 单行文本 | Input |
| TEXTAREA | 多行文本 | Textarea |
| NUMBER | 数字 | NumberPicker |
| TIME | 日期时间 | TimePicker |
| SELECT | 下拉选择 | Select |
| SWITCH | 开关 | Switch |
| RADIO | 单选 | Radio |
| UPLOAD | 文件上传 | Upload |
| PASSWORD | 密码 | Password |
| ACL | 关联引用 | ACL（关联其他表单） |

## MCP 工具

### 表单 CRUD
- `form__form_entity__create` — 创建表单
- `form__form_entity__get` — 获取表单详情（含字段和布局）
- `form__form_entity__list` — 列出表单
- `form__form_entity__list_by_pbc_id` — 按 PBC 列出表单
- `form__form_entity__update` — 更新表单
- `form__form_entity__delete` — 删除表单

### 字段管理
- `form__form_entity_field__create` — 创建单个字段
- `form__form_entity_field__create_by_batch` — 批量创建字段
- `form__form_entity_field__list` — 列出字段
- `form__form_entity_field__update` — 更新字段
- `form__form_entity_field__delete` — 删除字段

### 布局管理
- `form__form_entity_layout__create` — 创建布局
- `form__form_entity_layout__list` — 列出布局
- `form__form_entity_layout__get` — 获取布局详情
- `form__form_entity_layout__update` — 更新布局

## 操作流程

### 通过 API 创建完整表单
1. 确认 PBC 存在（`form__pbc__get`）
2. 调用 `form__form_entity__create` 创建表单实体
3. 调用 `form__form_entity_field__create_by_batch` 批量创建字段
4. 调用 `form__form_entity_layout__create` 创建布局（new/edit/view）
5. 调用 `form__form_entity__get` 验证


## 生成 schemaJson 和字段定义

本 SKILL 负责根据结构化的字段信息生成：
1. **Layout schemaJson** → 写入 `ceta-workspace/{project}/{pbcs}/{pbc}/{entity}-form.json`

字段定义信息记录在 analysis.json 中，由 assemble 脚本从 analysis.json 提取，不再单独生成 fields.json 文件。

输入可以来自任何上游 SKILL（HTML 分析器、图片分析器、需求文档分析器等），
只要提供了结构化的字段清单（token、label、type、required），本 SKILL 就能生成对应的 JSON。

当由 html-analyzer 调度时，HTML 原文仍在上下文中，样式可以直接从 HTML 提取并写入 JSON。
样式写法规范见 `schema-rules.md` 的「CETA 组件样式规范」章节。

### 需要读取的参考文档

| 什么时候读     | 读什么                                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------------------------- |
| 每次都读       | `skills/ceta/references/schema-rules.md`（含组件样式规范章节）                                                  |
| 遇到具体组件时 | `skills/ceta/references/components/input/{组件名}.md` 或 `skills/ceta/references/components/layout/{组件名}.md` |
| 涉及显示隐藏时 | `skills/ceta/references/hidden-rules.md`（hidden / readonly / disabled 条件配置）                               |

### 生成流程

#### 1. 确定布局模式

| 字段特征               | 布局模式      | JSON 结构                                                       |
| ---------------------- | ------------- | --------------------------------------------------------------- |
| 字段少（≤6个），无分组 | 简单平铺      | `{ form, fields: [输入组件...] }`                               |
| 字段有明确分组标题     | Card 分组     | `{ form, fields: [Card > Grid > 输入组件] }`                    |
| 字段多且有多个业务分组 | Collapse 折叠 | `{ form, fields: [Card > Collapse > items > Grid > 输入组件] }` |

#### 2. 构造字段

- 每个字段需要 id（camelCase）、title、组件类型
- 根据组件类型读取对应的组件文档确认 JSON 结构
- 多列布局用 Grid（通常 `colNumber: 2, blankChildPlace: false`）

#### 3. 输出字段定义列表

```json
[
  { "name": "用户名", "token": "username", "type": "TEXT_BOX" },
  { "name": "邮箱", "token": "email", "type": "TEXT_BOX" },
  { "name": "状态", "token": "status", "type": "SWITCH" }
]
```

type 值参照 `ceta-basic` 中的"字段类型映射"表。

## Layout schemaJson 结构

布局的 schemaJson 定义了表单的 UI 结构。统一使用 `{ form, fields }` 结构。
详细的 schemaJson 通用规范 → 读取 `skills/ceta/references/schema-rules.md`

### form 对象

```json
{
  "form": {
    "defaultSubmitButton": true,
    "defaultCancelButton": true,
    "browserTitle": { "notShowPbcName": true }
  }
}
```

### 三种布局模式

根据字段数量和分组需求选择：

#### 模式 A：简单平铺（字段 ≤6 个，无分组）

```json
{
  "form": { "defaultSubmitButton": true, "defaultCancelButton": true },
  "fields": [
    { "component": "Input", "id": "name", "title": "姓名", "validation": { "maxLength": 200 } },
    { "component": "Textarea", "id": "note", "title": "备注" },
    { "component": "DatePicker", "id": "date", "title": "日期" },
    { "component": "Select", "id": "status", "title": "状态",
      "componentProps": { "options": [{ "label": "成功", "value": "success" }, { "label": "失败", "value": "failure" }] } }
  ]
}
```

#### 模式 B：Card + Grid 分组（字段有明确分组）

```json
{
  "form": { "defaultSubmitButton": true, "defaultCancelButton": true },
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

#### 模式 C：Card + Collapse + Grid 折叠分组（字段多且有多个业务分组）

```json
{
  "form": { "defaultSubmitButton": true, "defaultCancelButton": true },
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
                "key": "key1", "label": "基本信息",
                "fields": [
                  { "component": "Grid", "componentProps": { "blankChildPlace": false, "colNumber": 2 },
                    "fields": [
                      { "id": "name", "title": "姓名", "component": "Input" },
                      { "id": "phone", "title": "电话", "component": "Input" }
                    ] }
                ]
              },
              {
                "key": "key2", "label": "详细信息",
                "fields": [
                  { "component": "Grid", "componentProps": { "blankChildPlace": false, "colNumber": 2 },
                    "fields": [
                      { "id": "address", "title": "地址", "component": "Input" },
                      { "id": "birthday", "title": "生日", "component": "DatePicker" }
                    ] }
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

### 组件名规范

统一使用新版组件名（Card 而非 CardLayout，Grid 而非 GridLayout，Stack 而非 FlowLayout）。
完整映射表 → 读取 `references/field-types.md`

## 参考文档
- schemaJson 通用结构规范 → 读取 `skills/ceta/references/schema-rules.md`
- 组件显示隐藏规则 → 读取 `skills/ceta/references/hidden-rules.md`
- 数据源配置规范 → 读取 `skills/ceta/references/data-source-rules.md`
- 字段类型映射 → 读取 `references/field-types.md`
- 组件详细文档（按需读取） → `skills/ceta/references/components/input/`、`skills/ceta/references/components/layout/`、`skills/ceta/references/components/display/`

## 字段校验（validation）

输入组件支持 `validation` 属性定义校验规则。不同组件支持的校验属性不同：

| 组件 | 支持的 validation 属性 |
|------|----------------------|
| Input | `required`, `maxLength`, `regex` |
| Textarea | `required`, `maxLength` |
| NumberPicker | `required`, `minValue`, `maxValue` |
| DatePicker | `required` |
| Select / Radio / Checkbox | `required` |
| Upload | `required` |
| Switch | — |

```json
{
  "component": "Input",
  "id": "email",
  "title": "邮箱",
  "validation": { "required": true, "maxLength": 200, "regex": "^[\\w.-]+@[\\w.-]+\\.\\w+$" }
}
```

```json
{
  "component": "NumberPicker",
  "id": "quantity",
  "title": "数量",
  "validation": { "required": true, "minValue": 1, "maxValue": 9999 }
}
```

## i18n 国际化约定

schemaJson 中任何对象都可以通过 `_i18n_` 后缀属性添加多语言翻译。
属性名格式：`{原属性名}_i18n_{语言代码}`

```json
{
  "component": "Input",
  "id": "name",
  "title": "姓名",
  "title_i18n_en": "Name",
  "title_i18n_ja": "名前"
}
```

```json
{
  "component": "Card",
  "componentProps": {
    "title": "基本信息",
    "title_i18n_en": "Basic Information"
  }
}
```

默认只生成三种语言：`zh-CN`（中文）、`en`（英文）、`ja`（日文）。
除非用户明确要求其他语言，否则不要生成额外的语言翻译。
i18n 属性是可选的，不影响默认语言的显示。

## 注意事项
- 字段的 storageField 由系统自动分配（value1, value2, ...），创建时不需要指定
- 字段 token 在表单内唯一，建议用 camelCase
- 一个表单通常需要多个 Layout：new（新建）、edit（编辑）、view（查看）
- Layout 的 schemaJson 是 JSON 字符串，不是 JSON 对象
- FormEntity 的 boolean 类字段（isInnerEntity, isMasterData, isAutoLock, isDeletableWhenReferenced, standalone 等）在 seed data 中必须用 int（0/1），不能用 true/false
- Field 的 autoLock 和 forbidToUpdate 是 boolean（true/false）
- 遇到无法映射的组件结构，用 `Text` 组件兜底并告知用户
- 输出文件使用格式化 JSON（2 空格缩进）

### 输出文件的用途（重要）

生成的 `-form.json` 是**中间产物**，不要直接通过 MCP API 创建到平台。

正确流程：
1. 本 SKILL 生成 `{entity}-form.json` 到 `ceta-workspace/` 目录
2. 由 `ceta_sync.sh assemble-pbc` 将这些文件拼装为 `pbc-seed-data.json`（字段定义从 analysis.json 提取）
3. 由 `ceta_sync.sh import-pbc` 统一导入到 CETA 平台

**禁止**对包含复杂 schemaJson 的表单直接调用 `form__form_entity_layout__create` 创建布局。
MCP API 参数有大小限制，大 schemaJson 会被截断。

只有在修改已有表单的单个字段或简单布局时，才使用 MCP API 逐个操作。
