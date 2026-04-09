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

## 文档加载规则

分析 HTML 时需要加载以下文档：

- `ceta-basic`（入口 SKILL，了解平台数据模型和字段类型映射）
- `skills/ceta/references/html-mapping-rules.md`（HTML 元素 → CETA 组件类型的映射）
- `skills/ceta/references/schema-rules.md`（schemaJson 结构规范）
- `skills/ceta/references/style-mapping-rules.md`（样式映射）
- `skills/ceta/references/tailwind-mapping-rules.md`（Tailwind CSS 强制解析流程 — **检测到 Tailwind 时必须加载**）
- 子 SKILL（ceta-form、ceta-page、ceta-app-config）

**必须加载的组件文档（每次都读）：**
- `skills/ceta/references/components/layout/` 下所有文件 — 理解每个布局组件的能力和适用场景
- `skills/ceta/references/components/display/` 下所有文件 — 理解每个展示组件的能力和适用场景
- `skills/ceta/references/components/input/` 下所有文件 — 理解每个输入组件的能力

**为什么必须全部加载？** 因为你需要了解每个组件的特性才能正确选择。不了解 Box 能做渐变色块，就会把图标降级为纯文本；不了解 Text 能做 Badge，就会丢失状态标签。

**可选参考的映射示例：**
- `templates/css/` — 真实项目的 HTML→JSON 映射示例，可作为参考但不要照搬
- `templates/htmlToJson/` — HTML 到 JSON 的完整转换示例

---

## 页面组件拆解方法论（核心 — 生成 JSON 前必须执行）

**这是整个转换流程中最重要的步骤。** 在写任何 JSON 之前，必须先完成组件拆解分析。

### 最高原则：忠实还原 HTML（Faithful Reproduction）

**HTML 写了什么，就生成什么。不要自作主张做"全局主题一致性"优化。**

这是整个转换流程中优先级最高的规则，凌驾于所有其他规则之上：

1. **不要统一配色**：HTML 中某个区块用了绿色渐变，就生成绿色渐变，不要因为"项目主题是深蓝色"就改成深蓝色
2. **不要合并元素**：HTML 中 NH6 和 Y 是两个独立的 `<span>`（不同背景色），就生成两个独立的 Text 组件，不要合并成一个 "NH6 Y" 文本
3. **不要省略元素**：HTML 中有删除按钮，就生成 Button 组件，不要因为"这是只读摘要区域"就省略
4. **不要过度设计**：HTML 中数字 "1" 只是一个粗体文字，就生成 Text 组件，不要自作主张改成圆形徽章
5. **Tailwind/Bootstrap class 必须精确解析**：`bg-emerald-500` 就是 `#10b981`，不要替换成项目主色 `#003366`

**违反此原则的典型错误：**

| 错误行为 | 原因 | 正确做法 |
|---------|------|---------|
| 把绿色渐变背景改成白色 | "统一主题" | 保留 `linear-gradient(to right, #ecfdf5, #f0fdfa)` |
| 把两个不同颜色的 badge 合并成一个文本 | "简化结构" | 保留两个独立的 Text 组件各自带背景色 |
| 省略 HTML 中的按钮/交互元素 | "这里不需要" | 保留所有 HTML 中存在的交互元素 |
| 把普通文字改成圆形徽章 | "看起来更好" | 按 HTML 原文生成，不做视觉升级 |
| 用项目主色替换 Tailwind 解析出的颜色 | "配色一致" | 用 Tailwind class 解析出的精确颜色值 |

### 三步法：拆解 → 选组件 → 加样式

```
Step A: 组件拆解 — 分析页面由哪些独立的视觉区块组成
Step B: 选择组件 — 根据每个区块的特性选择合适的 CETA 组件
Step C: 加样式   — 把 HTML/CSS/图片中的样式映射到对应组件的 style 上
```

**严格按顺序执行，不要跳步。** 先把结构搞对，再加样式。

### Step A: 组件拆解

对页面中的每个 HTML 元素，问自己：**这个元素在视觉上是一个独立的"东西"吗？**

- 如果是 → 它应该是一个独立的组件节点
- 如果不是（只是包裹层） → 它可能是一个布局组件（Stack/Grid/Box）

#### 拆解规则

1. **每个可见的文本片段** → 一个 `Text` 组件（不要把多段文本合并成一个 content 字符串）
2. **每个输入控件**（input/select/textarea 等） → 对应的输入组件（Input/Select/DatePicker 等）
3. **每个按钮** → 一个 `Button` 组件
4. **每个图片** → 一个 `Image` 组件
5. **每个有背景色/边框/圆角的装饰性容器** → 一个 `Box` 或 `Card` 组件
6. **每个包含子元素的容器** → 一个布局组件（Stack/Grid/Card）

#### 关键原则：一个 HTML 元素 = 一个组件节点

**HTML 中的每个可见元素（`<span>`、`<div>`、`<button>`、`<input>` 等）都必须对应一个独立的 CETA 组件节点。**

**禁止合并**：如果 HTML 中有两个独立的 `<span>`（即使文本内容相关），就必须生成两个独立的 Text 组件。
**禁止省略**：如果 HTML 中有一个 `<button>`，就必须生成一个 Button 组件，不能因为"不需要"而省略。
**禁止替换**：如果 HTML 中是一个普通 `<span>` 文字，就生成 Text 组件，不能自作主张改成 Box + Text 圆形徽章。

判断方法：**看 HTML 源码中有几个独立的标签，就生成几个组件。** 不要根据"业务语义"或"视觉效果"做合并/省略/替换。

