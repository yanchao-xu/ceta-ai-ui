---
name: ceta-app-config
description: >
  CETA 应用配置生成（app.config.json）。统一管理菜单（navbar items）、主题（themeConfig）、
  路由（routers）、国际化（i18n）、模板选择（template）等全局配置。
  在 HTML 转换完成后，根据生成的表单和页面信息自动创建菜单配置。
  不负责调用 API 创建资源，只负责生成 JSON 结构。
inclusion: manual
metadata:
  version: "2.0.0"
  tags: ["menu", "theme", "css", "navbar", "navigation", "app-config", "router", "i18n", "ceta"]
  layer: "conversion"
  dependencies:
    - "ceta-basic"
---

# CETA 应用配置生成（app.config.json）

## 概述

`app.config.json` 是 CETA 前端应用的核心配置文件，位于每个可部署包的 `appConfig/` 目录下。
它在构建时和运行时都会被读取，控制着应用的几乎所有全局行为。

本 SKILL 统一管理 `app.config.json` 的所有部分：菜单、主题、路由、国际化、模板等。

## app.config.json 在数据模型中的位置

```
Project
└── frontEndConfig
    └── config (app.config.json)
        └── appConfig
            ├── template / mobileTemplate    ← 页面模板选择
            ├── title / description / favicon ← 站点基础信息
            ├── baseUrl                       ← 构建路径（必须以 / 结尾）
            ├── appearance                    ← 外观模式（light / dark）
            ├── i18n                          ← 国际化配置
            ├── templateConfig                ← 模板布局配置（含菜单）
            │   ├── navbar.items              ← 桌面端菜单项
            │   ├── navbar.position           ← sidebar | top
            │   ├── header / footer           ← 页头页脚
            │   ├── sidebar                   ← 侧边栏配置
            │   ├── login                     ← 登录页配置
            │   └── userMenu                  ← 用户菜单
            ├── themeConfig                   ← 主题/样式配置
            │   ├── base                      ← 基础主题色名
            │   ├── token                     ← Ant Design 全局 token
            │   ├── components                ← 组件级 token
            │   ├── cssVars                   ← CSS 自定义变量
            │   ├── css                       ← 全局自定义 CSS 字符串
            │   └── agGridCssVars             ← AG Grid CSS 变量
            └── mobileTemplateConfig          ← 移动端配置
```

## 配置规则

- `baseUrl` 是必填项，且**必须以 `/` 结尾**，否则构建会直接报错退出
- 在 SaaS 模式下（`SAAS_MODE`），配置不从文件读取，而是通过 API `/form/api/front-end-config` 从服务端获取
- 运行时通过 `window.__icp_i18n__` 注入 i18n 覆盖，允许免构建修改语言配置
- 配置中的文本字段（title、description、logo alt、菜单 label 等）会被 i18n key 生成器扫描，自动生成翻译 key

### 初始模板最低字段要求（只能多不能少）

生成 `app-config.json` 时，**必须**以下面的结构为最低基线模板。所有字段都必须存在且有值，可以在此基础上增加字段，但**绝不能缺少任何字段**：

```json
{
  "appConfig": {
    "template": "Template2",
    "mobileTemplate": "MobileTemplate1",
    "title": "",
    "description": "",
    "baseUrl": "/ui/",
    "favicon": "public/favicon.ico",
    "projectName": "",
    "appearance": "light",
    "i18n": {
      "allowedLanguages": ["zh-CN"],
      "fallbackLng": "zh-CN"
    },
    "templateConfig": {
      "title": "",
      "logo": {
        "src": "",
        "alt": "Logo",
        "default": "default-logo-4"
      },
      "navbar": {
        "appearance": "light",
        "items": []
      },
      "header": {},
      "sidebar": {
        "collapsible": true
      },
      "footer": {},
      "login": {
        "template": "LoginSplit"
      }
    },
    "mobileTemplateConfig": {
      "carousel": {},
      "subApps": [],
      "navbar": {
        "items": [
          { "to": "/mobile", "label": "", "icon": "home" },
          { "to": "/mobile/my", "label": "", "icon": "person" }
        ]
      }
    },
    "themeConfig": {
      "base": "",
      "token": {},
      "components": {},
      "cssVars": {},
      "css": ""
    }
  },
  "routers": {
    "desktop": {
      "/": {
        "pageTitle": "",
        "isHomePage": true
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/confirm": {
        "pageTitle": "",
        "formType": "FLOW-FORM"
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/approval": {
        "pageTitle": "",
        "formType": "FLOW-APPROVAL"
      }
    },
    "mobile": {
      "/mobile": {
        "pageTitle": "",
        "isHomePage": true,
        "showNavBar": true
      }
    }
  }
}
```

