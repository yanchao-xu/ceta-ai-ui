# Text 文本

展示型组件，用于显示静态或动态文本内容。

## schemaJson

```json
{
  "component": "Text",
  "componentProps": {
    "content": "这是一段说明文字",
    "fontSize": 14,
    "fontWeight": 600,
    "color": "#333"
  },
  "style": { "marginBottom": 8 }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| content | number \| string | 文本内容，支持 Variable Pattern |
| fontSize | number \| string | 文字大小 |
| fontWeight | number \| string | 文字粗细 |
| color | string | 文字颜色 |
| formatter | `'localeDateTime'` \| `'localeDate'` \| `'localTime'` \| `'age'` | 文字格式化 |
| prefix | string | 文字前缀（不包含在链接内） |
| suffix | string | 文字后缀（不包含在链接内） |
| href | string | 文字链接地址，支持 Variable Pattern |
| hrefIsSiteBased | boolean | 是否为站点级跳转（跨 PBC） |
| suppressBasePath | boolean | 是否跳过 base path 拼接 |
| target | string | 链接的 target 属性（如 `_blank`） |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

## 样式规则

Text 有两种样式方式：

| 方式 | 放在哪里 | 说明 |
|------|---------|------|
| 快捷属性 | `componentProps` 的 fontSize/fontWeight/color | 直接设置文字样式 |
| 完整样式 | `style`（组件根层） | 所有 CSS 样式（背景、padding、border 等） |

注意：Text 的所有视觉样式放在 `style`（组件根层），不是 `componentProps.style`。

## 高级用法：状态徽章

```json
{
  "component": "Text",
  "className": "status-active",
  "componentProps": { "content": "已激活" },
  "style": {
    "padding": "4px 12px",
    "borderRadius": 20,
    "fontSize": 12,
    "fontWeight": 600
  }
}
```

配合 themeConfig.css：
```css
.status-active { background: #10B981; color: white; }
```

## 高级用法：等宽字体编号

```json
{
  "component": "Text",
  "componentProps": { "content": "QPERBI" },
  "style": {
    "fontFamily": "monospace",
    "fontSize": 15,
    "fontWeight": 700,
    "letterSpacing": "0.15em"
  }
}
```

## 高级用法：带链接的文本

```json
{
  "component": "Text",
  "componentProps": {
    "content": ":name",
    "href": "/form/:id/view",
    "color": "#1890ff"
  }
}
```

## HTML 识别

- CETA 原生：`.text-element`
- 通用 HTML：`<p>`、`<span>`、`<h1>`~`<h6>`、纯文本区域
