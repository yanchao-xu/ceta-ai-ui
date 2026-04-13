# Input 单行文本

Field type: `TEXT_BOX`

## schemaJson

```json
{
  "component": "Input",
  "id": "username",
  "title": "用户名",
  "validation": { "required": true, "maxLength": 200 },
  "componentProps": {
    "placeholder": "请输入用户名",
    "style": {
      "background": "#F8FAFC",
      "border": "2px solid #E2E8F0",
      "borderRadius": 12,
      "fontSize": 16
    }
  },
  "style": { "width": "50%" }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| placeholder | string | 占位文本 |
| disabled | boolean | 是否禁用 |
| addonBefore | string | 前缀文本 |
| addonAfter | string | 后缀文本 |
| style | object | 输入框本身的样式（背景、边框、圆角、字号等） |

## 样式规则

Input 有两个样式位置，作用不同：

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| 输入框本身（背景、边框、圆角、字号、字体） | `componentProps.style` | 控件内部样式 |
| 外层容器（宽度、margin） | `style`（组件根层） | 外层 wrapper |

### 高级样式：大号等宽输入框

适用于编号、代码类输入（如 RLOC、航班号）：

```json
{
  "component": "Input",
  "id": "rloc",
  "title": "参考编号",
  "componentProps": {
    "placeholder": "例如: QPERBI",
    "style": {
      "background": "#F8FAFC",
      "border": "2px solid #E2E8F0",
      "borderRadius": 12,
      "padding": "20px 24px",
      "fontSize": 24,
      "fontFamily": "monospace",
      "letterSpacing": "0.15em",
      "textTransform": "uppercase"
    }
  }
}
```

## HTML 识别

- CETA 原生：`.input-element` + `<input type="text">`
- 通用 HTML：`<input type="text">`、`<input type="email">`、`<input type="tel">`、`<input type="search">`
- 字段名从 `id` 或 `name` 属性提取
- 标题从 `.field-title-text` 或 `<label>` 提取

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| Textarea | 多行文本，`<textarea>` 或 `.textarea-element` | TEXTAREA |
| Password | 密码，`<input type="password">` | PASSWORD |
| RichText | 富文本，`contenteditable` 或 `.ql-editor` | RICH_TEXT |
| CodeEditor | 代码编辑器 | — |

### Textarea 示例

```json
{
  "component": "Textarea",
  "id": "description",
  "title": "描述",
  "componentProps": { "rows": 4 }
}
```
