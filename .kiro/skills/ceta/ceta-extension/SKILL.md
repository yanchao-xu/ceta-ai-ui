# CETA Extension 扩展组件开发

## 职责

本 SKILL 负责引导用户创建和开发 CETA Extension（扩展组件）项目。
当用户需要实现 CETA 内置组件无法覆盖的自定义 UI 时，使用本 SKILL。

## 何时加载本 SKILL

满足以下任一条件时加载：

- 用户明确说要创建 Extension / 扩展组件
- HTML 转换过程中发现无法映射到 CETA 内置组件的复杂 UI（如图表、自定义编辑器、第三方组件）
- 用户提到需要嵌入自定义 React/JS 组件到 CETA 表单或页面中

## 前置知识

加载本 SKILL 前应已加载 `ceta-basic`。
组件配置规范见 `skills/ceta/references/components/wrapper/Extension.md`。

---

## 流程总览

```
用户说要创建扩展组件
│
├── Step 1: 收集信息
│   ├── 组件名称（kebab-case，如 gantt-chart）
│   └── 组件用途简述
│
├── Step 2: 创建项目目录
│   └── 在工作区根目录下创建 extensions/{组件名}/ 文件夹
│
├── Step 3: 运行脚手架（交互式命令，需用户手动执行）
│   ├── 提示用户在终端中运行命令
│   └── 等待用户确认脚手架已完成
│
├── Step 4: 安装依赖
│   └── 在项目目录下运行 npm install
│
├── Step 5: 读取项目结构 + icp-extension.md
│   └── 理解脚手架生成的代码结构和开发规范
│
└── Step 6: 进入组件开发对话
    ├── 根据用户需求编写 mount 函数
    ├── 如需充当表单字段，实现 fieldApi 集成
    ├── 编写样式（CSS）
    └── 指导用户本地调试和上传到 CETA
```

---

## Step 1: 收集信息

向用户询问以下信息：

1. **组件名称**：用于创建文件夹和项目名，使用 kebab-case（如 `gantt-chart`、`code-editor`、`signature-pad`）
2. **组件用途**：简要描述组件要做什么（如"甘特图展示项目进度"、"代码编辑器支持语法高亮"）

可选信息（不问也行，后续开发时再确定）：
- 是否需要充当表单字段（即是否需要 fieldApi）
- 是否需要接收 extensionParams

---

## Step 2: 创建项目目录

在工作区根目录下创建文件夹：

```bash
mkdir -p extensions/{组件名}
```

例如用户要创建甘特图组件：`mkdir -p extensions/gantt-chart`

---

## Step 3: 运行脚手架

**脚手架是交互式命令，AI 无法直接代替用户完成交互。必须提示用户手动执行。**

向用户展示以下信息：

```
请在终端中执行以下命令来初始化扩展组件项目：

cd extensions/{组件名}
npx --registry https://nexus.dev.bizops.com.cn/repository/npm-group/ create-icp-extension@latest

脚手架会询问一些配置问题，按提示回答即可。
完成后告诉我，我来帮你安装依赖并开始开发。
```

**注意：必须带 `--registry` 参数指向内部 npm 仓库，否则找不到包。**
完整命令：`npx --registry https://nexus.dev.bizops.com.cn/repository/npm-group/ create-icp-extension@latest`

**等待用户确认脚手架已完成后再继续。**

如果用户不确定脚手架的某个问题怎么回答，可以给出建议：
- 项目名：默认即可（就是文件夹名）
- 模式选择：推荐选「远端通用」（remote-common），开发最灵活
- 其他选项：按默认值即可

---

## Step 4: 安装依赖

用户确认脚手架完成后，自动运行：

```bash
npm install
```

工作目录为 `extensions/{组件名}`。

---

## Step 5: 读取项目结构

安装完成后，执行以下操作：

1. **列出项目目录结构**：了解脚手架生成了哪些文件
2. **读取 `icp-extension.md`**：这是脚手架生成的 AI 提示词文件，包含该项目的开发规范和 API 说明
3. **读取入口文件**（通常是 `src/index.js` 或 `src/main.js`）：了解 mount 函数的模板代码
4. **读取 `package.json`**：了解可用的脚本命令（dev、build 等）

**重点：`icp-extension.md` 是后续开发的核心参考文档，必须读取并遵循其中的规范。**

---

## Step 6: 组件开发

根据用户描述的组件用途，在对话中引导完成开发。

### 开发要点

#### mount 函数结构

所有远端通用模式的 Extension 都通过 default export 的 mount 函数工作：

```javascript
export default function mount(element, { params, formApi, messageApi, restApi, i18nApi, routerApi }) {
  // 1. 使用 params 获取 extensionParams 传入的参数
  // 2. 在 element 上渲染自定义 UI
  // 3. 绑定事件、调用 API 等

  // 返回 unmount 函数或 Handle 对象
  return function unmount() {
    // 清理事件监听、定时器等
  };
}
```

#### 如果需要充当表单字段

mount 函数会额外收到 `fieldApi` 参数：

```javascript
export default function mount(element, { params, formApi, messageApi, restApi, i18nApi, routerApi, fieldApi }) {
  // fieldApi.setValue(value)  — 设置字段值
  // fieldApi.setTouched()     — 标记已操作，触发校验
  // fieldApi.id               — 字段 id
  // fieldApi.name             — 字段 name
  // fieldApi.defaultValue     — 默认值

  return {
    unmount() { /* 清理 */ },
    updateValue(value) { /* 外部更新值时调用 */ },
    validate(value) { /* 返回 true/false 或抛异常 */ }
  };
}
```

#### 本地调试

开发阶段可以：
1. 运行 `npm run dev` 启动本地 dev server
2. 在 CETA 表单设计器中添加 Extension 组件
3. 选择「远端通用」模式，将本地 dev server 地址填入 `extensionJsUrl`
4. 实时预览和调试

提示用户手动运行 `npm run dev`，不要用 AI 启动 dev server。

#### 构建和部署

开发完成后：
1. 运行 `npm run build` 构建产物
2. 如果使用 CETA 托管模式：通过表单设计器 GUI 上传 JS 和 CSS 文件
3. 如果使用远端通用模式：将构建产物部署到自己的服务器

---

## 与 HTML 转换流程的集成

当 `ceta-html-analyzer` 在转换 HTML 时遇到无法映射到 CETA 内置组件的元素（如 `<canvas>`、图表库、富交互控件），应：

1. 在 analysis.json 中标注该区域需要 Extension 组件
2. 在生成的 schemaJson 中放置 Extension 占位：

```json
{
  "component": "Extension",
  "componentProps": {
    "extensionType": "remote-common",
    "extensionJsUrl": "TODO: 待开发",
    "extensionParams": {}
  }
}
```

3. 提示用户：「这个区域需要自定义扩展组件，是否现在创建 Extension 项目？」
4. 如果用户同意，加载本 SKILL 开始创建流程

---

## 与 ceta-basic 的关系

```
用户要创建扩展组件
      │
      ▼
ceta-basic（入口 SKILL）
      │ 识别任务类型为"创建 Extension"
      ▼
ceta-extension（本 SKILL）
      │ 引导创建项目 → 安装依赖 → 开发组件
      │
      ├── 参考：skills/ceta/references/components/wrapper/Extension.md（组件配置规范）
      └── 参考：extensions/{组件名}/icp-extension.md（脚手架生成的开发规范）
```
