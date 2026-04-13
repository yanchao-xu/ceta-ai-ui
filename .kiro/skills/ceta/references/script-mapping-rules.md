# HTML Script → CETA 配置映射规则

> 本文件定义如何将 HTML 中 `<script>` 标签内的 JavaScript DOM 操作提取并映射为 CETA 平台的配置。
> 适用于 ceta-html-analyzer 在 Step 0 之前的 Script 预处理阶段。

---

## 概述

CETA 平台提供两层机制来处理 HTML 中的用户交互行为：

1. **声明式配置**（优先使用）— `hidden`、`disabled`、`readonly` 条件配置、Button action、validation 等
2. **form.script 脚本**（声明式不够用时）— 表单/页面级 JS 脚本，通过 `formApi`、`restApi`、`messageApi`、`i18nApi` 操作数据和组件

**映射决策流程**：

```
HTML 中的 <script> / 事件处理代码
│
├── 能用声明式配置表达吗？
│   ├── 简单条件显示/隐藏 → hidden 配置（第一部分）
│   ├── 简单条件禁用/启用 → disabled 配置（第一部分）
│   ├── 按钮点击行为 → Button action 19 种类型（第一部分）
│   ├── 字段校验规则 → validation 配置（第一部分）
│   └── 是 → 用声明式配置，不写 script
│
├── 声明式不够用？
│   ├── 跨字段联动校验（A 变化影响 B 的值） → form.script + fieldValueChange（第二部分）
│   ├── 表单加载时调 API 填充数据 → form.script + ready 事件（第二部分）
│   ├── 组件间联动（Table 选中行 → 其他组件） → form.script + getFieldApi（第二部分）
│   ├── 按钮触发自定义逻辑 → form.script + registerMethod + Button formMethod（第二部分）
│   ├── 复杂计算/数据转换 → form.script 内写函数（第二部分）
│   └── 是 → 写 form.script
│
└── 完全无法映射？
    └── 记录到 analysis.json 的 flowHints，提示用户需要 Flow 或自定义开发
```

### HTML 行为 → CETA 机制速查表

| HTML 行为 | 简单场景（声明式） | 复杂场景（form.script） |
|---|---|---|
| onclick 按钮 | Button action（19 种类型） | `formApi.registerMethod` + `formMethod` |
| onchange 联动显示/隐藏 | `hidden` / `disabled` 条件配置 | `formApi.on('fieldValueChange', ...)` |
| onchange 联动修改其他字段值 | — | `formApi.on('fieldValueChange', ...)` + `formApi.setValue` |
| onchange 跨字段校验 | `validation`（简单） | `fieldValueChange` + `messageApi.error` + `formApi.setValue(null)` |
| onload 初始化 | `defaultValue` | `formApi.on('ready', ...)` + `restApi` |
| onload 调 API 填充数据 | — | `formApi.on('ready', ...)` + `restApi.get/post` |
| onsubmit 提交 | Button `action.type: "submit"` | `formApi.registerMethod` + 自定义提交逻辑 |
| Table 选中行联动 | `dataFilters` 依赖字段 | `getFieldApi` + `gridApi.addEventListener` |
| Chart 点击联动 | — | `getFieldApi` + `chart.on('click', ...)` |
| Input 回车触发 | — | `getFieldApi` + `input.addEventListener('keydown', ...)` |
| 设备检测/环境判断 | — | `formApi.on('ready', ...)` + `navigator.userAgent` |
| 录音/摄像头等硬件 | — | `formApi.on('ready', ...)` + 浏览器原生 API |

---

## 第一部分：声明式配置映射（优先使用）

### 显示/隐藏 → CETA hidden 配置

#### JS 模式识别

以下 JS 模式都表示"显示/隐藏"意图：

```javascript
// 模式 1：直接操作 style.display
element.style.display = 'none';        // 隐藏
element.style.display = '';            // 显示
element.style.display = 'block';      // 显示

// 模式 2：操作 classList（最常见，Tailwind HTML 中大量使用）
element.classList.add('hidden');        // 隐藏
element.classList.remove('hidden');     // 显示
element.classList.toggle('hidden');     // 切换

// 模式 3：操作 visibility
element.style.visibility = 'hidden';   // 隐藏（保留占位）
element.style.visibility = 'visible';  // 显示

// 模式 4：操作 opacity
element.style.opacity = '0';           // 视觉隐藏
element.style.opacity = '1';           // 显示
```

