# Cascader 级联选择

Field type: `CASCADER`

## schemaJson — 静态选项

```json
{
  "component": "Cascader",
  "id": "region",
  "title": "省市区",
  "componentProps": {
    "options": [
      {
        "value": "zhejiang",
        "label": "浙江",
        "children": [
          { "value": "hangzhou", "label": "杭州" },
          { "value": "ningbo", "label": "宁波" }
        ]
      }
    ],
    "changeOnSelect": true,
    "placeholder": "请选择地区"
  }
}
```

## schemaJson — 数据源

```json
{
  "component": "Cascader",
  "id": "cascader",
  "title": "省市区级联",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/cascader-demo",
    "dataResponseKeyPath": "output",
    "changeOnSelect": true,
    "placeholder": "请输入所属地区"
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| options | `Array<{ value, label, children? }>` | | 静态级联选项 |
| dataUrl | string | | 通过 URL 获取数据，支持 Variable Pattern |
| dataResponseKeyPath | string | | 数据在 response 中的路径 |
| dataFilters | array | | 请求数据源的固定 filter 条件 |
| mapping | `{ value, label, children }` | `{ value: 'value', label: 'label', children: 'children' }` | 字段映射关系 |
| multiple | boolean | `false` | 是否多选 |
| useOriginValue | boolean | `false` | 是否保存映射关系的原始值 |
| stringEqual | boolean | `false` | 选项 id 是否转换成字符串比较 |
| buildTreeBy | string \| object | | 根据字段自动构建树状结构 |
| translateDataResponse | boolean | `false` | 是否翻译请求结果 |
| debounceTime | number | `200` | 请求数据的 debounce time |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Cascader](https://ant.design/components/cascader)。

## HTML 识别

- CETA 原生：`.cascader-element` 或 `.ant-cascader`
- 通用 HTML：多级联动的 `<select>` 组合

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| TreeSelect | 树选择（单层展开） | TREE_SELECT |
| Select | 单级下拉选择 | SELECT |
