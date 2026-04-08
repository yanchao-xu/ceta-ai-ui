# Collapse 折叠面板

通过 `componentProps.items` 定义多个折叠区域，每个 item 有自己的 `fields`。

**注意：Collapse 的子组件在 `componentProps.items[].fields` 中，不是自身的 `fields`。**

## schemaJson

```json
{
  "component": "Collapse",
  "componentProps": {
    "defaultActiveKey": ["key1", "key2"],
    "itemSplit": true,
    "items": [
      {
        "key": "key1",
        "label": "基本信息",
        "fields": [
          {
            "component": "Grid",
            "componentProps": { "colNumber": 2, "blankChildPlace": false },
            "fields": [
              { "id": "name", "title": "姓名", "component": "Input" },
              { "id": "age", "title": "年龄", "component": "NumberPicker" }
            ]
          }
        ]
      },
      {
        "key": "key2",
        "label": "联系方式",
        "fields": [
          {
            "component": "Grid",
            "componentProps": { "colNumber": 2, "blankChildPlace": false },
            "fields": [
              { "id": "phone", "title": "电话", "component": "Input" },
              { "id": "email", "title": "邮箱", "component": "Input" }
            ]
          }
        ]
      }
    ]
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| items | array | 折叠项数组，每项有 `key`、`label`、`fields` |
| defaultActiveKey | string[] | 默认展开的 key 列表 |
| itemSplit | boolean | 项之间是否有分割线 |

## HTML 识别

- CETA 原生：`.collapse-element` 或 `.ant-collapse`
- 每个 `.ant-collapse-item` → 一个 item
- `.ant-collapse-item-active` → 收集 key 到 `defaultActiveKey`
- `.ant-collapse-header-text` 文本 → item 的 `label`
- `.ant-collapse-content-box` 内容 → item 的 `fields`
- `.item-split` class → `itemSplit: true`

## 典型嵌套结构

```
Card
└── Collapse
    ├── item: "基本信息"
    │   └── Grid（colNumber: 2, blankChildPlace: false）
    │       ├── Input
    │       └── Input
    └── item: "详细信息"
        └── Grid（colNumber: 2, blankChildPlace: false）
            ├── Select
            └── DatePicker
```
