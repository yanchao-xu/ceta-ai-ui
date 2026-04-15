# CETA schemaJson 结构规范

> 此文件定义 CETA 平台 schemaJson 的顶层结构和通用规则。
> 所有组件的具体属性见 components-*.md 文件。
> 数据源配置规范见 `data-source-rules.md`。

---

## 顶层结构

CETA 的 schemaJson 统一使用 `{ form, fields }` 结构：

```json
{
  "form": { ... },
  "fields": [ ... ]
}
```

### form 对象

`form` 定义表单/页面级别的全局配置：

```json
{
  "form": {
    "title": "页面标题",
    "defaultSubmitButton": true,
    "defaultCancelButton": true,
    "labelLayout": "vertical",
    "browserTitle": {
      "notShowPbcName": true
    },
    "enableAutoLock": false
  }
}
```

| 字段 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| title | string | 页面/表单标题 | 无 |
| defaultSubmitButton | boolean | 是否显示默认提交按钮 | false |
| defaultCancelButton | boolean | 是否显示默认取消按钮 | false |
| labelLayout | string | 标签布局：`"vertical"` 或 `"horizontal"` | `"vertical"` |
| browserTitle | object | 浏览器标题配置 | 无 |
| enableAutoLock | boolean | 是否启用自动锁定 | false |

**表单页**（新建/编辑/查看）通常设置 `defaultSubmitButton: true, defaultCancelButton: true`。
**列表页/页面**通常只需要 `title`，但 `labelLayout: "vertical"` 应始终显式设置。

**重要：`labelLayout: "vertical"` 必须在所有 schemaJson 的 `form` 对象中显式设置，无论是表单布局（Layout）还是页面配置（FormEntityPage）。** 虽然平台默认值是 `"vertical"`，但省略会导致生成不一致，必须显式声明。

### fields 数组

`fields` 是组件树的根数组，每个元素是一个组件节点。组件通过 `fields` 属性嵌套子组件。

---

## 组件节点通用字段

每个组件节点的通用字段：