错误示范：
```
HTML: <span class="bg-emerald-500 text-white">NH6</span><span class="bg-slate-200">Y</span>
错误: { "component": "Text", "componentProps": { "content": "NH6 Y" } }  ← 合并了两个 span
正确: 两个独立的 Text 组件，各自带不同的背景色
```

### Step B: 选择组件

拆解完成后，确定每个区块用什么 CETA 组件。选择依据是**组件的特性**，不是具体的业务场景。

#### 布局组件选择规则

| 子元素的排列关系 | 选择的组件 | 关键属性 |
|----------------|-----------|---------|
| 纵向堆叠（上下排列） | `Stack` | `flexDirection: "column"` |
| 横向排列（左右并排） | `Stack` | `flexDirection: "row"` |
| 多列等宽/按比例排列 | `Grid` | `colNumber` + 子项 `span` |
| 有标题的分组区域 | `Card` | `title` |
| 纯装饰性容器（背景色块、圆角方块、圆形徽章） | `Box` | 无特殊属性，靠 `style` 实现视觉效果 |
| 可折叠的分组 | `Collapse` | `items[].fields` |
| 可切换的标签页 | `Tabs` | `items[].fields` |

#### 如何判断用 Stack 还是 Grid？

- **Stack**：子元素数量不固定，或者子元素宽度不需要精确控制。用 `gap` 控制间距。
- **Grid**：子元素需要按固定列数排列，或者需要精确控制宽度比例。用 `colNumber` + `span`。

常见场景：
- 表单字段两列排列 → `Grid colNumber: 2`
- 页面左右分栏（如 8:4 比例） → `Grid colNumber: 12` + `span: 8` / `span: 4`
- 按钮组横向排列 → `Stack flexDirection: "row"`
- 信息卡片内的多行内容 → `Stack flexDirection: "column"`

#### 展示组件选择规则

| 视觉元素的特征 | 选择的组件 | 说明 |
|--------------|-----------|------|
| 纯文本内容 | `Text` | 通过 `style` 控制字号/颜色/粗细 |
| 带背景色的文本标签（Badge） | `Text` | 通过 `style` 加 background + padding + borderRadius |
| 数据表格 | `Table` | 需要 columnDefs |
| 步骤指示器 | `Steps` | 需要 current + items |
| 按钮 | `Button` | 需要 action |

#### Box 的特殊用途

`Box` 是最灵活的容器，当其他组件都不合适时用 Box。典型场景：
- 渐变色背景块（如 logo 图标容器）：`Box` + `style.background: "linear-gradient(...)"`
- 圆形数字徽章：`Box` + `style.borderRadius: "50%"` + `style.width/height`
- 提示信息框：`Box` + `style.background` + `style.border`
- 任何需要特殊视觉效果但不需要 Card 标题的容器

### Step C: 加样式

组件结构确定后，把 HTML/CSS/图片中的样式映射到对应组件上。

#### 样式放在哪里？

这是最容易出错的地方。不同组件的样式位置不同：

| 组件类型 | 视觉样式放哪里 | 布局属性放哪里 |
|---------|--------------|--------------|
| Stack / Grid / Box | 外层 `style` | `componentProps` 顶层（flexDirection/gap/colNumber 等） |
| Card | `componentProps.style`（卡片整体）+ `componentProps.titleTextStyle` 等 | `componentProps` |
| Text / Button | 外层 `style` | 无 |
| 输入组件 | `componentProps.style`（控件本身）+ `style`（外层尺寸） | 无 |

详细规则见各组件的 `.md` 文档和 `style-mapping-rules.md`。

#### 样式提取顺序

1. 先从 HTML 的 Tailwind/Bootstrap class 精确解析（类型 E）
2. 再从 HTML 的 `<style>` 标签或 inline style 提取（类型 A/B/D）
3. 如果 HTML 无 CSS 也无框架 class 但有设计稿图片，从图片还原
4. 如果都没有，根据 class 名语义推断或询问用户

**样式提取的核心原则：忠实还原，不做替换。**
- Tailwind `bg-emerald-500` 解析为 `#10b981`，就写 `#10b981`，不要替换成项目主色
- HTML 中是绿色渐变，就写绿色渐变，不要改成蓝色渐变
- HTML 中某个元素没有特殊样式（只是普通文字），就不要自作主张加圆形背景、渐变色等装饰

---

## 多输入源处理（重要 — 必须在 Step 0 之前判断）

用户可能同时提供多种输入源。**必须在开始分析之前识别所有输入源，并确定各自的角色。**

### 输入源类型与优先级

| 输入源 | 角色 | 优先级 |
|--------|------|--------|
| 设计稿图片/截图 | **视觉真相来源**（Visual Truth） | 最高 — 颜色、布局、间距、字体以图片为准 |
| HTML + CSS（含 `<style>` 或 inline style） | 结构 + 样式来源 | 高 — 结构和样式都可信 |
| HTML（无 CSS） | **仅结构来源** | 中 — 只信任 HTML 的 DOM 结构和文本内容，不信任其视觉表现 |
| 自然语言描述 | 补充说明 | 低 — 用于消歧和补充 |

### 判断流程

