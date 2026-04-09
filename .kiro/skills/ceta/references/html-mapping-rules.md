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

#### Button 交互意图识别（重要 — 不能丢失）

HTML 中的按钮不只是视觉元素，它们承载了交互行为。
**必须分析按钮的上下文语义，确定其 action 类型，然后按 `Button.md` 的规范生成 action。**

不带 action 的 Button 在 CETA 中是死按钮，点击无反应。

##### 从 HTML 识别 action 类型

| 按钮/HTML 特征 | 推断的 action 类型 |
|---------------|-------------------|
| 文本含"搜索"/"查询"/"筛选" | 通常不需要 action |
| 文本含"创建"/"新建"/"新增" | `link` 或 `dialog` |
| 文本含"下一步" / 步骤条暗示流程 | `link` |
| 文本含"返回"/"上一步" | `link` 或 `cancel` |
| 文本含"提交"/"保存"/"完成" | `submit` |
| 文本含"取消" | `cancel` |
| 文本含"删除" | `confirm` |
| 文本含"编辑"/"修改" | `link` 或 `dialog` |
| 文本含"查看"/"详情" | `link` 或 `dialog` |
| `<a href="...">` / `onclick` 含路由 | `link` |
| 按钮关联了 modal/dialog 弹窗 | `dialog` |
| 按钮关联了确认弹窗 | `confirm` |

##### 如何判断按钮的交互目标

1. **显式线索**：`<a href="...">`、`onclick`、`data-target="#modalId"`、`aria-controls`
2. **流程线索**：步骤条（stepper）中的步骤顺序暗示页面间前后关系
3. **语义线索**：按钮文本中的目标名称（如"下一步: 旅客信息"）
4. **弹窗关联**：按钮附近有 `.modal` / `role="dialog"` / 隐藏的弹窗容器
5. **布局线索**：按钮在某个 Tab/视图中，点击后切换到另一个 Tab/视图

##### link vs dialog 的选择

| 场景 | 用 link | 用 dialog |
|------|---------|----------|
| 跳转到独立的完整页面 | ✅ | |
| 流程中的下一步（步骤条） | ✅ | |
| 在当前页面上方弹出编辑/查看表单 | | ✅ |
| 快速新建（不离开当前列表页） | | ✅ |
| 确认操作（删除、取消等） | | 用 `confirm` |

##### 确定 action 类型后

确定了 action 类型后，**加载 `skills/ceta/references/components/display/Button.md`** 获取该 action 类型的完整配置规范（参数、嵌套 successAction、Variable Pattern 等）。

本文件只负责从 HTML 识别出"这个按钮应该是什么 action 类型"以及"目标是谁"，
具体怎么配置 action JSON 由 Button.md 定义。

**如果无法确定交互目标，在 analysis.json 的 `navigationHints` 中记录，供用户确认。**

#### Modal/Dialog 弹窗元素识别

HTML 中的 modal/dialog 不是独立的 CETA 组件，而是 Button `dialog` action 的内容载体。
识别到弹窗后，需要：1) 关联触发按钮 2) 判断弹窗内容归属 3) 按 Button.md 的 dialog 规范配置 action。

##### HTML 中 modal 的典型特征

| HTML 特征 | 说明 |
|-----------|------|
| `.modal` / `.ant-modal` / `role="dialog"` | 弹窗容器 |
| `data-bs-toggle="modal"` / `data-target="#id"` | 按钮-弹窗关联 |
| `display: none` 的 `<div>` 内含表单/内容 | 隐藏的弹窗内容区 |
| 弹窗内有"确定"/"取消"按钮对 | 弹窗底部操作栏 |

##### 按钮与弹窗的关联方式

1. **显式关联**：`data-target="#modalId"` / `aria-controls="modalId"`
2. **位置关联**：弹窗紧跟在触发按钮后面
3. **语义关联**：按钮文本与弹窗标题对应

##### 弹窗内容的拆分归属