```json
{
  "component": "Input",
  "id": "username",
  "title": "用户名",
  "componentProps": {},
  "validation": {},
  "style": {},
  "hidden": false
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| component | string | 是 | 组件类型名（PascalCase），见组件清单 |
| id | string | 输入组件必填 | 字段标识符，对应 Field 的 token，camelCase |
| title | string | 输入组件必填 | 字段显示名称 |
| fields | array | 布局组件必填 | 子组件数组 |
| componentProps | object | 否 | 组件特有属性 |
| validation | object | 否 | 校验规则 |
| style | object | 否 | CSS 样式对象（组件根层） |
| className | string | 否 | CSS class 名，引用 themeConfig.css 中定义的全局 class |
| hidden | boolean \| ConditionalPropertyPropType | 否 | 是否隐藏，支持静态布尔值或条件表达式，详见 `hidden-rules.md` |

### 关键规则

1. **布局组件**（Card, Grid, Stack 等）通过 `fields` 嵌套子组件
2. **Collapse 和 Tabs** 通过 `componentProps.items[].fields` 嵌套子组件（不是自身的 `fields`）
3. **输入组件**（Input, Select 等）必须有 `id` 和 `title`
4. **展示组件**（Table, Text, Button 等）不需要 `id`（Table 除外，Table 需要 `id`）
5. `id` 值使用 camelCase，对应后端 Field 的 token
6. `componentProps` 放组件特有配置，不同组件的 props 不同

---

## validation 校验规则

```json
{
  "validation": {
    "required": true,
    "maxLength": 200,
    "minValue": 0,
    "maxValue": 100,
    "regex": "^[a-zA-Z]+$",
    "regexErrorMessage": "只允许英文字母"
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| required | boolean | 是否必填 |
| maxLength | number | 最大长度 |
| minValue | number | 最小值（数字字段） |
| maxValue | number | 最大值（数字字段） |
| regex | string | 正则校验 |
| regexErrorMessage | string | 正则校验失败提示 |

---

## 字段类型映射（component → Field API type）

当需要同时输出字段定义列表时，使用此映射：

| component | Field type |
|-----------|-----------|
| Input | TEXT_BOX |
| Textarea | TEXTAREA |
| NumberPicker | NUMBER |
| Password | PASSWORD |
| Select | SELECT |
| Radio | RADIO |
| Checkbox | CHECKBOX |
| Switch | SWITCH |
| DatePicker | DATE |
| TimePicker | TIME |
| Upload | UPLOAD |
| ACL | ACL |
| Cascader | CASCADER |
| TreeSelect | TREE_SELECT |
| RichText | RICH_TEXT |
| Rate | RATE |
| EditableTable | EDITABLE_GRID |
| Formula | FORMULA |
| Join | JOIN |
| Workflow | WORKFLOW |
| NumericPicker | NUMERIC |

---

## 两种 schemaJson 的区别

### 表单布局 schemaJson（用于 Layout）

用于 FormEntity 的 Layout（新建/编辑/查看表单），特点：
- `form.defaultSubmitButton` 和 `form.defaultCancelButton` 通常为 true
- `fields` 中可以包含任意组件：输入控件（映射数据库字段）+ 展示控件（Table、Button、Text 等）+ 布局控件
- 输入控件必须有 `id`（对应 Field token）和 `title`，展示控件不需要 `id`（Table 除外）

### 列表页 schemaJson（用于 FormEntityPage）

用于 FormEntityPage（列表页、仪表盘等），特点：
- `form` 必须设置 `title` 和 `labelLayout: "vertical"`
- `fields` 中主要是 Table 组件（含 columnDefs）
- Table 通过 `componentProps.dataSource.token` 关联 FormEntity
- Table / EditableTable 的高度应从 HTML 中读取，读不出来默认 `450px`，不要使用 `"height": "100%"`

---

## 组件名称规范

统一使用新版组件名，不使用旧名：

| 旧名（仍兼容） | 新名（必须使用） |
|---------------|---------------|
| CardLayout | Card |
| FlowLayout | Stack |
| GridLayout | Grid |
| NormalLayout | Box |
| MobileLayout | Mobile |
| Tab | Tabs |
| NavTab | NavTabs |


---

## CETA 组件样式规范

> 本章节定义 CETA schemaJson 中样式的写法规范。
> 无论输入来源是什么（HTML、图片、需求文档、手动编写），生成 schemaJson 时都必须遵循这些规则。

### 样式的两个位置

每个组件有两个可以放样式的位置：

| 位置 | 作用 | 渲染目标 |
|------|------|---------|
| `style` | 组件样式（视觉样式 + 间距 + 尺寸） | 组件根 `<div>` |
| `componentProps.style` | 组件内部样式（仅 Card 和输入组件使用） | 组件内容 `<div>` |

样式位置规则：
- Stack / Grid / Box：所有视觉样式（background、padding、border 等）放在外层 `style`
- Card：卡片整体样式放 `componentProps.style`，标题/内容区域用 `titleStyle`/`contentStyle` 等
- Text / Button：所有样式放在外层 `style`
- 输入组件（Input、Select 等）：`style` 作用于外层 wrapper，`componentProps.style` 作用于输入控件本身

### 各布局组件的样式插槽

#### Stack

flex 属性写在 `componentProps` 顶层，视觉样式写在外层 `style`：

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "alignItems": "center",
    "justifyContent": "space-between",
    "gap": 16,
    "flexWrap": "wrap"
  },
  "style": {
    "padding": 15,
    "background": "#f5f5f5",
    "borderRadius": 8,
    "marginBottom": 20
  }
}
```

Stack 的 `componentProps` 顶层属性：`flexDirection`、`alignItems`、`justifyContent`、`gap`、`flexWrap`

#### Grid

布局参数写在 `componentProps` 顶层，视觉样式写在外层 `style`：

```json
{
  "component": "Grid",
  "componentProps": {
    "colNumber": 3,
    "columnGap": 16,
    "rowGap": 12,
    "blankChildPlace": false
  },
  "style": { "padding": 12 }
}
```

Grid 的 `componentProps` 顶层属性：`colNumber`（建议最多不超过 4）、`columnGap`、`rowGap`、`gap`、`alignItems`、`justifyContent`、`justifyItems`、`alignContent`、`blankChildPlace`

Grid 子项通过 `span` 控制占列数（优先用 span，不用 width）。

#### Card

Card 有多个样式插槽，分别控制不同区域：

```json
{
  "component": "Card",
  "componentProps": {
    "title": "基本信息",
    "titleDesc": "描述文字",
    "style": { "backgroundColor": "#fff", "borderRadius": 8, "boxShadow": "0 2px 8px rgba(0,0,0,0.1)" },
    "titleStyle": { "borderBottom": "1px solid #e8e8e8", "padding": "12px 16px" },
    "titleTextStyle": { "fontSize": 16, "fontWeight": 600, "color": "#1a1a1a" },
    "titleDescStyle": { "fontSize": 12, "color": "#999" },
    "contentStyle": { "padding": "16px" }
  },
  "style": { "marginBottom": 20 }
}
```

| 插槽 | 控制区域 |
|------|---------|
| `componentProps.style` | 卡片整体（背景、边框、阴影） |
| `componentProps.titleStyle` | 标题栏区域 |
| `componentProps.titleTextStyle` | 标题文字 |
| `componentProps.titleDescStyle` | 描述文字 |
| `componentProps.contentStyle` | 内容区域 |
| `style` | 外层间距 |

#### Box

最简单的容器，样式放在外层 `style`：

```json
{
  "component": "Box",
  "style": { "padding": 16, "background": "#f0f2f5", "borderRadius": 4 }
}
```

### className 与 themeConfig.css

除了 inline style，CETA 支持通过 `className` 引用全局 CSS class。

组件有两个 className 位置：

| 位置 | 作用 |
|------|------|
| `className` | 组件根层 class，与 `style` 同级 |
| `componentProps.className` | 组件内部 class，与 `componentProps.style` 同级 |

```json
{
  "component": "Card",
  "className": "bg-header-card",
  "componentProps": { "title": "维护订单" },
  "fields": [ ... ]
}
```

CSS class 定义在 `app.config.json` 的 `appConfig.themeConfig.css` 中（原始 CSS 字符串，`\n` 换行）。

#### 何时用 className 而非 inline style

| 场景 | 用 className | 原因 |
|------|-------------|------|
| 伪类（:hover、:focus） | ✅ | inline style 无法实现 |
| 子元素选择器 | ✅ | 需要控制组件内部子元素 |
| 多组件复用同一套样式 | ✅ | 避免重复 |
| 动画（@keyframes） | ✅ | 只能在 CSS 中定义 |
| 单个组件的一次性样式 | ❌ 用 inline style | 更直接 |

className 命名规范：kebab-case，建议加业务前缀（如 `flight-card-header`）。
重复 class（`.bg-header-card.bg-header-card`）用于提高优先级覆盖默认样式。

### 样式值格式规则

| 值类型 | 写法 | 示例 |
|--------|------|------|
| 纯 px 数字 | 数字类型（去掉 px） | `16` |
| 非 px 单位 | 字符串 | `"1.5em"` |
| 百分比 | 字符串 | `"100%"` |
| 复合值 | 字符串 | `"8px 16px"` |
| 颜色 | 字符串 | `"#f5f5f5"`、`"rgba(0,0,0,0.1)"` |
| 渐变 | 字符串 | `"linear-gradient(135deg, #004B8D 0%, #0066CC 100%)"` |
| auto | 字符串 | `"auto"` |

### 常见样式模式

#### 页面级背景

用最外层 Stack 包裹整个页面内容：

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "gap": 16
  },
  "style": { "background": "#f0f2f5", "padding": 24, "minHeight": "100vh" },
  "fields": [ ... ]
}
```

