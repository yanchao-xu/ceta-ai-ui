# Status 状态标签

展示型组件，用于显示带颜色的状态标签。

## schemaJson

```json
{
  "component": "Status",
  "componentProps": {
    "enums": [
      { "value": "active", "label": "已激活", "color": "#52c41a" },
      { "value": "inactive", "label": "未激活", "color": "#d9d9d9" },
      { "value": "error", "label": "异常", "color": "#ff4d4f" }
    ]
  },
  "valueField": "status"
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| enums | `Array<{ value: string, label: string, color: string }>` | 状态列表 |
| statusColor | string | 当前状态颜色（不指定则从 enums 中通过 value 查找） |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

## HTML 识别

- CETA 原生：`.status-element`
- 通用 HTML：带颜色标记的状态文本、Badge 组件

## 示例：固定颜色

```json
{
  "component": "Status",
  "componentProps": {
    "statusColor": "#52c41a"
  },
  "valueField": "approvalStatus"
}
```
