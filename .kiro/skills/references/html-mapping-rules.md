# HTML → CETA schemaJson 映射规则

> 本文件定义 HTML 元素/class 到 CETA 组件的映射规则。
> 规则从 CETA 平台真实渲染的 HTML 中提炼而来。

---

## 一、CETA 平台原生 HTML 识别

当 HTML 来自 CETA 平台自身渲染时，根元素通常包含以下特征 class：

```
.form-renderer          — 表单渲染器根容器
.label-layout-horizontal — 水平标签布局
.label-layout-vertical   — 垂直标签布局
.library-ant-design      — 使用 Ant Design 组件库
```

识别到这些 class 时，可以更精确地还原 schemaJson。

---

## 二、布局组件映射

### Card 卡片

```
HTML class: .card-layout / .form-layout / .variant-pc
→ component: "Card"
```

- `style` 属性直接映射到 JSON 的 `componentProps.style`
- 内容在 `.card-content` 子元素中

### Grid 栅格

```
HTML class: .grid-layout / .form-layout
CSS 变量: --column-number, --column-gap, --row-gap
→ component: "Grid"
→ componentProps: { colNumber, columnGap, rowGap }
```

提取规则：
- `--column-number: 2` → `"colNumber": 2`
- `--column-gap: 8px` → `"columnGap": 8`（去掉 px）
- `--row-gap: 8px` → `"rowGap": 8`（去掉 px）
- 子元素在 `.grid-layout-item` 中，每个 item 的 `--span` 表示占几列

### Stack / FlowLayout 弹性布局

```
HTML: display: flex 容器 / .stack-layout
→ component: "Stack"
```

### Box / NormalLayout 盒子

```
HTML: 普通 <div> 容器
→ component: "Box"
```

---

## 三、展示组件映射

### Collapse 折叠面板

```
HTML class: .collapse-element / .ant-collapse
→ component: "Collapse"
```

提取规则：
- 每个 `.ant-collapse-item` 对应一个 item
- `.ant-collapse-item-active` 表示默认展开 → 收集其 key 到 `defaultActiveKey`
- `.ant-collapse-header-text` 的文本 → item 的 `label`
- `.ant-collapse-content-box` 内的内容 → item 的 `fields`
- `.item-split` class → `"itemSplit": true`

嵌套结构：
```json
{
  "component": "Collapse",
  "componentProps": {
    "defaultActiveKey": ["key1", "key2"],
    "itemSplit": true,
    "items": [
      {
        "key": "key1",
        "label": "分组标题",
        "fields": [ /* 子组件 */ ]
      }
    ]
  }
}
```

### Table 数据表格

```
HTML class: .icp-ag-table / .ag-theme-quartz / .table-element
→ component: "Table"
```

- `id` 属性 → Table 的 `id`
- 表格列从 `.ag-header-cell` 或列定义中提取
- 工具栏中的筛选按钮 `.icp-table-filter-item[data-id]` → `pinnedFilter` 数组
- "新增"按钮 `.icp-toolbar-add-button` → `addButtonHref`
- 详细结构见 `table-schema.md`

### Tabs 标签页

```
HTML class: .ant-tabs / role="tablist"
→ component: "Tabs"
```

### Button 按钮

```
HTML: <button> / .ant-btn
→ component: "Button"
```

- `.ant-btn-primary` → `"type": "primary"`
- 按钮文本 → `componentProps.content`

### Text 文本

```
HTML: <p> / <span>（非标签文本）
→ component: "Text"
```

---

## 四、输入组件映射

### 通用识别规则

CETA 平台渲染的输入组件有统一的 HTML 结构：

```html
<div class="{type}-element input-element form-element label-horizontal">
  <label class="field-title">
    <span class="field-title-text">字段标题</span>
  </label>
  <div>
    <!-- 具体输入控件 -->
  </div>
</div>
```

提取规则：
- `.field-title-text` 的文本 → `title`
- 输入控件的 `id` 或 `name` 属性 → `id`（即 Field token）
- 外层 class 中的 `{type}-element` → 判断组件类型

### 具体类型映射

| HTML 特征 | component | Field type |
|-----------|-----------|-----------|
| `.input-element` + `<input type="text">` | Input | TEXT_BOX |
| `.textarea-element` + `<textarea>` | Textarea | TEXTAREA |
| `.number-picker-element` / `<input type="number">` | NumberPicker | NUMBER |
| `<input type="password">` | Password | PASSWORD |
| `.select-element` / `.ant-select` | Select | SELECT |
| `.date-picker-element` / `.ant-picker` | DatePicker | DATE |
| `.date-picker-element.show-time` | DatePicker + `showTime: true` | DATE |
| `<input type="radio">` / `.ant-radio` | Radio | RADIO |
| `<input type="checkbox">` / `.ant-checkbox` | Checkbox | CHECKBOX |
| `.ant-switch` / `.switch-element` | Switch | SWITCH |
| `<input type="file">` / `.ant-upload` | Upload | UPLOAD |
| `.ant-picker-range` | DateRangePicker | DATE |
| 富文本区域 / `.ql-editor` / `contenteditable` | RichText | RICH_TEXT |

