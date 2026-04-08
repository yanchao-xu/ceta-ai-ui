---
name: ceta-basic
description: >
  CETA 低代码平台 schemaJson 生成入口。提供平台上下文、数据模型概述和 SKILL 导航。
  所有 CETA schemaJson 生成任务都应首先加载此 SKILL。
  不负责调用 API 创建资源，只负责生成 JSON 结构。
inclusion: auto
metadata:
  version: "1.0.0"
  tags: ["ceta", "platform", "entry-point", "schema"]
  layer: "entry"
---

# CETA 低代码平台

## 平台概述

CETA 是一个企业级低代码开发平台，提供三大引擎：

- 表单引擎 — 通过 JSON 配置设计和渲染表单
- 流程引擎 — 设计和渲染工作流/审批流
- 仪表盘引擎 — 设计和渲染数据仪表盘

用户通过配置表单、工作流、页面和数据源来构建业务应用，无需编写代码。

## 核心数据模型

```
Project（项目/应用）
└── PBC（业务组件，功能模块的最小发布单元）
    ├── FormEntity（表单实体）
    │   ├── fields（字段定义）
    │   └── layouts（表单布局，含 schemaJson）
    ├── FormEntityPage（页面配置，含 schemaJson）
    ├── FlowDefinition（工作流定义）
    ├── Permission（权限定义）
    ├── routesJson（前端路由配置）
    └── config（菜单配置）
```

### 核心概念

| 概念           | 说明                                 | 标识方式                     |
| -------------- | ------------------------------------ | ---------------------------- |
| Project        | 顶层应用容器                         | projectId / projectToken     |
| PBC            | 业务组件，功能模块的最小发布单元     | pbcId / pbcToken             |
| FormEntity     | 数据模型（表单），定义字段和布局     | formEntityId / token         |
| Field          | 表单字段                             | fieldId / token（camelCase） |
| Layout         | 表单的不同视图（新建、编辑、查看等） | layoutId / token             |
| FormEntityPage | 页面配置（列表页、详情页等）         | pageId / schemaId            |
| FlowDefinition | 工作流定义                           | flowDefinitionId / token     |

### 标识符约定

- `id` 是数据库自增主键
- `token` 是业务标识符，跨环境稳定，推荐使用
- token 使用 kebab-case（如 `user-management`、`leave-application`）
- 字段 token 使用 camelCase（如 `userName`、`birthDate`）

---

## schemaJson 概述

CETA 平台的 UI 结构通过 schemaJson 描述，统一使用 `{ form, fields }` 结构：

```json
{
  "form": { ... },
  "fields": [ ... ]
}
```

schemaJson 有两种用途：

- **表单布局**（Layout schemaJson）— 用于 FormEntity 的 Layout，描述新建/编辑/查看表单的 UI
- **列表页**（FormEntityPage schemaJson）— 用于 FormEntityPage，描述列表页、仪表盘等页面的 UI

schemaJson 最终在平台中存储为 JSON 字符串（`JSON.stringify`），但生成时输出为格式化的 JSON 对象（2 空格缩进）。

---

## SKILL 导航

根据任务类型，按需加载对应的 SKILL。不要一次性全部加载。

| 需要做什么              | 加载哪个 SKILL                              |
| ----------------------- | ------------------------------------------- |
| 分析复杂 HTML 结构并拆分 | `skills/ceta/ceta-html-analyzer/SKILL.md`   |
| 生成表单布局 schemaJson | `skills/ceta/ceta-form/SKILL.md`            |
| 生成列表页 schemaJson   | `skills/ceta/ceta-page/SKILL.md`            |
| 生成应用配置（菜单+主题+路由） | `skills/ceta/ceta-app-config/SKILL.md` |

### 判断规则

**HTML 结构分析**（优先判断） — 满足任一条件时，先加载 `ceta-html-analyzer`：

- HTML 包含导航栏（`<nav>`）或侧边栏菜单（`<aside>`），暗示是完整应用或模块
- HTML 包含 2 个以上独立的 `<form>` 或 `<table>`，暗示是 PBC 级别
- HTML 结构复杂，包含多个业务区域（如多个卡片/Tab 切换不同内容）
- 用户说"分析这个 HTML"、"拆分"、"转成 CETA 配置"

html-analyzer 会分析范围（项目/PBC/单页面），拆分结构后再调度下面的子 SKILL。

**表单布局** — 满足任一条件（且 HTML 结构简单，无需 html-analyzer）：

- 输入中包含 `<form>` 标签或多个输入字段（`<input>`、`<select>`、`<textarea>` 等）
- CETA 原生 HTML 中有 `.input-element`、`.select-element`、`.date-picker-element` 等
- 自然语言提到：表单、新建、编辑、查看、填写、录入、申请表

