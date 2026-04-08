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
| disabledDate | object | 禁用日期规则 `{ type: "before"/"after", value: "2023-01-01" }` |
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
