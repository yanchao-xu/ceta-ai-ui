# ListSelect 列表选择

输入型组件，以列表卡片形式展示选项供用户选择。

## schemaJson — 静态选项

```json
{
  "component": "ListSelect",
  "id": "plan",
  "title": "选择方案",
  "componentProps": {
    "mode": "single",
    "listStyle": "style1",
    "size": "medium",
    "showCheckMark": true,
    "options": [
      { "value": "basic", "label": "基础版" },
      { "value": "pro", "label": "专业版" },
      { "value": "enterprise", "label": "企业版" }
    ]
  }
}
```

## schemaJson — 数据源

```json
{
  "component": "ListSelect",
  "id": "template",
  "title": "选择模板",
  "componentProps": {
    "mode": "multiple",
    "dataSource": {
      "token": "template-form",
      "pbcToken": "template-management"
    },
    "mapping": { "value": "id", "label": "name" }
  },
  "fields": [
    { "component": "Text", "componentProps": { "content": ":name" } },
    { "component": "Text", "componentProps": { "content": ":description", "color": "#999" } }
  ]
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| mode | `'single'` \| `'multiple'` \| `'tags'` | `'single'` | 选择模式 |
| valueType | `'value'` \| `'item'` \| `'valueAlwaysArray'` \| `'itemAlwaysArray'` | `'itemAlwaysArray'` | 值的格式 |
| mapping | `{ value, label }` | `{ value: 'value', label: 'label' }` | 值映射关系 |
| useOriginValue | boolean | `false` | 是否保存原始映射值 |
| stringEqual | boolean | `true` | 比较选项时是否转字符串 |
| options | array | | 静态选项 |
| dataSource | `{ token, pbcToken, listUrl }` | | 数据源 |
| dataUrl | string | | 通过 URL 获取数据 |
| dataFilters | array | | 固定 filter 条件 |
| showCheckMark | boolean | `false` | 选中项是否显示打勾 |
| listStyle | `'style1'` \| `'style2'` \| `'style3'` \| `'style4'` | `'style1'` | 选项样式 |
| size | `'medium'` \| `'large'` | `'medium'` | 选项尺寸 |
| itemProps | object | | 传给每个子元素的属性 |
| selectedItemProps | object | | 传给选中子元素的属性 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| getOptions | `() => array` | 获取所有可选项 |
| refresh | `() => void` | 刷新数据 |
