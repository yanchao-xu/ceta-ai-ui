---
name: ceta-pbc-requirement
description: >
  PBC 业务信息文档生成器。为每个 PBC 生成包含 ER 图、业务流程、用户故事的 HTML 文档。
  文档用于展示 PBC 的业务全貌，可上传到 CETA 平台作为 PBC 简介。
license: MIT
metadata:
  openclaw:
    requires:
      env: ["CETA_API_BASE", "CETA_API_TOKEN"]
    version: "1.0.0"
    author: "CETA Team"
    tags: ["pbc", "requirement", "documentation", "html"]
    source: "builtin"
    dependencies:
      - "ceta/ceta-pbc"
---

# PBC 业务信息文档生成

## 概述

为项目中的每个 PBC 生成一份 HTML 格式的业务信息文档，包含三个核心部分：
1. **ER 图** — 数据实体、字段定义、实体间关系
2. **业务流程** — 核心业务操作的步骤流程
3. **用户故事** — 用户视角的功能场景描述

## 输出规则

- 每个 PBC 生成一个 HTML 文件
- 文件命名：`{pbcToken}.html`
- 输出目录：`ceta-workspace/{projectToken}/requirement/`
- 样式模板参考：`requirements/requirement_itinerary-booking.html`

## 信息来源

生成文档前，需要收集 PBC 的业务信息。信息来源优先级：

1. **CETA 平台查询**（最准确）：
   - `form__pbc__get` — 获取 PBC 详情，含 formEntityList、formEntityPageList、flowDefinitionList
   - `form__form_entity__get` — 获取表单详情，含 fields 和 layouts
   - `form__form_entity_page__list_by_pbc_id` — 获取页面列表
   - `flow__flow_definition__list` — 获取流程列表

2. **本地 workspace 文件**：
   - `ceta-workspace/{projectToken}/pbcs/{pbcToken}/` 下的 JSON 文件
   - `pbc-seed-data.json` — 完整的 PBC 配置
   - `{entity}-fields.json` — 字段定义
   - `design.md` — 设计文档

3. **用户提供的需求描述**（需要 AI 理解和整理）

## HTML 文档结构

### 1. Header（PBC 基本信息）

必须包含：
- PBC Token
- PBC 中文名
- 功能摘要（一句话描述核心能力，用中文顿号分隔关键功能）
- 所属项目名称/标签

### 2. Tab 导航

三个 Tab 页签：ER 图、业务流程、用户故事。默认显示 ER 图。

### 3. ER 图 Tab（数据实体关系图）

对 PBC 下的每个 FormEntity，展示：

- **序号** — 从 1 开始编号
- **Entity Token** — 如 `flight-availability`
- **中文名** — 如 `航班空席信息`
- **主键** — `PK: id (auto)`
- **外键** — 如 `FK: pnrRloc`（如果有 ACL 类型字段或业务关联）
- **字段列表** — 每个字段显示 `fieldToken` + `CETA 字段类型`（TEXT_BOX / DATE / NUMBER / SELECT / SWITCH 等）
- **实体间关系** — 用文字描述关联关系（如 "预订后生成航段"）

字段类型直接使用 CETA 的 Field API type（TEXT_BOX、TEXTAREA、NUMBER、DATE、TIME、SELECT、RADIO、SWITCH、UPLOAD、ACL 等）。

实体间关系的展示：在两个有关联的实体之间，插入一个关系描述标签。

### 4. 业务流程 Tab

对 PBC 的核心业务操作，展示步骤流程：

- **流程名称** — 如 "航程预订流程"
- **步骤列表** — 每个步骤包含：
  - 步骤编号
  - 步骤名称
  - 步骤描述（简要说明该步骤做什么）
  - 涉及的实体/页面（可选）
- **步骤间的流转关系** — 用箭头或文字描述

如果 PBC 有 FlowDefinition（审批流），也在此展示流程节点。

如果 PBC 没有明确的业务流程，根据页面和表单的功能推断典型的操作流程。

### 5. 用户故事 Tab

以用户视角描述 PBC 的功能场景：