**列表页** — 满足任一条件：

- 输入中包含 `<table>` 标签
- CETA 原生 HTML 中有 `.icp-ag-table`、`.ag-theme-quartz`
- 自然语言提到：列表、管理页、数据表、查询页、Table

**混合页面** — 同时包含表单和列表元素时，分别加载两个 SKILL 生成对应的 JSON。

### 后续扩展（规划中）

| 输出目标                 | 子 SKILL               | 状态   |
| ------------------------ | ---------------------- | ------ |
| 应用配置（菜单+主题+路由） | `ceta/ceta-app-config` | 可用   |
| 完整 PBC Seed Data       | `ceta/ceta-pbc`        | 规划中 |

---

## 输入类型识别

用户的输入分三种来源：

| 来源           | 识别特征                                                                                                                      |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| CETA 原生 HTML | 根元素 class 含 `form-renderer`，子元素含 `card-layout`、`grid-layout`、`collapse-element`、`icp-ag-table` 等 CETA 专有 class |
| 普通 HTML      | 标准 HTML 标签（`<form>`、`<table>`、`<input>`、`<select>` 等），不含 CETA 专有 class                                         |
| 自然语言       | 中文或英文描述，如"做一个用户管理的列表页"、"创建请假申请表单"                                                                |

---

## 共享参考文档

子 SKILL 在执行转换时，按需读取以下共享文档：

### 通用规范

| 文档                | 路径                                       | 内容                                                                      |
| ------------------- | ------------------------------------------ | ------------------------------------------------------------------------- |
| schemaJson 结构规范 | `skills/references/schema-rules.md`        | 顶层结构、组件节点通用字段、validation 校验、字段类型映射、组件名新旧对照 |
| HTML 映射规则       | `skills/references/html-mapping-rules.md`  | HTML class/元素 → CETA 组件的映射规则                                     |
| 样式映射规则        | `skills/references/style-mapping-rules.md` | HTML CSS 样式 → JSON style/componentProps.style 的映射规则                |

### 组件文档（按需读取）

| 目录                         | 包含组件                                          | 适用场景     |
| ---------------------------- | ------------------------------------------------- | ------------ |
| `skills/components/input/`   | Input、Select、DatePicker、Switch、Upload、others | 表单转换     |
| `skills/components/layout/`  | Card、Grid、Collapse、others                      | 表单和列表页 |
| `skills/components/display/` | Table、Tabs、Button、others                       | 列表页转换   |

**按需读取**：不要一次性加载所有组件文档。根据输入中出现的组件类型，只读取需要的那几个。

---

## 输出文件规范

将生成的 JSON 写入 `output/` 目录：

```
output/
└── {业务名称}/
    ├── {业务名称}-form.json       # 表单布局 schemaJson
    ├── {业务名称}-page.json       # 列表页 schemaJson
    └── {业务名称}-fields.json     # 字段定义列表
```

### 命名规则

- 业务名称使用 kebab-case（如 `user-management`、`leave-application`）
- 如果输入是 CETA 原生 HTML 且包含 entity token，优先使用该 token
- 如果无法确定业务名称，询问用户

---

## 通用约定

### 组件名称规范

统一使用新版组件名：

| 旧名（仍兼容） | 新名（必须使用） |
| -------------- | ---------------- |
| CardLayout     | Card             |
| FlowLayout     | Stack            |
| GridLayout     | Grid             |
| NormalLayout   | Box              |
| Tab            | Tabs             |
| NavTab         | NavTabs          |

### 字段类型映射（component → Field API type）

| component     | Field type    |
| ------------- | ------------- |
| Input         | TEXT_BOX      |
| Textarea      | TEXTAREA      |
| NumberPicker  | NUMBER        |
| Password      | PASSWORD      |
| Select        | SELECT        |
| Radio         | RADIO         |
| Checkbox      | CHECKBOX      |
| Switch        | SWITCH        |
| DatePicker    | DATE          |
| TimePicker    | TIME          |
| Upload        | UPLOAD        |
| ACL           | ACL           |
| Cascader      | CASCADER      |
| TreeSelect    | TREE_SELECT   |
| RichText      | RICH_TEXT     |
| Rate          | RATE          |
| EditableTable | EDITABLE_GRID |

### 其他约定

- 字段 `id` 对应 Field 的 `token`，使用 camelCase
- 遇到无法映射的 HTML 结构，用 `Text` 组件兜底并告知用户
- 输出文件使用格式化 JSON（2 空格缩进）
