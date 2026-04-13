# NumberPicker 数字输入

Field type: `NUMBER`

## schemaJson

```json
{
  "component": "NumberPicker",
  "id": "amount",
  "title": "金额",
  "validation": { "required": true, "minValue": 0, "maxValue": 999999 },
  "componentProps": {
    "precision": 2,
    "min": 0,
    "step": 0.01,
    "formatOptions": { "currency": "CNY" }
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| min | number | | 最小值 |
| max | number | | 最大值 |
| step | number | `1` | 每次改变步数，可以为小数 |
| precision | number | | 数值精度（小数位数） |
| formatOptions | `{ currency: string }` | | Intl.NumberFormat 的属性，用于货币格式化 |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [InputNumber](https://ant.design/components/input-number)。

## validation

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| required | boolean | `false` | 是否必填 |
| minValue | number | | 最小值校验 |
| maxValue | number | | 最大值校验 |

## 样式规则

| 样式目标 | 放在哪里 | 说明 |
|---------|---------|------|
| 输入框本身样式 | `componentProps.style` | 控件内部 |
| 外层容器（宽度、margin） | `style`（组件根层） | 外层 wrapper |

## HTML 识别

- CETA 原生：`.number-picker-element` 或 `.ant-input-number`
- 通用 HTML：`<input type="number">`
- `name` 属性 → `id`

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| NumericPicker | 与 NumberPicker 配置完全相同 | NUMERIC |
| Slider | 滑块选择数值 | — |
