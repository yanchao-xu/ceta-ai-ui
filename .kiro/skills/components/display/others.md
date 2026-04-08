# 其他展示组件

## Text 文本

```json
{
  "component": "Text",
  "componentProps": { "content": "这是一段说明文字" }
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
    "items": [
      { "key": "step1", "label": "步骤1" },
      { "key": "step2", "label": "步骤2" }
    ]
  }
}
```

注意：Steps 的 items 没有 `fields`（与 Collapse/Tabs 不同）。

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
| Image | 图片 | ★ |
| Steps | 步骤条 | ★ |
| Progress | 进度条 | ★ |
| Status | 状态标签 | ★ |
| NavTabs | 导航标签 | ★ |
| List | 列表 | ★ |
| Tree | 树形控件 | ★ |
| EChart | 图表 | ☆ |
| IFrame | 内嵌页面 | ☆ |
