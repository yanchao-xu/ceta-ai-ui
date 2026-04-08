# Card 卡片容器

最常用的布局容器，渲染为带标题的卡片区域。

## schemaJson

```json
{
  "component": "Card",
  "componentProps": {
    "title": "基本信息",
    "titleDesc": "请填写以下信息",
    "style": { "margin": "0 0 20px" }
  },
  "fields": [ ... ],
  "style": { "height": "100%" }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| title | string | 卡片标题 |
| titleDesc | string | 标题描述文字 |
| style | object | 卡片内部样式 |

## HTML 识别

- CETA 原生：`.card-layout` 或 `.form-layout.variant-pc`
- 通用 HTML：`<section>` 带标题、`<fieldset>` + `<legend>`
- `style` 属性直接映射到 `componentProps.style`
- 内容在 `.card-content` 子元素中

## 使用场景

- 表单页：作为字段分组容器，内部嵌套 Grid 或 Collapse
- 列表页：作为 Table 的外层容器，需要 `style: { "height": "100%" }`