- **故事编号** — US-001、US-002 等
- **故事标题** — 简短描述
- **角色** — 作为什么角色
- **目标** — 我想要做什么
- **价值** — 以便于达到什么目的
- **验收标准** — 关键的验收条件（2-4 条）

用户故事应覆盖 PBC 的主要功能点，通常 3-8 个故事。

## HTML 模板

使用以下 HTML 模板生成文档。模板使用 Tailwind CSS CDN + Nunito 字体，与现有样式保持一致。

**重要：以下模板中的占位符说明：**
- `{{pbcToken}}` — PBC 的 token
- `{{pbcName}}` — PBC 的中文名
- `{{pbcSummary}}` — 功能摘要
- `{{projectName}}` — 项目名称
- `{{erContent}}` — ER 图内容（多个实体卡片）
- `{{flowContent}}` — 业务流程内容
- `{{storyContent}}` — 用户故事内容


```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>PBC: {{pbcToken}} — {{pbcName}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&display=swap" rel="stylesheet" />
    <style>
      * { font-family: "Nunito", sans-serif; }
      .card-shadow { box-shadow: 0 10px 40px -10px rgba(79, 172, 254, 0.3); }
      .tab-active { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); color: white; transform: translateY(-2px); box-shadow: 0 8px 25px rgba(79, 172, 254, 0.4); }
      .tab-inactive { background: white; color: #64748b; }
      .tab-inactive:hover { background: #f0f9ff; color: #4facfe; }
      .entity-card { background: linear-gradient(135deg, #e0f2fe 0%, #bae6fd 100%); border-left: 4px solid #0ea5e9; }
      .flow-step { background: linear-gradient(135deg, #dbeafe 0%, #bfdbfe 100%); border: 2px solid #60a5fa; }
      .story-card { background: linear-gradient(135deg, #eff6ff 0%, #dbeafe 100%); border-top: 4px solid #3b82f6; }
      .badge { background: linear-gradient(135deg, #60a5fa 0%, #3b82f6 100%); }
      .field-tag { background: #e0f2fe; color: #0369a1; }
      .pk-tag { background: #fecaca; color: #dc2626; }
      .fk-tag { background: #bbf7d0; color: #16a34a; }
      .tab-content { display: none; animation: fadeIn 0.4s ease-out; }
      .tab-content.active { display: block; }
      @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
      .flow-arrow { color: #3b82f6; font-size: 1.5rem; }
      .ac-item { border-left: 3px solid #10b981; padding-left: 0.5rem; }
    </style>
  </head>
  <body class="bg-gradient-to-br from-blue-50 via-cyan-50 to-blue-100 min-h-screen">
    <!-- Header -->
    <div class="max-w-6xl mx-auto px-4 pt-6">
      <div class="bg-white rounded-2xl p-4 shadow-lg mb-4">
        <div class="flex items-center justify-between">
          <div>
            <h1 class="text-2xl font-bold text-slate-800">📋 PBC: {{pbcToken}}</h1>
            <p class="text-sm text-slate-500 mt-1">{{pbcName}} — {{pbcSummary}}</p>
          </div>
          <span class="bg-sky-100 text-sky-700 px-4 py-2 rounded-xl text-sm font-bold">{{projectName}}</span>
        </div>
      </div>
    </div>

    <!-- Tab Navigation -->
    <div class="max-w-6xl mx-auto px-4">
      <div class="bg-white rounded-2xl p-2 shadow-lg flex space-x-2">
        <button onclick="switchTab('er')" id="tab-er" class="tab-active flex-1 py-3 px-6 rounded-xl font-bold text-sm transition-all duration-300 flex items-center justify-center space-x-2">
          <span class="text-lg">🗄️</span><span>ER图</span>
        </button>
        <button onclick="switchTab('flow')" id="tab-flow" class="tab-inactive flex-1 py-3 px-6 rounded-xl font-bold text-sm transition-all duration-300 flex items-center justify-center space-x-2">
          <span class="text-lg">🔄</span><span>业务流程</span>
        </button>
        <button onclick="switchTab('story')" id="tab-story" class="tab-inactive flex-1 py-3 px-6 rounded-xl font-bold text-sm transition-all duration-300 flex items-center justify-center space-x-2">
          <span class="text-lg">📖</span><span>用户故事</span>
        </button>
      </div>
    </div>

    <!-- Tab Content -->
    <div class="max-w-6xl mx-auto px-4 py-6 pb-12">

      <!-- ER Tab -->
      <div id="content-er" class="tab-content active">
        <div class="bg-white rounded-3xl p-6 card-shadow">
          <h2 class="text-2xl font-bold text-blue-600 mb-4 flex items-center">
            <span class="mr-2">🗄️</span> 数据实体关系图
          </h2>
          {{erContent}}
        </div>
      </div>

      <!-- Flow Tab -->
      <div id="content-flow" class="tab-content">
        <div class="bg-white rounded-3xl p-6 card-shadow">
          <h2 class="text-2xl font-bold text-blue-600 mb-4 flex items-center">
            <span class="mr-2">🔄</span> 业务流程
          </h2>
          {{flowContent}}
        </div>
      </div>

      <!-- Story Tab -->
      <div id="content-story" class="tab-content">
        <div class="bg-white rounded-3xl p-6 card-shadow">
          <h2 class="text-2xl font-bold text-blue-600 mb-4 flex items-center">
            <span class="mr-2">📖</span> 用户故事
          </h2>
          {{storyContent}}
        </div>
      </div>

    </div>

    <script>
      function switchTab(tabName) {
        document.querySelectorAll(".tab-content").forEach(c => c.classList.remove("active"));
        document.getElementById("content-" + tabName).classList.add("active");
        document.querySelectorAll('[id^="tab-"]').forEach(t => { t.classList.remove("tab-active"); t.classList.add("tab-inactive"); });
        document.getElementById("tab-" + tabName).classList.remove("tab-inactive");
        document.getElementById("tab-" + tabName).classList.add("tab-active");
      }
    </script>
  </body>
</html>
```

