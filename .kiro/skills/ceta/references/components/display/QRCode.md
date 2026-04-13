# QRCode 二维码

展示型组件，用于生成和显示二维码。

## schemaJson

```json
{
  "component": "QRCode",
  "componentProps": {
    "text": "https://example.com",
    "type": "text",
    "title": "扫码访问",
    "description": "使用手机扫描二维码",
    "width": 200,
    "height": 200,
    "QRProps": { "margin": 2, "width": 180 }
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| text | string | 转换成二维码的文本信息 |
| type | `'text'` \| `'internalUrl'` | 二维码类型 |
| title | string | 二维码标题 |
| description | string | 二维码描述信息 |
| width | number | 二维码宽度 |
| height | number | 二维码高度 |
| QRProps | `{ margin: number, width: number }` | qrcode toDataURL 方法的属性 |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

## HTML 识别

- CETA 原生：`.qrcode-element`
- 通用 HTML：`<canvas>` 二维码容器
