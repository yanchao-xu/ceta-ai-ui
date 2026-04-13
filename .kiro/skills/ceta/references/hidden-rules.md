# CETA 组件显示隐藏规则

> 本文件定义 CETA 平台 schemaJson 中组件 `hidden` 属性的配置规范。
> 适用于所有组件（输入组件、展示组件、布局组件）。

---

## 概述

`hidden` 属性的类型是 `ConditionalPropertyPropType`，支持三种配置形式：
1. **布尔值** — 静态隐藏/显示
2. **条件对象** — 基于数据或权限动态隐藏/显示
3. **条件列表** — 多组条件的数组形式

运行时由 `useConditionalProperty` hook 解析 `hidden` 值，结果为 `true` 时组件不渲染（返回 `null`）。

---

## 方式一：静态隐藏（布尔值）

```json
{ "hidden": true }
```

- `true`：组件永远隐藏
- `false` 或不设置：组件永远显示

适用场景：字段需要存在于表单数据中但不需要在 UI 上展示。

```json
{
  "component": "Input",
  "id": "onedriveLink",
  "title": "OneDrive链接",
  "validation": { "maxLength": 200, "required": true },
  "readonly": true,
  "hidden": true
}
```

---

## 方式二：条件对象（动态隐藏）

### 基本结构

