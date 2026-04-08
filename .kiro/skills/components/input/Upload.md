# Upload 文件上传

Field type: `UPLOAD`

## schemaJson

```json
{
  "component": "Upload",
  "id": "attachment",
  "title": "附件",
  "componentProps": {
    "listType": "picture-card",
    "maxCount": 5
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| listType | string | 展示类型：`"text"` / `"picture"` / `"picture-card"` |
| maxCount | number | 最大文件数 |
| accept | string | 接受的文件类型 |

## HTML 识别

- CETA 原生：`.ant-upload` 或 `.upload-element`
- 通用 HTML：`<input type="file">`

## 同类组件

| component | 说明 | Field type |
|-----------|------|-----------|
| UserAvatar | 用户头像上传 | UPLOAD |
| IconUpload | 图标上传 | — |