### 特殊属性提取

- `maxlength="200"` → `"validation": { "maxLength": 200 }`
- `<textarea rows="4">` → `"componentProps": { "rows": 4 }`
- `.show-time` class → `"componentProps": { "showTime": true }`
- `.ant-select` 的选项 → `"componentProps": { "options": [...] }`

---

## 五、表单级属性映射

### 提交/取消按钮

```
HTML class: .form-renderer-default-button
→ form.defaultSubmitButton: true
→ form.defaultCancelButton: true
```

当 HTML 底部存在 `.form-renderer-default-button` 容器（内含"提交"和"取消"按钮）时，
在 `form` 对象中设置 `defaultSubmitButton: true` 和 `defaultCancelButton: true`。

### 标签布局

```
.label-layout-horizontal → form.labelLayout: "horizontal"（可省略，是默认值）
.label-layout-vertical   → form.labelLayout: "vertical"
```

---

## 六、通用 HTML 元素映射（非 CETA 原生 HTML）

当 HTML 不是来自 CETA 平台渲染时，使用以下通用映射：

| HTML 元素 | CETA component |
|-----------|---------------|
| `<form>` | 识别为表单页，生成 Layout schemaJson |
| `<table>` | Table（需构造 columnDefs） |
| `<input type="text">` | Input |
| `<input type="email">` | Input |
| `<input type="tel">` | Input |
| `<input type="number">` | NumberPicker |
| `<input type="password">` | Password |
| `<input type="date">` | DatePicker |
| `<input type="file">` | Upload |
| `<input type="checkbox">` | Switch（单个）或 Checkbox（多个） |
| `<input type="radio">` | Radio |
| `<textarea>` | Textarea |
| `<select>` | Select（提取 `<option>` 为 options） |
| `<section>` 带标题 | Card |
| `<fieldset>` + `<legend>` | Card（legend 文本作为 title） |
| `<div>` 多列布局 | Grid（根据列数设置 colNumber） |
| `<button>` | Button |
| `<img>` | Image |
| `<hr>` | Divider |
| `<p>` / `<span>` | Text |

---

## 七、样式提取规则

转换 HTML 时，必须同时提取组件的样式信息。详细规则见 `skills/references/style-mapping-rules.md`。

### 核心原则

1. **不要丢弃样式**：HTML 中的 inline style 和可推断的视觉样式都应映射到 JSON
2. **区分位置**：布局属性（flex、grid 参数）放 `componentProps` 顶层，视觉样式放 `componentProps.style`，外层间距放 `style`
3. **属性名转换**：CSS kebab-case → JSON camelCase（如 `background-color` → `backgroundColor`）
4. **值转换**：纯 px 数字去单位变数字，其他保留字符串

### 快速参考

| 组件 | 布局属性位置 | 视觉样式位置 | 外层间距位置 |
|------|------------|------------|------------|
| Stack | `componentProps.{flexDirection,gap,...}` | `componentProps.style` | `style` |
| Grid | `componentProps.{colNumber,columnGap,...}` | `componentProps.style` | `style` |
| Card | `componentProps.{title,...}` | `componentProps.style` + `titleStyle` + `contentStyle` 等 | `style` |
| Box | 无 | `componentProps.style` | `style` |
| 输入组件 | 无 | `componentProps.style`（控件本身） | `style`（外层 wrapper） |

---

## 八、典型转换模式

### 模式 A：简单平铺表单（带样式）

输入字段直接平铺，无分组。

```
HTML: .form-renderer > 多个 .input-element
→ JSON: { form: { defaultSubmitButton: true, defaultCancelButton: true }, fields: [ 输入组件... ] }
```

### 模式 B：Collapse 分组表单

字段按业务分组，使用折叠面板。

```
HTML: .form-renderer > .card-layout > .collapse-element > 多个 .ant-collapse-item
→ JSON: { form: {...}, fields: [{ component: "Card", fields: [{ component: "Collapse", componentProps: { items: [...] } }] }] }
```

每个 collapse item 内部通常嵌套 Grid 实现多列布局：
```
item.fields → [{ component: "Grid", componentProps: { colNumber: 2 }, fields: [ 输入组件... ] }]
```

### 模式 C：Table 列表页

数据表格页面。

```
HTML: .form-renderer > .card-layout > .icp-ag-table
→ JSON: { form: { title: "..." }, fields: [{ component: "Card", fields: [{ component: "Table", ... }], style: { height: "100%" } }] }
```

Table 外层 Card 和 Table 自身都需要 `style: { "height": "100%" }`。