```json
{
  "hidden": {
    "dataPredicate": { ... },
    "permissionPredicate": { ... },
    "valueIfPositive": true | false,
    "valueIfNegative": true | false
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| dataPredicate | object | 基于数据的条件判断 |
| permissionPredicate | object | 基于权限的条件判断 |
| valueIfPositive | boolean | 条件满足时 hidden 的值 |
| valueIfNegative | boolean | 条件不满足时 hidden 的值 |

`dataPredicate` 和 `permissionPredicate` 可以同时使用，也可以只用其中一个。

### valueIfPositive / valueIfNegative 常见模式

| 意图 | valueIfPositive | valueIfNegative |
|------|----------------|-----------------|
| 条件满足时**显示**，不满足时**隐藏** | `false` | `true` |
| 条件满足时**隐藏**，不满足时**显示** | `true` | `false` |

> 最常见的模式是 `"valueIfPositive": false, "valueIfNegative": true`，即"满足条件才显示"。

---

## dataPredicate 配置

### 结构

```json
{
  "dataPredicate": {
    "operator": "AND" | "OR",
    "conditions": [
      {
        "dataField": "fieldId",
        "condition": "equals",
        "value": "someValue"
      }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| operator | `"AND"` \| `"OR"` | `AND` 全部满足，`OR` 满足其一。不写时默认 `AND` |
| conditions | array | 条件列表 |

### condition 条件项

每个条件项的字段：

| 字段 | 类型 | 支持 Variable Pattern | 说明 |
|------|------|:---:|------|
| dataField | string | ✗ | 数据字段 id，直接从 formData 取值 |
| field | string | ✔ | 数据字段 id，支持 Variable Pattern 语法 |
| condition | string | ✗ | 比较操作符 |
| value | string \| number \| boolean | ✔ | 比较值 |

`dataField` 和 `field` 二选一：
- `dataField`：简单场景，直接引用表单字段（如 `"dataField": "status"`）
- `field`：复杂场景，支持 Variable Pattern（如 `"field": ":context.userProfile.email"`）

### 可用的 condition 操作符

| 操作符 | 说明 | 是否需要 value |
|--------|------|:-----------:|
| equals | 相等（宽松比较） | ✔ |
| stringEquals | 字符串严格相等 | ✔ |
| notEqual | 不等（宽松比较） | ✔ |
| stringNotEqual | 字符串严格不等 | ✔ |
| contains | 包含 | ✔ |
| notContains | 不包含 | ✔ |
| startsWith | 以...开头 | ✔ |
| endsWith | 以...结尾 | ✔ |
| lessThan | 小于 | ✔ |
| lessThanOrEqual | 小于等于 | ✔ |
| greaterThan | 大于 | ✔ |
| greaterThanOrEqual | 大于等于 | ✔ |
| blank | 为空 | ✗ |
| notBlank | 不为空 | ✗ |

---

## permissionPredicate 配置

基于用户权限判断：

```json
{
  "permissionPredicate": {
    "hasAnyOf": ["PERMISSION_CODE_1", "PERMISSION_CODE_2"]
  }
}
```

用户拥有列表中任一权限时，predicate 为 positive。

---

## Variable Pattern 语法

在 `field` 和 `value` 中可以使用 Variable Pattern，以冒号 `:` 开头：

| 变量 | 说明 |
|------|------|
| `:currentData` 或 `:fieldId` | 当前数据字段（`:currentData` 可省略） |
| `:formData` | 表单数据字段 |
| `:context.userProfile` | 用户信息（如 `.id`, `.email`, `.roles`） |
| `:context.pageType` | 页面类型（`form-create`, `form-edit`, `form-view` 等） |
| `:context.pbcToken` | 当前 PBC token |
| `:context.formEntityToken` | 当前表单 token |
| `:context.formEntityDataId` | 当前表单数据 id |
| `:params` | URL 路径参数 |
| `:searchParams` | URL 搜索参数 |

支持深层路径访问，如 `:relatedContractNumber[0].contractCategory[0].value`。

---

## 配置示例

### 示例 1：单条件 — 字段值包含某值时显示

当 `officialSealType` 包含 `"mainBusinessRelated"` 时显示组件：

```json
{
  "hidden": {
    "dataPredicate": {
      "operator": "AND",
      "conditions": [
        {
          "dataField": "officialSealType",
          "condition": "contains",
          "value": "mainBusinessRelated"
        }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 2：多条件 AND — 多个条件同时满足时显示

当 `status` 等于 `"waitingApproval"` 且 `sealApplicationType` 等于 `"个人用章申请"` 时显示：

```json
{
  "hidden": {
    "dataPredicate": {
      "operator": "AND",
      "conditions": [
        {
          "dataField": "status",
          "condition": "equals",
          "value": "waitingApproval"
        },
        {
          "dataField": "sealApplicationType",
          "condition": "equals",
          "value": "个人用章申请"
        }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 3：多条件 OR — 任一条件满足时显示

当 `sealType` 包含三种印章类型中的任意一种时显示：

```json
{
  "hidden": {
    "dataPredicate": {
      "operator": "OR",
      "conditions": [
        { "dataField": "sealType", "condition": "contains", "value": "bidGuangzhouSeal" },
        { "dataField": "sealType", "condition": "contains", "value": "bidBeijingSeal" },
        { "dataField": "sealType", "condition": "contains", "value": "bidShanghaiSeal" }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 4：字段为空时隐藏（notBlank）

当 `relatedApprovalLink` 不为空时显示链接：

```json
{
  "hidden": {
    "dataPredicate": {
      "operator": "AND",
      "conditions": [
        {
          "dataField": "relatedApprovalLink",
          "condition": "notBlank"
        }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 5：使用 Variable Pattern 的 field — 引用深层数据

使用 `field` 引用关联数据的嵌套字段：

```json
{
  "hidden": {
    "dataPredicate": {
      "operator": "OR",
      "conditions": [
        {
          "condition": "equals",
          "value": "businessSOCO",
          "field": ":relatedContractNumber[0].contractCategory[0].value"
        },
        {
          "condition": "equals",
          "value": "salesContractFW",
          "field": ":relatedContractNumber[0].contractCategory[0].value"
        }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 6：使用 Variable Pattern 的 value — 与上下文数据比较

当前行的 email 等于当前登录用户的 email，且 isAdmin 不等于 "1" 时显示：

```json
{
  "hidden": {
    "dataPredicate": {
      "conditions": [
        {
          "field": ":email",
          "condition": "stringEquals",
          "value": ":context.userProfile.email"
        },
        {
          "field": ":isAdmin",
          "condition": "stringNotEqual",
          "value": "1"
        }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

### 示例 7：权限 + 数据复合条件

用户拥有 `CYBER_SECURITY_APPLY` 权限，且角色不是"开发"，且状态为"已完成"时隐藏：

```json
{
  "hidden": {
    "permissionPredicate": {
      "hasAnyOf": ["CYBER_SECURITY_APPLY"]
    },
    "dataPredicate": {
      "operator": "AND",
      "conditions": [
        {
          "field": ":context.userProfile.roles",
          "condition": "notEqual",
          "value": "开发"
        },
        {
          "dataField": "status",
          "condition": "equals",
          "value": "已完成"
        }
      ]
    },
    "valueIfPositive": true,
    "valueIfNegative": false
  }
}
```

---

## 注意事项

1. `dataField` 和 `field` 二选一，不要同时使用。需要 Variable Pattern 时用 `field`，简单字段引用用 `dataField`
2. `blank` 和 `notBlank` 操作符不需要 `value` 字段
3. `operator` 不写时默认为 `AND`
4. `valueIfPositive` 和 `valueIfNegative` 必须成对出现
5. `hidden` 属性同样适用于 `readonly`、`disabled` 等支持 `ConditionalPropertyPropType` 的属性，配置方式完全相同
6. 表单级的 `disabled`/`readonly` 与组件级做 OR 合并，任一为 true 即生效
