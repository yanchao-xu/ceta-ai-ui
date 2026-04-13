# ACL 关联引用

Field type: `ACL`

## schemaJson — 基础用法

```json
{
  "component": "ACL",
  "id": "relatedUser",
  "title": "关联用户",
  "componentProps": {
    "dataSource": {
      "token": "user-form",
      "pbcToken": "user-management"
    },
    "mapping": { "value": "id", "label": "username" },
    "multiple": false,
    "placeholder": "请选择用户"
  }
}
```

## schemaJson — 通过 dataUrl

```json
{
  "component": "ACL",
  "id": "relatedItem",
  "title": "关联项目",
  "componentProps": {
    "dataUrl": "/form/api/v2/form-entity-data/my-pbc/my-form/list",
    "mapping": { "value": "id", "label": "name" },
    "multiple": true,
    "stringEqual": true
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| dataSource | `{ token, pbcToken, listUrl?, listIdsUrl?, listByIdsUrl? }` | | 指定 FormEntity 数据源 |
| dataUrl | string | | 一次性获取所有数据的 URL |
| idListUrl | string | | 懒加载模式：先获取所有 id |
| mapping | `{ value: string, label: string }` | `{ value: 'value', label: 'label' }` | 值映射关系 |
| useOriginValue | boolean | `false` | 是否保存原始映射值 |
| stringEqual | boolean | `true` | 比较选项时是否转字符串 |
| multiple | boolean | `false` | 是否多选 |
| columnDefs | array | | ACL 弹窗表格的列定义（ag-grid） |
| placeholder | string | | 输入框占位文本 |
| dataFilters | array | | 固定 filter 条件 |
| dataExclusion | `Array<string>` | | 排除某些数据 |
| dataResponseKeyPath | string | | 数据在 response 中的路径 |
| supportImport | boolean | `false` | 是否支持导入 |
| supportExport | boolean | `false` | 是否支持导出 |
| exportColumns | `Array<{ value, label }>` | | 导出的列 |
| unit | string | `'项'` | 选择器的单位 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

## dataSource 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| token | string | FormEntity token |
| pbcToken | string | PBC token |
| listUrl | string | 获取列表详细数据的地址 |
| listIdsUrl | string | 获取所有数据 id 的地址 |
| listByIdsUrl | string | 根据 id 获取详细数据的地址 |

## HTML 识别

- CETA 原生：`.acl-element` 或 `.ant-select`（带弹窗表格）
- 通用 HTML：带搜索和弹窗选择的关联字段
