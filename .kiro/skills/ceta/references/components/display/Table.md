# Table 数据表格

列表页的核心组件，基于 ag-grid 封装。

> 基于 `icp-doc-website/docs/components/table.md` 和 `TableElement.js` 源码整理。

## schemaJson

```json
{
  "component": "Table",
  "id": "{entityToken}-table",
  "componentProps": {
    "dataSource": { "token": "{entityToken}" },
    "columnDefs": [ ... ],
    "addButtonHref": "/form/{entityToken}/layout/new",
    "pinnedFilter": [],
    "pagination": true
  },
  "style": { "height": "450px" }
}
```

> Table / EditableTable 的高度应从 HTML 中读取实际值，如果读不出来则默认设为 `450px`，不要使用 `"height": "100%"`。

## componentProps

> 数据源配置（`dataSource` / `dataUrl`）的完整规范见 `skills/ceta/references/data-source-rules.md`。

| 属性 | 类型 | 默认值 | 必填 | Variable Pattern | 说明 |
|------|------|--------|------|:---:|------|
| dataSource | `{ token, pbcToken?, listUrl? }` | | 是 | | 数据源，`token` 指向 FormEntity。与 `dataUrl` 二选一 |
| columnDefs | array | | 是 | | 列定义数组（见下方详解） |
| defaultColDef | object | | 否 | | 默认列属性，会与内置默认值合并 |
| dataUrl | string | | 否 | ✔️ | 通过自定义 URL 一次性获取所有表格数据 |
| pinnedTopDataUrl | string | | 否 | ✔️ | 固定到表格顶部的行数据 URL |
| pinnedBottomDataUrl | string | | 否 | ✔️ | 固定到表格底部的行数据 URL |
| dataFilters | array | | 否 | ✔️ | 请求数据源隐藏的固定 filter 条件，无法通过界面改变 |
| defaultFilterModel | array | | 否 | ✔️ | 表格默认的 filter 条件，可以通过界面改变 |
| addButtonHref | string | | 否 | ✔️ | "新增"按钮跳转路径 |
| addButtonContent | string | | 否 | | 新增按钮的文字 |
| pinnedFilter | string[] | | 否 | | 固定显示在 Filter Panel 的筛选字段 token，无法被删除 |
| pagination | boolean | `false` | 否 | | 是否分页 |
| paginationPageSize | number | `30` | 否 | | 分页每页行数 |
| pageSizeOptions | number[] | | 否 | | 自定义每页行数的选项 |
| rowModelType | `'serverSide'` \| `'clientSide'` | `'serverSide'` | 否 | | 数据加载模式 |
| httpMethod | `'get'` \| `'post'` | `'get'` | 否 | | 获取数据的 HTTP 方法 |
| httpBody | object \| string | | 否 | ✔️ | POST 请求时的请求体，覆盖自动生成的 body |
| selectColId | string[] | | 否 | | 只加载指定字段，不配置则加载所有 |
| needCount | boolean | `true` | 否 | | 请求是否需要查询数量 |
| searchTextColIds | string[] | | 否 | | 指定模糊搜索的字段，不指定则全局搜索 |
| translateDataResponse | boolean | `false` | 否 | | 是否翻译请求结果 |
| transformDataResponse | string | | 否 | | 转换数据响应的 eval 表达式 |
| tableSize | `'default'` \| `'small'` | `'default'` | 否 | | 表格尺寸 |
| interval | boolean | `false` | 否 | | 是否定期自动刷新数据 |
| intervalTime | number | `10` | 否 | | 自动刷新间隔（秒） |
| fuzzySearchPlaceholder | string | | 否 | | 模糊搜索输入框 placeholder |
| fuzzySearchOpen | boolean | `false` | 否 | | 模糊搜索是否永远展开 |
| excelExportFilename | string | `'export'` | 否 | | 导出 Excel 文件名 |
| excelExportMaxSizePerReq | number | `1000` | 否 | | 导出 Excel 分批请求行数 |
| settingKey | string | | 否 | | 表格设置保存的 key，默认使用 id |
| alwaysUseApiFilter | boolean | `false` | 否 | | clientSide 模式下也使用 API 过滤 |
| multiValueKey | string | `'label'` | 否 | | ACL/Select 列过滤和分组时使用的数据属性 |
| toolbarFields | array | | 否 | | 工具栏自定义元素（JSON 配置） |
| toolbarChildren | node | | 否 | | 工具栏自定义元素（JSX） |
| deleteButtonUrl | string | | 否 | | 工具栏删除按钮的 API 地址 |

