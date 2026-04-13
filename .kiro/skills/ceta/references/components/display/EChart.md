# EChart 图表

展示型组件，基于 Apache ECharts 渲染各类图表。

## schemaJson — 轴坐标系（柱状图/折线图）

```json
{
  "component": "EChart",
  "id": "salesChart",
  "componentProps": {
    "category": "axis",
    "chartSettings": {
      "xAxisField": "month",
      "yAxisField": "amount"
    },
    "echartOption": {
      "title": { "text": "月度销售" }
    },
    "series": {
      "type": "bar"
    },
    "dataSource": {
      "token": "sales-data",
      "pbcToken": "report-pbc"
    }
  },
  "style": { "height": 400 }
}
```

## schemaJson — 饼图

```json
{
  "component": "EChart",
  "id": "pieChart",
  "componentProps": {
    "category": "pie",
    "chartSettings": {
      "legendField": "category",
      "valueField": "count"
    },
    "dataSource": {
      "token": "category-stats",
      "pbcToken": "report-pbc"
    }
  },
  "style": { "height": 300 }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| category | `'axis'` \| `'pie'` \| `'generic'` | | axis=轴坐标系，pie=饼图，generic=直接用数据源结果作为 option |
| chartSettings | object | | axis 配横纵轴字段，pie 配 legend/value 字段 |
| echartOption | object | | ECharts 的 option 设置 |
| series | object | | ECharts 的 series 设置 |
| dataSource | `{ token, pbcToken, listUrl }` | | 数据源 |
| dataUrl | string | | 通过 URL 获取数据 |
| dataFilters | array | | 固定 filter 条件 |
| sortModel | `Array<{ colId, sort, sortType }>` | | 排序方式 |
| dataResponseKeyPath | string | `'results'` | 数据在 response 中的路径 |
| transformDataResponse | string | | 数据转换表达式 |
| translateDataResponse | boolean | `true` | 是否翻译请求结果 |
| debounceTime | number | `200` | 请求 debounce time |
| httpMethod | `'get'` \| `'post'` | `'post'` | HTTP 方法 |
| selectColId | `Array<string>` | | 加载的数据字段 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| chart | object | EChart 实例 |

## HTML 识别

- CETA 原生：`.echart-element`
- 通用 HTML：`<canvas>` 图表容器