### 实体卡片模板（ER 图中每个 FormEntity）

序号颜色交替使用：蓝色（bg-blue-500）、绿色（bg-green-500）、紫色（bg-purple-500）、橙色（bg-orange-500），循环使用。

```html
<!-- 单个实体卡片 -->
<div class="entity-card rounded-2xl p-4 mb-4">
  <div class="flex items-center justify-between mb-3">
    <h3 class="font-bold text-blue-800 text-lg flex items-center">
      <span class="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center text-sm mr-2">1</span>
      entity-token
      <span class="text-xs font-normal text-blue-600 ml-2">(中文名)</span>
    </h3>
    <div class="flex space-x-1">
      <span class="pk-tag px-2 py-1 rounded-lg text-xs font-bold">PK: id (auto)</span>
      <!-- 如果有外键 -->
      <span class="fk-tag px-2 py-1 rounded-lg text-xs font-bold">FK: foreignKeyField</span>
    </div>
  </div>
  <div class="grid grid-cols-2 md:grid-cols-4 gap-2 text-sm">
    <!-- 每个字段一个 tag -->
    <div class="field-tag px-2 py-1 rounded">fieldToken <span class="text-xs">TEXT_BOX</span></div>
    <!-- 更多字段... -->
  </div>
</div>
```

### 实体间关系标签

在两个有关联的实体之间插入：

```html
<div class="flex justify-center my-2">
  <div class="bg-blue-100 rounded-full px-4 py-2 text-blue-700 text-sm font-bold">
    🔗 关系描述文字
  </div>
</div>
```

### 业务流程模板

```html
<!-- 单个流程 -->
<div class="mb-6">
  <h3 class="font-bold text-blue-800 text-lg mb-4">流程名称</h3>
  <div class="space-y-3">
    <!-- 步骤 -->
    <div class="flow-step rounded-xl p-4">
      <div class="flex items-center mb-2">
        <span class="bg-blue-500 text-white rounded-full w-7 h-7 flex items-center justify-center text-xs font-bold mr-3">1</span>
        <span class="font-bold text-blue-800">步骤名称</span>
      </div>
      <p class="text-sm text-slate-600 ml-10">步骤描述，说明该步骤做什么。</p>
      <p class="text-xs text-slate-400 ml-10 mt-1">涉及：entity-token / page-schema-id</p>
    </div>
    <!-- 箭头 -->
    <div class="flex justify-center"><span class="flow-arrow">↓</span></div>
    <!-- 下一个步骤... -->
  </div>
</div>
```

