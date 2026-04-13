# 字段类型映射（component ↔ Field API type）

## 组件 → 字段类型

| component | Field type | 说明 |
|-----------|-----------|------|
| Input | TEXT_BOX | 单行文本 |
| Textarea | TEXTAREA | 多行文本 |
| NumberPicker | NUMBER | 数字 |
| NumericPicker | NUMERIC | 高精度数字 |
| Password | PASSWORD | 密码 |
| Select | SELECT | 下拉选择 |
| Radio | RADIO | 单选 |
| Checkbox | CHECKBOX | 复选框 |
| Switch | SWITCH | 开关 |
| DatePicker | DATE | 日期 |
| TimePicker | TIME | 时间 |
| Upload | UPLOAD | 文件上传 |
| ACL | ACL | 关联引用 |
| Cascader | CASCADER | 级联选择 |
| TreeSelect | TREE_SELECT | 树形选择 |
| RichText | RICH_TEXT | 富文本 |
| Rate | RATE | 评分 |
| EditableTable | EDITABLE_GRID | 可编辑表格 |
| Formula | FORMULA | 公式 |
| Join | JOIN | 关联 |
| Workflow | WORKFLOW | 工作流 |

## 校验属性（validation）

不同组件支持的 validation 属性：

| component | required | maxLength | regex | minValue | maxValue |
|-----------|----------|-----------|-------|----------|----------|
| Input | ✅ | ✅ | ✅ | — | — |
| Textarea | ✅ | ✅ | — | — | — |
| NumberPicker | ✅ | — | — | ✅ | ✅ |
| DatePicker | ✅ | — | — | — | — |
| Select | ✅ | — | — | — | — |
| Radio | ✅ | — | — | — | — |
| Checkbox | ✅ | — | — | — | — |
| Upload | ✅ | — | — | — | — |
| ACL | ✅ | — | — | — | — |
| Switch | — | — | — | — | — |

## 组件名新旧对照

统一使用新版组件名：

| 旧名（仍兼容） | 新名（必须使用） |
|---------------|---------------|
| CardLayout | Card |
| FlowLayout | Stack |
| GridLayout | Grid |
| NormalLayout | Box |
| MobileLayout | Mobile |
| Tab | Tabs |
| NavTab | NavTabs |
