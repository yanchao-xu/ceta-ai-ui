# 其他布局组件

> Stack 已独立为 `Stack.md`，详见该文件。

## Box 盒子布局

最基础的容器，无特殊布局逻辑，仅作为分组容器或样式容器。

```json
{
  "component": "Box",
  "componentProps": {
    "style": {
      "padding": 16,
      "backgroundColor": "#fafafa",
      "border": "1px solid #e8e8e8",
      "borderRadius": 4
    }
  },
  "fields": [ ... ]
}
```

### componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| className | string | 自定义 class |
| style | object | 内部样式 |

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