```
拿到用户输入
│
├── 1. 检查是否有图片附件（设计稿/截图）
│   ├── 有 → 标记为「有视觉真相」，图片是样式的最终权威
│   └── 无 → 样式只能从 HTML/CSS 提取或推断
│
├── 2. 检查 HTML 是否有 CSS
│   ├── 有 <style> 或 inline style → 样式可信
│   └── 无 CSS → 标记为「结构-only HTML」
│
└── 3. 确定生成策略
    ├── 有图片 + 有 CSS 的 HTML → 以图片为视觉基准，HTML CSS 作为参考
    ├── 有图片 + 无 CSS 的 HTML → 结构从 HTML 提取，样式从图片还原（最常见场景）
    ├── 无图片 + 有 CSS 的 HTML → 结构和样式都从 HTML 提取
    └── 无图片 + 无 CSS 的 HTML → 结构从 HTML 提取，样式必须询问用户或根据语义推断
```

### 「有图片 + 无 CSS 的 HTML」场景的处理规则（最常见）

**这是最容易出错的场景。** HTML 只提供了 DOM 结构（哪些元素、嵌套关系、文本内容），
但没有任何视觉信息。如果不看图片，AI 会生成灰白色的"安全"默认样式，与设计稿严重不符。

#### 必须遵循的规则

1. **结构从 HTML 提取**：DOM 层级、元素类型、文本内容、输入字段、按钮等
2. **样式从图片还原**：颜色、渐变、圆角、阴影、间距、字体大小/粗细、布局比例等
3. **逐区块对照**：将 HTML 的每个主要区块（Card、Section、Form 等）与图片中对应的视觉区域一一对照
4. **不要用默认灰白色**：如果图片中某个区域有明确的颜色（如绿色渐变、蓝色背景），必须还原到 JSON 中
5. **布局结构以图片为准**：如果 HTML 结构暗示单列但图片显示两栏，以图片的两栏布局为准

#### 图片样式提取清单

对图片中的每个视觉区域，必须提取以下信息：

| 视觉属性 | 提取方法 | 写入位置 |
|---------|---------|---------|
| 背景色/渐变 | 从图片中识别颜色值 | `style.background` 或 `componentProps.style.backgroundColor` |
| 文字颜色 | 从图片中识别 | `style.color` |
| 文字大小/粗细 | 从图片中估算 | `style.fontSize` / `style.fontWeight` |
| 圆角 | 从图片中估算 | `style.borderRadius` |
| 阴影 | 从图片中判断 | `style.boxShadow` |
| 边框 | 从图片中判断 | `style.border` |
| 间距（padding/margin） | 从图片中估算 | `style.padding` / `style.margin` |
| 布局方向（横/纵） | 从图片中判断 | Stack 的 `flexDirection` 或 Grid 的 `colNumber` |
| 元素宽度比例 | 从图片中估算 | Grid 的 `span` 或 `style.width` |
| 状态标签/Badge | 从图片中识别颜色和文字 | Text 组件 + 对应样式 |

#### 常见错误与纠正

| 错误 | 原因 | 纠正 |
|------|------|------|
| 所有卡片都是白色背景 + 灰色边框 | 没看图片，用了默认样式 | 从图片提取实际背景色（可能是渐变、彩色） |
| 复杂两栏布局变成单列 | 只看了 HTML 的线性结构 | 从图片确认实际布局是几栏 |
| 按钮全是深蓝色 | 用了统一的 primary 色 | 从图片区分不同按钮的颜色（绿色确认、红色取消等） |
| 状态标签只是纯文本 | 没有还原 Badge 样式 | 从图片提取 Badge 的背景色、圆角、padding |
| 编号圆圈变成纯数字 | 没有还原圆形背景样式 | 从图片提取圆形背景色、尺寸 |

---

## 第一步：范围判定

拿到 HTML 后，首先判断它描述的是哪个层级的内容。

### 判定规则

| 范围级别                  | 判定条件                                                                                                                                      | 示例                                           |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **项目级（Project）**     | HTML 包含导航栏/侧边栏菜单 + 多个独立业务区域；或包含多个 `<nav>`、多组不相关的表单/表格；或有明显的应用壳（header/sidebar/content 三栏布局） | 完整的 ERP 系统原型、带侧边栏的管理后台        |
| **PBC 级（业务组件）**    | HTML 包含同一业务域下的多个相关页面（如列表页+表单页）；或有 Tab 切换的多视图；或包含 2+ 个相关但独立的表单/表格                              | 用户管理模块（用户列表+新建用户表单+角色管理） |
| **单页面级（Page/Form）** | HTML 只包含一个表单、一个列表页、或一个仪表盘；结构单一，无导航                                                                               | 一个请假申请表单、一个订单列表页               |

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
ceta-workspace/
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

