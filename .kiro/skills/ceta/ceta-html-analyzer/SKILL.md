---
name: ceta-html-analyzer
description: >
  HTML 结构分析器。接收用户输入的 HTML，判断其业务范围（项目级/PBC级/单页面级），
  按 CETA 平台层级（Project → PBC → FormEntity/Page）进行结构拆分，
  然后自顶向下调度对应的子 SKILL 生成各层级的配置。
inclusion: manual
metadata:
  version: "1.0.0"
  tags: ["ceta", "html", "analyzer", "dispatcher"]
  layer: "analyzer"
---

# HTML 结构分析器

## 职责

本 SKILL 是 HTML 输入的**分析与调度层**，位于 `ceta-basic`（入口）和各模块 SKILL（ceta-form、ceta-page 等）之间。

核心职责：
1. 接收用户提供的 HTML（可能是完整应用原型、单个页面、或单个表单）
2. 分析 HTML 的业务范围和复杂度
3. 按 CETA 平台层级拆分结构
4. 自顶向下调度子 SKILL 逐层生成配置

## 两阶段文档加载规则（重要）

**阶段一（分析）只需要加载：**
- `ceta-basic`（入口 SKILL，了解平台数据模型和字段类型映射）
- `skills/references/html-mapping-rules.md`（HTML 元素 → CETA 组件类型的映射，用于识别字段类型）

**阶段一不需要加载：**
- 任何组件文档（`skills/components/` 下的 Card.md、Grid.md、Button.md、Table.md 等）
- `skills/references/style-mapping-rules.md`
- 子 SKILL（ceta-form、ceta-page、ceta-app-config）

阶段一的目标只是识别"有几个表单、几个页面、多少字段、什么类型"，不需要知道组件的 JSON 写法。

**阶段二（用户确认后生成 JSON）才按需加载：**
- 子 SKILL（ceta-form、ceta-page、ceta-app-config）
- `skills/references/schema-rules.md`（schemaJson 结构规范）
- `skills/references/style-mapping-rules.md`（样式映射）
- 具体组件文档（只读取实际用到的组件，如遇到 Grid 布局才读 Grid.md）


---

## 第一步：范围判定

拿到 HTML 后，首先判断它描述的是哪个层级的内容。

### 判定规则

| 范围级别 | 判定条件 | 示例 |
|---------|---------|------|
| **项目级（Project）** | HTML 包含导航栏/侧边栏菜单 + 多个独立业务区域；或包含多个 `<nav>`、多组不相关的表单/表格；或有明显的应用壳（header/sidebar/content 三栏布局） | 完整的 ERP 系统原型、带侧边栏的管理后台 |
| **PBC 级（业务组件）** | HTML 包含同一业务域下的多个相关页面（如列表页+表单页）；或有 Tab 切换的多视图；或包含 2+ 个相关但独立的表单/表格 | 用户管理模块（用户列表+新建用户表单+角色管理） |
| **单页面级（Page/Form）** | HTML 只包含一个表单、一个列表页、或一个仪表盘；结构单一，无导航 | 一个请假申请表单、一个订单列表页 |

### 判定流程

```
1. 扫描 HTML 顶层结构
   ├── 有 <nav>/<aside> + 多个内容区域？ → 项目级
   ├── 有多个独立的 <form>/<table> 且业务相关？ → PBC 级
   └── 只有单个 <form> 或 <table>？ → 单页面级

2. 辅助信号（加强判定）
   ├── 多个不同的菜单项/路由链接 → 倾向项目级
   ├── Tab/NavTab 切换多个视图 → 倾向 PBC 级
   ├── 页面标题暗示模块名（如"用户管理"） → 倾向 PBC 级
   └── 无导航、无菜单、单一内容 → 倾向单页面级

3. 如果无法确定，询问用户：
   "这个 HTML 看起来包含了 [X] 和 [Y]，请问这是一个完整应用的原型，
    还是某个模块的页面？"
```


---

## 第二步：结构拆分

根据范围级别，将 HTML 拆分为 CETA 平台的层级结构。

