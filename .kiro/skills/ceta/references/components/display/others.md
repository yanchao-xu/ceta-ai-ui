# 其他展示组件

> 以下组件已有独立文档：Text、Divider、Image、Steps、Progress、Status、Icon、EChart、List、NavTabs、Audio、QRCode、Gantt、ListSelect。
> Data 已迁移至 `wrapper/Data.md`。

---

## Tree 树形控件

```json
{
  "component": "Tree",
  "id": "orgTree",
  "componentProps": {
    "dataSource": { "token": "org-form", "pbcToken": "org-management" },
    "mapping": { "value": "id", "label": "name", "children": "children" }
  }
}
```

HTML 识别：`.tree-element` 或 `.ant-tree`

---

## IFrame 内嵌页面

```json
{
  "component": "IFrame",
  "componentProps": {
    "src": "https://example.com/embed",
    "style": { "width": "100%", "height": 500, "border": "none" }
  }
}
```

---

## Video 视频

```json
{
  "component": "Video",
  "componentProps": {
    "src": "https://example.com/video.mp4"
  }
}
```

---

## Carousel 轮播

```json
{
  "component": "Carousel",
  "componentProps": {},
  "fields": [ ... ]
}
```

---

## Tooltip 提示

```json
{
  "component": "Tooltip",
  "componentProps": {
    "title": "提示内容"
  },
  "fields": [ ... ]
}
```

---

## TodoList 待办列表

```json
{
  "component": "TodoList",
  "id": "todos",
  "componentProps": {}
}
```

---

## 完整展示组件清单

| component | 说明 | 独立文档 |
|-----------|------|---------|
| Table | 数据表格 | ✔️ Table.md |
| Tabs | 标签页 | ✔️ Tabs.md |
| Button | 按钮 | ✔️ Button.md |
| Text | 文本 | ✔️ Text.md |
| Divider | 分割线 | ✔️ Divider.md |
| Steps | 步骤条 | ✔️ Steps.md |
| Image | 图片 | ✔️ Image.md |
| Progress | 进度条 | ✔️ Progress.md |
| Status | 状态标签 | ✔️ Status.md |
| Icon | 图标 | ✔️ Icon.md |
| EChart | 图表 | ✔️ EChart.md |
| List | 列表 | ✔️ List.md |
| NavTabs | 导航标签 | ✔️ NavTabs.md |
| Audio | 音频播放器 | ✔️ Audio.md |
| QRCode | 二维码 | ✔️ QRCode.md |
| Gantt | 甘特图 | ✔️ Gantt.md |
| ListSelect | 列表选择 | ✔️ ListSelect.md |
| Data | 数据容器 | 已迁移至 wrapper/Data.md |
| Tree | 树形控件 | 本文件 |
| IFrame | 内嵌页面 | 本文件 |
| Video | 视频 | 本文件 |
| Carousel | 轮播 | 本文件 |
| Tooltip | 提示 | 本文件 |
| TodoList | 待办列表 | 本文件 |
