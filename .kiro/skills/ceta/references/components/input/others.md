# 其他输入组件

> 以下组件已有独立文档：Input、Select、DatePicker、Switch、Upload、Radio、Checkbox、Cascader、NumberPicker、RichText、Textarea、ACL、Rate、EditableTable。

---

## Password 密码

Field type: `PASSWORD`

```json
{
  "component": "Password",
  "id": "password",
  "title": "密码",
  "componentProps": {}
}
```

透传给 Ant Design [Input.Password](https://ant.design/components/input#components-input-demo-password-input)。

HTML 识别：`<input type="password">`

---

## DateRangePicker 日期范围

Field type: `DATE`

```json
{
  "component": "DateRangePicker",
  "id": "dateRange",
  "title": "日期范围",
  "componentProps": { "showTime": true }
}
```

| 属性     | 类型              | 说明         |
| -------- | ----------------- | ------------ |
| showTime | boolean \| object | 是否显示时间 |

透传给 Ant Design [DatePicker.RangePicker](https://ant.design/components/date-picker#components-date-picker-demo-range-picker)。

HTML 识别：`.ant-picker-range`

---

## TimePicker 时间选择

Field type: `TIME`

```json
{
  "component": "TimePicker",
  "id": "startTime",
  "title": "开始时间",
  "componentProps": {}
}
```

HTML 识别：`<input type="time">`

---

## TimeRangePicker 时间范围

```json
{
  "component": "TimeRangePicker",
  "id": "timeRange",
  "title": "时间范围",
  "componentProps": {}
}
```

透传给 Ant Design [TimePicker.RangePicker](https://ant.design/components/time-picker)。

---

## TreeSelect 树选择

Field type: `TREE_SELECT`

```json
{
  "component": "TreeSelect",
  "id": "department",
  "title": "部门",
  "componentProps": {
    "dataSource": { "token": "dept-form", "pbcToken": "org-management" },
    "mapping": { "value": "id", "label": "name", "children": "children" }
  }
}
```

---

## Slider 滑块

```json
{
  "component": "Slider",
  "id": "progress",
  "title": "进度",
  "componentProps": {}
}
```

透传给 Ant Design [Slider](https://ant.design/components/slider)。

---

## NumericPicker 数字选择

Field type: `NUMERIC`

与 NumberPicker 配置完全相同，参见 NumberPicker.md。

---

## Formula 公式

Field type: `FORMULA`

```json
{
  "component": "Formula",
  "id": "total",
  "title": "合计",
  "componentProps": {}
}
```

---

## CodeEditor 代码编辑器

```json
{
  "component": "CodeEditor",
  "id": "code",
  "title": "代码",
  "componentProps": {}
}
```

---

## OrgTree 组织树选择

```json
{
  "component": "OrgTree",
  "id": "org",
  "title": "组织",
  "componentProps": {}
}
```

## 完整输入组件清单

| component       | Field type    | 独立文档            |
| --------------- | ------------- | ------------------- |
| Input           | TEXT_BOX      | ✔️ Input.md         |
| Textarea        | TEXTAREA      | ✔️ Textarea.md      |
| NumberPicker    | NUMBER        | ✔️ NumberPicker.md  |
| Select          | SELECT        | ✔️ Select.md        |
| DatePicker      | DATE          | ✔️ DatePicker.md    |
| Switch          | SWITCH        | ✔️ Switch.md        |
| Upload          | UPLOAD        | ✔️ Upload.md        |
| Radio           | RADIO         | ✔️ Radio.md         |
| Checkbox        | CHECKBOX      | ✔️ Checkbox.md      |
| Cascader        | CASCADER      | ✔️ Cascader.md      |
| RichText        | RICH_TEXT     | ✔️ RichText.md      |
| ACL             | ACL           | ✔️ ACL.md           |
| Rate            | RATE          | ✔️ Rate.md          |
| EditableTable   | EDITABLE_GRID | ✔️ EditableTable.md |
| Password        | PASSWORD      | 本文件              |
| DateRangePicker | DATE          | 本文件              |
| TimePicker      | TIME          | 本文件              |
| TimeRangePicker | —             | 本文件              |
| TreeSelect      | TREE_SELECT   | 本文件              |
| Slider          | —             | 本文件              |
| NumericPicker   | NUMERIC       | 本文件              |
| Formula         | FORMULA       | 本文件              |
| CodeEditor      | —             | 本文件              |
| OrgTree         | —             | 本文件              |
