# Stack 弹性布局

最常用的布局组件，基于 CSS Flexbox，控制子组件的排列方向和间距。
页面级容器、水平排列、按钮组、信息行等场景都靠 Stack 实现。

## schemaJson

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "alignItems": "center",
    "justifyContent": "space-between",
    "gap": 16,
    "flexWrap": "wrap"
  },
  "style": {
    "padding": "16px 24px",
    "background": "#f5f5f5",
    "borderRadius": 8,
    "minHeight": 60,
    "marginBottom": 20
  },
  "fields": [ ... ]
}
```

## componentProps

| 属性 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| flexDirection | string | 排列方向：`"row"` / `"column"` / `"row-reverse"` / `"column-reverse"` | — |
| gap | number/string | 子项间距(px) | 8 |
| alignItems | string | 交叉轴对齐：`"stretch"` / `"center"` / `"flex-start"` / `"flex-end"` / `"baseline"` | — |
| justifyContent | string | 主轴对齐：`"flex-start"` / `"center"` / `"flex-end"` / `"space-between"` / `"space-around"` / `"space-evenly"` | — |
| flexWrap | string | 是否换行：`"wrap"` / `"nowrap"` | — |
| className | string | 自定义 class，引用 themeConfig.css 中的全局 class | — |

注意：`componentProps` 中不放 `style`，所有视觉样式放在外层 `style`。

## 样式映射

**重要**：Stack 的 flex 属性直接写在 `componentProps` 顶层，视觉样式放在外层 `style`。

从 HTML 提取样式时：

| HTML CSS 属性 | 映射到 | 说明 |
|--------------|--------|------|
| `display: flex` | 识别为 Stack 组件 | 忽略此属性本身 |
| `flex-direction` | `componentProps.flexDirection` | 顶层属性 |
| `align-items` | `componentProps.alignItems` | 顶层属性 |
| `justify-content` | `componentProps.justifyContent` | 顶层属性 |
| `gap` | `componentProps.gap` | 顶层属性，数字（去掉 px） |
| `flex-wrap` | `componentProps.flexWrap` | 顶层属性 |
| `padding`、`background-color`、`border` 等 | `style` | 外层 style |
| `margin` | `style` | 外层 style |

## HTML 识别

- CETA 原生：`.stack-layout` 或 `.form-layout` + `display: flex`
- 通用 HTML：任何 `display: flex` 的容器

## 常见用法

### 垂直容器（默认）

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "gap": 16
  },
  "fields": [ ... ]
}
```

### 水平排列

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "gap": 8,
    "alignItems": "center"
  },
  "fields": [ ... ]
}
```

### 按钮组（右对齐）

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "gap": 8,
    "justifyContent": "flex-end"
  },
  "style": {
    "padding": "12px 0"
  },
  "fields": [
    { "component": "Button", "componentProps": { "content": "取消" } },
    { "component": "Button", "componentProps": { "content": "提交", "type": "primary" } }
  ]
}
```

### 页面级背景容器

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "gap": 16
  },
  "style": {
    "background": "#f0f2f5",
    "padding": 24,
    "minHeight": "100vh"
  },
  "fields": [ ... ]
}
```

### 页面级渐变背景（推荐 — 用 className + CSS 伪元素）

用 CSS `::before` 伪元素创建顶部渐变色背景，比 inline style 效果好得多：

```json
{
  "component": "Stack",
  "className": "wave-bg-page",
  "componentProps": {
    "flexDirection": "column",
    "gap": 24
  },
  "style": {
    "background": "#F8FAFC",
    "padding": 32,
    "minHeight": "100vh"
  },
  "fields": [ ... ]
}
```

themeConfig.css：
```css
.wave-bg-page.wave-bg-page {
  position: relative;
  overflow: hidden;
}
.wave-bg-page.wave-bg-page::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 300px;
  background: linear-gradient(135deg, #004B8D 0%, #0066CC 50%, #00A8E8 100%);
  clip-path: ellipse(120% 100% at 50% 0%);
  z-index: -1;
  pointer-events: none;
}
```

### 顶部导航栏

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "justifyContent": "space-between",
    "alignItems": "center"
  },
  "style": {
    "background": "#1a237e",
    "padding": 15,
    "position": "sticky",
    "top": 0,
    "zIndex": "var(--max-z-index)"
  },
  "fields": [ ... ]
}
```

### 带 className 的复用样式

当需要 :hover 等伪类或多处复用时，用 `className` 引用 `themeConfig.css` 中的 class：

```json
{
  "component": "Stack",
  "className": "hover-lift",
  "componentProps": {
    "flexDirection": "column",
    "gap": 12
  },
  "fields": [ ... ]
}
```

对应 themeConfig.css：
```css
.hover-lift:hover {
  position: relative;
  top: -10px;
}
```
