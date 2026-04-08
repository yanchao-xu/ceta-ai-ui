# Grid 栅格布局

多列布局，常用于表单字段并排显示。

## schemaJson

```json
{
  "component": "Grid",
  "componentProps": {
    "colNumber": 2,
    "columnGap": 16,
    "rowGap": 12,
    "blankChildPlace": false,
    "alignItems": "start",
    "justifyContent": "stretch",
    "style": {
      "padding": "12px",
      "backgroundColor": "#fafafa"
    }
  },
  "fields": [
    { "id": "field1", "title": "字段1", "component": "Input", "span": 1 },
    { "id": "field2", "title": "字段2", "component": "Input", "span": 1 }
  ]
}
```

## componentProps

| 属性 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| colNumber | number | 列数 | 12 |
| gap | number | 统一间距(px) | 8 |
| columnGap | number | 列间距(px)，覆盖 gap | 同 gap |
| rowGap | number | 行间距(px)，覆盖 gap | 同 gap |
| blankChildPlace | boolean | 隐藏子项是否占位 | true |
| justifyItems | string | 单元格水平对齐 | 无 |
| alignItems | string | 单元格垂直对齐 | 无 |
| justifyContent | string | 网格整体水平对齐 | 无 |
| alignContent | string | 网格整体垂直对齐 | 无 |
| className | string | 自定义 class | 无 |
| style | object | 内部样式（背景、内边距等） | 无 |

## 样式映射

Grid 的布局参数在 `componentProps` 顶层，视觉样式在 `componentProps.style`：

| HTML CSS 属性 | 映射到 |
|--------------|--------|
| `--column-number` | `componentProps.colNumber` |
| `--column-gap` | `componentProps.columnGap`（去掉 px） |
| `--row-gap` | `componentProps.rowGap`（去掉 px） |
| `grid-template-columns` | 从列数推断 `componentProps.colNumber` |
| `gap` | `componentProps.gap` |
| `align-items` | `componentProps.alignItems` |
| `justify-content` | `componentProps.justifyContent` |
| `padding`, `background` 等 | `componentProps.style` |
| `margin` | `style`（组件根层） |

子项的 `--span` CSS 变量 → 子组件的 `span` 属性。

## HTML 识别

- CETA 原生：`.grid-layout`，从 CSS 变量提取参数
- 通用 HTML：CSS `display: grid` 容器、多列表单布局

## 使用场景

- 表单中字段并排显示（最常见 `colNumber: 2`）
- 通常嵌套在 Card 或 Collapse item 内部
