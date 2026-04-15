# Icon 图标

展示型组件，用于在页面中展示纯装饰性或信息性图标。支持产品图标库、Octicons、Material Icons、Solar Icons、Ant Design Icons 和 Emoji。

> 此图标命名规则同样适用于 Button 组件的 `icon` 属性和 navbar 菜单项的 `icon` 属性，整个 CETA 平台的图标体系是统一的。

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

更多示例：

```json
{
  "component": "Icon",
  "componentProps": {
    "name": "material:emoji-food-beverage",
    "size": 24
  }
}
```

```json
{
  "component": "Icon",
  "componentProps": {
    "name": "ant:audit-outlined",
    "size": 24
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| name | string | | Icon 名称（见图标命名规则） |
| size | number | | Icon 尺寸 |
| color | string | `'currentColor'` | Icon 颜色 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 `<svg>` 元素。

## 图标命名规则

| 前缀 | 来源 | 示例 |
|------|------|------|
| 无前缀 | 产品图标库 | `"home"`, `"person"`, `"mining"` |
| `oct:` | [Octicons](https://primer.github.io/octicons/) | `"oct:accessibility"`, `"oct:people"`, `"oct:checklist"`, `"oct:home"` |
| `material:` | [Material Icons](https://fonts.google.com/icons) | `"material:search"`, `"material:flight-takeoff"`, `"material:group-sharp"`, `"material:engineering-rounded"` |
| `solar:` | [Solar Icons](https://www.figma.com/community/file/1166831539721848736) | `"solar:database-outline"`, `"solar:users-group-rounded-outline"` |
| `ant:` | [Ant Design Icons](https://ant.design/components/icon) | `"ant:account-book-outlined"`, `"ant:api-outlined"`, `"ant:audit-outlined"` |
| Emoji | 直接使用 | `"😀"` |

推荐优先使用 `material:` 和 `oct:` 前缀的图标。

## 使用场景

Icon 组件适用于页面中需要纯展示图标的场景，例如：
- 标题旁的装饰图标
- 状态指示图标
- 信息卡片中的图标标识
- 列表项前的图标

## HTML 识别

- CETA 原生：`.icon-element`
- 通用 HTML：`<svg>` 图标、`<i>` 图标类（如 Font Awesome `fa-*`、Material Icons 等）
- 转换时需根据图标语义从 CETA 支持的图标库中选择对应图标，而非直接复制 HTML 中的图标 class
