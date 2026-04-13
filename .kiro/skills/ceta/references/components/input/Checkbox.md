# Checkbox 多选框组

Field type: `CHECKBOX`

## schemaJson

```json
{
  "component": "Checkbox",
  "id": "hobbies",
  "title": "兴趣爱好",
  "componentProps": {
    "options": [
      { "label": "篮球", "value": "1" },
      { "label": "足球", "value": "2" },
      { "label": "羽毛球", "value": "3" }
    ],
    "direction": "horizontal",
    "gap": "middle"
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| options | `Array<{ value: string \| number, label: string }>` | | 可选项列表 |
| direction | `'horizontal'` \| `'vertical'` | | Checkbox 元素的排列方向 |
| gap | `'small'` \| `'middle'` \| `'large'` \| number | `'small'` | Checkbox 之间的间距 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Checkbox.Group](https://ant.design/components/checkbox)。

## 样式规则

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| Checkbox 组件内部样式 | `componentProps.style` | 控件本身 |
| 外层容器（宽度、margin） | `style`（组件根层） | 外层 wrapper |

## HTML 识别

- CETA 原生：`.ant-checkbox-group` 或 `.checkbox-element`
- 通用 HTML：多个 `<input type="checkbox">` 且语义为选项组
- 注意：单个 checkbox 且语义为开关（"是否XX"）应映射为 `Switch`，不是 `Checkbox`
- `name` 属性 → `id`

## 示例：禁用状态

```json
{
  "component": "Checkbox",
  "id": "hobbies",
  "title": "兴趣爱好",
  "disabled": true,
  "componentProps": {
    "options": [
      { "label": "篮球", "value": "1" },
      { "label": "足球", "value": "2" }
    ],
    "direction": "horizontal"
  }
}
```

## 示例：联动（全选）

```json
{
  "component": "Checkbox",
  "id": "selectAll",
  "title": "选择",
  "onFieldValueChange": "this.keyPath=='0'?this.value='1':''",
  "componentProps": {
    "options": [
      { "label": "全选", "value": "0" },
      { "label": "选项A", "value": "1" },
      { "label": "选项B", "value": "2" }
    ],
    "direction": "horizontal"
  }
}
```
