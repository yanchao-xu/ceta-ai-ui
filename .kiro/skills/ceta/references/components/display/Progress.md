# Progress 进度条

展示型组件，用于显示进度百分比。

## schemaJson

```json
{
  "component": "Progress",
  "componentProps": {
    "percent": 75,
    "status": "success",
    "progressStyle": "style1"
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| percent | number \| string | | 进度百分比 |
| status | `'default'` \| `'warning'` \| `'success'` \| `'error'` | | 状态，支持 Variable Pattern |
| progressStyle | `'style1'` \| `'style2'` \| `'style3'` \| `'style4'` | `'style1'` | 外观样式，支持 Variable Pattern |
| className | string | | 组件的 className，支持 Variable Pattern |
| style | object | | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Progress](https://ant.design/components/progress)。

## HTML 识别

- CETA 原生：`.progress-element` 或 `.ant-progress`
- 通用 HTML：`<progress>` 标签、进度条 UI
