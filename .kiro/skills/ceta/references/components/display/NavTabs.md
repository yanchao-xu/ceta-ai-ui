# NavTabs 导航标签

展示型组件，用于页面级导航标签切换，通过链接跳转不同页面。

## schemaJson

```json
{
  "component": "NavTabs",
  "componentProps": {
    "activeKey": "tab1",
    "centered": false,
    "showUnderLine": true,
    "items": [
      { "key": "tab1", "label": "概览", "href": "/page/overview" },
      { "key": "tab2", "label": "详情", "href": "/page/detail" },
      { "key": "tab3", "label": "历史", "href": "/page/history" }
    ]
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| activeKey | string | | 当前激活的 Tab |
| centered | boolean | `false` | 是否居中显示 |
| items | `Array<ItemProps>` | | Tabs 的内容 |
| showUnderLine | object | | 底部是否显示边框 |

## ItemProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| key | number | | Tab 的 key |
| label | string | | Tab 的名字 |
| href | string | | 链接地址 |
| hrefIsSiteBased | boolean | `false` | 是否为站点级跳转（跨 PBC） |
| suppressInheritIncludeDeleted | boolean | `false` | 是否禁止继承 include_deleted 参数 |
| hidden | boolean | `false` | 是否隐藏，支持条件表达式 |

## 与 Tabs 的区别

| 组件 | 用途 | 切换方式 |
|------|------|---------|
| NavTabs | 页面级导航 | 通过 href 跳转页面 |
| Tabs | 内容区域切换 | 切换显示不同 fields |

## HTML 识别

- CETA 原生：`.nav-tabs-element`
- 通用 HTML：导航标签栏
