# CETA schemaJson 结构规范

> 此文件定义 CETA 平台 schemaJson 的顶层结构和通用规则。
> 所有组件的具体属性见 components-*.md 文件。

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
| labelLayout | string | 标签布局：`"vertical"` 或 `"horizontal"` | `"horizontal"` |
| browserTitle | object | 浏览器标题配置 | 无 |
| enableAutoLock | boolean | 是否启用自动锁定 | false |

**表单页**（新建/编辑/查看）通常设置 `defaultSubmitButton: true, defaultCancelButton: true`。
**列表页**通常只设置 `title`。

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
| hidden | boolean | 否 | 是否隐藏，默认 false |

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
- `fields` 中主要是输入组件 + 布局组件
- 每个输入组件有 `id`（对应 Field token）和 `title`

### 列表页 schemaJson（用于 FormEntityPage）

用于 FormEntityPage（列表页、仪表盘等），特点：
- `form` 通常只有 `title`
- `fields` 中主要是 Table 组件（含 columnDefs）
- Table 通过 `componentProps.dataSource.token` 关联 FormEntity
- Table 外层 Card 和 Table 自身都需要 `style: { "height": "100%" }`

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