**关键约束：**
- `template`、`mobileTemplate`、`baseUrl`、`favicon`、`appearance`、`i18n`、`templateConfig`、`mobileTemplateConfig`、`themeConfig`、`routers` 等字段**缺一不可**
- 即使某些字段值为空字符串或空对象，也必须保留字段本身
- 生成时可以在此基础上添加更多字段（如 `navbar.position`、`userMenu`、`agGridCssVars` 等），但不能删减上述任何字段
- 这是最低保障结构，确保前端应用能正常启动和渲染

---

## 一、模板选择

| 字段 | 说明 | 可选值 |
|------|------|--------|
| `template` | 桌面端布局模板 | `Template2`（推荐）、`Template1` |
| `mobileTemplate` | 移动端布局模板 | `MobileTemplate1` |

路由系统根据 template 值渲染对应的 Layout 组件。

---

## 二、站点基础信息

| 字段 | 类型 | 说明 |
|------|------|------|
| `title` | string | 页面 `<title>` |
| `description` | string | `<meta>` 描述 |
| `favicon` | string | favicon 图标路径（通常 `"public/favicon.ico"`） |
| `projectName` | string | 项目名称（可为空） |
| `baseUrl` | string | webpack publicPath 和开发服务器路径前缀，**必须以 `/` 结尾** |
| `appearance` | string | `"light"` 或 `"dark"` |

---

## 三、国际化配置（i18n）

```json
{
  "i18n": {
    "allowedLanguages": ["zh-CN", "en-US"],
    "fallbackLng": "zh-CN"
  }
}
```

- `allowedLanguages`：支持的语言列表，构建时 webpack 据此决定打包哪些 intl locale 数据
- `fallbackLng`：默认回退语言

---

## 四、菜单配置（navbar）

### 菜单在配置中的位置

```
appConfig.templateConfig.navbar
├── items: [...]        ← 桌面端菜单项
├── position: "sidebar" | "top"
├── appearance: "light" | "dark"
└── searchEnabled: boolean
```

### NavbarItem 字段定义

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `to` | string | 是 | 路由路径。叶子菜单必须有值；纯分组菜单可为 `""` |
| `label` | string | 是 | 菜单显示名称（默认语言） |
| `label_i18n_zh-CN` | string | 否 | 中文标签 |
| `label_i18n_en-US` | string | 否 | 英文标签 |
| `label_i18n_ja-JP` | string | 否 | 日文标签 |
| `icon` | string | 否 | 图标名称（见图标规范） |
| `key` | string | 否 | 菜单项唯一标识（8位十六进制，如 `"793341b4"`） |
| `children` | NavbarItem[] | 否 | 子菜单项，支持多级嵌套 |
| `activeBasePath` | string | 否 | 激活状态匹配的基础路径 |
| `category` | string \| null | 否 | 菜单分类（通常为 `null`） |
| `accessPermissionPredicate` | object | 否 | 权限控制 |
| `hidden` | boolean | 否 | 是否隐藏菜单项（隐藏但路由仍可访问） |
| `noNeedAuth` | boolean | 否 | 是否无需登录即可访问 |
| `target` | string | 否 | 链接打开方式（如 `"_blank"`） |

### 权限配置

```json
{
  "accessPermissionPredicate": {
    "hasAnyOf": ["permission-token_R", "permission-token_C"]
  }
}
```

- `hasAnyOf`：用户拥有列表中任一权限即可看到该菜单
- 权限 token 格式：`{entityToken}_R`（读）、`{entityToken}_C`（创建）、`{entityToken}_U`（更新）、`{entityToken}_D`（删除）
- 特殊值 `"***"` 表示所有已登录用户可见
- 特殊值 `"USER_CREATE"` 是系统内置管理员权限
- 不设置 `accessPermissionPredicate` 则所有用户可见

### 图标规范

| 图标库 | 格式 | 示例 |
|--------|------|------|
| Octicons | `oct:{name}` | `oct:people`, `oct:checklist`, `oct:home` |
| Material | `material:{name}` | `material:engineering-rounded`, `material:group-sharp` |
| Solar | `solar:{name}` | `solar:database-outline`, `solar:users-group-rounded-outline` |
| Ant Design | `ant:{name}` | `ant:api-outlined` |
| 内置 | 直接名称 | `home`, `person`, `mining` |

