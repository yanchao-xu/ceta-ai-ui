# EditableTable 可编辑表格

Field type: `EDITABLE_GRID`

## schemaJson

```json
{
  "component": "EditableTable",
  "id": "lineItems",
  "title": "明细行",
  "fieldConfig": {
    "pbcToken": "order-management",
    "formEntityToken": "order-line-form",
    "relations": [
      {
        "colId": "parentDataId",
        "filterType": "text",
        "type": "equals",
        "filter": "$CURRENT_FORM_DATA_ID"
      }
    ]
  },
  "componentProps": {
    "simpleMode": false,
    "canAddRow": true,
    "canDeleteRow": true,
    "maxRowCount": 100,
    "columnDefs": [
      { "field": "itemName", "headerName": "品名", "editable": true },
      { "field": "quantity", "headerName": "数量", "editable": true },
      { "field": "price", "headerName": "单价", "editable": true }
    ]
  }
}
```

## 特殊属性 fieldConfig

EditableTable 需要 `fieldConfig` 来关联子表数据：

```json
{
  "fieldConfig": {
    "pbcToken": "",
    "formEntityToken": "",
    "relations": [
      {
        "colId": "parentDataId",
        "filterType": "text",
        "type": "equals",
        "filter": "$CURRENT_FORM_DATA_ID"
      }
    ]
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| simpleMode | boolean | `false` | `true` 提交时全量发送；`false` 自动识别增量 `{ inserted, updated, deleted }` |
| columnDefs | array | | 列定义，支持 ag-grid 所有 columnDefs |
| defaultColDef | object | | ag-grid 默认列属性 |
| dataUrl | string | | 通过 URL 获取数据 |
| defaultValueDataUrl | string | | 表格默认值 URL，加载后放在 inserted 内 |
| dataFilters | array | | 请求数据源的固定 filter 条件 |
| dataResponseKeyPath | string | `'results'` | 数据在 response 中的路径 |
| canAddRow | boolean | `true` | 是否支持增加行 |
| newRowDefaultValues | object | | 新增行的默认值 |
| maxRowCount | number | `Infinity` | 可增加的最大行数 |
| canDeleteRow | boolean | `true` | 是否支持删除行 |
| transformDataResponse | string | | 数据转换表达式 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| gridApi | object | Ag-Grid 的 api 接口 |
| refresh | `() => void` | 刷新表格数据 |
| getRowData | `() => array` | 获取表格所有数据 |
| getOldRowData | `() => array` | 获取编辑前的所有数据 |
| getChangedData | `() => { inserted, updated, deleted }` | 获取修改数据 |
| addRow | `(rows) => void` | 添加一行或多行 |
| updateRow | `(rows) => void` | 修改行数据（靠 id 标识） |
| removeRow | `(rows) => void` | 删除行数据（靠 id 标识） |
| getSelectedIds | `() => array` | 返回选中行的 id 数组 |

### 事件

| Name | callback | Description |
|------|----------|-------------|
| dataFetched | `(formApi) => array` | 数据源获取到数据后触发 |
| addRow | `(formApi) => array` | 新增行时触发 |
| deleteRow | `(formApi) => array` | 删除行时触发 |

## HTML 识别

- CETA 原生：`.editable-table-element`
- 通用 HTML：可编辑的 `<table>` 或带增删行按钮的表格