#### 提取流程

```
扫描 <script> 中的显示/隐藏操作
│
├── 1. 识别目标元素
│   ├── document.getElementById('xxx') → 通过 id 定位 HTML 元素
│   ├── document.querySelector('.xxx') → 通过 class/选择器定位
│   └── document.querySelectorAll('.view-section') → 批量操作
│
├── 2. 识别触发条件
│   ├── 在 if 语句内 → 提取条件表达式
│   ├── 在事件回调内（onclick/onchange） → 提取触发字段
│   └── 在函数内被其他函数调用 → 追溯调用链找到触发源
│
├── 3. 映射为 CETA hidden 配置
│   ├── 条件基于表单字段值 → dataPredicate
│   ├── 条件基于用户权限 → permissionPredicate
│   ├── 条件基于页面类型 → field: ":context.pageType"
│   └── 无法确定条件 → 记录到 analysis.json，标注需要用户确认
│
└── 4. 写入对应组件的 hidden 属性
```

#### 映射规则表

| JS 条件模式 | CETA hidden 配置 |
|------------|-----------------|
| `if (fieldValue === 'xxx')` 时显示 | `dataPredicate: { conditions: [{ dataField: "field", condition: "equals", value: "xxx" }] }, valueIfPositive: false, valueIfNegative: true` |
| `if (fieldValue !== 'xxx')` 时显示 | `dataPredicate: { conditions: [{ dataField: "field", condition: "notEqual", value: "xxx" }] }, valueIfPositive: false, valueIfNegative: true` |
| `if (fieldValue)` 时显示（非空） | `dataPredicate: { conditions: [{ dataField: "field", condition: "notBlank" }] }, valueIfPositive: false, valueIfNegative: true` |
| `if (!fieldValue)` 时显示（为空） | `dataPredicate: { conditions: [{ dataField: "field", condition: "blank" }] }, valueIfPositive: false, valueIfNegative: true` |
| `if (a === 'x' && b === 'y')` 时显示 | `dataPredicate: { operator: "AND", conditions: [...] }, valueIfPositive: false, valueIfNegative: true` |
| `if (a === 'x' \|\| b === 'y')` 时显示 | `dataPredicate: { operator: "OR", conditions: [...] }, valueIfPositive: false, valueIfNegative: true` |

### 声明式案例

#### 案例 1：Tab/视图切换 → Tabs 组件或路由

```javascript
// JS 原文
function switchTab(tabName) {
    document.querySelectorAll('.view-section').forEach(el => {
        el.classList.add('hidden');
    });
    document.getElementById('view-' + tabName).classList.remove('hidden');
}
```

**CETA 映射**：

方案 A（推荐）：拆为独立页面，通过路由切换
```
view-search  → pnr-search-page.json   （路由：/pnr-management/page/pnr-search）
view-booking → flight-booking-page.json（路由：/pnr-management/page/flight-booking）
```

方案 B：使用 Tabs 组件（视图间关联紧密时）
```json
{
  "component": "Tabs",
  "componentProps": {
    "items": [
      { "key": "search", "label": "PNR搜索", "fields": [...] },
      { "key": "booking", "label": "航班预订", "fields": [...] }
    ]
  }
}
```

#### 案例 2：弹窗显示/关闭 → Button action dialog

```javascript
// JS 原文
function showMCTCheck() {
    document.getElementById('mct-modal').classList.remove('hidden');
}
```

```json
{
  "component": "Button",
  "componentProps": {
    "content": "MCT检查",
    "action": {
      "type": "dialog",
      "openFormProps": { "formEntityToken": "mct-check" },
      "openDialogProps": { "title": "MCT 最短中转时间检查" }
    }
  }
}
```

#### 案例 3：条件显示字段

```javascript
// JS 原文
const extraFields = (isChild || isInfant) ? `<input type="number" class="pax-age">` : '';
```