推荐优先使用 `oct:` 和 `material:` 前缀的图标。

### 菜单层级设计原则

1. 一级菜单：业务模块或功能大类
2. 二级菜单：模块下的具体页面
3. 三级菜单：仅在必要时使用
4. 建议不超过 3 级嵌套

---

## 五、路由系统

整个路由和菜单体系围绕两个核心概念：`pbcToken`（业务组件标识）和各类 `formToken`（表单/页面标识），分三层运作。

### 路由地址构建规则

路由地址的基本模式是 `/{pbcToken}/{类型}/{具体token}`，由 `generalRoutes.js` 定义通用路由模板：

| 场景 | 路由模式 | 示例 |
|------|---------|------|
| 普通页面 | `/{pbcToken}/page/{schemaId}` | `/contract/page/contract-list` |
| PBC 首页 | `/{pbcToken}/` | `/contract/`（默认加载 schemaId 为 entry 的页面） |
| 表单新建 | `/{pbcToken}/form/{formEntityToken}/layout/{layoutToken}` | `/contract/form/contract-form/layout/default` |
| 表单编辑 | `/{pbcToken}/form/{formEntityToken}/layout/{layoutToken}/{dataId}` | `/contract/form/contract-form/layout/default/123` |
| 表单查看 | `/{pbcToken}/form/{formEntityToken}/{dataId}/view` | `/contract/form/contract-form/123/view` |
| 流程新建 | `/{pbcToken}/flow/{flowToken}/new` | `/contract/flow/approval-flow/new` |
| 流程审批 | `/{pbcToken}/flow/{flowInstanceId}/approval` | `/contract/flow/456/approval` |
| 移动端 | `/mobile/{pbcToken}/...` | `/mobile/contract/page/list` |

通用路由模板在 `composeAllRoutesConfig.js` 中通过 `icpRoutes()` 函数自动为每个 PBC 生成完整路由。
`flattenRoutes.js` 负责将每个 PBC 自定义的路由拼接上 pbcToken 前缀，合并到全局路由表中。

### 菜单与路由的关系

**菜单和路由是解耦的：**
- 路由是自动全量注册的（每个 PBC 都会拥有所有通用路由模式），不需要手动注册
- 菜单只是导航入口，`to` 字段指向某个已存在的路由地址
- 菜单是按需配置的（只把用户需要看到的入口放进去）

### routers 配置

`app.config.json` 中的 `routers` 字段用于配置特殊路由行为：

```json
{
  "routers": {
    "desktop": {
      "/": {
        "pageTitle": "",
        "pbcToken": "dashboard",
        "schemaId": "entry",
        "isHomePage": true
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/confirm": {
        "pageTitle": "",
        "formType": "FLOW-FORM"
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/approval": {
        "pageTitle": "",
        "formType": "FLOW-APPROVAL"
      }
    },
    "mobile": {
      "/mobile": {
        "pageTitle": "",
        "isHomePage": true,
        "showNavBar": true
      }
    }
  }
}
```

- 根路由 `/` 通常指向首页 PBC 的 entry 页面
- 流程相关路由（`/todo-list/form/...`）是固定模板，大多数项目都需要
- 移动端根路由 `/mobile` 配置首页和导航栏显示

---

## 六、主题配置（themeConfig）

### themeConfig 字段定义

| 字段 | 类型 | 说明 |
|------|------|------|
| `base` | string | 基础主题色名（如 `"orange"`），可为空 |
| `token` | object | Ant Design 5 全局 Design Token |
| `components` | object | Ant Design 5 组件级 Token（按组件名分组） |
| `cssVars` | object | 自定义 CSS 变量，键值对，全局可用 |
| `css` | string | 自定义 CSS 字符串，全局注入到 `<style>` 标签 |
| `agGridCssVars` | object | AG Grid 专用 CSS 变量 |

### token 常用字段

```json
{
  "token": {
    "colorPrimary": "#1a237e",
    "colorPrimaryBg": "#FFEBE8",
    "colorPrimaryHover": "#D42C3D",
    "colorInfo": "#BE1E2D",
    "colorLink": "#BE1E2D",
    "borderRadius": 6
  }
}
```

### components 示例

```json
{
  "components": {
    "Menu": {
      "colorBgContainer": "#23164B",
      "itemSelectedBg": "#392B63",
      "itemSelectedColor": "#FFF",
      "itemColor": "#FFF",
      "itemHoverColor": "#F68443",
      "itemHoverBg": "#392B63",
      "fontSize": 16
    }
  }
}
```