| 概念                   | 本质                                              | 字段是否存数据库            | 典型场景                                            |
| ---------------------- | ------------------------------------------------- | --------------------------- | --------------------------------------------------- |
| FormEntity（表单实体） | 数据模型，定义字段，和数据库表一一映射            | ✅ 是，每个字段对应数据库列 | 新建订单、编辑用户信息、填写申请表                  |
| FormEntity Layout      | 同一套数据（同一个 FormEntity）的不同 UI 表现形式 | 共享同一套字段              | new（新建视图）、edit（编辑视图）、view（只读视图） |
| FormEntityPage（页面） | 纯 UI 交互页面，字段不存数据库                    | ❌ 否，仅用于用户交互       | 搜索页、筛选页、仪表盘、列表页、首页                |

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
│   │   └── 输出：{entity-token}-form.json（布局 schemaJson）
│   │
│   └── 否 → Page
│       ├── 字段仅用于用户交互（如搜索条件、筛选器）
│       ├── 有"搜索"/"查询"/"筛选"按钮，触发查询而非保存
│       ├── 数据表格/列表展示（Table 组件）
│       ├── 仪表盘/统计卡片
│       └── 输出：{page-token}-page.json（页面配置）
```

#### 常见误判场景

| 场景               | 错误判断      | 正确判断                     | 原因                                  |
| ------------------ | ------------- | ---------------------------- | ------------------------------------- |
| 搜索页面有输入框   | ❌ FormEntity | ✅ Page                      | 搜索条件不存数据库，只是查询参数      |
| 列表页上方有筛选区 | ❌ FormEntity | ✅ Page                      | 筛选条件是 UI 交互，不是数据模型      |
| 登录页有用户名密码 | ❌ FormEntity | ✅ Page                      | 登录凭证不是业务数据模型              |
| 新建订单表单       | ✅ FormEntity | ✅ FormEntity                | 订单数据需要存库                      |
| 编辑用户信息       | ✅ FormEntity | ✅ FormEntity                | 用户数据需要存库，编辑是另一个 Layout |
| 订单详情只读页     | ❌ Page       | ✅ FormEntity 的 view Layout | 展示的是同一套数据模型，只是只读视图  |

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
ceta-workspace/{project-token}/pbcs/{pbc-token}/
├── {entity-token}-form.json       # FormEntity 的表单布局 schemaJson
└── {page-token}-page.json         # Page 的页面配置 schemaJson
```

**文件名 = token + 类型后缀**，app-config.json 中引用的 token 必须和文件名一一对应：

- FormEntity token `passenger-info` → 文件 `passenger-info-form.json`
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
ceta-workspace/
└── {业务名称}/
    ├── analysis.json              # 分析结果（scope: "page"）
    ├── {业务名称}-form.json       # 表单布局 schemaJson（如有）
    └── {业务名称}-page.json       # 列表页 schemaJson（如有）
```

---

## 输出目录硬性规范

**所有生成物必须写入 `ceta-workspace/` 目录，不允许写到其他位置。**

### 目录结构总览（按范围级别）

| 范围     | 输出根目录                | 说明                                             |
| -------- | ------------------------- | ------------------------------------------------ |
| 项目级   | `ceta-workspace/{project-token}/` | 含 analysis.json + frontend-config + pbcs 子目录 |
| PBC 级   | `ceta-workspace/{pbc-token}/`     | 含 analysis.json + form/page/fields json         |
| 单页面级 | `ceta-workspace/{业务名称}/`      | 含 analysis.json + form 或 page json             |

---

## 执行流程（核心机制）

**本 SKILL 采用「一次生成，确认后导入」的流程。CSS 预处理、分析和生成同步进行，不分两个阶段。**

```
CSS 预处理 → 分析 HTML → 同时生成 analysis.json + 所有 JSON 文件 → 展示 analysis 给用户确认
                                                                      │
                                            用户说"确认" ──→ 进入导入平台流程（Step 3）
                                            用户说"改一下 XX" ──→ 修改对应的 JSON 文件 → 重新确认
                                            用户说"不要了" ──→ 结束
