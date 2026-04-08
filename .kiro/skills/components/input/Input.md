# Input 单行文本

Field type: `TEXT_BOX`

## schemaJson

```json
{
  "component": "Input",
  "id": "username",
  "title": "用户名",
  "validation": { "required": true, "maxLength": 200 }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| placeholder | string | 占位文本 |
| disabled | boolean | 是否禁用 |
| addonBefore | string | 前缀文本 |
| addonAfter | string | 后缀文本 |

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
