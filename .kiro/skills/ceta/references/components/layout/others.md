# 其他布局组件

> Stack 已独立为 `Stack.md`，详见该文件。

## Box 盒子布局

最基础的容器，无特殊布局逻辑。常用作样式容器、装饰性元素（圆形徽章、渐变色块等）。

### 基础用法

```json
{
  "component": "Box",
  "style": {
    "padding": 16,
    "background": "#fafafa",
    "border": "1px solid #e8e8e8",
    "borderRadius": 4
  },
  "fields": [ ... ]
}
```

### componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| className | string | 自定义 class |

注意：Box 的视觉样式放在外层 `style`，不放在 `componentProps.style`。

### 样式规则

| 样式目标 | 放在哪里 |
|---------|---------|
| 所有视觉样式（背景、边框、圆角、padding、阴影、尺寸） | `style` |

### 高级用法：渐变色图标/徽章

用 Box 做圆角渐变色背景块，放文字或图标：

```json
{
  "component": "Box",
  "style": {
    "width": 64,
    "height": 64,
    "borderRadius": 16,
    "background": "linear-gradient(135deg, #004B8D 0%, #0066CC 50%, #00A8E8 100%)",
    "display": "flex",
    "alignItems": "center",
    "justifyContent": "center",
    "color": "white",
    "textAlign": "center"
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": "NH" },
      "style": { "fontSize": 14, "fontWeight": 700, "color": "white" }
    }
  ]
}
```

### 高级用法：圆形数字徽章

```json
{
  "component": "Box",
  "style": {
    "width": 28,
    "height": 28,
    "borderRadius": "50%",
    "background": "#003366",
    "display": "flex",
    "alignItems": "center",
    "justifyContent": "center"
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": "1" },
      "style": { "fontSize": 14, "fontWeight": 600, "color": "white" }
    }
  ]
}
```

### 高级用法：提示信息框

```json
{
  "component": "Box",
  "style": {
    "padding": "10px 14px",
    "background": "#e6f0fa",
    "borderRadius": 6,
    "border": "1px solid #b3d4f0"
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": "提示信息内容" },
      "style": { "fontSize": 13, "color": "#003366" }
    }
  ]
}
```

HTML 识别：普通 `<div>` 容器（无 flex/grid 布局）

---

## Mobile 移动端布局

移动端专用布局容器。

```json
{
  "component": "Mobile",
  "fields": [ ... ]
}
```
