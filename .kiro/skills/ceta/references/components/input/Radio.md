# Radio 单选按钮组

Field type: `RADIO`

## schemaJson

```json
{
  "component": "Radio",
  "id": "gender",
  "title": "性别",
  "componentProps": {
    "options": [
      { "label": "男", "value": "male" },
      { "label": "女", "value": "female" }
    ],
    "direction": "horizontal",
    "gap": "middle"
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| options | `Array<{ value: string \| number \| bool, label: string }>` | | 可选项列表 |
| direction | `'horizontal'` \| `'vertical'` | | Radio 元素的排列方向 |
| gap | `'small'` \| `'middle'` \| `'large'` \| number | `'small'` | Radio 之间的间距 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Radio.Group](https://ant.design/components/radio)。

## 样式规则

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| Radio 组件内部样式 | `componentProps.style` | 控件本身 |
| 外层容器（宽度、margin） | `style`（组件根层） | 外层 wrapper |

## HTML 识别

- CETA 原生：`.ant-radio-group` 或 `.radio-element`
- 通用 HTML：多个 `<input type="radio">` 且 `name` 相同
- 从 `<label>` 或相邻文本提取 options 的 label
- `name` 属性 → `id`

## 示例：垂直排列

```json
{
  "component": "Radio",
  "id": "priority",
  "title": "优先级",
  "componentProps": {
    "options": [
      { "label": "高", "value": "high" },
      { "label": "中", "value": "medium" },
      { "label": "低", "value": "low" }
    ],
    "direction": "vertical",
    "gap": "middle"
  }
}
```
