# Select 下拉选择

Field type: `SELECT`

## schemaJson — 静态选项

```json
{
  "component": "Select",
  "id": "status",
  "title": "状态",
  "componentProps": {
    "options": [
      { "label": "启用", "value": "active" },
      { "label": "禁用", "value": "inactive" }
    ],
    "multiple": false,
    "stringEqual": true
  }
}
```

## schemaJson — 引用数据源

```json
{
  "component": "Select",
  "id": "department",
  "title": "部门",
  "componentProps": {
    "dataSource": { "token": "department-form", "pbcToken": "org-management" },
    "mapping": { "value": "id", "label": "name" }
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| options | array | 静态选项 `[{ label, value }]` |
| multiple | boolean | 是否多选 |
| stringEqual | boolean | 值比较是否用字符串 |
| dataSource | object | 引用其他表单数据源 `{ token, pbcToken }` |
| mapping | object | 数据源字段映射 `{ value, label }` |

## HTML 识别

- CETA 原生：`.select-element` 或 `.ant-select`
- 通用 HTML：`<select>` 标签，从 `<option>` 提取 options
- `name` 属性 → `id`

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| Radio | 单选按钮组，`<input type="radio">` 或 `.ant-radio` | RADIO |
| Checkbox | 多选框组，`<input type="checkbox">` 或 `.ant-checkbox` | CHECKBOX |
| Cascader | 级联选择 | CASCADER |
| TreeSelect | 树选择 | TREE_SELECT |
| ListSelect | 列表选择 | — |

### Radio 示例

```json
{
  "component": "Radio",
  "id": "gender",
  "title": "性别",
  "componentProps": {
    "options": [
      { "label": "男", "value": "male" },
      { "label": "女", "value": "female" }
    ]
  }
}
```
