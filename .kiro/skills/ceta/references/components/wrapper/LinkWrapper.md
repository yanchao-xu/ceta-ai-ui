# LinkWrapper 链接包装器

别名：`Link`（`Link` 和 `LinkWrapper` 是同一个组件）。

用于给子组件添加点击跳转能力的包装器组件。本身不渲染可见 UI，仅在子组件外层包裹一个链接。

## 典型场景

1. **List + LinkWrapper**：List 组件按模板渲染数组数据，模板内嵌套 LinkWrapper，使列表中每个元素可点击跳转。
2. **Dashboard / 首页卡片跳转**：给任意组件（Card、Box、Image 等）外层包裹 LinkWrapper，点击即可跳转到目标页面。

## schemaJson

### 基础用法：包裹子组件实现跳转

```json
{
  "component": "LinkWrapper",
  "componentProps": {
    "href": "/some-page/:id"
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": ":name" }
    }
  ]
}
```

### 与 List 结合：列表每项可点击跳转

```json
{
  "component": "List",
  "componentProps": {
    "dataSource": { "token": "order-form", "pbcToken": "order-management" }
  },
  "fields": [
    {
      "component": "LinkWrapper",
      "componentProps": {
        "href": "/order-detail/:id"
      },
      "fields": [
        {
          "component": "Card",
          "fields": [
            { "component": "Text", "componentProps": { "content": ":orderNo" } },
            { "component": "Text", "componentProps": { "content": ":status" } }
          ]
        }
      ]
    }
  ]
}
```

### Dashboard 卡片跳转

```json
{
  "component": "LinkWrapper",
  "componentProps": {
    "href": "/dashboard/report",
    "target": "_blank"
  },
  "fields": [
    {
      "component": "Box",
      "style": { "padding": 16, "background": "#f5f5f5", "borderRadius": 8, "cursor": "pointer" },
      "fields": [
        { "component": "Icon", "componentProps": { "type": "BarChartOutlined" } },
        { "component": "Text", "componentProps": { "content": "查看报表" } }
      ]
    }
  ]
}
```

### 外部链接

```json
{
  "component": "LinkWrapper",
  "componentProps": {
    "href": "https://example.com",
    "target": "_blank",
    "suppressBasePath": true
  },
  "fields": [
    { "component": "Text", "componentProps": { "content": "访问外部网站" } }
  ]
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| href | string \| ConditionalProperty | | 链接地址，支持 Variable Pattern（如 `/detail/:id`） |
| hrefSelector | array | | （已废弃）条件选择不同的 href |
| hrefIsSiteBased | boolean | `false` | 为 `false` 时自动拼接 PBC token 作为 basename，移动端自动拼接 `mobile` 前缀 |
| suppressBasePath | boolean | `false` | 是否跳过 react-router 的 base path，用于外部链接或绝对路径 |
| suppressInheritIncludeDeleted | boolean | `false` | 是否禁止自动继承当前页面 URL 中的 `include_deleted` 参数 |
| suppressLinkColor | boolean | `true` | 是否抑制默认链接颜色（蓝色下划线） |
| target | string | | HTML `<a>` 标签的 target 属性（`_blank`、`_self` 等） |
| replace | boolean | | react-router Link 的 replace 属性，为 `true` 时替换当前历史记录而非新增 |
| style | object | | 链接元素的内联样式 |
| className | string | | 链接元素的自定义 class |

## 链接类型自动判断

组件根据 `href` 的值自动选择渲染方式：

| 条件 | 渲染为 | 说明 |
|------|--------|------|
| 无 href | `<span>` | 不可点击，仅作为容器 |
| `suppressBasePath=true` 或 href 以 `http` 开头或 `mailto:` | `<a>` | 原生链接，直接跳转 |
| 其他 | react-router `<Link>` | SPA 内部路由跳转 |

## HTML 识别

`.link-wrapper` 或 `<a>` 标签包裹子元素

## 注意事项

- 在设计模式（design mode）下点击会被 `preventDefault`，不会真正跳转
- `href` 支持 Variable Pattern，如 `:id`、`:context.xxx`，运行时会替换为实际数据值
- 默认 `suppressLinkColor=true`，子组件不会显示蓝色链接样式
