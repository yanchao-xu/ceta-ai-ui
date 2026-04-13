# Switch 开关

Field type: `SWITCH`

## schemaJson

```json
{
  "component": "Switch",
  "id": "enabled",
  "title": "是否启用"
}
```

## HTML 识别

- CETA 原生：`.ant-switch` 或 `.switch-element`
- 通用 HTML：单个 `<input type="checkbox">` 且语义为开关（"是否XX"）
- 注意：多个 checkbox 组成的选项组应映射为 `Checkbox`，不是 `Switch`