### 项目级拆分

```
输入 HTML
│
├── 1. 提取应用壳（App Shell）
│   ├── 导航栏 → 菜单配置（navbar items）
│   ├── 侧边栏菜单项 → 路由配置（routers）+ 菜单配置
│   └── 全局样式/主题 → themeConfig（均由 ceta-app-config SKILL 统一生成）
│
├── 2. 按菜单/导航拆分为 PBC
│   ├── 每个一级菜单项 = 一个 PBC
│   │   例：侧边栏有"用户管理"、"订单管理"、"报表" → 3 个 PBC
│   └── 为每个 PBC 生成 token（kebab-case）
│
└── 3. 对每个 PBC 递归执行「PBC 级拆分」
```

#### 项目级输出结构

```
output/
└── {project-token}/
    ├── analysis.json              # 分析结果摘要
    ├── {project-token}-app-config.json  # 完整 app.config.json（菜单+主题+路由，唯一的全局配置文件）
    └── pbcs/
        ├── {pbc-token-1}/         # 每个 PBC 的子目录
        │   ├── ...（PBC 级输出）
        └── {pbc-token-2}/
            └── ...
```

**注意：不单独生成 menu.json、theme.json、theme-css.css。菜单、主题、CSS 全部写入 app-config.json。**

### PBC 级拆分

#### 核心概念：FormEntity vs Page 的区分

**这是最关键的判断，必须准确。**

| 概念 | 本质 | 字段是否存数据库 | 典型场景 |
|------|------|----------------|---------|
| FormEntity（表单实体） | 数据模型，定义字段，和数据库表一一映射 | ✅ 是，每个字段对应数据库列 | 新建订单、编辑用户信息、填写申请表 |
| FormEntity Layout | 同一套数据（同一个 FormEntity）的不同 UI 表现形式 | 共享同一套字段 | new（新建视图）、edit（编辑视图）、view（只读视图） |
| FormEntityPage（页面） | 纯 UI 交互页面，字段不存数据库 | ❌ 否，仅用于用户交互 | 搜索页、筛选页、仪表盘、列表页、首页 |

#### 判断规则

```
HTML 中的一个区域包含输入字段时，问自己：
│
├── 这些字段的数据需要持久化存储到数据库吗？
│   │
│   ├── 是 → FormEntity
│   │   ├── 用户填写后数据会被保存（如创建订单、录入旅客信息）
│   │   ├── 有"提交"/"保存"按钮，提交后数据入库
│   │   ├── 同一套数据可能有多个视图（新建/编辑/查看）→ 这些是 Layout
│   │   └── 输出：{entity-token}-form.json（布局）+ {entity-token}-fields.json（字段定义）
│   │
│   └── 否 → Page
│       ├── 字段仅用于用户交互（如搜索条件、筛选器）
│       ├── 有"搜索"/"查询"/"筛选"按钮，触发查询而非保存
│       ├── 数据表格/列表展示（Table 组件）
│       ├── 仪表盘/统计卡片
│       └── 输出：{page-token}-page.json（页面配置）
```

#### 常见误判场景

| 场景 | 错误判断 | 正确判断 | 原因 |
|------|---------|---------|------|
| 搜索页面有输入框 | ❌ FormEntity | ✅ Page | 搜索条件不存数据库，只是查询参数 |
| 列表页上方有筛选区 | ❌ FormEntity | ✅ Page | 筛选条件是 UI 交互，不是数据模型 |
| 登录页有用户名密码 | ❌ FormEntity | ✅ Page | 登录凭证不是业务数据模型 |
| 新建订单表单 | ✅ FormEntity | ✅ FormEntity | 订单数据需要存库 |
| 编辑用户信息 | ✅ FormEntity | ✅ FormEntity | 用户数据需要存库，编辑是另一个 Layout |
| 订单详情只读页 | ❌ Page | ✅ FormEntity 的 view Layout | 展示的是同一套数据模型，只是只读视图 |

#### Layout 的含义