#### 渐变背景区域

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "alignItems": "center"
  },
  "style": {
    "background": "linear-gradient(135deg, #004B8D 0%, #0066CC 50%, #00A8E8 100%)",
    "borderRadius": 20,
    "padding": "40px 0 32px"
  },
  "fields": [ ... ]
}
```

#### 按钮组

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "gap": 8,
    "justifyContent": "flex-end"
  },
  "fields": [
    { "component": "Button", "componentProps": { "content": "取消" } },
    { "component": "Button", "componentProps": { "content": "提交", "type": "primary" }, "style": { "borderRadius": 12, "fontWeight": 600 } }
  ]
}
```

#### 输入组件自定义样式

```json
{
  "component": "Input",
  "id": "rloc",
  "title": "参考编号",
  "componentProps": {
    "placeholder": "例如: QPERBI",
    "style": {
      "fontSize": 24,
      "fontFamily": "monospace",
      "letterSpacing": "0.15em",
      "textTransform": "uppercase",
      "background": "#F8FAFC",
      "border": "2px solid #E2E8F0",
      "borderRadius": 12
    }
  },
  "style": { "width": "50%" }
}
```

### 平台自动处理的样式（不需要写）

| 样式 | 原因 |
|------|------|
| `display: flex` | Stack 组件自带 |
| `display: grid` | Grid 组件自带 |
| `box-sizing: border-box` | 全局默认 |
| `font-family`（除非特殊需求） | 平台全局字体 |
| `outline`、`cursor` | 交互样式由组件处理 |
| `-webkit-*` 前缀 | 平台 PostCSS 自动添加 |

### 样式完整性要求（重要）

生成 schemaJson 时，必须为每个组件设置完整的视觉样式。不允许生成"裸"组件（只有 component 和 id，没有任何样式）。

**最高原则：忠实还原 HTML 中的样式，不做"全局主题一致性"替换。**
- HTML 中的颜色值（Tailwind 解析值、inline style 值）必须原样写入 JSON
- 不要用项目主色替换 HTML 中的其他颜色
- 不要合并 HTML 中独立的视觉元素
- 不要省略 HTML 中存在的交互元素
- 不要添加 HTML 中不存在的视觉效果（如把普通文字改成圆形徽章）

具体要求：
1. 页面级：必须有背景色、padding、minHeight
2. Card：必须有 borderRadius、background/backgroundColor，有标题时必须有 titleTextStyle
3. 按钮：必须有 borderRadius、fontWeight，主要按钮需要有 background 和 color
4. 输入组件：如果有特殊视觉要求（如大字号、等宽字体），必须通过 componentProps.style 设置
5. 文本组件：必须有 fontSize、color，标题类文本需要 fontWeight
6. 间距：组件之间必须有合理的 gap 或 margin，不能紧贴

如果是由 html-analyzer 调度生成，HTML 原文仍在上下文中，样式应直接从 HTML 提取并写入 JSON。