### suppress / support 开关属性

以下属性均支持 ConditionalProperty（可根据条件动态控制），默认值均为 `false`：

| 属性 | 说明 |
|------|------|
| suppressToolbar | 禁止整个 Toolbar |
| suppressToolbarActions | 禁止 Toolbar 所有操作按钮 |
| suppressAddButton | 禁止"新增"按钮 |
| suppressDeleteButton | 禁止"删除"按钮（默认 `true`） |
| suppressFuzzySearch | 禁止模糊搜索 |
| suppressFuzzySearchSpeech | 禁止语音搜索 |
| suppressColumnSelect | 禁止"显示/隐藏"列 |
| suppressExcelExport | 禁止导出 Excel |
| suppressToolbarRefresh | 禁止"刷新"按钮 |
| suppressTableSetting | 禁止设置功能 |
| suppressFullscreen | 禁止全屏功能 |
| suppressSaveSetting | 禁止保存表格设置 |
| suppressFilterPanel | 禁止 Filter 面板 |
| suppressFavoriteView | 禁止保存多种 View |
| suppressRefreshDataWhenFilterChange | 禁止 Filter 变更后自动刷新 |
| suppressStatusbar | 禁止状态栏 |
| supportShowDeleted | 支持"显示已删除"功能 |

#### ⚠️ suppressAddButton 生成规则

`suppressAddButton` 默认值为 `false`，这意味着**不设置此属性时，平台会自动渲染一个"新增"按钮**。
生成 Table JSON 时，必须根据业务场景判断是否需要新增按钮，并显式设置：

| 场景 | 设置 | 原因 |
|------|------|------|
| 标准 CRUD 列表页（有关联 FormEntity，用户可新建数据） | 不设置（或 `false`），同时配置 `addButtonHref` | 需要新增按钮 |
| 只读展示表格（搜索结果、确认页航段表、SSR 表、历史记录等） | `"suppressAddButton": true` | 不需要新增按钮 |
| 仪表盘/统计表格 | `"suppressAddButton": true` | 不需要新增按钮 |
| 嵌入在其他页面中的关联数据表格（非独立列表页） | `"suppressAddButton": true` | 不需要新增按钮 |

**判断方法：看 HTML 原文中该 `<table>` 区域是否有"新建"/"新增"/"创建"/"添加"按钮。如果没有，就设置 `"suppressAddButton": true`。如果有，就配置 `addButtonHref` 指向新建表单。**


## columnDefs 列定义