一个 FormEntity 可以有多个 Layout，每个 Layout 是同一套字段的不同 UI 表现：
- `new` — 新建视图（所有字段可编辑，有提交按钮）
- `edit` — 编辑视图（部分字段可编辑，有保存按钮）
- `view` — 查看视图（所有字段只读）
- 自定义 Layout — 如审批视图（只显示部分字段 + 审批按钮）

**Layout 不是独立的 FormEntity**，它们共享同一套 fields。
如果 HTML 中同一业务数据有"新建"和"编辑"两个表单，它们是同一个 FormEntity 的两个 Layout，不是两个 FormEntity。

#### 拆分流程

```
PBC 对应的 HTML 片段
│
├── 1. 识别 FormEntity（数据模型）
│   ├── 判断标准：字段数据需要持久化存储
│   ├── 提取字段定义（fields）— 这些字段和数据库列一一对应
│   │   ├── <input type="text"> → Input (TEXT_BOX)
│   │   ├── <select> → Select (SELECT)
│   │   ├── <textarea> → Textarea (TEXTAREA)
│   │   ├── <input type="number"> → NumberPicker (NUMBER)
│   │   ├── <input type="date"> → DatePicker (DATE)
│   │   ├── <input type="checkbox"> → Checkbox (CHECKBOX)
│   │   ├── <input type="radio"> → Radio (RADIO)
│   │   ├── <input type="file"> → Upload (UPLOAD)
│   │   └── 其他 → 根据上下文推断或用 Text 兜底
│   ├── 识别 Layout（同一数据的不同视图）
│   │   ├── 有新建表单 → new layout
│   │   ├── 有编辑表单 → edit layout
│   │   ├── 有只读详情 → view layout
│   │   └── 多个视图共享同一套 fields
│   └── 识别布局结构
│       ├── 有分区/卡片包裹 → Card 布局
│       ├── 有栅格/多列 → Grid 布局
│       └── 有折叠面板 → Collapse 布局
│
├── 2. 识别 Page（纯 UI 页面）
│   ├── 判断标准：字段仅用于用户交互，不存数据库
│   ├── 搜索/筛选页面 → Page（搜索条件是 UI 交互组件）
│   ├── 数据列表页（<table>）→ Page（Table 组件 + 关联 FormEntity）
│   ├── 仪表盘/统计页 → Page
│   └── 首页/欢迎页 → Page
│
├── 3. 识别流程线索（FlowDefinition）
│   ├── 有"提交"→"审批"→"完成"等按钮/状态 → 可能需要流程
│   └── 记录流程线索，但不在此 SKILL 生成（交给 ceta-flow）
│
└── 4. 记录菜单信息供 ceta-app-config 使用
    ├── 每个页面/表单 = 一个潜在菜单项
    └── 菜单项对应路由路径
```

#### PBC 级输出结构与文件命名

```
output/{project-token}/pbcs/{pbc-token}/
├── {entity-token}-form.json       # FormEntity 的表单布局 schemaJson
├── {entity-token}-fields.json     # FormEntity 的字段定义（和数据库映射）
└── {page-token}-page.json         # Page 的页面配置 schemaJson
```

**文件名 = token + 类型后缀**，app-config.json 中引用的 token 必须和文件名一一对应：
- FormEntity token `passenger-info` → 文件 `passenger-info-form.json` + `passenger-info-fields.json`
- Page token `pnr-search` → 文件 `pnr-search-page.json`
- app-config.json 中的路由：`/pnr-management/page/pnr-search` 对应 `pnr-search-page.json`
- app-config.json 中的路由：`/pnr-management/form/passenger-info/layout/new` 对应 `passenger-info-form.json`

菜单和路由配置统一在项目级的 app-config.json 中生成（由 ceta-app-config SKILL 处理）。

### 单页面级拆分

直接判断是表单还是列表页，跳过项目/PBC 层级的拆分：

