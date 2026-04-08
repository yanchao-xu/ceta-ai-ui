# Tabs 标签页

通过 `componentProps.items` 定义多个标签页，每个 tab 有自己的 `fields`。

**注意：和 Collapse 一样，子组件在 `componentProps.items[].fields` 中。**

## schemaJson

```json
{
  "component": "Tabs",
  "componentProps": {
    "items": [
      {
        "key": "tab1",
        "label": "标签1",
        "fields": [ ... ]
      },
      {
        "key": "tab2",
        "label": "标签2",
        "fields": [ ... ]
      }
    ]
  }
}
```

## HTML 识别

- CETA 原生：`.ant-tabs`、`role="tablist"`
- 通用 HTML：tab 导航结构

## 同类组件

| component | 说明 |
|-----------|------|
| NavTabs | 导航标签，结构类似 Tabs |
