# Steps 步骤条

展示流程进度的组件，支持水平/垂直方向，可由外部数据驱动。

## schemaJson — 基础用法

```json
{
  "component": "Steps",
  "componentProps": {
    "current": 1,
    "items": [
      { "title": "第一步", "description": "描述信息" },
      { "title": "第二步", "description": "描述信息" },
      { "title": "第三步" }
    ]
  },
  "style": { "margin": "12px 0" }
}
```

## schemaJson — 由数据驱动

通过 `valueField` 和 `valueMap` 将表单数据映射到步骤编号：

```json
{
  "component": "Steps",
  "valueField": "status",
  "componentProps": {
    "valueMap": {
      "pendingApproval": 1,
      "firstPass": 2,
      "firstReject": 1,
      "secondPass": 3
    },
    "stepItems": [
      { "title": "提交" },
      { "title": "一级审批" },
      { "title": "二级审批" },
      { "title": "发布" }
    ]
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| current | number | | 当前步骤编号（从 0 开始） |
| status | `'wait'` \| `'process'` \| `'finish'` \| `'error'` | | 当前步骤的状态 |
| direction | `'horizontal'` \| `'vertical'` | `'horizontal'` | 方向 |
| labelPlacement | `'horizontal'` \| `'vertical'` | `'vertical'` | 标题和描述的位置 |
| items | `Array<ItemProps>` | | 步骤列表（antd steps 属性） |
| stepItems | `Array<ItemProps>` | | 步骤列表（与 valueMap 配合使用） |
| valueMap | object | | 将 value 映射到 step 编号 |
| progressDot | boolean | | 是否使用点状步骤条 |
| size | `'default'` \| `'small'` | | 尺寸 |
| className | string | | 组件的 className |
| style | object | | 组件的 style |

## ItemProps

| 属性 | 类型 | 说明 |
|------|------|------|
| title | string | 步骤标题 |
| description | string | 步骤描述 |
| icon | string | 步骤图标 |
| hidden | boolean | 是否隐藏 |

注意：Steps 的 items 没有 `fields`（与 Collapse/Tabs 不同）。

## 示例：垂直方向

```json
{
  "component": "Steps",
  "componentProps": {
    "direction": "vertical",
    "labelPlacement": "vertical",
    "items": [
      { "title": "第一步" },
      { "title": "第二步" },
      { "title": "第三步" }
    ]
  }
}
```

## 示例：自定义图标

```json
{
  "component": "Steps",
  "componentProps": {
    "waitIcon": "oct:stopwatch",
    "current": 2,
    "items": [
      { "title": "第一步", "icon": "play" },
      { "title": "第二步" },
      { "title": "第三步" }
    ]
  }
}
```

## 示例：配合 Data 组件由外部数据驱动

```json
{
  "component": "Data",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/get-step-status"
  },
  "fields": [
    {
      "component": "Steps",
      "valueField": "result.status",
      "componentProps": {
        "valueMap": { "审批通过": 2, "approved": 2 },
        "items": [
          { "title": "待提交" },
          { "title": "待审批" },
          { "title": "审批通过" }
        ]
      }
    }
  ]
}
```

## HTML 识别

- CETA 原生：`.steps-element` 或 `.ant-steps`
- 通用 HTML：步骤条/进度指示器应映射为 Steps，不要用多个 Text 手动拼
