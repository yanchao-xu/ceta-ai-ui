# Table 数据表格

列表页的核心组件。

## schemaJson

```json
{
  "component": "Table",
  "id": "{entityToken}-table",
  "componentProps": {
    "dataSource": { "token": "{entityToken}" },
    "columnDefs": [ ... ],
    "addButtonHref": "/form/{entityToken}/layout/new",
    "pinnedFilter": ["field1", "field2"],
    "pagination": true
  },
  "style": { "height": "100%" }
}
```

## componentProps

| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| dataSource | object | 是 | `{ token }` 指向 FormEntity 的 token |
| columnDefs | array | 是 | 列定义数组 |
| addButtonHref | string | 否 | "新增"按钮跳转路径 |
| pinnedFilter | string[] | 否 | 固定显示的筛选字段 token |
| pagination | boolean | 否 | 是否分页 |
| dataFilters | array | 否 | 默认数据过滤条件 |

## columnDefs 列定义

```json
{
  "type": "TEXT_COLUMN",
  "colId": "fieldToken",
  "field": "fieldToken",
  "headerName": "列标题",
  "width": 120,
  "minWidth": 60,
  "pinned": "left"
}
```

### 列类型

| type | 说明 | 对应 Field type |
|------|------|----------------|
| TEXT_COLUMN | 文本列 | TEXT_BOX, TEXTAREA, PASSWORD |
| DATE_COLUMN | 日期列 | DATE, TIME |
| NUMBER_COLUMN | 数字列 | NUMBER |
| AVATAR_COLUMN | 头像列 | UPLOAD（头像） |
| ACL_COLUMN | 关联引用列 | ACL |
| SWITCH_COLUMN | 开关列 | SWITCH |
| STATUS_COLUMN | 状态列 | SELECT（状态） |
| ACTION_COLUMN | 操作列 | 无（固定最后） |

### ACTION_COLUMN

```json
{
  "type": "ACTION_COLUMN",
  "width": 240,
  "headerName": "操作",
  "cellRendererParams": {
    "displayLikeButton": true,
    "showIcon": true,
    "actions": [
      { "type": "EDIT", "href": "/form/{entityToken}/layout/edit/:id" },
      { "type": "VIEW", "href": "/form/:id/view" },
      { "type": "DELETE" }
    ]
  },
  "pinned": "right"
}
```

### AVATAR_COLUMN

```json
{
  "type": "AVATAR_COLUMN",
  "colId": "avatar",
  "field": "avatar",
  "headerName": "头像",
  "cellRendererParams": {
    "componentProps": { "listType": "picture-card", "maxCount": 1 },
    "avatarStyle": "avatar-label"
  },
  "width": 80,
  "minWidth": 60
}
```

### dataFilters

```json
{
  "dataFilters": [
    {
      "conditions": [
        { "id": "fieldToken", "filterType": "text", "type": "equals", "filter": "value", "colId": "fieldToken" }
      ],
      "operator": "OR"
    }
  ]
}
```

## href 路径规则

| 场景 | href 格式 |
|------|----------|
| 新增 | `/form/{entityToken}/layout/{layoutToken}` |
| 编辑 | `/form/{entityToken}/layout/{layoutToken}/:id` |
| 查看 | `/form/:id/view` |

## HTML 识别

- CETA 原生：`.icp-ag-table`、`.ag-theme-quartz`、`.table-element`
- `id` 属性 → Table 的 `id`
- `.icp-table-filter-item[data-id]` → `pinnedFilter` 数组
- `.icp-toolbar-add-button` → `addButtonHref`
- 通用 HTML：`<table>` 标签，从 `<th>` 提取列定义