### cssVars 示例

```json
{
  "cssVars": {
    "--card-padding": "20px",
    "--helper-color": "rgb(178,109,10)"
  }
}
```

在 schemaJson 中通过 `var()` 引用：`{ "style": { "padding": "var(--card-padding)" } }`

平台内置 CSS 变量（无需定义即可使用）：
- `--primary-color`、`--border-color`、`--border-color-light`
- `--info-color`、`--error-color`、`--warning-color`、`--success-color`
- `--max-z-index`

### css 字段详解

`css` 是一个 CSS 字符串，使用 `\n` 换行：

```json
{
  "css": ".my-class {\n  background: #f5f5f5;\n}\n\n.my-class:hover {\n  background: #e8e8e8;\n}"
}
```

适用场景：伪类样式（:hover/:focus）、子元素选择器、多组件复用 class、动画 @keyframes、提高优先级覆盖默认样式。

CSS class 命名规范：
- 使用 kebab-case（如 `bg-header-card`、`cell-brown`）
- 建议加业务前缀避免冲突
- 避免使用平台已有 class 名（`card-layout`、`stack-layout`、`grid-layout`、`form-layout`）

提高优先级技巧 — 重复 class：
```css
.bg-header-card.bg-header-card .card-title {
  background: #5c6bc0;
  color: white;
}
```

### 判断 inline style vs className 的规则

| 条件 | 用 inline style | 用 className |
|------|----------------|-------------|
| 只有一个组件使用 | ✅ | |
| 多个组件复用同一样式 | | ✅ |
| 需要 :hover 等伪类 | | ✅ |
| 需要控制子元素样式 | | ✅ |
| 需要动画 @keyframes | | ✅ |
| 简单的 padding/margin/color | ✅ | |
| 需要覆盖平台默认样式 | | ✅ |

---

## 七、模板布局配置（templateConfig）

### login 登录页配置

```json
{
  "login": {
    "template": "LoginCentered",
    "title": "系统名称",
    "subTitle": "系统描述",
    "useTemplatePrimaryColor": true,
    "logo": { "src": "https://..." },
    "showRegister": false,
    "registerLink": ""
  }
}
```

可选模板：`LoginCentered`（居中）、`LoginSplit`（左右分栏）

### sidebar 侧边栏配置

```json
{
  "sidebar": {
    "collapsible": true,
    "appearance": "light"
  }
}
```

### mobileTemplateConfig 移动端配置

```json
{
  "mobileTemplateConfig": {
    "carousel": {},
    "subApps": [],
    "navbar": {
      "items": [
        { "to": "/mobile", "label": "", "icon": "home" },
        { "to": "/mobile/my", "label": "", "icon": "person" }
      ]
    },
    "tabBar": {
      "items": []
    },
    "tabBarUsed": false
  }
}
```

移动端菜单项的 `to` 会自动加上 `/mobile` 前缀。

---

## 八、菜单自动生成规则（核心）

在 HTML 转换完成后，根据生成的表单（FormEntity）和页面（FormEntityPage）信息，自动创建菜单配置。

### 菜单项的三种创建时机

| 时机 | 触发条件 | 生成规则 |
|------|---------|---------|
| 创建 PBC 时 | `createPbc()` 调用 `insertPbcToMenu()` | 默认生成 `{ to: "/{pbcToken}", label: pbc.name, icon: "oct:home" }`。如果 PBC config 里自定义了 menu 数组，则使用自定义菜单 |
| 表单设计器保存时 | 全新表单实体首次保存后，`needAutoPageMenu()` 返回 true | 自动创建列表页（schemaId 为 `{formEntityToken}-list`），生成菜单项 `{ to: "/{pbcToken}/page/{formEntityToken}-list", label: "...", icon: "oct:home" }` |
| 手动添加 | 通过 AddNavItemPopover 组件 | 支持选择 PBC、选择具体页面、自定义链接 |

### AI 生成菜单的自动化规则

当 HTML 转换流程完成后（已生成 form.json、page.json），本 SKILL 根据 `analysis.json` 中的信息自动生成菜单：

#### 规则 1：每个有列表页的 FormEntity 生成一个菜单项

```
对 analysis.json 中每个 page（type: "list"）：
→ 生成菜单项 {
    to: "/{pbcToken}/page/{formEntityToken}-form-list",
    label: page.pageName,
    icon: "oct:home",
    key: 随机8位hex
  }
```