```json
{
  "component": "Input",
  "id": "age",
  "title": "年龄",
  "hidden": {
    "dataPredicate": {
      "operator": "OR",
      "conditions": [
        { "dataField": "type", "condition": "equals", "value": "CNN" },
        { "dataField": "type", "condition": "equals", "value": "INF" }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

#### 案例 4：按钮禁用/启用

```javascript
// JS 原文
btn.disabled = !isValid;
```

```json
{
  "component": "Button",
  "componentProps": { "content": "下一步" },
  "disabled": {
    "dataPredicate": {
      "operator": "AND",
      "conditions": [
        { "dataField": "surname", "condition": "notBlank" },
        { "dataField": "given", "condition": "notBlank" }
      ]
    },
    "valueIfPositive": false,
    "valueIfNegative": true
  }
}
```

> `disabled`、`readonly` 和 `hidden` 使用完全相同的 `ConditionalPropertyPropType` 配置格式。详见 `hidden-rules.md`。

#### 案例 5：下拉菜单显示/隐藏 → 不映射

```javascript
function toggleDropdown(id) { document.getElementById(id).classList.toggle('hidden'); }
```

不需要映射。CETA 的 Select 组件自带下拉能力，这类纯 UI 交互由组件内部处理。

---

## 第二部分：form.script 脚本（声明式不够用时）

### 概述

每个表单布局（FormEntity Layout）和页面（FormEntityPage）都有一个 `script` 属性，可以写 JS 脚本。
脚本写在 JSON 的 `form.script` 字段中，是一个 JS 字符串，运行时直接在浏览器执行。

```json
{
  "form": {
    "script": "formApi.on('ready', function() {\n  // 你的代码\n});"
  },
  "fields": [...]
}
```

**脚本中所有浏览器 DOM API 均可用**，和本地写 JS 文件无任何区别。

此外，平台注入了四个全局 API：

| API | 用途 | 说明 |
|-----|------|------|
| `formApi` | 表单数据操作、事件监听、字段 API | 最核心的 API |
| `restApi` | HTTP 请求 | `get`/`post`/`put`/`delete`，返回 Promise |
| `messageApi` | 消息提示 | Ant Design message API（success/error/warning） |
| `i18nApi` | 国际化 | 获取当前语言等 |

### formApi 核心方法

| 方法 | 类型 | 说明 |
|------|------|------|
| `on(eventType, callback)` | 事件监听 | 监听表单事件（见下方事件列表） |
| `off(eventType, callback)` | 取消监听 | 取消事件监听 |
| `getValue(id)` | 读数据 | 获取单个字段的值 |
| `setValue(id, newValue)` | 写数据 | 设置单个字段的值 |
| `getData()` | 读数据 | 获取表单所有数据 |
| `setData(newData)` | 写数据 | 替换表单所有数据 |
| `getContext()` | 读上下文 | 获取上下文信息（userProfile、pageType 等） |
| `getFieldApi(id)` | 获取组件 API | 获取单个组件的接口（Table 的 gridApi、EChart 的 chart 等） |
| `getFieldApis()` | 获取所有组件 API | 获取所有可用的字段 API |
| `registerMethod(name, func)` | 注册方法 | 注册函数，供 Button `formMethod` action 调用 |
| `getMethods()` | 获取方法 | 获取所有注册的函数 |
| `validateForm()` | 验证 | 立即验证表单，返回 Promise |
| `isDataValid()` | 验证 | 当前表单数据是否合法 |
| `isDirty()` | 状态 | 表单数据是否被修改过 |
| `submit({ saveOnly, handler })` | 提交 | 提交表单 |
| `refresh()` | 刷新 | 刷新表单数据 |
| `cancel()` | 取消 | 取消并返回上一页 |
| `setFieldComponentProps(id, props)` | 动态设置 | 通过 JS 动态设置组件的 componentProps |
| `setError(id, error)` | 设置错误 | 设置某个字段的错误信息 |
| `setErrors(errors)` | 设置错误 | 设置表单所有错误信息 |
| `node` | DOM 元素 | 表单根 DOM 元素 |

### formApi 事件列表

| 事件名 | 回调签名 | 说明 | 触发时机 |
|--------|---------|------|---------|
| `ready` | `(formApi) => void` | **最常用**。表单 API 及所有异步组件 API 可用时触发 | 推荐所有代码放在 ready 下 |
| `firstRendered` | `(formApi) => void` | 表单第一次渲染时 | 早于 ready |
| `fieldValueChange` | `(keyPath, newValue) => void` | 某个字段值改变时 | 用户输入或 setValue 触发 |
| `change` | `(allData) => void` | 表单数据发生任何改变时 | 参数为所有表单数据 |
| `submitted` | `(response) => void` | 表单提交成功后 | submit 成功后 |
| `validationFail` | `(errors) => void` | 表单验证失败时 | 参数为所有错误信息 |
| `schemaChanged` | `(schema) => void` | 表单 schema 发生改变时 | 动态 schema 场景 |
| `unmount` | `() => void` | 表单卸载时 | 清理资源用 |

### restApi 方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `get` | `(url, config) => Promise` | GET 请求 |
| `getStatic` | `(url, config) => Promise` | GET 请求（带缓存） |
| `post` | `(url, data, config) => Promise` | POST 请求 |
| `put` | `(url, data, config) => Promise` | PUT 请求 |
| `delete` | `(url, config) => Promise` | DELETE 请求 |

---

### 何时用声明式 vs form.script

| 场景 | 用声明式 | 用 form.script |
|------|---------|---------------|
| 字段 A 的值决定字段 B 是否显示 | ✅ `hidden` 条件配置 | — |
| 字段 A 的值决定字段 B 是否禁用 | ✅ `disabled` 条件配置 | — |
| 按钮点击提交/跳转/弹窗/请求 | ✅ Button action | — |
| 字段必填/长度校验 | ✅ `validation` | — |
| 字段 A 变化时修改字段 B 的值 | — | ✅ `fieldValueChange` + `setValue` |
| 字段 A 变化时校验与字段 B 的关系 | — | ✅ `fieldValueChange` + `messageApi.error` |
| 表单加载时调 API 填充数据 | — | ✅ `ready` + `restApi` |
| 按钮点击执行自定义逻辑 | — | ✅ `registerMethod` + `formMethod` |
| Table 选中行联动其他组件 | — | ✅ `getFieldApi` + `gridApi` 事件 |
| Chart 点击联动 | — | ✅ `getFieldApi` + `chart.on` |
| 设备检测/环境判断 | — | ✅ `ready` + `navigator` |

**原则：能用声明式就用声明式，声明式不够用时才写 script。两者可以共存于同一个表单/页面。**

---

### form.script 映射案例

#### 案例 1：表单加载时根据用户角色调 API 填充数据

**HTML 原文**：
```javascript
window.onload = function() {
    fetch('/api/sales-manager-info').then(res => res.json()).then(data => {
        document.getElementById('team').value = data.team;
        document.getElementById('manager').value = data.managerName;
    });
}
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('ready', () => {\n  const currentUser = formApi.getContext().userProfile.nickName;\n  restApi.get('/flow/api/flow-rest/sales-manager-info-flow').then(function(data) {\n    const result = data.results.find(item => item.nickName === currentUser);\n    formApi.setValue('team', result.team);\n    formApi.setValue('teamManager', result.nickNameM);\n    formApi.setValue('teamManagerEmail', result.emailM);\n  });\n});"
  }
}
```

**映射要点**：
- `window.onload` / `DOMContentLoaded` → `formApi.on('ready', ...)`
- `fetch()` / `XMLHttpRequest` → `restApi.get()` / `restApi.post()`
- `document.getElementById('xxx').value = yyy` → `formApi.setValue('xxx', yyy)`
- 获取当前用户信息 → `formApi.getContext().userProfile`

#### 案例 2：onchange 跨字段联动校验

**HTML 原文**：
```javascript
document.getElementById('signingDate').addEventListener('change', function() {
    const biddingDate = document.getElementById('biddingDate').value;
    const signingDate = this.value;
    if (signingDate < biddingDate) {
        this.value = '';
        alert('签约时间不能早于招标时间');
    }
});
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('fieldValueChange', (keyPath, value) => {\n  if (keyPath === 'signingDate') {\n    const biddingDate = formApi.getValue('biddingDate');\n    const signingDate = formApi.getValue('signingDate');\n    if (biddingDate && signingDate && signingDate < biddingDate) {\n      formApi.setValue('signingDate', null);\n      messageApi.error('签约时间不能早于招标时间');\n    }\n  }\n});"
  }
}
```

**映射要点**：
- `element.addEventListener('change', ...)` → `formApi.on('fieldValueChange', (keyPath, value) => { if (keyPath === 'fieldId') { ... } })`
- `element.value` → `formApi.getValue('fieldId')`
- `element.value = ''` → `formApi.setValue('fieldId', null)`
- `alert('xxx')` → `messageApi.error('xxx')` 或 `messageApi.warning('xxx')`

#### 案例 3：onchange 联动修改其他字段值

**HTML 原文**：
```javascript
document.getElementById('salesOwner').addEventListener('change', function() {
    const selected = this.options[this.selectedIndex];
    document.getElementById('salesManager').value = selected.dataset.manager;
    document.getElementById('team').value = selected.dataset.team;
});
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('fieldValueChange', (keyPath, value) => {\n  if (keyPath === 'salesOwner') {\n    const teamValue = formApi.getValue('salesOwner[0].team');\n    const nickNameM = formApi.getValue('salesOwner[0].nickNameM');\n    const emailM = formApi.getValue('salesOwner[0].emailM');\n    formApi.setValue('salesManager', [{\n      label: nickNameM,\n      value: emailM,\n      email: emailM,\n      jobTitle: '销售经理',\n      team: teamValue\n    }]);\n  }\n});"
  }
}
```

**映射要点**：
- Select 的 onchange 联动 → `fieldValueChange` 监听对应字段
- 从选中项取关联数据 → `formApi.getValue('field[0].subField')` 深层路径访问
- 设置关联字段 → `formApi.setValue` 可以设置复杂对象/数组

#### 案例 4：按钮点击执行自定义逻辑（registerMethod + formMethod）

**HTML 原文**：
```javascript
document.getElementById('exportBtn').addEventListener('click', function() {
    const filterModel = getTableFilterModel();
    fetch('/api/export', {
        method: 'POST',
        body: JSON.stringify({ filterModel })
    }).then(() => alert('导出成功'));
});
```

**CETA 映射**（分两部分）：

Button JSON：
```json
{
  "component": "Button",
  "componentProps": {
    "content": "自定义导出",
    "icon": "DownloadOutlined",
    "action": {
      "type": "formMethod",
      "methodName": "customExport"
    }
  }
}
```

form.script：
```json
{
  "form": {
    "script": "formApi.registerMethod('customExport', function() {\n  const theTable = formApi.getFieldApi('myTable');\n  const tableState = theTable.getTableState();\n  const filterModel = tableState.filterModel;\n  restApi.post('/api/export', { filterModel }).then(function() {\n    messageApi.success('导出成功');\n  }).catch(function() {\n    messageApi.error('导出失败');\n  });\n});"
  }
}
```

**映射要点**：
- 按钮点击 → 自定义逻辑的桥梁是 `registerMethod` + `formMethod`
- Button 的 `action.type: "formMethod"` + `methodName` 调用 script 中注册的方法
- `registerMethod` 的回调接收 `{ buttonApi, response, currentData }` 参数
- `buttonApi.setLoading(true/false)` 可以控制按钮加载状态
- 方法可以返回 Promise，平台会等待 Promise 完成

#### 案例 5：Table 选中行联动其他组件

**HTML 原文**：
```javascript
table.addEventListener('click', function(e) {
    const row = e.target.closest('tr');
    const gender = row.dataset.gender;
    filterTable2ByGender(gender);
});
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('ready', function() {\n  const firstTable = formApi.getFieldApi('jsDemoTable1');\n  firstTable.gridApi.addEventListener('selectionChanged', function() {\n    const rowData = firstTable.gridApi.getSelectedRows()[0];\n    if (rowData && rowData.gender && rowData.gender[0]) {\n      formApi.setValue('filterInput', rowData.gender[0].label);\n    } else {\n      formApi.setValue('filterInput', '');\n    }\n  });\n});"
  }
}
```

**映射要点**：
- `formApi.getFieldApi('tableId')` 获取 Table 组件的 API
- Table 组件暴露 `gridApi`（Ag-Grid API），可以监听 `selectionChanged` 等事件
- 通过 `formApi.setValue` 设置一个隐藏字段的值，另一个 Table 的 `dataFilters` 依赖该字段会自动刷新
- Ag-Grid API 参考：https://www.ag-grid.com/javascript-data-grid/grid-api/

#### 案例 6：EChart 点击联动 Table

**HTML 原文**：
```javascript
chart.on('click', function(params) {
    const category = params.data.category;
    filterTableByCategory(category);
});
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('ready', function() {\n  const theChart = formApi.getFieldApi('productPieChart');\n  theChart.chart.on('click', function(params) {\n    const category = params.data && params.data.productCategory;\n    formApi.setValue('hiddenCategoryValue', category);\n  });\n});"
  }
}
```

**映射要点**：
- `formApi.getFieldApi('chartId')` 获取 EChart 组件的 API
- EChart 组件暴露 `chart`（ECharts 实例），可以监听 `click` 等事件
- ECharts API 参考：https://echarts.apache.org/zh/api.html
- `formApi.setValue` 可以设置任意 key（不需要对应一个真实组件），其他组件的 dataFilters 可以依赖它

#### 案例 7：Input 回车触发按钮点击

**HTML 原文**：
```javascript
document.getElementById('searchInput').addEventListener('keydown', function(e) {
    if (e.key === 'Enter') {
        document.getElementById('searchBtn').click();
    }
});
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('ready', function() {\n  const inputApi = formApi.getFieldApi('name');\n  inputApi.input.addEventListener('keydown', function(event) {\n    if (event.key === 'Enter') {\n      event.stopPropagation();\n      const button = formApi.getFieldApi('sendButton').node;\n      button.click();\n    }\n  });\n});"
  }
}
```

**映射要点**：
- Input 组件的 `getFieldApi` 暴露 `input`（原生 input DOM 元素）
- Button 组件的 `getFieldApi` 暴露 `node`（按钮 DOM 元素）
- 可以直接操作原生 DOM 事件

#### 案例 8：设备检测 — 移动端/桌面端判断

**HTML 原文**：
```javascript
if (/Mobi|Android|iPhone/.test(navigator.userAgent)) {
    document.getElementById('mobileView').style.display = 'block';
    document.getElementById('desktopView').style.display = 'none';
}
```

**CETA form.script 映射**：
```json
{
  "form": {
    "script": "formApi.on('ready', () => {\n  function isMobileDevice() {\n    return /Mobi|Android|iPhone|iPad|iPod/i.test(navigator.userAgent);\n  }\n  if (isMobileDevice()) {\n    formApi.setValue('terminalType', 'mobile');\n  } else {\n    formApi.setValue('terminalType', 'desktop');\n  }\n});"
  }
}
```

配合隐藏字段和条件显示：
```json
{
  "component": "Input",
  "id": "terminalType",
  "title": "终端类型",
  "hidden": true
}
```

其他组件通过 `hidden` 条件配置依赖 `terminalType` 字段值来决定显示/隐藏。

#### 案例 9：表单加载时设置默认值

**HTML 原文**：
```javascript
window.onload = function() {
    document.getElementById('createDate').value = new Date().toISOString().split('T')[0];
    document.getElementById('status').value = 'draft';
}
```

**CETA 映射**：

简单默认值用声明式 `defaultValue`：
```json
{ "component": "Input", "id": "status", "defaultValue": "draft" }
```

需要计算的默认值用 form.script：
```json
{
  "form": {
    "script": "formApi.on('ready', () => {\n  const today = new Date().toISOString().split('T')[0];\n  if (formApi.getValue('createDate') == null) {\n    formApi.setValue('createDate', today);\n  }\n});"
  }
}
```

**映射要点**：
- 静态默认值 → 用 `defaultValue` 属性（声明式）
- 动态默认值（当前日期、当前用户等） → 用 `formApi.on('ready', ...)` + `setValue`
- 注意判断 `getValue == null` 避免覆盖已有数据（编辑模式）

---

## 第三部分：HTML Script 提取流程（ceta-html-analyzer 使用）

### Script 预处理阶段（在 CSS 预处理 Step 0 之前执行）

```
拿到 HTML
│
├── Step -1: Script 预处理
│   │
│   ├── -1.1 提取 <script> 标签内容
│   │   ├── 忽略外部脚本引用（<script src="...">）— 框架代码不分析
│   │   ├── 提取 inline <script> 标签内的 JS 代码
│   │   └── 提取 HTML 元素上的 inline 事件属性（onclick="..."、onchange="..." 等）
│   │
│   ├── -1.2 分类 JS 代码片段
│   │   │
│   │   │  对每个 JS 代码片段，判断属于哪一类：
│   │   │
│   │   ├── 类别 A：显示/隐藏操作
│   │   │   ├── 识别标志：style.display、classList.add/remove('hidden')、visibility
│   │   │   └── 映射目标：组件的 hidden 条件配置（声明式）
│   │   │
│   │   ├── 类别 B：禁用/启用操作
│   │   │   ├── 识别标志：element.disabled = true/false、classList 操作 disabled 相关 class
│   │   │   └── 映射目标：组件的 disabled 条件配置（声明式）
│   │   │
│   │   ├── 类别 C：表单验证
│   │   │   ├── 识别标志：required 检查、正则匹配、长度检查、自定义验证函数
│   │   │   ├── 简单验证 → validation 配置（声明式）
│   │   │   └── 跨字段验证 → form.script + fieldValueChange
│   │   │
│   │   ├── 类别 D：字段值联动
│   │   │   ├── 识别标志：onchange 回调中修改其他字段的值
│   │   │   └── 映射目标：form.script + fieldValueChange + setValue
│   │   │
│   │   ├── 类别 E：API 调用
│   │   │   ├── 识别标志：fetch()、XMLHttpRequest、axios
│   │   │   ├── 页面加载时调用 → form.script + ready + restApi
│   │   │   └── 按钮点击时调用 → Button action request 或 registerMethod + formMethod
│   │   │
│   │   ├── 类别 F：Tab/视图切换
│   │   │   ├── 识别标志：批量隐藏 + 单个显示、Tab 按钮点击
│   │   │   └── 映射目标：Tabs 组件或拆为独立页面（声明式）
│   │   │
│   │   ├── 类别 G：弹窗操作
│   │   │   ├── 识别标志：modal 显示/隐藏、dialog 操作
│   │   │   └── 映射目标：Button action dialog（声明式）
│   │   │
│   │   ├── 类别 H：组件间联动
│   │   │   ├── 识别标志：Table 选中行事件、Chart 点击事件、组件间数据传递
│   │   │   └── 映射目标：form.script + getFieldApi + 组件原生事件
│   │   │
│   │   └── 类别 I：复杂业务逻辑 / 浏览器原生 API
│   │       ├── 识别标志：复杂计算、第三方库调用、MediaRecorder、Canvas 等
│   │       └── 映射目标：form.script 内直接写（浏览器 API 均可用）
│   │
│   ├── -1.3 生成 scriptMap（Script 预处理结果）
│   │   │
│   │   │  将分类结果记录到 scriptMap，供后续步骤使用：
│   │   │
│   │   ├── declarativeRules: [] — 可以用声明式配置的规则
│   │   │   ├── { type: "hidden", targetElement: "xxx", condition: {...} }
│   │   │   ├── { type: "disabled", targetElement: "xxx", condition: {...} }
│   │   │   └── { type: "validation", targetElement: "xxx", rules: {...} }
│   │   │
│   │   ├── scriptCode: "" — 需要写入 form.script 的代码
│   │   │   ├── ready 事件代码（初始化、API 调用）
│   │   │   ├── fieldValueChange 事件代码（字段联动）
│   │   │   └── registerMethod 代码（自定义方法）
│   │   │
│   │   └── unmappable: [] — 无法映射的逻辑
│   │       └── { description: "xxx", sourceCode: "...", suggestion: "建议通过 Flow 实现" }
│   │
│   └── -1.4 输出
│       ├── declarativeRules 在 Step 1 生成组件 JSON 时写入对应组件的 hidden/disabled/validation
│       ├── scriptCode 在生成 form JSON 时写入 form.script 字段
│       └── unmappable 写入 analysis.json 的 scriptHints 字段
```

### script 代码生成规范

生成 form.script 代码时，必须遵循以下规范：

1. **所有代码放在 `formApi.on('ready', ...)` 内**（推荐），确保异步组件 API 可用
2. **使用 `formApi` 操作数据**，不要直接操作 DOM 获取/设置表单值
3. **使用 `restApi` 发请求**，不要用 `fetch` 或 `XMLHttpRequest`
4. **使用 `messageApi` 提示用户**，不要用 `alert`/`confirm`/`prompt`
5. **script 是 JSON 字符串**，换行用 `\n`，引号用单引号或转义双引号
6. **多个 `fieldValueChange` 监听可以合并**到一个回调中，用 `if (keyPath === 'xxx')` 分支
7. **registerMethod 的方法名**必须和 Button action 的 `methodName` 一致
8. **ES5 兼容**：如果不确定浏览器支持，用 `var` 和 `function`；现代浏览器可以用 `const`/`let`/箭头函数

### script 代码模板

#### 模板 1：ready 事件 + API 调用填充数据

```
formApi.on('ready', () => {
  const context = formApi.getContext();
  const currentUser = context.userProfile.nickName;
  
  restApi.get('/flow/api/flow-rest/xxx-flow').then(function(data) {
    formApi.setValue('fieldA', data.valueA);
    formApi.setValue('fieldB', data.valueB);
  });
});
```

#### 模板 2：fieldValueChange 跨字段校验

```
formApi.on('fieldValueChange', (keyPath, value) => {
  if (keyPath === 'fieldA' || keyPath === 'fieldB') {
    const a = formApi.getValue('fieldA');
    const b = formApi.getValue('fieldB');
    if (a < b) {
      formApi.setValue('fieldB', null);
      messageApi.error('fieldA 不能小于 fieldB');
    }
  }
});
```

#### 模板 3：registerMethod + formMethod

```
formApi.registerMethod('myMethod', function(params) {
  const { buttonApi, response } = params;
  buttonApi.setLoading(true);
  
  return restApi.post('/flow/api/flow-rest/xxx', {
    id: formApi.getValue('id')
  }).then(function(data) {
    messageApi.success('操作成功');
    buttonApi.setLoading(false);
  }).catch(function(err) {
    messageApi.error('操作失败');
    buttonApi.setLoading(false);
  });
});
```

对应的 Button：
```json
{
  "component": "Button",
  "componentProps": {
    "content": "执行操作",
    "action": { "type": "formMethod", "methodName": "myMethod" }
  }
}
```

#### 模板 4：Table 联动

```
formApi.on('ready', function() {
  const tableApi = formApi.getFieldApi('myTable');
  tableApi.gridApi.addEventListener('selectionChanged', function() {
    const selectedRow = tableApi.gridApi.getSelectedRows()[0];
    if (selectedRow) {
      formApi.setValue('filterField', selectedRow.someValue);
    }
  });
});
```

#### 模板 5：ready 事件 + 条件默认值

```
formApi.on('ready', () => {
  const idValue = formApi.getValue('id');
  if (idValue == null) {
    // 新建模式，设置默认值
    const today = new Date().toISOString().split('T')[0];
    formApi.setValue('createDate', today);
    formApi.setValue('status', 'draft');
  }
});
```

---

## 第四部分：analysis.json 中的 Script 相关字段

当 HTML 中包含 `<script>` 时，analysis.json 需要增加以下字段：

```json
{
  "scriptAnalysis": {
    "hasScript": true,
    "declarativeCount": 5,
    "scriptCodeCount": 3,
    "unmappableCount": 1,
    "summary": "提取了 5 条声明式规则（hidden/disabled），3 段 form.script 代码（字段联动、API 调用），1 处无法映射的逻辑"
  },
  "scriptHints": [
    {
      "description": "复杂的价格计算逻辑，涉及多个字段的级联计算",
      "suggestion": "建议通过 CETA Flow 实现计算逻辑，或在 form.script 中编写",
      "sourceCode": "function calculatePrice() { ... }"
    }
  ]
}
```

---

## 参考文档

- `icp-doc-website/docs/platform/frontend/js-code.md` — JS 脚本使用说明和示例
- `icp-doc-website/docs/components/api-for-props/js-api.md` — formApi/restApi/messageApi 完整 API
- `icp-doc-website/docs/platform/frontend/javascript-coding-specification.md` — JS 编码规范
- `templates/script/` — 真实项目的 form.script 示例
- `skills/ceta/references/hidden-rules.md` — hidden/disabled/readonly 条件配置详细规范
- `skills/ceta/references/components/display/Button.md` — Button action 19 种类型详解
