# CETA 数据源（DataSource）配置规范

> 数据源是 CETA 中多个组件共有的概念，定义了组件展示数据的来源。
> 适用组件：Table、Select、ACL、Data、EChart、List、ListSelect、Gantt。

---

## 核心概念

数据源回答的问题是：**这个组件要展示的数据从哪里来？**

CETA 提供两种主要的数据获取方式，配置在 `componentProps` 中：

| 方式 | 配置字段 | 适用场景 |
|------|---------|---------|
| FormEntity 数据源 | `dataSource` | 数据直接来自某个表单（数据库表） |
| URL 数据源 | `dataUrl` | 数据来自自定义 API（Flow REST 或其他接口） |

两者**二选一**，不同时使用。

---

## 方式一：FormEntity 数据源（dataSource）

通过指定表单 token，直接从该表单对应的数据库表拉取数据。

### 基本配置

```json
{
  "componentProps": {
    "dataSource": {
      "token": "user-info-form",
      "pbcToken": "user-management"
    }
  }
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| token | string | 是 | FormEntity 的 token，标识哪个表单（对应哪张数据库表） |
| pbcToken | string | 否 | 表单所属 PBC 的 token。同 PBC 内可省略，跨 PBC 引用时必填 |
| listUrl | string | 否 | 自定义列表数据获取地址（覆盖默认） |
| listIdsUrl | string | 否 | 自定义获取所有 ID 的地址（ACL 懒加载用） |
| listByIdsUrl | string | 否 | 自定义根据 ID 批量获取数据的地址（ACL 懒加载用） |

### 使用场景

- Table 展示某个表单的所有记录
- Select / ACL 从某个表单拉取选项列表
- Data 容器获取某个表单的数据供子组件使用
- EChart 基于某个表单数据绘图

### 示例

Table 使用 FormEntity 数据源：
```json
{
  "component": "Table",
  "componentProps": {
    "dataSource": {
      "token": "seal-application-form"
    },
    "columnDefs": [...]
  }
}
```

Select 跨 PBC 引用数据源：
```json
{
  "component": "Select",
  "componentProps": {
    "dataSource": {
      "token": "service-type-schema",
      "pbcToken": "setting"
    },
    "mapping": {
      "label": "productTypeName",
      "value": "id"
    }
  }
}
```

ACL 跨 PBC 引用数据源：
```json
{
  "component": "ACL",
  "componentProps": {
    "dataSource": {
      "token": "city-type-schema",
      "pbcToken": "setting"
    },
    "mapping": {
      "label": "companyName",
      "value": "id"
    },
    "columnDefs": [
      {
        "colId": "companyName",
        "field": "companyName",
        "headerName": "公司名称"
      }
    ]
  }
}
```

---

## 方式二：URL 数据源（dataUrl）

通过指定 URL 调用自定义 API 获取数据。常用于 Flow REST 接口。

### 基本配置

```json
{
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/get-staff-utilization"
  }
}
```

### URL 格式

| 类型 | URL 模式 | 说明 |
|------|---------|------|
| Flow REST | `/flow/api/flow-rest/{flow-name}` | CETA 平台的 Flow 暴露的 REST 端点，类似自定义后端 API |
| 平台内置 API | `/user-management/api/user/list` 等 | 平台自带的管理接口 |

### URL 中的变量引用

dataUrl 支持 Variable Pattern（`:变量名`）动态传参：

```json
{
  "dataUrl": "/flow/api/flow-rest/business-forecast?parentDataId=:params.formEntityDataId&createDate=:createDate"
}
```

常用变量：
- `:params.formEntityDataId` — URL 参数中的表单数据 ID
- `:id` — 当前记录 ID
- `:fieldToken` — 当前表单中某个字段的值

### 使用场景

- 数据需要跨表聚合、计算（单个表单无法直接提供）
- 需要自定义后端逻辑处理数据
- 调用平台内置管理接口

### 示例

Table 使用 Flow 数据源：
```json
{
  "component": "Table",
  "componentProps": {
    "rowModelType": "clientSide",
    "dataUrl": "/flow/api/flow-rest/get-staff-utilization",
    "suppressAddButton": true
  }
}
```

Data 容器使用 Flow 数据源：
```json
{
  "component": "Data",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/total-number-of-product-projects-in-execution"
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": ":count" }
    }
  ]
}
```

EChart 使用 Flow 数据源（直接配在 EChart 上）：
```json
{
  "component": "EChart",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/subject-scores-flow",
    "chartSettings": { ... }
  }
}
```

EChart 通过 Data 容器间接获取数据：
```json
{
  "component": "Data",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/subject-scores-flow",
    "setToFormData": true
  },
  "fields": [
    {
      "component": "EChart",
      "id": "results",
      "componentProps": { ... }
    }
  ]
}
```

ACL 使用 URL 数据源：
```json
{
  "component": "ACL",
  "componentProps": {
    "dataUrl": "/flow/api/flow-rest/wbslist",
    "mapping": {
      "value": "wbsId",
      "label": "description"
    },
    "columnDefs": [
      { "colId": "description", "field": "description", "headerName": "名称" }
    ]
  }
}
```

---

## 辅助配置（两种方式通用）

以下配置可与 `dataSource` 或 `dataUrl` 搭配使用：

### dataFilters — 数据过滤

对返回数据做固定过滤，类似 SQL WHERE 条件。用户无法在界面上修改这些过滤条件。

```json
{
  "dataFilters": [
    {
      "type": "notBlank",
      "colId": "productTypeId"
    },
    {
      "type": "blank",
      "colId": "token"
    },
    {
      "values": ["resource", "3rdParty"],
      "filterType": "set",
      "colId": "parameter_8"
    },
    {
      "colId": "companyCode",
      "filterType": "text",
      "type": "notBlank"
    }
  ]
}
```

| 字段 | 说明 |
|------|------|
| colId | 要过滤的字段 |
| type | 过滤类型：`blank`（为空）、`notBlank`（不为空）等 |
| filterType | 过滤器类型：`set`（集合匹配）、`text`（文本匹配） |
| values | set 类型的匹配值数组 |

### mapping — 字段映射

将数据源返回的字段名映射为组件需要的标准字段名。Select 和 ACL 必须配置。

```json
{
  "mapping": {
    "value": "id",
    "label": "companyName",
    "contactEmail": "contactEmail"
  }
}
```

- `value` — 选中后保存的值对应的源字段
- `label` — 显示文本对应的源字段
- 其他自定义映射 — 选中后额外携带的字段

### dataResponseKeyPath — 响应数据路径

从 API 返回的 JSON 中提取特定路径的数据：

```json
{
  "dataResponseKeyPath": "results[0]"
}
```

### transformDataResponse — 数据转换

用 JS 函数对返回数据做二次加工：

```json
{
  "transformDataResponse": "(function transform(data) {\n  return data.map(item => ({ ...item, total: Number(item.total) }));\n})(this);"
}
```

### sortModel — 排序

```json
{
  "sortModel": [
    { "colId": "createDate", "sort": "desc" }
  ]
}
```

---

## 选择指南

| 判断条件 | 使用方式 |
|---------|---------|
| 数据直接来自某个表单的全部/部分记录 | `dataSource` + token |
| 数据来自其他 PBC 的表单 | `dataSource` + token + pbcToken |
| 数据需要跨表聚合、计算、自定义逻辑 | `dataUrl` + Flow REST |
| 数据来自平台内置管理接口 | `dataUrl` + 平台 API 路径 |

### 注意事项

1. 使用 `dataUrl`（Flow）时，Table 通常需要配合 `rowModelType: "clientSide"` 和 `suppressAddButton: true`
2. Select / ACL 使用 `dataSource` 时必须配置 `mapping`（至少 `value` 和 `label`）
3. Data 容器获取的数据，子组件通过 `:fieldName` 变量模式访问
4. EChart 既可以自己配 `dataUrl`，也可以放在 Data 容器内通过父级获取数据
