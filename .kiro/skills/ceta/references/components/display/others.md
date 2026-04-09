# 其他展示组件

## Text 文本

```json
{
  "component": "Text",
  "componentProps": { "content": "这是一段说明文字" },
  "style": { "fontSize": 14, "color": "#333" }
}
```

### componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| content | string | 文本内容，支持 Variable Pattern |

### 样式规则

Text 的所有样式放在 `style`（组件根层），不是 `componentProps.style`：

| 样式 | 放在哪里 | 示例 |
|------|---------|------|
| fontSize、fontWeight、color | `style` | `"style": { "fontSize": 20, "fontWeight": 700, "color": "#1e293b" }` |
| background、padding、borderRadius | `style` | 用于做状态徽章效果 |
| margin | `style` | 外层间距 |
| fontFamily | `style` | 如 monospace 字体 |

### 高级用法：状态徽章

```json
{
  "component": "Text",
  "className": "status-hk",
  "componentProps": { "content": "HK2" },
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
.status-hk { background: #10B981; color: white; }
.status-ss { background: #F59E0B; color: white; }
.status-nn { background: #6B7280; color: white; }
```

### 高级用法：等宽字体编号

```json
{
  "component": "Text",
  "componentProps": { "content": "QPERBI" },
  "style": { "fontFamily": "monospace", "fontSize": 15, "fontWeight": 700, "letterSpacing": "0.15em" }
}
```

HTML 识别：`<p>`、`<span>`、纯文本区域

---

## Divider 分割线

```json
{
  "component": "Divider",
  "componentProps": { "content": "分割线标题" }
}
```

HTML 识别：`<hr>`、`.ant-divider`

---

## Image 图片

```json
{
  "component": "Image",
  "componentProps": { "src": "https://example.com/img.jpg", "alt": "描述" }
}
```

HTML 识别：`<img>` 标签

---

## Steps 步骤条

```json
{
  "component": "Steps",
  "componentProps": {
    "current": 1,
    "items": [
      { "key": "step1", "label": "步骤1" },
      { "key": "step2", "label": "步骤2" },
      { "key": "step3", "label": "步骤3" }
    ],
    "style": {
      "padding": "12px 40px",
      "backgroundColor": "#f8fafc",
      "borderBottom": "1px solid #e2e8f0"
    }
  }
}
```

### componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| current | number | 当前步骤索引（从 0 开始） |
| items | array | 步骤列表，每项有 key 和 label |
| style | object | 步骤条容器样式 |

注意：Steps 的 items 没有 `fields`（与 Collapse/Tabs 不同）。
HTML 中的步骤条/进度指示器应映射为 Steps 组件，不要用多个 Text 手动拼。

---

## Progress 进度条

```json
{
  "component": "Progress",
  "componentProps": { "percent": 75 }
}
```

---

## 完整展示组件清单

| component | 说明 | 常用程度 |
|-----------|------|---------|
| Table | 数据表格 | ★★★ |
| Tabs | 标签页 | ★★ |
| Button | 按钮 | ★★ |
| Text | 文本 | ★★ |
| Divider | 分割线 | ★★ |
| Steps | 步骤条 | ★★ |
| Image | 图片 | ★ |
| Progress | 进度条 | ★ |
| Status | 状态标签 | ★ |
| NavTabs | 导航标签 | ★ |
| List | 列表 | ★ |
| Tree | 树形控件 | ★ |
| EChart | 图表 | ☆ |
| IFrame | 内嵌页面 | ☆ |
