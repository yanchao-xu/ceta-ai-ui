# List 列表

展示型组件，用于渲染数据列表，每个列表项可包含自定义子组件。

## schemaJson

```json
{
  "component": "List",
  "id": "userList",
  "componentProps": {
    "dataSource": {
      "token": "user-form",
      "pbcToken": "user-management"
    },
    "empty": true,
    "lazyLoading": false,
    "style": {
      "flexDirection": "column",
      "gap": 12
    }
  },
  "fields": [
    {
      "component": "Card",
      "componentProps": { "title": ":name" },
      "fields": [
        { "component": "Text", "componentProps": { "content": ":email" } }
      ]
    }
  ]
}
```

## componentProps

> 数据源配置（`dataSource` / `dataUrl`）的完整规范见 `skills/ceta/references/data-source-rules.md`。两者二选一。

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| dataSource | `{ token, pbcToken, listUrl }` | | 数据源，与 `dataUrl` 二选一 |
| dataUrl | string | | 通过 URL 获取数据，与 `dataSource` 二选一 |
| dataFilters | array | | 固定 filter 条件 |
| sortModel | `Array<{ colId, sort, sortType }>` | | 排序方式 |
| dataResponseKeyPath | string | `'results'` | 数据在 response 中的路径 |
| transformDataResponse | string | | 数据转换表达式 |
| translateDataResponse | boolean | `true` | 是否翻译请求结果 |
| debounceTime | number | `200` | 请求 debounce time |
| httpMethod | `'get'` \| `'post'` | `'get'` | HTTP 方法 |
| selectColId | `Array<string>` | | 加载的数据字段 |
| empty | boolean \| object | `false` | 无数据时的占位符（antd Empty 属性） |
| itemProps | object | | 传给每个子元素顶层 div 的属性 |
| lazyLoading | boolean | `false` | 是否开启懒加载 |
| cacheBlockSize | number | `30` | 懒加载每次请求的数据量 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

注意：`direction`、`wrap`、`gap` 已废弃，请使用 `style.flexDirection`、`style.flexWrap`、`style.gap` 替代。

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| getData | `() => any` | 获取所有数据 |
| refresh | `() => void` | 刷新数据 |

## HTML 识别

- CETA 原生：`.list-element`
- 通用 HTML：重复结构的列表容器
