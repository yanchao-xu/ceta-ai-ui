# RichText 富文本编辑器

Field type: `RICH_TEXT`

## schemaJson

```json
{
  "component": "RichText",
  "id": "content",
  "title": "内容",
  "componentProps": {
    "toolBarProps": {},
    "editorProps": {},
    "acceptProps": {
      "img": ".jpg,.jpeg,.png,.gif",
      "video": ".mp4,.webm",
      "file": ".pdf,.doc,.docx"
    },
    "sizeLimitProps": {
      "img": 5,
      "video": 100,
      "file": 20
    }
  }
}
```

## componentProps

| 属性 | 类型 | 说明 |
|------|------|------|
| toolBarProps | object | wangeditor toolbar 的属性 |
| editorProps | object | wangeditor editor 的属性 |
| acceptProps | `{ img: string, video: string, file: string }` | 上传文件接受的文件类型 |
| sizeLimitProps | `{ img: number, video: number, file: number }` | 上传文件的大小限制（MB） |
| className | string | 组件的 className，支持 Variable Pattern |
| style | object | 组件的 style |

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |
| editor | object | 编辑器实例 |

## HTML 识别

- CETA 原生：`.richtext-element`、`.ql-editor`、`contenteditable`
- 通用 HTML：`contenteditable="true"` 的 div、富文本编辑器容器

## 示例：基础用法

```json
{
  "component": "RichText",
  "id": "description",
  "title": "详细描述",
  "componentProps": {}
}
```