```
输入 HTML
│
├── 包含 <form> 或输入字段？
│   └── 是 → 表单，调度到 ceta-form SKILL
│
├── 包含 <table> 或数据列表？
│   └── 是 → 列表页，调度到 ceta-page SKILL
│
└── 两者都有？
    └── 混合页面，分别调度两个 SKILL
```

#### 单页面级输出结构

```
output/
└── {业务名称}/
    ├── analysis.json              # 分析结果（scope: "page"）
    ├── {业务名称}-form.json       # 表单布局 schemaJson（如有）
    ├── {业务名称}-fields.json     # 字段定义（如有）
    └── {业务名称}-page.json       # 列表页 schemaJson（如有）
```

---

## 输出目录硬性规范

**所有生成物必须写入 `output/` 目录，不允许写到其他位置。**

### 目录结构总览（按范围级别）

| 范围 | 输出根目录 | 说明 |
|------|-----------|------|
| 项目级 | `output/{project-token}/` | 含 analysis.json + frontend-config + pbcs 子目录 |
| PBC 级 | `output/{pbc-token}/` | 含 analysis.json + form/page/fields json |
| 单页面级 | `output/{业务名称}/` | 含 analysis.json + form 或 page json |

---

## 两阶段执行流程（核心机制）

**本 SKILL 采用「先分析确认，再生成」的两阶段流程，不允许跳过确认直接生成。**

```
阶段一：分析 → 输出 analysis.json → 展示给用户 → 等待确认
                                                    │
                              用户说"确认"/"可以"/"没问题" ──→ 阶段二
                              用户说"改一下 XX" ──→ 修改 analysis.json → 重新确认
                              用户说"不要了" ──→ 结束
                                                    │
阶段二：按 analysis.json 蓝图 → 调度子 SKILL → 逐个生成文件到 output/
```

### 阶段一：分析并等待确认

完成第一步（范围判定）和第二步（结构拆分）后：

1. **生成 analysis.json** 到 `output/{根目录}/analysis.json`
2. **向用户展示分析摘要**，格式如下：

```
📋 HTML 分析结果：

范围：PBC 级（业务组件）
业务名称：user-management（用户管理）

📦 将创建 1 个 PBC：
  └── user-management（用户管理）

📝 将创建 2 个表单实体（FormEntity）— 字段存数据库：
  1. system-user（系统用户）— 5 个字段，布局：new / edit / view
     ├── userName（用户名）- TEXT_BOX，必填
     ├── email（邮箱）- TEXT_BOX，必填
     ├── phone（电话）- TEXT_BOX
     ├── role（角色）- SELECT
     └── enabled（是否启用）- SWITCH

  2. user-role（用户角色）— 3 个字段，布局：new / edit / view
     ├── roleName（角色名）- TEXT_BOX，必填
     ├── roleCode（角色编码）- TEXT_BOX，必填
     └── description（描述）- TEXTAREA

📄 将创建 3 个页面（Page）— 纯 UI 交互，字段不存数据库：
  1. user-list（用户列表）— 列表页，关联表单 system-user
  2. role-list（角色列表）— 列表页，关联表单 user-role
  3. user-search（用户搜索）— 搜索页，搜索条件仅用于查询

📁 将生成以下文件到 output/：
  output/user-management/
  ├── analysis.json
  ├── system-user-form.json          ← FormEntity: system-user 的布局
  ├── system-user-fields.json        ← FormEntity: system-user 的字段定义
  ├── user-role-form.json            ← FormEntity: user-role 的布局
  ├── user-role-fields.json          ← FormEntity: user-role 的字段定义
  ├── user-list-page.json            ← Page: 用户列表
  ├── role-list-page.json            ← Page: 角色列表
  ├── user-search-page.json          ← Page: 用户搜索
  └── user-management-app-config.json（含菜单、主题、路由）

⚠️ 注意：app-config.json 中的路由 token 和上面的文件名一一对应：
  - /user-management/form/system-user/layout/new → system-user-form.json
  - /user-management/page/user-list → user-list-page.json

请确认以上分析是否正确？如需调整请告诉我。
```

3. **停下来等待用户回复**，不做任何文件生成（analysis.json 除外）

