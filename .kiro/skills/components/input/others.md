# 其他输入组件

低频输入组件汇总。

## NumberPicker 数字输入

Field type: `NUMBER`

```json
{
  "component": "NumberPicker",
  "id": "amount",
  "title": "金额",
  "validation": { "minValue": 0, "required": true },
  "componentProps": { "precision": 2 }
}
```

HTML 识别：`<input type="number">` 或 `.number-picker-element`

---

## ACL 关联引用

Field type: `ACL`

```json
{
  "component": "ACL",
  "id": "relatedUser",
  "title": "关联用户",
  "componentProps": {
    "dataSource": { "token": "system-user-form" },
    "mapping": { "value": "id", "label": "username" }
  }
}
```

---

## Rate 评分

Field type: `RATE`

```json
{ "component": "Rate", "id": "score", "title": "评分" }
```

---

## Slider 滑块

```json
{ "component": "Slider", "id": "progress", "title": "进度" }
```

---

## EditableTable 可编辑表格

Field type: `EDITABLE_GRID`

```json
{ "component": "EditableTable", "id": "lineItems", "title": "明细行" }
```

---

## 完整输入组件清单

| component | Field type | 常用程度 |
|-----------|-----------|---------|
| Input | TEXT_BOX | ★★★ |
| Textarea | TEXTAREA | ★★★ |
| NumberPicker | NUMBER | ★★★ |
| Select | SELECT | ★★★ |
| DatePicker | DATE | ★★★ |
| Switch | SWITCH | ★★★ |
| Radio | RADIO | ★★ |
| Upload | UPLOAD | ★★ |
| Password | PASSWORD | ★★ |
| ACL | ACL | ★★ |
| Checkbox | CHECKBOX | ★★ |
| RichText | RICH_TEXT | ★ |
| DateRangePicker | DATE | ★ |
| TimePicker | TIME | ★ |
| Cascader | CASCADER | ★ |
| TreeSelect | TREE_SELECT | ★ |
| Rate | RATE | ★ |
| Slider | — | ★ |
| EditableTable | EDITABLE_GRID | ★ |
| NumericPicker | NUMERIC | ☆ |
| Formula | FORMULA | ☆ |
| CodeEditor | — | ☆ |
