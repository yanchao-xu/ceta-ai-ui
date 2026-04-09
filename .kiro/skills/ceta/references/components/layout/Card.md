# Card 卡片容器

最常用的布局容器，渲染为带标题的卡片区域。

## schemaJson

```json
{
  "component": "Card",
  "className": "glass-card",
  "componentProps": {
    "title": "基本信息",
    "titleDesc": "请填写以下信息",
    "style": {
      "borderRadius": 16,
      "background": "rgba(255,255,255,0.95)",
      "boxShadow": "0 8px 32px rgba(0,75,141,0.08)",
      "border": "1px solid #e2e8f0"
    },
    "titleStyle": {
      "borderBottom": "1px solid #e8e8e8",
      "padding": "12px 16px"
    },
    "titleTextStyle": {
      "fontSize": 16,
      "fontWeight": 600,
      "color": "#1a1a1a"
    },
    "titleDescStyle": {
      "fontSize": 12,
      "color": "#999"
    },
    "contentStyle": {
      "padding": "16px 24px"
    }
  },
  "fields": [ ... ]
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| title | string | 卡片标题 |
| titleDesc | string | 标题描述文字 |
| style | object | 卡片整体样式（背景、边框、阴影、圆角） |
| titleStyle | object | 标题栏区域样式（`.card-title` div） |
| titleTextStyle | object | 标题文字样式（`.title-text` div） |
| titleDescStyle | object | 描述文字样式（`.title-desc` div） |
| contentStyle | object | 内容区域样式（`.card-content` div） |
| className | string | 自定义 class，引用 themeConfig.css |

## 样式规则

### 样式插槽位置

Card 有 5 个独立的样式插槽，必须分别设置：

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| 卡片整体（背景/边框/阴影/圆角） | `componentProps.style` | 卡片根容器 |
| 标题栏区域 | `componentProps.titleStyle` | 标题栏 div |
| 标题文字 | `componentProps.titleTextStyle` | 标题文字 |
| 描述文字 | `componentProps.titleDescStyle` | 描述文字 |
| 内容区域 | `componentProps.contentStyle` | 内容 div |
| 外层间距/尺寸 | `style`（组件根层） | margin、width、height |

### 推荐样式（毛玻璃卡片）

高质量 UI 推荐使用半透明白色 + 大圆角 + 柔和阴影：

```json
{
  "component": "Card",
  "className": "glass-card",
  "componentProps": {
    "style": {
      "borderRadius": 16,
      "background": "rgba(255,255,255,0.95)",
      "boxShadow": "0 8px 32px rgba(0,75,141,0.08)",
      "padding": 32
    }
  }
}
```

配合 themeConfig.css：
```css
.glass-card {
  border: 1px solid rgba(255,255,255,0.5);
}
.glass-card:hover {
  border-color: #00A8E8;
  box-shadow: 0 4px 20px rgba(0,168,232,0.08);
}
```

### 带彩色标题栏的卡片

用 className 控制标题栏背景色（需要重复 class 提高优先级）：

```json
{
  "component": "Card",
  "className": "bg-header-card",
  "componentProps": { "title": "航段信息" }
}
```

themeConfig.css：
```css
.bg-header-card.bg-header-card .card-title {
  background: #5c6bc0;
  color: white;
  padding: 10px 16px;
}
.bg-header-card.bg-header-card .card-title .title-text {
  font-size: 14px;
  font-weight: 600;
}
```

## HTML 识别

- CETA 原生：`.card-layout` 或 `.form-layout.variant-pc`
- 通用 HTML：`<section>` 带标题、`<fieldset>` + `<legend>`
- 内容在 `.card-content` 子元素中

## 使用场景

- 表单页：作为字段分组容器，内部嵌套 Grid 或 Collapse
- 列表页：作为 Table 的外层容器，Card 和 Table 都需要 `style: { "height": "100%" }`
- 信息展示：搜索结果卡片、摘要卡片