每个列定义对象支持 ag-grid 的所有 [Column Properties](https://www.ag-grid.com/react-data-grid/column-properties/)，以下为常用属性：

```json
{
  "type": "TEXT_COLUMN",
  "colId": "fieldToken",
  "field": "fieldToken",
  "headerName": "列标题",
  "width": 120,
  "minWidth": 60,
  "pinned": "left",
  "hide": false,
  "sortable": true,
  "filter": true,
  "cellRendererParams": { ... }
}
```

| 属性 | 类型 | 说明 |
|------|------|------|
| type | string | 列类型（见下方列类型表） |
| colId | string | 列唯一标识，通常与 field 相同 |
| field | string | 数据字段名 |
| headerName | string | 列标题 |
| width | number | 列宽度（px） |
| minWidth | number | 最小宽度，默认 100 |
| flex | number | 弹性宽度，默认 1 |
| pinned | `'left'` \| `'right'` | 固定列位置 |
| hide | boolean | 是否隐藏 |
| hidden | ConditionalProperty | 条件隐藏（在 formatColumnDefs 中过滤） |
| sortable | boolean | 是否可排序，默认 true |
| filter | boolean \| string | 过滤器类型，默认 `'agTextColumnFilter'` |
| cellRendererParams | object | 传递给 cellRenderer 的参数（各列类型不同） |
| children | array | 列分组的子列定义 |

### 列类型完整对照表

| type | 说明 | 对应 Field type | cellRenderer | cellEditor | filter |
|------|------|----------------|-------------|------------|--------|
| TEXT_COLUMN | 文本列 | TEXT_BOX, PASSWORD | TextCellRenderer | TextCellEditor | agTextColumnFilter |
| TEXTAREA_COLUMN | 多行文本列 | TEXTAREA | AutoCellRenderer | TextareaCellEditor | agTextColumnFilter |
| DATE_COLUMN | 日期列 | DATE | DateCellRenderer | DateCellEditor | agDateColumnFilter |
| TIME_COLUMN | 时间列 | TIME | AutoCellRenderer | TimeCellEditor | agTextColumnFilter |
| NUMBER_COLUMN | 数字列 | NUMBER | NumberCellRenderer | NumberCellEditor | agNumberColumnFilter |
| NUMERIC_COLUMN | 高精度数字列 | NUMERIC | NumericCellRenderer | NumericCellEditor | agNumberColumnFilter |
| ENUM_COLUMN | 枚举列 | SELECT（带颜色/图标） | EnumCellRenderer | — | agSetColumnFilter |
| SELECT_COLUMN | 选择列 | SELECT | MultiValueCellRenderer | SelectCellEditor | 自动 |
| ACL_COLUMN | 关联引用列 | ACL | MultiValueCellRenderer | ACLCellEditor | 自动 |
| AVATAR_COLUMN | 头像列 | UPLOAD（头像） | AvatarCellRenderer | — | — |
| UPLOAD_COLUMN | 上传列 | UPLOAD | UploadCellRenderer | UploadCellEditor | 无 |
| PROGRESS_COLUMN | 进度条列 | NUMBER（百分比） | ProgressCellRenderer | — | agNumberColumnFilter |
| CHECKBOX_COLUMN | 复选框列 | CHECKBOX | AutoCellRenderer | SelectCellEditor | agTextColumnFilter |
| RADIO_COLUMN | 单选列 | RADIO | AutoCellRenderer | SelectCellEditor | agTextColumnFilter |
| AUTO_INDEX_COLUMN | 自动序号列 | 无（自动生成） | AutoIndexCellRenderer | — | 无 |
| AUTO_COLUMN | 自动列 | 通用 | AutoCellRenderer | TextCellEditor | agTextColumnFilter |
| ACTION_COLUMN | 操作列 | 无（固定最后） | ActionCellRenderer | — | 无 |
| STATUS_COLUMN | 状态列（⚠️ 已废弃） | SELECT | StatusCellRenderer | — | agSetColumnFilter |
| MULTI_VALUE_COLUMN | 多值列（⚠️ 已废弃） | — | MultiValueCellRenderer | SelectCellEditor | — |

> `STATUS_COLUMN` 已废弃，请使用 `ENUM_COLUMN` 替代。`MULTI_VALUE_COLUMN` 已废弃。

### 内置默认列属性 (DEFAULT_COL_DEF)

```json
{
  "flex": 1,
  "minWidth": 100,
  "resizable": true,
  "sortable": true,
  "filter": "agTextColumnFilter",
  "enableRowGroup": false,
  "cellDataType": false
}
```

## 各列类型 cellRendererParams 详解

### TEXT_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| href | string | 点击跳转链接，支持 ConditionalProperty |
| hrefSelector | array | 条件链接选择器 |
| hrefIsSiteBased | boolean | 是否为站点级跳转 |
| target | string | 链接 target（如 `_blank`） |
| replace | boolean | 是否替换路由历史 |
| suppressBasePath | boolean | 跳过 base path 拼接 |
| formatter | object \| string | 文本格式化器，支持 Text 组件的 formatterTypes |

```json
{
  "type": "TEXT_COLUMN",
  "colId": "name",
  "field": "name",
  "headerName": "名称",
  "cellRendererParams": {
    "href": "/form/:id/view",
    "target": "_blank"
  }
}
```

### NUMBER_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| formatOptions | object | Intl.NumberFormat 的所有属性 |
| formatOptions.currency | string | 货币代码，支持冒号语法引用表单数据 |
| href | string | 点击跳转链接 |
| target | string | 链接 target |

```json
{
  "type": "NUMBER_COLUMN",
  "colId": "amount",
  "field": "amount",
  "headerName": "金额",
  "cellRendererParams": {
    "formatOptions": {
      "style": "currency",
      "currency": "CNY",
      "minimumFractionDigits": 2,
      "maximumFractionDigits": 2
    }
  }
}
```

### DATE_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| dateValueType | `'ISO'` \| `'timestamp'` \| `'unix_timestamp'` \| `'time'` | 数据值类型 |
| formatterType | `'date'` \| `'time'` \| `'datetime'` | 显示格式类型 |
| dateFormat | string | 自定义 dayjs 格式字符串（如 `'YYYY-MM-DD'`） |
| formatOptions | object | Intl.DateTimeFormat 的所有属性 |

```json
{
  "type": "DATE_COLUMN",
  "colId": "createdAt",
  "field": "createdAt",
  "headerName": "创建时间",
  "cellRendererParams": {
    "formatterType": "datetime"
  }
}
```

### ENUM_COLUMN

用于带颜色标签和图标的枚举显示，比 STATUS_COLUMN 功能更丰富。

| 属性 | 类型 | 说明 |
|------|------|------|
| enums | array | 枚举选项数组 `[{ value, label, color, icon }]` |
| remoteEnum | object | 远程枚举配置 `{ dataUrl, dataResponseKeyPath?, transformDataResponse?, translateDataResponse? }` |
| defaultValue | string \| number | 默认值 |
| enumStyle | `'style1'` \| `'style2'` | 显示样式 |
| rounded | boolean | 是否圆角，默认 `true` |

```json
{
  "type": "ENUM_COLUMN",
  "colId": "status",
  "field": "status",
  "headerName": "状态",
  "cellRendererParams": {
    "enums": [
      { "value": "active", "label": "启用", "color": "green", "icon": "ant:check-circle-outlined" },
      { "value": "inactive", "label": "停用", "color": "red", "icon": "ant:close-circle-outlined" }
    ]
  }
}
```

远程枚举：

```json
{
  "type": "ENUM_COLUMN",
  "colId": "category",
  "field": "category",
  "headerName": "分类",
  "cellRendererParams": {
    "remoteEnum": {
      "dataUrl": "/api/categories",
      "dataResponseKeyPath": "results"
    }
  }
}
```

### SELECT_COLUMN / ACL_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| separator | string | 多值分隔符，默认 `', '` |
| displayTextKeyPath | string | 显示文本的取值路径，默认 `'label'` |
| options | array | 选项列表（用于显示文本匹配） |
| href | string | 点击跳转链接 |
| target | string | 链接 target |

```json
{
  "type": "ACL_COLUMN",
  "colId": "assignee",
  "field": "assignee",
  "headerName": "负责人",
  "cellRendererParams": {
    "displayTextKeyPath": "label"
  }
}
```

### AVATAR_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| avatarStyle | `'auto'` \| `'avatar'` \| `'avatar-label'` \| `'group'` | 头像显示样式，默认 `'auto'` |
| componentProps | object | Upload 组件属性（如 `{ listType, maxCount }`） |

```json
{
  "type": "AVATAR_COLUMN",
  "colId": "avatar",
  "field": "avatar",
  "headerName": "头像",
  "cellRendererParams": {
    "avatarStyle": "avatar-label"
  },
  "width": 80,
  "minWidth": 60
}
```

### PROGRESS_COLUMN

| 属性 | 类型 | 说明 |
|------|------|------|
| progressStyle | `'style1'` \| `'style2'` \| `'style3'` \| `'style4'` | 进度条样式，style4 为圆环 |
| inHundred | boolean | 值是否已经是百分比（0-100），默认 `false`（0-1） |
| defaultZero | boolean | 无值时是否显示 0%，默认 `false` |
| colorConfig | object | 状态颜色映射 `{ default, warning, success, error }` |

数据格式支持 `number`（纯数值）或 `{ percent: number, status: string }`。

```json
{
  "type": "PROGRESS_COLUMN",
  "colId": "progress",
  "field": "progress",
  "headerName": "进度",
  "cellRendererParams": {
    "progressStyle": "style1",
    "inHundred": false
  }
}
```

### AUTO_INDEX_COLUMN

自动序号列，无需配置 field，自动显示行号。固定宽度 70px，不可排序、不可过滤、不可隐藏。

```json
{
  "type": "AUTO_INDEX_COLUMN",
  "headerName": "#",
  "width": 70
}
```

### ACTION_COLUMN

操作列，固定不可排序、不可过滤。支持内置操作类型和自定义 Button/Switch 组件。

```json
{
  "type": "ACTION_COLUMN",
  "width": 240,
  "headerName": "操作",
  "cellRendererParams": {
    "displayLikeButton": true,
    "showIcon": true,
    "actions": [
      { "type": "EDIT", "href": "/form/{entityToken}/layout/edit/:id" },
      { "type": "VIEW", "href": "/form/:id/view" },
      { "type": "DELETE" }
    ]
  },
  "pinned": "right"
}
```

#### cellRendererParams

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| displayLikeButton | boolean | `false` | 操作项显示为按钮样式（而非链接） |
| showIcon | boolean | `false` | 是否显示操作图标 |
| iconOnly | boolean | `false` | 只显示图标不显示文字（自动启用 showIcon） |
| actions | array | | 操作项数组 |

#### actions 操作项

| 属性 | 类型 | 说明 |
|------|------|------|
| type | string | 操作类型，支持 ConditionalProperty |
| href | string | 跳转链接（VIEW/EDIT/APPROVAL 使用） |
| hrefSelector | array | 条件链接选择器 |
| hrefIsSiteBased | boolean | 是否站点级跳转 |
| suppressBasePath | boolean | 跳过 base path |
| target | string | 链接 target（如 `_blank`） |
| hidden | ConditionalProperty | 条件隐藏 |
| value | string | 自定义按钮文字 |
| url | string | DELETE/TREE_DELETE 的自定义删除 API 地址 |
| title | string | 删除确认弹窗标题 |
| content | string | 删除确认弹窗内容，支持 Variable Pattern |
| msg | string | 操作成功提示，设为 `null` 不显示 |

#### 内置操作类型

| type | 说明 | 默认图标 |
|------|------|---------|
| VIEW | 查看详情 | `oct:eye` |
| EDIT | 编辑 | `oct:pencil` |
| DELETE | 删除（带确认弹窗） | `oct:trash` |
| TREE_DELETE | 级联删除（删除当前节点及所有子孙节点） | `oct:trash` |
| APPROVAL | 审批 | `recipe-step-approval` |
| CALL_GLOBAL_METHOD | 调用全局方法（⚠️ 不推荐） | — |

#### 自定义 Button 操作

actions 数组中可以直接使用 Button 组件配置：

```json
{
  "type": "ACTION_COLUMN",
  "cellRendererParams": {
    "showIcon": true,
    "actions": [
      { "type": "VIEW", "href": "/form/:id/view" },
      {
        "component": "Button",
        "componentProps": {
          "content": "审批",
          "icon": "ant:check-outlined",
          "action": {
            "type": "dialog",
            "openFormProps": {
              "formEntityToken": "approval-form",
              "formEntityDataId": "{{currentData.id}}"
            },
            "successAction": { "type": "refreshTable" }
          }
        }
      }
    ]
  }
}
```

## dataFilters

Table 支持通过 `dataFilters` 设置默认数据过滤条件，只显示符合条件的数据。

### 结构定义

```json
{
  "dataFilters": [
    {
      "id": "fieldToken",
      "filterType": "text",
      "type": "equals",
      "filter": "value"
    }
  ]
}
```

### 属性说明

| 属性 | 类型 | 说明 |
|------|------|------|
| `id` | string | 过滤的字段 token |
| `filterType` | `"text"` / `"number"` | 字段数据类型 |
| `type` | string | 过滤操作符（见下表） |
| `filter` | string | 过滤值，支持 `:fieldName` 引用当前表单其他字段的值 |

### 过滤操作符

| filterType | 支持的 type |
|-----------|-------------|
| `text` | `equals`, `notEqual`, `contains`, `notContains`, `startsWith`, `endsWith` |
| `number` | `equals`, `notEqual`, `lessThan`, `lessThanOrEqual`, `greaterThan`, `greaterThanOrEqual` |

### 示例

静态过滤（只显示状态为 active 的数据）：
```json
{
  "dataFilters": [
    { "id": "status", "filterType": "text", "type": "equals", "filter": "active" }
  ]
}
```

引用字段值（`:fieldName` 语法，过滤值来自当前表单的另一个字段）：
```json
{
  "dataFilters": [
    { "id": "departmentId", "filterType": "text", "type": "equals", "filter": ":department" }
  ]
}
```

复合条件（AND/OR）：
```json
{
  "dataFilters": [
    {
      "conditions": [
        { "id": "status", "filterType": "text", "type": "equals", "filter": "active", "colId": "status" },
        { "id": "priority", "filterType": "text", "type": "equals", "filter": "high", "colId": "priority" }
      ],
      "operator": "OR"
    }
  ]
}
```

## JS API

通过 `formApi.getFieldApi('{tableId}')` 获取的接口：

| 方法 | 类型 | 说明 |
|------|------|------|
| node | HTML Element | 组件的根 DOM 元素 |
| gridApi | object | ag-grid 的 [Grid API](https://www.ag-grid.com/react-data-grid/grid-api/) |
| getComponentProps | `() => object` | 获取表格的 componentProps |
| updateComponentProps | `(obj) => void` | 修改表格的 componentProps（合并更新） |
| getRowData | `() => array` | 获取当前所有行数据 |
| getTableState | `() => { tableSize, columnState, filterModel, paginationPageSize }` | 获取当前表格状态 |
| setTableState | `(newState) => void` | 设置表格状态（合并更新） |
| getSearchText | `() => string` | 获取当前模糊搜索文本 |
| getSelectedIds | `() => array` | 获取选中行的 id 数组 |
| refresh | `() => void` | 刷新表格数据 |
| on | `(type, listener) => void` | 监听事件，支持 `'dataFetched'` |

## href 路径规则

| 场景 | href 格式 |
|------|----------|
| 新增 | `/form/{entityToken}/layout/{layoutToken}` |
| 编辑 | `/form/{entityToken}/layout/{layoutToken}/:id` |
| 查看 | `/form/:id/view` |

## HTML 识别

- CETA 原生：`.icp-ag-table`、`.ag-theme-quartz`、`.table-element`
- `id` 属性 → Table 的 `id`
- `.icp-table-filter-item[data-id]` → `pinnedFilter` 数组
- `.icp-toolbar-add-button` → `addButtonHref`
- 通用 HTML：`<table>` 标签，从 `<th>` 提取列定义

## 完整示例

### 基础列表页表格

```json
{
  "component": "Table",
  "id": "user-table",
  "componentProps": {
    "dataSource": { "token": "user" },
    "columnDefs": [
      { "type": "AUTO_INDEX_COLUMN", "headerName": "#" },
      {
        "type": "AVATAR_COLUMN",
        "colId": "avatar",
        "field": "avatar",
        "headerName": "头像",
        "cellRendererParams": { "avatarStyle": "avatar-label" },
        "width": 80
      },
      {
        "type": "TEXT_COLUMN",
        "colId": "name",
        "field": "name",
        "headerName": "姓名",
        "cellRendererParams": { "href": "/form/:id/view" },
        "width": 150,
        "pinned": "left"
      },
      {
        "type": "ENUM_COLUMN",
        "colId": "status",
        "field": "status",
        "headerName": "状态",
        "cellRendererParams": {
          "enums": [
            { "value": "active", "label": "启用", "color": "green" },
            { "value": "inactive", "label": "停用", "color": "red" }
          ]
        },
        "width": 100
      },
      {
        "type": "DATE_COLUMN",
        "colId": "createdAt",
        "field": "createdAt",
        "headerName": "创建时间",
        "cellRendererParams": { "formatterType": "datetime" },
        "width": 180
      },
      {
        "type": "ACTION_COLUMN",
        "width": 200,
        "headerName": "操作",
        "cellRendererParams": {
          "showIcon": true,
          "actions": [
            { "type": "EDIT", "href": "/form/user/layout/edit/:id" },
            { "type": "VIEW", "href": "/form/:id/view" },
            { "type": "DELETE" }
          ]
        },
        "pinned": "right"
      }
    ],
    "addButtonHref": "/form/user/layout/new",
    "pinnedFilter": ["status"],
    "pagination": true,
    "paginationPageSize": 20
  },
  "style": { "height": "600px" }
}
```
