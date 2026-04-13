# Divider 分割线

展示型组件，用于分隔内容区域。

## schemaJson

```json
{
  "component": "Divider",
  "componentProps": {
    "content": "分割线标题"
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| content | string | Divider 中显示的文字 |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Divider](https://ant.design/components/divider)。

## HTML 识别

- CETA 原生：`.divider-element` 或 `.ant-divider`
- 通用 HTML：`<hr>` 标签