注意：列表页的 schemaId 命名规则是 `{formEntityToken}-form-list`（平台自动创建时的默认命名）。
如果 analysis.json 中 page 有自定义 schemaId，则使用自定义值。

#### 规则 2：同一 PBC 下多个页面归入同一父菜单

```
如果一个 PBC 下有 2+ 个页面：
→ 创建父菜单 { to: "", label: pbc.name, icon: 选择合适图标, children: [...] }
→ 各页面作为 children

如果一个 PBC 下只有 1 个页面：
→ 直接作为一级菜单，不需要父菜单包裹
```

#### 规则 3：项目级别自动添加用户管理菜单

大多数项目都需要用户管理模块（`user-management-new` 是平台内置 PBC），自动追加：

```json
{
  "to": "",
  "label": "用户与角色",
  "label_i18n_zh-CN": "用户与角色",
  "label_i18n_en-US": "Users and Roles",
  "key": "793341b4",
  "icon": "material:group-sharp",
  "children": [
    {
      "to": "/user-management-new/page/system-user-form-list",
      "label": "用户",
      "label_i18n_zh-CN": "用户",
      "label_i18n_en-US": "User",
      "key": "74e20a44",
      "icon": "oct:passkey-fill"
    },
    {
      "to": "/user-management-new/page/system-role-form-list",
      "label": "角色",
      "label_i18n_zh-CN": "角色",
      "label_i18n_en-US": "Role",
      "key": "ad696683",
      "icon": "material:people-alt-outlined"
    }
  ]
}
```

#### 规则 4：根据业务语义选择图标

| 业务类型 | 推荐图标 |
|---------|---------|
| 用户/人员 | `oct:people`、`solar:users-group-rounded-outline` |
| 订单/清单 | `oct:checklist` |
| 数据/主数据 | `solar:database-outline` |
| 设置/配置 | `material:perm-data-setting-rounded` |
| 文件/文档 | `oct:file` |
| 公司/组织 | `oct:briefcase` |
| 连接器 | `ant:api-outlined` |
| 首页/默认 | `oct:home` |
| 星标/推荐 | `oct:star` |
| 任务/待办 | `oct:tasklist` |

---

## 九、输出文件规范

将生成的配置写入 `ceta-workspace/` 目录：

```
ceta-workspace/
└── {项目名称}/
    └── {项目名称}-app-config.json   # 完整 app.config.json（菜单+主题+路由，唯一输出文件）
```

**不单独生成 menu.json、theme.json、theme-css.css。所有配置统一写入 app-config.json。**

### 命名规则

- 项目名称使用 kebab-case（如 `trade-boost`、`plant-maintenance`）
- 如果用户已有同项目的其他输出文件（如 `-form.json`、`-page.json`），放在同一目录下
- 输出文件使用格式化 JSON（2 空格缩进）

### 输出内容说明

- `-app-config.json`：完整的 `app.config.json` 结构，包含菜单（navbar items）、主题（themeConfig 含 css 字符串）、路由（routers），可直接用于项目配置

---

## 十、生成流程

### 场景 A：HTML 转换后自动生成菜单

当 `ceta-html-analyzer` 完成分析并生成了 form/page 文件后，自动触发菜单生成：

```
ceta-html-analyzer 完成
      │
      ▼
读取 analysis.json
      │
      ├── 提取所有 PBC 及其页面信息
      ├── 按「菜单自动生成规则」创建菜单项
      ├── 如果 HTML 中有样式/主题信息 → 生成 themeConfig
      ├── 配置 routers（根路由指向第一个 PBC 的 entry）
      └── 输出完整 app-config.json + 单独的 menu.json / theme.json
```

### 场景 B：用户直接要求生成菜单

用户提供 PBC 列表和页面信息，或自然语言描述菜单结构：

```
1. 确定菜单层级结构（哪些是分组，哪些是叶子页面）
2. 为每个叶子菜单项生成路由路径 /{pbcToken}/page/{schemaId}
3. 为分组菜单选择合适的图标
4. 如果项目支持多语言，添加 label_i18n_* 字段
5. 根据需要配置权限 accessPermissionPredicate
6. 为每个菜单项生成 8 位十六进制 key
7. 输出 menu.json
```

### 场景 C：用户要求生成主题/样式

用户提供 HTML（含 `<style>` 标签）、设计稿颜色、或自然语言描述：

```
1. 分析输入中的样式需求
2. 区分 inline style vs 全局 CSS class（见判断规则）
3. 生成 CSS 规则和对应的 className
4. 如果有主题色、圆角等全局配置，生成 token 和 cssVars
5. 将 CSS 规则转为单行字符串写入 themeConfig.css
6. 输出 theme.json 和 theme-css.css
```

