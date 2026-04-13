# Icon 图标

展示型组件，支持产品图标库、Octicons、Ant Design Icons 和 Emoji。

## schemaJson

```json
{
  "component": "Icon",
  "componentProps": {
    "name": "home",
    "size": 24,
    "color": "#1890ff"
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| name | string | | Icon 名称（见命名规则） |
| size | number | | Icon 尺寸 |
| color | string | `'currentColor'` | Icon 颜色 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 `<svg>` 元素。

## 图标命名规则

| 前缀 | 来源 | 示例 |
|------|------|------|
| 无前缀 | 产品图标库 | `"home"` |
| `oct:` | [Octicons](https://primer.github.io/octicons/) | `"oct:accessibility"` |
| `ant:` | [Ant Design Icons](https://ant.design/components/icon) | `"ant:account-book-outlined"` |
| Emoji | 直接使用 | `"😀"` |

## HTML 识别

- CETA 原生：`.icon-element`
- 通用 HTML：`<svg>` 图标、`<i>` 图标类