### 用户确认的几种情况

| 用户回复 | 处理方式 |
|---------|---------|
| "确认" / "可以" / "没问题" / "OK" / "生成吧" | 进入阶段二，开始生成 |
| "XX 字段不要了" / "加一个 YY 字段" / "表单名改成 ZZ" | 修改 analysis.json，重新展示摘要，再次等待确认 |
| "这个表单不需要" / "去掉 XX 页面" | 从 analysis.json 中移除，重新展示，再次等待确认 |
| "不要了" / "取消" | 结束流程，不生成任何文件 |

### 阶段二：按蓝图生成

用户确认后，按 analysis.json 中的 `outputFiles` 列表，自顶向下调度子 SKILL 生成文件：

```
确认后的执行顺序
│
├── Step 1: 全局配置（如项目级）
│   ├── 加载 ceta-app-config SKILL
│   └── 生成完整 app-config.json（含菜单、主题、路由）
│
├── Step 2: 逐个 PBC 处理
│   │
│   ├── Step 2.1: 表单实体（FormEntity）
│   │   ├── 加载 ceta-form SKILL
│   │   ├── 对 analysis.json 中每个 formEntity：
│   │   │   ├── 生成 {entity-token}-fields.json（字段定义）
│   │   │   └── 生成 {entity-token}-form.json（表单布局 schemaJson）
│   │   └── 逐个处理，不跳过
│   │
│   ├── Step 2.2: 页面（FormEntityPage）
│   │   ├── 加载 ceta-page SKILL
│   │   ├── 对 analysis.json 中每个 page：
│   │   │   └── 生成 {page-token}-page.json（列表页 schemaJson）
│   │   └── 关联到对应的 FormEntity
│   │
│   ├── Step 2.3: 应用配置（菜单+主题+路由）
│   │   ├── 加载 ceta-app-config SKILL
│   │   └── 根据已生成的 form/page 自动创建菜单、主题、路由配置
│   │
│   └── Step 2.4: 流程（如有线索）
│       └── 提示用户可进一步配置流程
│
└── Step 3: 汇总报告
    └── 告知用户：生成了 X 个 form、Y 个 page，文件列表
```

### 调度规则

| 识别到的内容 | 调度目标 | 传递信息 |
|-------------|---------|---------|
| 表单/输入字段 | `skills/ceta/ceta-form/SKILL.md` | analysis.json 中的字段清单 + HTML 片段 + 布局结构 |
| 数据表格/列表 | `skills/ceta/ceta-page/SKILL.md` | analysis.json 中的列定义 + HTML 片段 + 关联 FormEntity |
| 全局样式/主题 | `skills/ceta/ceta-app-config/SKILL.md` | CSS 变量 + 颜色 + 字体 |
| 菜单/导航 | `skills/ceta/ceta-app-config/SKILL.md` | 菜单项列表 + 路由映射 |
| 审批/流程按钮 | 提示用户 | 流程线索描述 |

### 样式保留要求（重要）

**生成 form.json 和 page.json 时，必须保留 HTML 中的视觉样式。**

CETA 的 schemaJson 支持通过 `style` 和 `componentProps.style` 配置内联样式，
通过 `className` 和 `componentProps.className` 引用全局 CSS class。
参考 `skills/references/style-mapping-rules.md` 和 `templates/css/` 下的示例。

#### 样式在 JSON 中的位置

| 位置 | 作用 | 适用场景 |
|------|------|---------|
| `style` | 组件外层容器样式（margin、外层尺寸） | 所有组件 |
| `componentProps.style` | 组件内部样式（padding、背景色、边框等） | 所有组件 |
| `className` | 引用全局 CSS class（伪类、hover、复用样式） | 需要 :hover 等效果时 |

#### 必须提取的样式

从 HTML 中提取以下样式并写入 JSON：