### 用户故事卡片模板

```html
<!-- 单个用户故事 -->
<div class="story-card rounded-2xl p-5 mb-4">
  <div class="flex items-center justify-between mb-3">
    <h3 class="font-bold text-blue-800 text-lg">US-001 故事标题</h3>
    <span class="badge text-white px-3 py-1 rounded-full text-xs font-bold">核心功能</span>
  </div>
  <p class="text-sm text-slate-700 mb-3">
    作为 <strong>角色</strong>，我想要 <strong>做什么</strong>，以便于 <strong>达到什么目的</strong>。
  </p>
  <div class="mt-3">
    <p class="text-xs font-bold text-slate-500 mb-2">验收标准：</p>
    <div class="space-y-1">
      <div class="ac-item text-sm text-slate-600 py-1">验收条件 1</div>
      <div class="ac-item text-sm text-slate-600 py-1">验收条件 2</div>
    </div>
  </div>
</div>
```

## 生成流程

### 步骤 1：收集 PBC 信息

通过 MCP API 或本地文件获取 PBC 的完整信息：

```
1. 获取项目信息 → form__project__get / form__project__get_by_token
2. 获取 PBC 列表 → form__pbc__list_by_project_id
3. 对每个 PBC：
   a. form__pbc__get(pbcId) → 获取 formEntityList、formEntityPageList、flowDefinitionList
   b. 对每个 FormEntity → 提取 fields（token、type）、layouts
   c. 分析实体间关系（ACL 字段、业务关联）
```

### 步骤 2：整理业务信息

对每个 PBC，整理出：

- **基本信息**：token、中文名、功能摘要
- **ER 图数据**：每个实体的字段列表、PK/FK、实体间关系
- **业务流程**：根据页面功能和流程定义，梳理核心操作步骤
- **用户故事**：根据 PBC 的功能点，编写用户故事

### 步骤 3：生成 HTML

使用上面的模板，将占位符替换为实际内容，生成完整的 HTML 文件。

### 步骤 4：输出文件

将生成的 HTML 文件写入 `ceta-workspace/{projectToken}/requirement/{pbcToken}.html`。

### 步骤 5：上传到平台（可选）

调用 `pbc_upload_ai_info` 将 HTML 上传到 CETA 平台：

```
pbc_upload_ai_info(
  projectId: "项目ID",
  pbcId: "PBC ID",
  htmlFilePath: "ceta-workspace/{projectToken}/requirement/{pbcToken}.html"
)
```

注意：不是所有 CETA 实例都支持此接口，调用失败时忽略错误继续。

## 批量生成

当需要为整个项目生成所有 PBC 的业务信息文档时：

```
1. form__project__get_by_token(projectToken) → 获取 projectId
2. form__pbc__list_by_project_id(projectId) → 获取所有 PBC
3. 过滤掉系统 PBC（category = "SYSTEM"，如 user-management）
4. 对每个业务 PBC（category = "BUSINESS"）：
   a. 收集信息
   b. 生成 HTML
   c. 写入文件
   d. 上传到平台（可选）
```

## 注意事项

- 只为业务 PBC（category = BUSINESS）生成文档，跳过系统 PBC（如 user-management）
- 字段类型直接使用 CETA 的 Field API type，不要转换为组件名
- 如果 PBC 信息不完整（如刚创建还没有表单），ER 图部分显示"暂无数据实体"
- 业务流程和用户故事需要 AI 根据 PBC 的功能进行理解和编写，不是机械地从数据中提取
- HTML 文件必须是自包含的（使用 CDN 引入样式），不依赖本地资源
- 生成的 HTML 应该能直接在浏览器中打开查看