弹窗内容仍遵循 FormEntity vs Page 的判断规则：

| 弹窗内容 | 归属 |
|---------|------|
| 含输入字段 + 提交（数据存库） | 生成独立 FormEntity |
| 含交互内容（不存库） | 生成独立 Page（标注 `usage: "dialog"`） |
| 确认提示（"确定要XX吗？"） | 不生成文件，触发按钮用 `confirm` action |

弹窗内容的 Page/FormEntity 在 analysis.json 中标注 `"usage": "dialog"`，不在菜单中显示。

**弹窗的具体 action 配置（openFormProps、openDialogProps 等）见 Button.md 的 dialog 章节。**

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
2. **区分位置**：布局属性（flex、grid 参数）放 `componentProps` 顶层，视觉样式（background、padding、border 等）放外层 `style`（Card 除外，Card 用 `componentProps.style`）
3. **属性名转换**：CSS kebab-case → JSON camelCase（如 `background-color` → `backgroundColor`）
4. **值转换**：纯 px 数字去单位变数字，其他保留字符串

### 快速参考

| 组件 | 布局属性位置 | 视觉样式位置 | 说明 |
|------|------------|------------|------|
| Stack | `componentProps.{flexDirection,gap,...}` | `style` | 所有视觉样式放外层 style |
| Grid | `componentProps.{colNumber,columnGap,...}` | `style` | 所有视觉样式放外层 style |
| Card | `componentProps.{title,...}` | `componentProps.style` + `titleStyle` + `contentStyle` 等 | Card 是特例，用 componentProps 内的样式插槽 |
| Box | 无 | `style` | 所有视觉样式放外层 style |
| Text | 无 | `style` | fontSize/fontWeight/color 等全放 style |
| Button | 无 | `style` | background/color/fontWeight 等全放 style |
| 输入组件 | 无 | `componentProps.style`（控件本身） + `style`（外层 wrapper 尺寸） | 输入控件样式放 componentProps.style |

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

---

## 九、组件拆解原则（通用规则）

### 最高原则：忠实还原 HTML

**HTML 中的每个可见标签都必须对应一个独立的 CETA 组件节点。不要合并、省略或替换。**

- 两个 `<span>` 就是两个 Text，即使文本内容相关也不能合并
- 一个 `<button>` 就是一个 Button，不能因为"不需要"而省略
- 一个普通 `<span>` 就是 Text，不能自作主张改成 Box + Text 圆形徽章
- HTML 中的颜色（Tailwind 解析值、inline style 值）必须原样写入 JSON，不能替换为项目主色

### 核心原则

**HTML 中每个视觉上独立的元素，在 JSON 中都必须是一个独立的组件节点。**

判断方法：**看 HTML 源码中有几个独立的标签，就生成几个组件。** 不要根据"业务语义"或"视觉效果"做合并/省略/替换。

### 拆解流程

1. 识别页面中所有可见的文本片段 → 每个是一个 `Text`
2. 识别所有输入控件 → 每个是对应的输入组件
3. 识别所有按钮 → 每个是一个 `Button`
4. 识别所有装饰性容器（有背景色/边框/圆角的 div） → `Box` 或 `Card`
5. 确定容器之间的排列关系 → 纵向用 `Stack(column)`，横向用 `Stack(row)`，多列用 `Grid`

### 布局组件选择

| 子元素排列关系 | 组件 | 关键属性 |
|--------------|------|---------|
| 纵向堆叠 | Stack | `flexDirection: "column"` |
| 横向并排 | Stack | `flexDirection: "row"` |
| 多列等宽/按比例 | Grid | `colNumber` + `span` |
| 有标题的分组 | Card | `title` |
| 纯装饰容器 | Box | 靠 `style` 实现视觉效果 |

### 参考模板

`templates/css/` 和 `templates/htmlToJson/` 目录下有真实项目的 HTML→JSON 映射示例，可作为参考。