1. 背景色、渐变 → `componentProps.style.backgroundColor` 或 `componentProps.style.background`
2. 边框、圆角 → `componentProps.style.border`、`componentProps.style.borderRadius`
3. 内边距、外边距 → `componentProps.style.padding`、`style.margin`
4. 字体大小、粗细、颜色 → `style.fontSize`、`style.fontWeight`、`style.color`
5. 阴影 → `componentProps.style.boxShadow`
6. 宽高 → `style.width`、`style.height`
7. 布局属性 → Stack 的 `componentProps.flexDirection`、`componentProps.gap` 等

#### 示例：带样式的搜索区域

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "gap": 12
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": "PNR 搜索" },
      "style": {
        "fontWeight": "bolder",
        "fontSize": 20,
        "paddingLeft": 20,
        "borderLeft": "4px solid #003367"
      }
    },
    {
      "component": "Stack",
      "componentProps": { "flexDirection": "row" },
      "fields": [
        {
          "component": "Input",
          "id": "rloc",
          "title": "参考编号",
          "componentProps": {
            "style": { "background": "white" }
          }
        },
        {
          "component": "Button",
          "componentProps": {
            "content": "搜索",
            "icon": "material:search-two-tone"
          },
          "style": {
            "background": "#003367",
            "color": "white",
            "fontWeight": "bolder"
          }
        }
      ]
    }
  ],
  "style": {
    "background": "#f0f6ff",
    "padding": 15,
    "borderRadius": 5,
    "border": "1px solid #dbe9fe"
  }
}
```

#### 不要生成"裸"JSON

以下是错误示范（缺少样式）：
```json
{
  "component": "Input",
  "id": "rloc",
  "title": "参考编号"
}
```

正确做法是保留 HTML 中的视觉样式：
```json
{
  "component": "Input",
  "id": "rloc",
  "title": "参考编号",
  "componentProps": {
    "style": { "background": "white" }
  },
  "style": { "width": "50%" }
}
```


---

## 输出 analysis.json

analysis.json 在阶段一生成，是整个流程的蓝图。

### 关键要求

- `analysis.json` 必须包含 `summary` 统计字段，明确列出将要生成多少个 form、多少个 page
- 每个 formEntity 必须列出完整的字段清单（token、label、type、required）
- `status` 字段标记当前状态：`"pending_confirmation"` 或 `"confirmed"`
- 每个 page 必须标明关联的 formEntity 和列定义
- `outputFiles` 数组必须列出所有将要生成的文件路径（相对于 output/）

### analysis.json 完整结构

```json
{
  "status": "pending_confirmation",
  "scope": "project | pbc | page",
  "projectToken": "erp-system",
  "projectName": "ERP 系统",
  "summary": {
    "totalPbcs": 1,
    "totalFormEntities": 2,
    "totalFormEntityFields": 8,
    "totalPages": 3,
    "hasFlowHints": true,
    "note": "FormEntity = 数据存库的表单；Page = 纯 UI 交互页面（搜索、列表、仪表盘等）"
  },
  "pbcs": [
    {
      "pbcToken": "user-management",
      "pbcName": "用户管理",
      "formEntities": [
        {
          "entityToken": "system-user",
          "entityName": "系统用户",
          "reason": "用户数据需要持久化存储到数据库",
          "fieldCount": 5,
          "fields": [
            { "token": "userName", "label": "用户名", "type": "TEXT_BOX", "required": true },
            { "token": "email", "label": "邮箱", "type": "TEXT_BOX", "required": true },
            { "token": "phone", "label": "电话", "type": "TEXT_BOX", "required": false },
            { "token": "role", "label": "角色", "type": "SELECT", "required": false },
            { "token": "enabled", "label": "是否启用", "type": "SWITCH", "required": false }
          ],
          "layouts": ["new", "edit", "view"],
          "outputFiles": {
            "form": "system-user-form.json",
            "fields": "system-user-fields.json"
          },
          "sourceHtmlHint": "新建/编辑用户的 <form> 区域"
        },
        {
          "entityToken": "user-role",
          "entityName": "用户角色",
          "reason": "角色数据需要持久化存储到数据库",
          "fieldCount": 3,
          "fields": [
            { "token": "roleName", "label": "角色名", "type": "TEXT_BOX", "required": true },
            { "token": "roleCode", "label": "角色编码", "type": "TEXT_BOX", "required": true },
            { "token": "description", "label": "描述", "type": "TEXTAREA", "required": false }
          ],
          "layouts": ["new", "edit", "view"],
          "outputFiles": {
            "form": "user-role-form.json",
            "fields": "user-role-fields.json"
          },
          "sourceHtmlHint": "角色管理的 <form> 区域"
        }
      ],
      "pages": [
        {
          "pageToken": "user-list",
          "pageName": "用户列表",
          "type": "list",
          "reason": "列表展示页面，字段不存数据库",
          "relatedEntity": "system-user",
          "columns": ["userName", "email", "phone", "role", "enabled"],
          "outputFile": "user-list-page.json",
          "sourceHtmlHint": "<table> 区域"
        },
        {
          "pageToken": "role-list",
          "pageName": "角色列表",
          "type": "list",
          "reason": "列表展示页面，字段不存数据库",
          "relatedEntity": "user-role",
          "columns": ["roleName", "roleCode", "description"],
          "outputFile": "role-list-page.json",
          "sourceHtmlHint": "角色 <table> 区域"
        },
        {
          "pageToken": "user-search",
          "pageName": "用户搜索",
          "type": "search",
          "reason": "搜索条件仅用于查询，不存数据库",
          "relatedEntity": "system-user",
          "searchFields": ["userName", "email", "role"],
          "outputFile": "user-search-page.json",
          "sourceHtmlHint": "搜索/筛选区域"
        }
      ],
      "flowHints": [
        "检测到'提交'和'审批'按钮，可能需要审批流程"
      ]
    }
  ],
  "globalConfig": {
    "hasNavbar": true,
    "hasSidebar": true,
    "themeHints": {
      "primaryColor": "#1890ff",
      "layout": "sidebar"
    }
  },
  "outputFiles": [
    "erp-system/analysis.json",
    "erp-system/erp-system-app-config.json",
    "erp-system/pbcs/user-management/system-user-form.json",
    "erp-system/pbcs/user-management/system-user-fields.json",
    "erp-system/pbcs/user-management/user-role-form.json",
    "erp-system/pbcs/user-management/user-role-fields.json",
    "erp-system/pbcs/user-management/user-list-page.json",
    "erp-system/pbcs/user-management/role-list-page.json",
    "erp-system/pbcs/user-management/user-search-page.json"
  ]
}
```

### outputFiles 与 app-config.json 的 token 对应规则

**文件名 = token + 类型后缀**，app-config.json 中的路由必须和文件名一一对应：

| 类型 | token 示例 | 输出文件名 | app-config 路由 |
|------|-----------|-----------|----------------|
| FormEntity | `system-user` | `system-user-form.json` + `system-user-fields.json` | `/user-management/form/system-user/layout/new` |
| Page（列表） | `user-list` | `user-list-page.json` | `/user-management/page/user-list` |
| Page（搜索） | `user-search` | `user-search-page.json` | `/user-management/page/user-search` |

每个 formEntity 和 page 内部都有 `outputFile` / `outputFiles` 字段，明确标注对应的文件名，确保不会出现 token 和文件名不匹配的情况。

---

## HTML 元素 → CETA 概念映射速查

### 导航与布局

| HTML 元素/结构 | CETA 概念 | 层级 |
|---------------|----------|------|
| `<nav>` + 多个链接 | 菜单配置（config） | Project / PBC |
| `<aside>` 侧边栏菜单 | 路由配置（routesJson） | Project |
| `<header>` 应用头部 | frontEndConfig.appConfig | Project |
| Tab/标签页切换 | 同一 PBC 下的多个页面 | PBC |
| 面包屑导航 | 路由层级关系 | Project / PBC |

### 表单相关

| HTML 元素/结构 | CETA 概念 | 层级 | 判断关键 |
|---------------|----------|------|---------|
| `<form>` + 提交/保存按钮 | FormEntity + Layout | PBC → FormEntity | 数据需要存库 |
| `<form>` + 搜索/查询按钮 | Page（搜索页） | PBC → Page | 搜索条件不存库 |
| `<input>` / `<select>` / `<textarea>` 在存库表单中 | Field（字段） | FormEntity | 和数据库列映射 |
| `<input>` / `<select>` 在搜索区域中 | Page 的 UI 组件 | Page | 仅用于查询交互 |
| `<fieldset>` / 卡片分区 | Card 布局组件 | Layout | — |
| 多列排列的字段 | Grid 布局组件 | Layout | — |
| 折叠面板 | Collapse 布局组件 | Layout | — |
| 同一数据的新建/编辑/查看视图 | 同一 FormEntity 的不同 Layout | FormEntity | 共享同一套 fields |

### 列表/页面相关

| HTML 元素/结构 | CETA 概念 | 层级 |
|---------------|----------|------|
| `<table>` | FormEntityPage（列表页） | PBC → Page |
| 表头 `<th>` | Table 列定义 | Page |
| 操作列（编辑/删除按钮） | ACTION_COLUMN | Page |
| 搜索/筛选区域 | Table 的 filter 配置 | Page |
| 分页器 | Table 的 pagination 配置 | Page |
| 统计卡片/图表 | Dashboard portlet | Page |

---

## 特殊情况处理

### 1. CETA 原生 HTML

如果输入的 HTML 包含 CETA 专有 class（如 `form-renderer`、`card-layout`、`icp-ag-table`），
说明这是从 CETA 平台导出的 HTML，可以更精确地映射：

- `form-renderer` → 整个表单的 Layout
- `card-layout` → Card 组件
- `grid-layout` → Grid 组件
- `collapse-element` → Collapse 组件
- `icp-ag-table` → Table 组件（列表页）
- `input-element` → Input 字段
- `select-element` → Select 字段

此时跳过范围判定，直接按 CETA 组件结构还原。

### 2. 多文件 HTML

如果用户提供多个 HTML 文件：
- 每个文件视为一个独立页面
- 如果文件名暗示同一业务域（如 `user-list.html` + `user-form.html`），归入同一 PBC
- 如果文件名暗示不同业务域，归入不同 PBC

### 3. 不完整 HTML

如果 HTML 只是片段（没有 `<html>`、`<body>` 标签）：
- 直接分析内容区域
- 默认为单页面级

### 4. 含有业务逻辑的 HTML

如果 HTML 中包含 JavaScript 逻辑（如表单验证、条件显示）：
- 提取验证规则 → 映射到 field 的 validation 配置
- 提取条件显示逻辑 → 记录在 analysis.json 中供后续配置
- 不尝试完整还原 JS 逻辑

---

## 与 ceta-basic 的关系

```
用户输入 HTML
      │
      ▼
ceta-basic（入口 SKILL）
      │ 识别输入类型为 HTML
      │ 判断需要结构分析
      ▼
ceta-html-analyzer（本 SKILL）
      │ 分析范围、拆分结构
      │ 生成 analysis.json
      ▼
按分析结果调度子 SKILL
      ├── ceta-form（表单）
      ├── ceta-page（列表页）
      ├── ceta-app-config（菜单+主题+路由）
      └── ceta-flow（流程，提示用户）
```

### 何时加载本 SKILL

在 `ceta-basic` 的判断规则基础上，增加以下条件：

**需要加载 html-analyzer** — 满足任一条件：
- HTML 内容较复杂，包含多个独立的表单或表格
- HTML 包含导航/菜单结构
- HTML 看起来是一个完整应用或模块的原型
- 用户明确说"分析这个 HTML"或"把这个 HTML 转成 CETA 配置"

**不需要加载 html-analyzer**（直接用 ceta-form 或 ceta-page）：
- HTML 明显只是一个简单表单
- HTML 明显只是一个简单列表页
- 用户明确说"生成表单 JSON"或"生成列表页 JSON"