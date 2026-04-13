# Image 图片

展示型组件，用于显示图片。

## schemaJson

```json
{
  "component": "Image",
  "componentProps": {
    "src": "https://example.com/img.jpg",
    "alt": "描述",
    "width": 200,
    "height": 150
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| src | string | 图片地址，支持 Variable Pattern |
| alt | string | 图片的 alt 描述 |
| width | number \| string | 图片宽度 |
| height | number \| string | 图片高度 |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

其余 `componentProps` 会透传给 `<img>` 元素。

## HTML 识别

- CETA 原生：`.image-element`
- 通用 HTML：`<img>` 标签
- `src` 属性 → `componentProps.src`
- `alt` 属性 → `componentProps.alt`