```

### 为什么不分两个阶段？

之前的流程是"阶段一只生成 analysis.json → 用户确认 → 阶段二才生成 JSON"。
问题是到阶段二时，HTML 原文的样式细节容易丢失（上下文过长被截断、样式信息不在 analysis.json 中）。

现在改为：分析 HTML 的同时就生成所有 JSON 文件（带完整样式），analysis.json 只是给用户看的索引/摘要。
用户确认的是"要不要把这些已经生成好的文件导入平台"，而不是"要不要开始生成"。

### 执行步骤

```
拿到 HTML
│
├── Step 0: CSS 预处理（在分析结构之前必须先完成）
│   │
│   │  **这一步是防止样式丢失的关键。必须在分析 HTML 结构之前完成。**
│   │
│   ├── 0.1 判断样式来源类型
│   │   ├── 类型 A：HTML 中有 <style> 标签 → 提取 CSS 规则，建立 class→样式映射表
│   │   ├── 类型 B：HTML 元素有 inline style 属性 → 直接可用，无需预处理
│   │   ├── 类型 C：HTML 只有 class 引用但无 <style> 标签也无 inline style
│   │   │   └── **这是最危险的类型，最容易生成"裸"JSON。必须严格处理：**
│   │   │       ├── 先执行类型 E 检测（是否为已知 CSS 框架）
│   │   │       ├── 如果不是已知框架，检查是否有设计稿图片：
│   │   │       │   ├── 有图片 → 从图片还原样式（见「多输入源处理」章节），不需要询问用户
│   │   │       │   └── 无图片 → **必须停下来询问用户**：
│   │   │       │       "HTML 中引用了 CSS class 但未包含样式定义，也没有设计稿图片。
│   │   │       │        能否提供对应的 CSS 文件或设计稿截图？
│   │   │       │        否则我将根据 class 名语义和 HTML 结构推断样式（可能不准确）。"
│   │   │       └── **禁止在类型 C 且无图片时直接生成默认灰白样式然后跳过。**
│   │   │           必须要么从图片还原、要么询问用户、要么明确标注样式是推断的。
│   │   ├── 类型 D：混合（既有 <style> 又有 inline style）→ 两者都提取，inline 优先级更高
│   │   └── 类型 E：已知 CSS 框架（Tailwind CSS / Bootstrap 等）
│   │       │
│   │       │  **这是最常见的情况。** 现代 HTML 原型大量使用 Tailwind CSS 等 utility-first 框架，
│   │       │  class 名本身就是样式定义，不需要额外的 CSS 文件。
│   │       │
│   │       ├── 检测方法：扫描 HTML 中的 class 属性，如果大量出现以下模式则判定为对应框架：
│   │       │   ├── Tailwind CSS：`bg-*`、`text-*`、`p-*`、`m-*`、`rounded-*`、`flex`、`grid`、
│   │       │   │   `w-*`、`h-*`、`gap-*`、`border-*`、`shadow-*`、`font-*`、`space-*` 等
│   │       │   └── Bootstrap：`btn-*`、`col-*`、`row`、`container`、`card`、`form-control` 等
│   │       │
│   │       ├── 处理方式：
│   │       │   ├── 将每个 HTML 元素的 Tailwind class 逐个解析为精确的 CSS 属性值
│   │       │   ├── Tailwind class → CSS 值的速查表见 `skills/ceta/references/tailwind-mapping-rules.md`
│   │       │   ├── 颜色值使用 Tailwind 官方色板精确值（AI 已知），不确定时标注 `_unsure` 而非猜测
│   │       │   ├── 解析结果在 Step 1.3「逐元素样式解析」中统一输出（与其他样式来源走同一流程）
│   │       │   ├── 转换后的样式写入 inlineStyleMap，和类型 A 的处理结果合并
│   │       │   └── Tailwind 的伪类变体（hover:*、focus:*、dark:*）→ 保留为 className，
│   │       │       写入 themeConfig.css（将 Tailwind class 转为等效的原生 CSS）
│   │       │
│   │       └── 注意：类型 E 可以和类型 A/B/D 共存（HTML 同时有 Tailwind class 和 <style> 标签）
│   │
│   ├── 0.2 提取 <style> 标签中的 CSS 规则（类型 A/D）
│   │   ├── 解析每条 CSS 规则，建立映射表：{ selector → { 属性: 值, ... } }
│   │   ├── 示例：.bg-blue { background: #003366; color: white; }
│   │   │   → cssMap[".bg-blue"] = { background: "#003366", color: "white" }
│   │   └── 保留伪类规则（:hover 等）和子元素选择器规则，标记为「需要 className」
│   │
│   ├── 0.3 对每条 CSS 规则做分类决策
│   │   │
│   │   │  对 cssMap 中的每条规则，判断应该转为 inline style 还是保留为 className：
│   │   │
│   │   ├── 转为 inline style 的条件（满足全部）：
│   │   │   ├── 选择器是简单 class（如 .card-bg），不含伪类、子元素选择器、组合选择器
│   │   │   ├── 该 class 在 HTML 中只被 1-2 个元素引用（不是大量复用）
│   │   │   └── CSS 属性都是 CETA style 支持的（background、padding、color 等）
│   │   │
│   │   ├── 保留为 className 的条件（满足任一）：
│   │   │   ├── 选择器包含伪类（:hover、:focus、:active、::before 等）
│   │   │   ├── 选择器包含子元素/后代选择器（如 .card .title）
│   │   │   ├── 选择器包含组合选择器（如 .a.b、.a > .b）
│   │   │   ├── 该 class 在 HTML 中被 3+ 个元素引用（复用样式）
│   │   │   ├── CSS 包含 @keyframes 动画
│   │   │   └── CSS 包含 CETA inline style 不支持的属性（如 content、::before）
│   │   │
│   │   └── 分类结果记录到两个列表：
│   │       ├── inlineStyleMap: { htmlElement → { 属性: 值 } }  — 后续写入组件的 style
│   │       └── classNameRules: [ { selector, css } ]  — 后续写入 themeConfig.css
│   │
│   └── 0.4 输出 CSS 预处理结果
│       ├── inlineStyleMap 在后续 Step 1 生成组件 JSON 时使用
│       ├── classNameRules 在生成 app-config.json 时写入 themeConfig.css
│       └── 如果有 classNameRules，在组件 JSON 中对应元素加上 className 属性
│
├── Step 1: 分析 + 生成（同步进行，使用 Step 0 的样式数据）
│   │
│   ├── 1.1 范围判定（项目级/PBC级/单页面级）
│   ├── 1.2 结构拆分（识别 FormEntity、Page、菜单、主题）
│   ├── 1.3 **逐元素样式解析（强制 — 不可跳过）**
│   │   │
│   │   │  **这是防止样式丢失/替换的关键步骤。不管样式来源是什么（Tailwind、<style>、inline、图片），
│   │   │  都必须在生成 JSON 之前，先对每个页面的关键 HTML 元素输出样式解析表。**
│   │   │
│   │   │  对每个页面/表单中的关键元素（有背景色、有文字色、有边框、有圆角的元素），
│   │   │  输出如下格式的解析结果：
│   │   │
│   │   │  ```
│   │   │  [pnr-search-page] 搜索结果卡片:
│   │   │    <div class="flex items-center justify-between p-3 bg-emerald-50 rounded-xl border border-emerald-100">
│   │   │    → Stack, style: { padding: 12, background: "#ecfdf5", borderRadius: 12, border: "1px solid #d1fae5" }
│   │   │
│   │   │    <span class="text-emerald-700 font-bold">1</span>
│   │   │    → Text "1", style: { color: "#047857", fontWeight: 700 }
│   │   │  ```
│   │   │
│   │   │  **规则：**
│   │   │  - Tailwind class → 查 Tailwind 官方色板精确值（见 `tailwind-mapping-rules.md` 速查表）
│   │   │  - <style> 标签 → 直接从 cssMap 中取值
│   │   │  - inline style → 直接提取
│   │   │  - 设计稿图片 → 从图片中识别颜色值
│   │   │  - **不确定的值标注 `_unsure`，不要猜**
│   │   │  - **解析表中写了什么值，后续 JSON 就必须写什么值，禁止在生成 JSON 时"优化"或"统一"颜色**
│   │   │
│   ├── 1.4 生成所有 JSON 文件到 ceta-workspace/（严格按 1.3 的解析结果填写样式）
│   │   ├── 加载 ceta-form SKILL → 生成每个 {entity}-form.json
│   │   ├── 加载 ceta-page SKILL → 生成每个 {page}-page.json
│   │   └── 加载 ceta-app-config SKILL → 生成 app-config.json
│   │       **将 classNameRules 写入 themeConfig.css**
│   └── 1.5 生成 analysis.json（索引/摘要，给用户确认用）
│
├── Step 2: 展示 analysis 给用户，等待确认
│   │
│   ├── 用户说"确认" → 进入 Step 3
│   ├── 用户说"改一下 XX" → 修改对应的 JSON 文件，重新确认
│   └── 用户说"不要了" → 结束
│
├── Step 3: 推送到 CETA 平台（用户确认后才执行）
│   │
│   │  **禁止逐个调用 MCP API 创建表单/页面。统一走 seed data 导入。**
│   │
│   ├── Step 3.1: 确认/创建 Project
│   │   └── 通过 MCP API form__project__get_by_token 检查，不存在则 form__project__create
│   │
│   ├── Step 3.2: 运行 assemble-pbc 拼装 pbc-seed-data.json
│   │   └── $ ceta_sync.sh assemble-pbc {project-token} {pbc-token}
│   │
│   ├── Step 3.3: 修正 pbc-seed-data.json
│   │   ├── assemble 只生成 default layout，需用 Python 脚本改为 new/edit/view 三个 layout
│   │   ├── 补充 routesJson（PBC 级路由配置）
│   │   └── 补充 config（PBC 级菜单配置）
│   │   └── 详细修正逻辑见 skills/ceta/ceta-api/SKILL.md 的「post-assemble 修正」章节
│   │
│   ├── Step 3.4: 运行 import-pbc 导入到平台
│   │   └── $ ceta_sync.sh import-pbc {project-token} {pbc-token}
│   │
│   ├── Step 3.5: 更新 frontEndConfig（菜单、路由、主题）
│   │   └── 通过 MCP API form__front_end_config__update 更新
│   │
│   └── Step 3.6: 验证导入结果
│       └── 通过 MCP API form__form_entity_page__list_by_pbc_id 确认页面 schemaJson 完整
│
└── Step 4: 汇总报告
    └── 告知用户：生成了 X 个 form、Y 个 page，已导入平台，附项目 UI 链接
```

### 展示给用户的确认摘要

所有文件生成完毕后，向用户展示 analysis.json 的摘要，格式如下：

```
📋 分析结果（所有 JSON 文件已生成到 ceta-workspace/）：

范围：PBC 级（业务组件）
业务名称：user-management（用户管理）

📝 表单实体（FormEntity）— 字段存数据库：
  1. system-user（系统用户）— 5 个字段
  2. user-role（用户角色）— 3 个字段

📄 页面（Page）— 纯 UI 交互：
  1. user-list（用户列表）
  2. role-list（角色列表）
  3. user-search（用户搜索）

📁 已生成文件：
  ceta-workspace/user-management/
  ├── analysis.json
  ├── system-user-form.json
  ├── user-role-form.json
  ├── user-list-page.json
  ├── role-list-page.json
  ├── user-search-page.json
  └── user-management-app-config.json

请检查生成的 JSON 文件，确认无误后我将导入到 CETA 平台。
如需调整请告诉我（如"XX 字段不要了"、"改一下 YY 页面的样式"）。
```

### 用户确认的几种情况

| 用户回复                                             | 处理方式                                        |
| ---------------------------------------------------- | ----------------------------------------------- |
| "确认" / "可以" / "没问题" / "OK" / "导入吧"         | 进入 Step 3，开始导入平台                       |
| "XX 字段不要了" / "加一个 YY 字段" / "表单名改成 ZZ" | 修改对应的 JSON 文件和 analysis.json，重新确认   |
| "这个表单不需要" / "去掉 XX 页面"                    | 删除对应文件，更新 analysis.json，重新确认       |
| "不要了" / "取消"                                    | 结束流程，不导入平台（已生成的文件保留在本地）   |

### 调度规则

| 识别到的内容  | 调度目标                               | 传递信息                                               |
| ------------- | -------------------------------------- | ------------------------------------------------------ |
| 表单/输入字段 | `skills/ceta/ceta-form/SKILL.md`       | analysis.json 中的字段清单 + HTML 片段 + 布局结构      |
| 数据表格/列表 | `skills/ceta/ceta-page/SKILL.md`       | analysis.json 中的列定义 + HTML 片段 + 关联 FormEntity |
| 全局样式/主题 | `skills/ceta/ceta-app-config/SKILL.md` | CSS 变量 + 颜色 + 字体                                 |
| 菜单/导航     | `skills/ceta/ceta-app-config/SKILL.md` | 菜单项列表 + 路由映射                                  |
| 审批/流程按钮 | 提示用户                               | 流程线索描述                                           |

### 样式保留要求（重要）

**生成 form.json 和 page.json 时，必须忠实还原 HTML 中的视觉样式。**
**HTML 写了什么颜色就用什么颜色，不要为了"全局主题一致性"而替换颜色。**
**如果有设计稿图片，以图片为视觉真相来源，HTML 只提供结构。**

**样式忠实还原的具体要求：**
1. Tailwind class 解析出的颜色值必须原样写入 JSON，不能替换为项目主色
2. HTML 中的渐变色、背景色、文字色必须原样保留
3. HTML 中每个独立的视觉元素（不同背景色的 badge、不同样式的文字）必须是独立的组件节点
4. 不要因为"看起来更好"而添加 HTML 中不存在的视觉效果（如把普通文字改成圆形徽章）

**样式是在 Step C（加样式）中处理的，前提是 Step A（组件拆解）和 Step B（选组件）已经正确完成。**
**如果组件拆解就错了（把多个元素合并成一个 Text），样式加得再好也没用。**

CETA 的 schemaJson 支持两种样式方式，必须根据 Step 0 的 CSS 预处理结果选择：

| 样式方式 | 适用场景 | 写入位置 |
|---------|---------|---------|
| inline style（`style` / `componentProps.style`） | 简单 class 样式、inline style、单次使用的样式 | 组件 JSON 的 `style` 或 `componentProps.style` |
| className + themeConfig.css | 伪类（:hover）、子元素选择器、3+ 处复用的样式 | 组件 JSON 的 `className` + app-config.json 的 `themeConfig.css` |

**关键流程**：Step 0 的 CSS 预处理会将所有 CSS 规则分为两类（inlineStyleMap 和 classNameRules）。
Step 1 生成组件 JSON 时，必须查阅这两个列表，确保每个组件都带上完整样式。

参考 `skills/ceta/references/style-mapping-rules.md` 的「第六节：从 HTML 提取样式的完整流程」。

#### 组件完整配置要求（不仅是样式）

**生成每个组件的 JSON 时，必须读取对应的组件 `.md` 文档（`skills/ceta/references/components/`），完整配置该组件的所有属性——包括样式、功能属性和交互行为。不要只配样式。**

每个组件的 `.md` 文档定义了该组件的完整能力。`html-mapping-rules.md` 只是路由表（HTML 元素 → 组件类型），组件怎么配置由各自的文档决定。

| 组件 | 除了样式，还必须配置的属性 | 推断依据 |
|------|-------------------------|---------|
| Button | `action`（submit/link/dialog/request/confirm 等） | 按钮文本语义 + 所在上下文（表单内→submit、搜索区→request、导航→link） |
| Table | `columnDefs`、`dataSource`、`addButtonHref`、`pinnedFilter` | 表格列头 + 关联 FormEntity |
| Select | `options` 或 `dataSource` | `<option>` 标签或业务语义 |
| Tabs | `items[].key`、`items[].label`、`items[].fields` | Tab 标签文本 + 各 Tab 内容 |
| Collapse | `items[].key`、`items[].label`、`items[].fields`、`defaultActiveKey` | 折叠面板标题 + 内容 |
| DatePicker | `showTime` | 是否包含时间选择 |
| Input | `placeholder`、`validation` | HTML 属性 + 上下文 |

##### Button action 推断规则（重点）

Button 支持 19 种 action 类型（详见 `components/display/Button.md`），必须根据按钮的文本和上下文推断：

| 按钮文本/上下文 | 推断的 action |
|---------------|-------------|
| "提交"/"保存"/"完成" 在表单内 | `{ "type": "submit" }` |
| "取消"/"返回" | `{ "type": "cancel" }` |
| "搜索"/"查询" | `{ "type": "request", "method": "get", "url": "..." }` 或 `{ "type": "formMethod" }` |
| "新建"/"创建" 在列表页 | `{ "type": "link", "href": "/form/{entityToken}/layout/new" }` 或 `{ "type": "dialog" }` |
| "编辑" 在表格行 | `{ "type": "link", "href": "/form/{entityToken}/layout/edit/:id" }` |
| "删除" | `{ "type": "confirm", "successAction": { "type": "request", "method": "delete" } }` |
| "查看"/"详情" | `{ "type": "link", "href": "/form/:id/view" }` 或 `{ "type": "dialog", "openFormProps": { "disabled": true } }` |
| "下载"/"导出" | `{ "type": "download" }` |
| "导入"/"上传" | `{ "type": "upload" }` |
| "刷新" | `{ "type": "refreshTable" }` 或 `{ "type": "refresh" }` |
| 导航类（"下一步"/"去XX页"） | `{ "type": "link", "href": "..." }` |
| "确认XX"（危险操作） | `{ "type": "confirm", "successAction": { ... } }` |

如果无法确定 action 类型，至少配一个 `{ "type": "link", "href": "#" }` 占位，并在 analysis.json 中标注需要用户确认。

#### 样式在 JSON 中的位置

样式位置的详细规则定义在每个组件的 `.md` 文件中（`skills/ceta/references/components/`）。
生成 JSON 时，遇到具体组件必须读取对应的组件文档确认样式放在哪里。

通用原则：

| 位置                   | 作用                                        | 适用组件             |
| ---------------------- | ------------------------------------------- | -------------------- |
| `style`                | 组件所有视觉样式（background、padding、border、margin、尺寸、字体等） | Stack、Grid、Box、Text、Button |
| `componentProps.style` | 组件内部样式插槽 | Card（卡片整体样式）、Input/Select 等输入组件（控件本身样式） |
| `className`            | 引用全局 CSS class（伪类、hover、复用样式） | 需要 :hover 等效果时 |

**关键区别**：Stack/Grid/Box 的视觉样式（background、padding、border 等）放在外层 `style`，不放在 `componentProps.style`。Card 是特例，用 `componentProps.style` 控制卡片整体样式。

#### 必须提取的样式

从 HTML 中提取以下样式并写入 JSON（具体放在哪个位置，查阅对应组件的 `.md` 文档）：

1. 背景色、渐变
2. 边框、圆角
3. 内边距、外边距
4. 字体大小、粗细、颜色
5. 阴影
6. 宽高
7. 布局属性（flex、grid 参数）

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
- `outputFiles` 数组必须列出所有将要生成的文件路径（相对于 ceta-workspace/）

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
            {
              "token": "userName",
              "label": "用户名",
              "type": "TEXT_BOX",
              "required": true
            },
            {
              "token": "email",
              "label": "邮箱",
              "type": "TEXT_BOX",
              "required": true
            },
            {
              "token": "phone",
              "label": "电话",
              "type": "TEXT_BOX",
              "required": false
            },
            {
              "token": "role",
              "label": "角色",
              "type": "SELECT",
              "required": false
            },
            {
              "token": "enabled",
              "label": "是否启用",
              "type": "SWITCH",
              "required": false
            }
          ],
          "layouts": ["new", "edit", "view"],
          "outputFiles": {
            "form": "system-user-form.json"
          },
          "sourceHtmlHint": "新建/编辑用户的 <form> 区域"
        },
        {
          "entityToken": "user-role",
          "entityName": "用户角色",
          "reason": "角色数据需要持久化存储到数据库",
          "fieldCount": 3,
          "fields": [
            {
              "token": "roleName",
              "label": "角色名",
              "type": "TEXT_BOX",
              "required": true
            },
            {
              "token": "roleCode",
              "label": "角色编码",
              "type": "TEXT_BOX",
              "required": true
            },
            {
              "token": "description",
              "label": "描述",
              "type": "TEXTAREA",
              "required": false
            }
          ],
          "layouts": ["new", "edit", "view"],
          "outputFiles": {
            "form": "user-role-form.json"
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
      "flowHints": ["检测到'提交'和'审批'按钮，可能需要审批流程"]
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
    "erp-system/pbcs/user-management/user-role-form.json",
    "erp-system/pbcs/user-management/user-list-page.json",
    "erp-system/pbcs/user-management/role-list-page.json",
    "erp-system/pbcs/user-management/user-search-page.json"
  ]
}
```

### outputFiles 与 app-config.json 的 token 对应规则

**文件名 = token + 类型后缀**，app-config.json 中的路由必须和文件名一一对应：

| 类型         | token 示例    | 输出文件名                                          | app-config 路由                                |
| ------------ | ------------- | --------------------------------------------------- | ---------------------------------------------- |
| FormEntity   | `system-user` | `system-user-form.json` | `/user-management/form/system-user/layout/new` |
| Page（列表） | `user-list`   | `user-list-page.json`                               | `/user-management/page/user-list`              |
| Page（搜索） | `user-search` | `user-search-page.json`                             | `/user-management/page/user-search`            |

每个 formEntity 和 page 内部都有 `outputFile` / `outputFiles` 字段，明确标注对应的文件名，确保不会出现 token 和文件名不匹配的情况。

---

## HTML 元素 → CETA 概念映射速查

### 导航与布局

| HTML 元素/结构       | CETA 概念                | 层级          |
| -------------------- | ------------------------ | ------------- |
| `<nav>` + 多个链接   | 菜单配置（config）       | Project / PBC |
| `<aside>` 侧边栏菜单 | 路由配置（routesJson）   | Project       |
| `<header>` 应用头部  | frontEndConfig.appConfig | Project       |
| Tab/标签页切换       | 同一 PBC 下的多个页面    | PBC           |
| 面包屑导航           | 路由层级关系             | Project / PBC |

### 表单相关

| HTML 元素/结构                                     | CETA 概念                     | 层级             | 判断关键          |
| -------------------------------------------------- | ----------------------------- | ---------------- | ----------------- |
| `<form>` + 提交/保存按钮                           | FormEntity + Layout           | PBC → FormEntity | 数据需要存库      |
| `<form>` + 搜索/查询按钮                           | Page（搜索页）                | PBC → Page       | 搜索条件不存库    |
| `<input>` / `<select>` / `<textarea>` 在存库表单中 | Field（字段）                 | FormEntity       | 和数据库列映射    |
| `<input>` / `<select>` 在搜索区域中                | Page 的 UI 组件               | Page             | 仅用于查询交互    |
| `<fieldset>` / 卡片分区                            | Card 布局组件                 | Layout           | —                 |
| 多列排列的字段                                     | Grid 布局组件                 | Layout           | —                 |
| 折叠面板                                           | Collapse 布局组件             | Layout           | —                 |
| 同一数据的新建/编辑/查看视图                       | 同一 FormEntity 的不同 Layout | FormEntity       | 共享同一套 fields |

### 列表/页面相关

| HTML 元素/结构          | CETA 概念                | 层级       |
| ----------------------- | ------------------------ | ---------- |
| `<table>`               | FormEntityPage（列表页） | PBC → Page |
| 表头 `<th>`             | Table 列定义             | Page       |
| 操作列（编辑/删除按钮） | ACTION_COLUMN            | Page       |
| 搜索/筛选区域           | Table 的 filter 配置     | Page       |
| 分页器                  | Table 的 pagination 配置 | Page       |
| 统计卡片/图表           | Dashboard portlet        | Page       |

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
