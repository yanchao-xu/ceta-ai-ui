# Rate 评分

Field type: `RATE`

## schemaJson

```json
{
  "component": "Rate",
  "id": "score",
  "title": "评分",
  "componentProps": {
    "count": 5
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| count | number | 总分数（星星数量） |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

其余 `componentProps` 会透传给 Ant Design [Rate](https://ant.design/components/rate)，常用透传属性：

| 属性 | 类型 | 说明 |
|------|------|------|
| allowHalf | boolean | 是否允许半星 |
| allowClear | boolean | 是否允许再次点击清除 |
| character | string | 自定义字符替换星星 |
| tooltips | `Array<string>` | 每个等级的提示文字 |

## HTML 识别

- CETA 原生：`.rate-element` 或 `.ant-rate`
- 通用 HTML：星级评分组件

## 示例：半星 + 自定义字符

```json
{
  "component": "Rate",
  "id": "rating",
  "title": "评级",
  "componentProps": {
    "allowHalf": true,
    "character": "A"
  },
  "defaultValue": 2.5
}
```

## 示例：带提示文字

```json
{
  "component": "Rate",
  "id": "satisfaction",
  "title": "满意度",
  "componentProps": {
    "tooltips": ["很差", "差", "一般", "好", "很好"]
  }
}
```
