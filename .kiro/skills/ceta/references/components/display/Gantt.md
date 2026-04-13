# Gantt 甘特图

展示型组件，基于 DHTMLX Gantt 渲染甘特图。

## schemaJson

```json
{
  "component": "Gantt",
  "id": "projectGantt",
  "componentProps": {
    "dataSources": {
      "task": {
        "token": "task-form",
        "pbcToken": "project-management"
      }
    },
    "mapping": {
      "text": "taskName",
      "start_date": "startDate",
      "end_date": "endDate",
      "duration": "duration",
      "progress": "progress",
      "parent": "parentId"
    },
    "columnDefs": [
      { "field": "taskName", "headerName": "任务名称", "width": 200 },
      { "field": "startDate", "headerName": "开始日期", "width": 120 }
    ],
    "ganttConfig": {},
    "pinnedFilter": ["status"]
  },
  "style": { "height": 600 }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| dataSources | `{ task, link?, calendar? }` | 数据源（任务、关联、日历） |
| mapping | object | 数据字段映射（text, start_date, end_date, duration, progress, parent, type） |
| columnDefs | array | 左侧表格列定义（ag-grid） |
| relation | `{ colId, value }` | 作为表单子表的关系配置 |
| ganttConfig | object | DHTMLX Gantt 配置 |
| dateOptions | object | 日期配置（noTimeZone, timeZone, showTime, endDateIncludeSelf） |
| toolbarFields | array | 工具栏自定义元素（JSON 配置） |
| pinnedFilter | `Array<string>` | 默认显示的 filter 列 |
| suppressAddButton | boolean | 禁止默认新增按钮 |
| addButtonProps | object | 新增按钮属性 |
| className | string | 组件的 className |
| style | object | 组件的 style |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| gantt | object | DHTMLX Gantt 的 api 接口 |
| refresh | `() => void` | 刷新数据 |