---

## 十一、完整 app.config.json 示例

```json
{
  "appConfig": {
    "template": "Template2",
    "mobileTemplate": "MobileTemplate1",
    "title": "项目标题",
    "description": "",
    "baseUrl": "/ui/",
    "favicon": "public/favicon.ico",
    "projectName": "",
    "appearance": "light",
    "i18n": {
      "allowedLanguages": ["zh-CN", "en-US"],
      "fallbackLng": "zh-CN"
    },
    "templateConfig": {
      "title": "项目标题",
      "logo": { "src": "", "alt": "Logo" },
      "navbar": {
        "position": "sidebar",
        "appearance": "light",
        "items": [
          {
            "to": "/order-management/page/order-form-list",
            "label": "订单管理",
            "label_i18n_zh-CN": "订单管理",
            "label_i18n_en-US": "Order Management",
            "key": "a1b2c3d4",
            "icon": "oct:checklist"
          },
          {
            "to": "",
            "label": "主数据",
            "label_i18n_zh-CN": "主数据",
            "label_i18n_en-US": "Master Data",
            "key": "e5f6a7b8",
            "icon": "solar:database-outline",
            "children": [
              {
                "to": "/master-data/page/product-form-list",
                "label": "产品列表",
                "label_i18n_zh-CN": "产品列表",
                "label_i18n_en-US": "Product List",
                "key": "c9d0e1f2"
              },
              {
                "to": "/master-data/page/supplier-form-list",
                "label": "供应商列表",
                "label_i18n_zh-CN": "供应商列表",
                "label_i18n_en-US": "Supplier List",
                "key": "a3b4c5d6"
              }
            ]
          },
          {
            "to": "",
            "label": "用户与角色",
            "label_i18n_zh-CN": "用户与角色",
            "label_i18n_en-US": "Users and Roles",
            "key": "793341b4",
            "icon": "material:group-sharp",
            "children": [
              {
                "to": "/user-management-new/page/system-user-form-list",
                "label": "用户",
                "label_i18n_zh-CN": "用户",
                "label_i18n_en-US": "User",
                "key": "74e20a44",
                "icon": "oct:passkey-fill"
              },
              {
                "to": "/user-management-new/page/system-role-form-list",
                "label": "角色",
                "label_i18n_zh-CN": "角色",
                "label_i18n_en-US": "Role",
                "key": "ad696683",
                "icon": "material:people-alt-outlined"
              }
            ]
          }
        ]
      },
      "header": {},
      "sidebar": { "collapsible": true },
      "footer": {},
      "login": {
        "template": "LoginCentered",
        "title": "项目标题"
      },
      "userMenu": { "items": [] }
    },
    "mobileTemplateConfig": {
      "carousel": {},
      "subApps": [],
      "navbar": {
        "items": [
          { "to": "/mobile", "label": "", "icon": "home" },
          { "to": "/mobile/my", "label": "", "icon": "person" }
        ]
      },
      "tabBar": { "items": [] },
      "tabBarUsed": false
    },
    "themeConfig": {
      "base": "",
      "token": {},
      "components": {},
      "cssVars": {},
      "css": "",
      "agGridCssVars": {}
    }
  },
  "routers": {
    "desktop": {
      "/": {
        "pageTitle": "",
        "pbcToken": "order-management",
        "schemaId": "entry",
        "isHomePage": true
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/confirm": {
        "pageTitle": "",
        "formType": "FLOW-FORM"
      },
      "/todo-list/form/:flowInstanceId/:flowThreadId/approval": {
        "pageTitle": "",
        "formType": "FLOW-APPROVAL"
      }
    },
    "mobile": {
      "/mobile": {
        "pageTitle": "",
        "isHomePage": true,
        "showNavBar": true
      }
    }
  }
}
```

## 注意事项

- `key` 字段使用 8 位随机十六进制字符串，确保同一菜单树内唯一
- 子菜单项可以省略 `icon`，父菜单建议都设置 `icon`
- `activeBasePath` 用于路由路径与 `to` 不完全匹配时的菜单高亮
- 用户管理模块（`user-management-new`）是平台内置 PBC，大多数项目都会包含
- 连接器管理（`/connector`）和流程挖掘（`/mining/process-mining`）是平台内置功能，按需添加
- 根路由 `/` 的 `pbcToken` 和 `schemaId` 应指向项目的首页 PBC

