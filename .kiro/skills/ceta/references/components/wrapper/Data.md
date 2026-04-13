# Data 数据容器

特殊组件，用于从数据源获取数据并提供给子组件使用。不渲染可见 UI，仅作为数据提供者。

## schemaJson

```json
{
  "component": "Data",
  "componentProps": {
    "alias": "requestData",
    "setToFormData": { "merge": false },
    "dataSource": {
      "token": "apply-form",
      "pbcToken": "todo-list-prepare"
    },
    "dataResponseKeyPath": "results[0]"
  },
  "fields": [
    { "component": "Text", "componentProps": { "content": ":name" } }
  ]
}
```

## 数据访问方式

假设返回数据为 `{ "id": 12345, "name": "A name", "description": "Some text" }`：

1. 子组件中通过 Variable Pattern 直接访问：`:name`、`:description`
2. 通过 `setToFormData` 放入表单数据（input 组件可用，随表单提交）
3. 通过 `alias` 放入 context，子组件外可通过 `:context.dataSource.requestData.name` 访问

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| alias | string \| object | | 将数据放入 context.dataSource 下的属性名 |
| setToFormData | object | | 将数据放入表单数据（`{ merge: bool }`） |
| intervalTime | number | | 自动刷新间隔（毫秒） |
| renderWhileLoading | boolean | `false` | loading 时也渲染子元素 |
| dataSource | `{ token, pbcToken, listUrl }` | | 数据源 |
| dataUrl | string | | 通过 URL 获取数据 |
| dataFilters | array | | 固定 filter 条件 |
| sortModel | `Array<{ colId, sort, sortType }>` | | 排序方式 |
| dataResponseKeyPath | string | | 数据在 response 中的路径 |
| transformDataResponse | string | | 数据转换表达式 |
| translateDataResponse | boolean | `true` | 是否翻译请求结果 |
| debounceTime | number | `200` | 请求 debounce time |
| defaultValue | object | | 默认数据，与 API 返回数据 merge |
| httpMethod | `'get'` \| `'post'` | `'get'` | HTTP 方法 |
| selectColId | `Array<string>` | | 加载的数据字段 |
| multiDataSource | array | | 多数据源并发请求 |
| multiDataSourceFlat | boolean | `false` | 是否将多数据源 flatten 成一个数组 |
| className | string | | Root div 的 className |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| getData | func | 获取数据 |
| refresh | func | 刷新数据 |

### 事件

| Name | callback | Description |
|------|----------|-------------|
| dataFetched | `(formApi) => any` | 获取数据成功时触发 |
| childrenFirstRendered | `(formApi) => void` | 子元素第一次渲染时触发 |

## 使用场景

- 为展示型组件（Text、Image 等）提供动态数据
- 为 Steps、Status 等组件提供数据驱动
- 跨组件共享数据（通过 alias）
