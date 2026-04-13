# DatePicker 日期选择

Field type: `DATE`

## schemaJson

```json
{
  "component": "DatePicker",
  "id": "startDate",
  "title": "开始日期",
  "componentProps": { "showTime": true }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| showTime | boolean | 是否显示时间选择 |
| disabledDate | object | 禁用日期规则（见下方详细说明） |

### disabledDate 详细说明

`disabledDate` 用于限制可选日期范围。

| 属性 | 类型 | 说明 |
|------|------|------|
| type | `"before"` / `"after"` | `before` = 禁用指定日期之前，`after` = 禁用指定日期之后 |
| value | string | ISO-8601 日期，或 `:fieldName` 引用其他字段的值 |

固定日期：
```json
{ "disabledDate": { "type": "before", "value": "2025-01-01" } }
```

引用其他字段（如结束日期不能早于开始日期）：
```json
{
  "component": "DatePicker",
  "id": "endDate",
  "title": "结束日期",
  "componentProps": {
    "disabledDate": { "type": "before", "value": ":startDate" }
  }
}
```
| format | string | 日期格式 |

## HTML 识别

- CETA 原生：`.date-picker-element` 或 `.ant-picker`
- CETA 原生带时间：`.date-picker-element.show-time` → `showTime: true`
- 通用 HTML：`<input type="date">`、`<input type="datetime-local">`

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| DateRangePicker | 日期范围，`.ant-picker-range` | DATE |
| TimePicker | 时间选择 | TIME |
| TimeRangePicker | 时间范围 | TIME |
