# Textarea 多行文本

Field type: `TEXTAREA`

## schemaJson

```json
{
  "component": "Textarea",
  "id": "description",
  "title": "描述",
  "componentProps": {
    "size": "default",
    "simpleMode": false
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| size | `'default'` \| `'small'` | | 尺寸 |
| simpleMode | boolean | `false` | simple mode 不使用 `{ longText, value }` 格式 |
| i18nInputLayout | `'collapsed'` \| `'flat'` | `'collapsed'` | 多语言输入的布局方式 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [TextArea](https://ant.design/components/input#components-input-demo-textarea)。

## 特殊容器属性

| 属性 | 类型 | 说明 |
|------|------|------|
| allowI18nValue | boolean | 是否允许输入多种语言 |

## 样式规则

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| 输入框本身样式 | `componentProps.style` | 控件内部 |
| 外层容器（宽度、margin） | `style`（组件根层） | 外层 wrapper |

## HTML 识别

- CETA 原生：`.textarea-element`
- 通用 HTML：`<textarea>` 标签
- `name` 属性 → `id`

## 示例：只读展示

```json
{
  "component": "Textarea",
  "id": "notes",
  "title": "备注",
  "readonly": true,
  "defaultValue": { "longText": "这是一段只读文本" },
  "style": { "width": 800 }
}
```
