# HTML 样式 → CETA schemaJson 样式映射规则

> 本文件定义如何从 HTML 的 CSS 样式中提取信息，并映射到 CETA 组件的 `style` 和 `componentProps.style` 属性。
> 所有组件都支持样式，转换时必须保留原始 HTML 中的视觉样式。

---

## 一、样式的两个位置

CETA 组件有两个可以放样式的位置，作用不同：

| 位置 | 作用 | 渲染目标 |
|------|------|---------|
| `style` | 组件最外层容器样式（含 label 区域） | 组件根 `<div>` |
| `componentProps.style` | 组件内部内容区域样式 | 组件内容 `<div>` |

### 样式位置规则

- 布局组件（Stack、Grid、Box）：所有视觉样式（background、padding、border 等）放在外层 `style`
- Card：卡片有多个样式插槽（`componentProps.style`、`titleStyle`、`contentStyle` 等），详见 Card 章节
- 输入组件（Input、Select 等）：`style` 作用于外层 wrapper，`componentProps.style` 作用于输入控件本身
- Text / Button：所有样式放在外层 `style`

### ⚠️ 关键规则速查（生成 JSON 前必读）

| 组件类型 | background/padding/border/borderRadius 放哪里 | margin/width/height 放哪里 |
|---------|----------------------------------------------|---------------------------|
| Stack / Grid / Box | `style` | `style` |
| Card | `componentProps.style`（卡片整体）+ `componentProps.titleStyle` 等 | `style` |
| Text | `style`（文字样式也放这里：fontSize/fontWeight/color） | `style` |
| Button | `style`（background/color/borderRadius/fontWeight） | `style` |
| Input / Select / DatePicker | `componentProps.style`（控件本身样式） | `style`（外层 wrapper） |

**正确示范**（布局组件视觉样式放在 style 中）：
```json
{ "component": "Stack", "style": { "background": "#f0f4f8", "padding": 24, "borderRadius": 5, "border": "1px solid #dbe9fe" } }
```

**错误示范**（不要把布局组件的视觉样式放在 componentProps.style 中）：
```json
{ "component": "Stack", "componentProps": { "style": { "background": "#f0f4f8", "padding": 24 } } }
```

---

## 二、各布局组件的样式属性

### Stack（Flex 布局）

Stack 的 flex 属性直接写在 `componentProps` 顶层，不在 `style` 里。
其他视觉样式（background、padding、border 等）放在外层 `style`：

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "alignItems": "center",
    "justifyContent": "space-between",
    "gap": 16,
    "flexWrap": "wrap"
  },
  "style": {
    "padding": 15,
    "background": "#f5f5f5",
    "borderRadius": 8,
    "border": "1px solid #e6e7eb",
    "marginBottom": 20
  }
}
```

| HTML CSS 属性 | 映射到 | 说明 |
|--------------|--------|------|
| `flex-direction` | `componentProps.flexDirection` | 顶层属性，不在 style 里 |
| `align-items` | `componentProps.alignItems` | 顶层属性 |
| `justify-content` | `componentProps.justifyContent` | 顶层属性 |
| `gap` | `componentProps.gap` | 顶层属性，数字类型（去掉 px） |
| `flex-wrap` | `componentProps.flexWrap` | 顶层属性 |
| `padding` | `style.padding` | 外层 style |
| `background` / `background-color` | `style.background` | 外层 style |
| `border` | `style.border` | 外层 style |
| `border-radius` | `style.borderRadius` | 外层 style |
| `margin` | `style.margin` | 外层 style |

### Grid（栅格布局）

Grid 的布局参数在 `componentProps` 顶层，视觉样式在外层 `style`：

```json
{
  "component": "Grid",
  "componentProps": {
    "colNumber": 3,
    "columnGap": 16,
    "rowGap": 12,
    "alignItems": "start",
    "justifyContent": "stretch"
  },
  "style": {
    "padding": 12,
    "background": "#fafafa"
  }
}
```

| HTML CSS 属性 | 映射到 | 说明 |
|--------------|--------|------|
| `grid-template-columns` | `componentProps.colNumber` | 从列数推断 |
| `--column-number` | `componentProps.colNumber` | CETA 原生 CSS 变量 |
| `--column-gap` | `componentProps.columnGap` | 数字，去掉 px |
| `--row-gap` | `componentProps.rowGap` | 数字，去掉 px |
| `gap` | `componentProps.gap` | 统一间距 |
| `align-items` | `componentProps.alignItems` | 顶层属性 |
| `justify-content` | `componentProps.justifyContent` | 顶层属性 |
| `justify-items` | `componentProps.justifyItems` | 顶层属性 |
| `align-content` | `componentProps.alignContent` | 顶层属性 |
| 其他视觉样式 | `style.*` | 外层 style |

### Card（卡片容器）

Card 有多个样式插槽：

```json
{
  "component": "Card",
  "componentProps": {
    "title": "基本信息",
    "style": {
      "backgroundColor": "#fff",
      "borderRadius": 8,
      "boxShadow": "0 2px 8px rgba(0,0,0,0.1)"
    },
    "titleStyle": {
      "borderBottom": "1px solid #e8e8e8",
      "padding": "12px 16px"
    },
    "titleTextStyle": {
      "fontSize": 16,
      "fontWeight": 600,
      "color": "#1a1a1a"
    },
    "titleDescStyle": {
      "fontSize": 12,
      "color": "#999"
    },
    "contentStyle": {
      "padding": "16px"
    }
  },
  "style": {
    "marginBottom": 20
  }
}
```

| 样式目标 | 映射到 | 说明 |
|---------|--------|------|
| 卡片整体背景/边框/阴影 | `componentProps.style` | 卡片根容器 |
| 标题栏区域 | `componentProps.titleStyle` | `.card-title` div |
| 标题文字 | `componentProps.titleTextStyle` | `.title-text` div |
| 描述文字 | `componentProps.titleDescStyle` | `.title-desc` div |
| 内容区域 | `componentProps.contentStyle` | `.card-content` div |
| 外层间距 | `style` | 组件根容器 |

### Box（盒子容器）

最简单的容器，样式放在外层 `style`：

```json
{
  "component": "Box",
  "style": {
    "padding": 16,
    "background": "#f0f2f5",
    "borderRadius": 4
  }
}
```

---

## 三、CSS 属性名转换规则

HTML 的 CSS 属性名使用 kebab-case，JSON 中使用 camelCase：

| CSS (kebab-case) | JSON (camelCase) |
|------------------|-----------------|
| `background-color` | `backgroundColor` |
| `font-size` | `fontSize` |
| `font-weight` | `fontWeight` |
| `line-height` | `lineHeight` |
| `text-align` | `textAlign` |
| `border-radius` | `borderRadius` |
| `box-shadow` | `boxShadow` |
| `margin-bottom` | `marginBottom` |
| `padding-left` | `paddingLeft` |
| `min-height` | `minHeight` |
| `max-width` | `maxWidth` |
| `flex-direction` | `flexDirection` |
| `align-items` | `alignItems` |
| `justify-content` | `justifyContent` |
| `border-bottom` | `borderBottom` |
| `white-space` | `whiteSpace` |
| `overflow-x` | `overflowX` |
| `z-index` | `zIndex` |

---

## 四、CSS 值转换规则

### 数字值

纯数字的 px 值去掉单位，转为数字类型：

| CSS 值 | JSON 值 | 说明 |
|--------|---------|------|
| `16px` | `16` | 纯数字 |
| `1.5em` | `"1.5em"` | 非 px 保留字符串 |
| `100%` | `"100%"` | 百分比保留字符串 |
| `0` | `0` | 数字 |
| `auto` | `"auto"` | 字符串 |
| `8px 16px` | `"8px 16px"` | 复合值保留字符串 |
| `0 0 20px` | `"0 0 20px"` | 复合值保留字符串 |

### 颜色值

颜色值保留为字符串：

| CSS 值 | JSON 值 |
|--------|---------|
| `#f5f5f5` | `"#f5f5f5"` |
| `rgb(0,0,0)` | `"rgb(0,0,0)"` |
| `rgba(0,0,0,0.1)` | `"rgba(0,0,0,0.1)"` |
| `transparent` | `"transparent"` |
| `red` | `"red"` |

---

## 五、常见样式场景

### 场景 1：页面级背景色

HTML 中页面有背景色时，用最外层 Stack 包裹：

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "column",
    "gap": 16
  },
  "style": {
    "background": "#f0f2f5",
    "padding": 24,
    "minHeight": "100vh"
  },
  "fields": [ ... ]
}
```

### 场景 2：水平排列的按钮组

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "gap": 8,
    "justifyContent": "flex-end"
  },
  "style": {
    "padding": "12px 0"
  },
  "fields": [
    { "component": "Button", "componentProps": { "content": "取消" } },
    { "component": "Button", "componentProps": { "content": "提交", "type": "primary" } }
  ]
}
```

### 场景 3：带样式的信息展示区

```json
{
  "component": "Card",
  "componentProps": {
    "title": "订单摘要",
    "style": {
      "backgroundColor": "#e6f7ff",
      "border": "1px solid #91d5ff",
      "borderRadius": 8
    },
    "titleTextStyle": {
      "color": "#1890ff",
      "fontSize": 16,
      "fontWeight": 600
    },
    "contentStyle": {
      "padding": "16px 24px"
    }
  },
  "fields": [ ... ]
}
```

### 场景 4：等宽多列布局带间距

```json
{
  "component": "Grid",
  "componentProps": {
    "colNumber": 3,
    "columnGap": 24,
    "rowGap": 16
  },
  "style": {
    "padding": "16px 0"
  },
  "fields": [ ... ]
}
```

### 场景 5：组件自身的宽度/高度控制

输入组件通过 `style` 控制外层尺寸：

```json
{
  "component": "Input",
  "id": "name",
  "title": "姓名",
  "style": {
    "width": "50%"
  }
}
```

Grid 子项通过 `span` 控制占列数（优先用 span，不用 width）：

```json
{
  "component": "Grid",
  "componentProps": { "colNumber": 12 },
  "fields": [
    { "component": "Input", "id": "name", "title": "姓名", "span": 6 },
    { "component": "Input", "id": "phone", "title": "电话", "span": 6 }
  ]
}
```

---

## 六、从 HTML 提取样式的完整流程

### 最高原则：忠实还原，不做替换

**从 HTML 提取的样式必须原样写入 JSON，不能为了"全局主题一致性"而替换颜色或修改视觉效果。**

- Tailwind `bg-emerald-500` 解析为 `#10b981`，就写 `#10b981`，不要替换成项目主色 `#003366`
- HTML 中是 `from-emerald-50 to-teal-50` 绿色渐变，就写绿色渐变，不要改成蓝色渐变
- HTML 中某个元素只是普通粗体文字，就只设置 fontWeight，不要自作主张加圆形背景
- 两个不同背景色的 `<span>` 必须生成两个独立的 Text 组件，不能合并成一个

### 前置步骤：CSS 预处理（最重要，必须在分析 HTML 结构之前完成）

HTML 中的样式可能来自三个地方，必须全部处理，否则会丢失样式：

| 样式来源 | 识别方式 | 处理方法 |
|---------|---------|---------|
| `<style>` 标签 | HTML 中存在 `<style>...</style>` | 解析 CSS 规则，建立 class→样式映射表 |
| inline style | 元素有 `style="..."` 属性 | 直接提取键值对 |
| 纯 class 引用 | 元素有 `class="..."` 但无对应 `<style>` 或 inline style | 询问用户提供 CSS，或根据语义推断 |

#### 步骤 0.1：提取 `<style>` 标签中的 CSS 规则

如果 HTML 中包含 `<style>` 标签，必须先解析其中的 CSS 规则：

```html
<style>
  .header-bar { background: linear-gradient(135deg, #003366, #004080); padding: 12px 24px; }
  .header-bar .title { color: white; font-size: 18px; font-weight: 700; }
  .search-card { border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
  .result-item { padding: 14px 20px; border-bottom: 1px solid #f0f0f0; cursor: pointer; }
  .result-item:hover { background-color: #f8fafc; }
  .pnr-code { font-family: monospace; letter-spacing: 2px; font-weight: 700; color: #003366; }
  .btn-primary { background: #003366; color: white; border-radius: 8px; font-weight: 600; }
</style>
```

解析为映射表：

```
cssMap = {
  ".header-bar":         { background: "linear-gradient(135deg, #003366, #004080)", padding: "12px 24px" }
  ".header-bar .title":  { color: "white", fontSize: 18, fontWeight: 700 }
  ".search-card":        { borderRadius: 12, boxShadow: "0 1px 3px rgba(0,0,0,0.1)" }
  ".result-item":        { padding: "14px 20px", borderBottom: "1px solid #f0f0f0", cursor: "pointer" }
  ".result-item:hover":  { backgroundColor: "#f8fafc" }
  ".pnr-code":           { fontFamily: "monospace", letterSpacing: 2, fontWeight: 700, color: "#003366" }
  ".btn-primary":        { background: "#003366", color: "white", borderRadius: 8, fontWeight: 600 }
}
```

#### 步骤 0.2：对每条 CSS 规则做 inline style vs className 分类

| 条件 | 处理方式 | 原因 |
|------|---------|------|
| 简单 class 选择器（`.search-card`），无伪类，引用次数少 | → 转为 inline style | 直接写入组件的 `style` / `componentProps.style` |
| 包含伪类（`.result-item:hover`） | → 保留为 className | inline style 无法实现伪类 |
| 包含子元素选择器（`.header-bar .title`） | → 保留为 className | 需要 CSS 选择器能力 |
| 同一 class 被 3+ 个元素引用 | → 保留为 className | 复用样式，避免重复 |
| 包含 @keyframes 动画 | → 保留为 className | inline style 不支持 |

分类结果：

```
转为 inline style（写入组件 JSON）：
  .header-bar     → 对应 HTML 元素的 componentProps.style / style
  .search-card    → 对应 HTML 元素的 componentProps.style
  .pnr-code       → 对应 HTML 元素的 style
  .btn-primary    → 对应 HTML 元素的 style

保留为 className（写入 themeConfig.css）：
  .result-item:hover  → className="result-item" + themeConfig.css 中定义 hover 规则
  .header-bar .title  → className="header-bar" + themeConfig.css 中定义子元素规则
```

#### 步骤 0.3：生成组件 JSON 时应用样式

转换 HTML 元素为 CETA 组件时，查阅 cssMap 填入样式：

```html
<!-- HTML -->
<div class="header-bar">
  <span class="header-bar title">航空订座系统</span>
</div>
```

```json
{
  "component": "Stack",
  "className": "header-bar",
  "componentProps": {
    "flexDirection": "row",
    "style": {
      "background": "linear-gradient(135deg, #003366, #004080)",
      "padding": "12px 24px"
    }
  },
  "fields": [
    {
      "component": "Text",
      "componentProps": { "content": "航空订座系统" },
      "style": { "color": "white", "fontSize": 18, "fontWeight": 700 }
    }
  ]
}
```

注意：即使 `.header-bar` 的基础样式已转为 inline，如果它还有子元素选择器规则（`.header-bar .title`），
仍然需要在组件上保留 `className="header-bar"`，因为 themeConfig.css 中的子元素规则需要这个 class 才能生效。

#### 步骤 0.4：将 className 规则写入 themeConfig.css

保留为 className 的 CSS 规则，最终写入 `app-config.json` 的 `themeConfig.css`：

```json
{
  "themeConfig": {
    "css": ".result-item:hover {\n  background-color: #f8fafc;\n}\n\n.header-bar.header-bar .title {\n  color: white;\n  font-size: 18px;\n  font-weight: 700;\n}"
  }
}
```

### 步骤 1：识别 inline style

```html
<div style="display: flex; gap: 16px; padding: 24px; background-color: #f5f5f5;">
```

提取所有 `style` 属性中的 CSS 键值对。

**注意**：如果同一个元素既有 inline style 又有 class 引用，两者都要提取。
inline style 优先级更高，会覆盖 class 中的同名属性。

### 步骤 2：合并 class 样式和 inline style

对每个 HTML 元素：
1. 从 cssMap 中查找该元素的 class 对应的样式（步骤 0 的结果）
2. 提取该元素的 inline style
3. 合并两者（inline style 覆盖 class 样式中的同名属性）
4. 将合并后的样式写入组件 JSON

### 步骤 3：识别 class 暗示的样式

某些 class 暗示了特定样式，不需要显式提取：

| class | 暗示的样式 | 处理方式 |
|-------|----------|---------|
| `.stack-layout` | `display: flex` | 已由 Stack 组件处理 |
| `.grid-layout` | `display: grid` | 已由 Grid 组件处理 |
| `.card-layout` | 卡片样式 | 已由 Card 组件处理 |
| `.ant-btn-primary` | 主按钮样式 | 映射为 `type: "primary"` |

### 步骤 4：分离布局属性和视觉属性

将提取的 CSS 分为三类：

**布局属性**（映射到 `componentProps` 顶层）：
- `flex-direction`, `align-items`, `justify-content`, `gap`, `flex-wrap`（Stack）
- `grid-template-columns`, CSS 变量（Grid）

**视觉属性**（映射到外层 `style`，Card 除外）：
- `background-color` / `background`, `color`, `font-size`, `font-weight`
- `padding`, `border`, `border-radius`, `box-shadow`
- `width`, `height`, `min-height`, `max-width`
- `margin`, `margin-top`, `margin-bottom` 等
- `opacity`, `overflow`, `text-align`

**Card 特殊处理**：Card 的卡片整体样式放 `componentProps.style`，标题/内容区域用 `titleStyle`/`contentStyle` 等。

**输入组件特殊处理**：输入控件本身的样式（background、border 等）放 `componentProps.style`，外层尺寸放 `style`。

### 步骤 5：转换属性名和值

- 属性名：kebab-case → camelCase
- 值：纯 px 数字去单位，其他保留字符串

---

## 六-B、样式来源缺失时的处理策略

当 HTML 中的 class 引用找不到对应的 CSS 定义时（无 `<style>` 标签、无 inline style），
按以下优先级处理：

### 策略 0：从设计稿图片还原（最高优先级）

**如果用户同时提供了设计稿图片/截图，直接从图片还原样式，不需要询问用户。**

图片是视觉真相来源。HTML 只提供 DOM 结构（元素类型、嵌套关系、文本内容），
样式（颜色、间距、字体、布局比例等）全部从图片中提取。

从图片还原时的注意事项：
- 逐区块对照：HTML 的每个主要区块对应图片中的哪个视觉区域
- 颜色要准确：从图片中识别具体的颜色值（如 #10b981 而非笼统的"绿色"）
- 布局要准确：如果图片显示两栏布局，即使 HTML 结构是线性的，也要用 Grid/Stack 还原两栏
- Badge/标签要还原：图片中的彩色标签（如"成人"、"SS2"）必须用 Text + 背景色还原
- 圆形编号要还原：图片中的圆形数字编号必须用 Text + borderRadius: "50%" 还原

### 策略 1：询问用户（推荐，当无图片时）

```
"HTML 中引用了以下 CSS class 但未包含样式定义：
  .header-bar, .search-card, .result-item, .btn-primary
能否提供对应的 CSS 文件？这样我可以准确还原样式。"
```

### 策略 2：根据 class 名语义推断

如果用户无法提供 CSS，根据 class 名的语义推断常见样式：

| class 名模式 | 推断的样式 |
|-------------|-----------|
| `*-primary`, `*-blue` | 主色调背景/文字色 |
| `*-card`, `*-panel` | 白色背景、圆角、阴影 |
| `*-header`, `*-bar` | 深色背景、白色文字 |
| `*-muted`, `*-secondary` | 灰色文字 |
| `*-sm`, `*-small` | 小字号 |
| `*-lg`, `*-large` | 大字号 |
| `*-bold`, `*-strong` | 粗体 |
| `*-rounded` | 圆角 |
| `*-shadow` | 阴影 |
| `*-border` | 边框 |

**重要**：推断的样式必须在 analysis.json 中标注 `"styleSource": "inferred"`，
告知用户这些样式是推断的，可能需要调整。

**禁止在推断时做"全局主题统一"**：推断只能基于 class 名本身的语义，不能因为"项目主色是蓝色"就把所有推断的颜色都改成蓝色。

### 策略 3：从 HTML 结构和上下文推断

即使没有 CSS 定义，也可以从 HTML 结构推断一些样式：

| HTML 结构特征 | 推断的样式 |
|-------------|-----------|
| 元素是页面顶部的 header/nav | 深色背景、白色文字、固定高度 |
| 元素包含 `<input>` 和 `<button>` | 表单区域，白色背景、边框、内边距 |
| 元素是卡片容器（包裹独立内容块） | 白色背景、圆角、阴影、内边距 |
| 元素是列表项（重复结构） | 底部边框分隔、hover 效果 |
| 元素是按钮且文字暗示主操作 | primary 类型、主色调背景 |

---

## 七、className 与 themeConfig.css（全局 CSS class）

除了 inline style，CETA 还支持通过 `className` 引用全局 CSS class。这些 class 定义在 `app.config.json` 的 `appConfig.themeConfig.css` 字段中，是一个原始 CSS 字符串，平台启动时注入到页面 `<style>` 标签中全局生效。

### 两种样式方式对比

| 方式 | 适用场景 | 位置 |
|------|---------|------|
| inline style（`style` / `componentProps.style`） | 单个组件的一次性样式 | schemaJson 组件节点 |
| className（`className` / `componentProps.className`） | 可复用样式、伪类（:hover）、子元素选择器、动画 | schemaJson 引用 + themeConfig.css 定义 |

### className 在 schemaJson 中的使用

组件有两个 className 位置，与 style 对应：

| 位置 | 作用 |
|------|------|
| `className` | 组件根层 class，与 `style` 同级 |
| `componentProps.className` | 组件内部 class，与 `componentProps.style` 同级 |

两者最终都会合并到组件渲染的 `<div>` 的 `class` 属性上。

```json
{
  "component": "Card",
  "className": "bg-header-card",
  "componentProps": {
    "title": "维护订单"
  },
  "fields": [ ... ]
}
```

### themeConfig.css 定义

CSS class 在 `app.config.json` 的 `appConfig.themeConfig.css` 中定义：

```json
{
  "appConfig": {
    "themeConfig": {
      "css": ".bg-header-card {\n  overflow: hidden;\n}\n\n.bg-header-card.bg-header-card .card-title {\n  background: #5c6bc0;\n  color: white;\n  padding-top: 8px;\n  padding-bottom: 6px;\n}\n\n.bg-header-card.bg-header-card .card-title .title-text {\n  font-size: 14px;\n}"
    }
  }
}
```

格式化后的 CSS 内容：

```css
.bg-header-card {
  overflow: hidden;
}

.bg-header-card.bg-header-card .card-title {
  background: #5c6bc0;
  color: white;
  padding-top: 8px;
  padding-bottom: 6px;
}

.bg-header-card.bg-header-card .card-title .title-text {
  font-size: 14px;
}
```

注意：CSS 字符串中使用 `\n` 换行、`\r\n` 也可以。

### themeConfig 完整结构

```json
{
  "appConfig": {
    "themeConfig": {
      "base": "",
      "token": {
        "colorPrimary": "#1a237e",
        "borderRadius": 6
      },
      "components": {
        "Menu": {
          "colorBgContainer": "#23164B",
          "itemSelectedBg": "#392B63",
          "itemColor": "#FFF"
        }
      },
      "cssVars": {
        "--card-padding": "20px",
        "--helper-color": "rgb(178,109,10)"
      },
      "css": "/* 全局自定义 CSS class */",
      "agGridCssVars": {
        "--ag-header-background-color": "white"
      }
    }
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `base` | string | 基础主题色名（如 `"orange"`），可为空 |
| `token` | object | Ant Design 5 全局 token（colorPrimary、borderRadius 等） |
| `components` | object | Ant Design 5 组件级 token（按组件名分组） |
| `cssVars` | object | 自定义 CSS 变量，全局可用 |
| `css` | string | 自定义 CSS 字符串，全局注入 |
| `agGridCssVars` | object | AG Grid 专用 CSS 变量 |

### 何时用 className 而非 inline style

优先用 className 的场景：

1. **伪类样式**（:hover、:focus、:active）— inline style 无法实现
   ```css
   .company-card:hover {
     position: relative;
     top: -10px;
   }
   ```

2. **子元素选择器** — 需要控制组件内部子元素的样式
   ```css
   .bg-header-card.bg-header-card .card-title {
     background: #5c6bc0;
     color: white;
   }
   ```
   注意：重复 class（`.bg-header-card.bg-header-card`）用于提高优先级覆盖默认样式。

3. **多组件复用** — 同一套样式被多个组件引用
   ```json
   { "component": "Card", "className": "bg-header-card", ... }
   { "component": "Card", "className": "bg-header-card", ... }
   ```

4. **动画** — @keyframes 只能在 CSS 中定义
   ```css
   @keyframes pulse {
     0%, 100% { opacity: 0.5; }
     50% { opacity: 1; }
   }
   ```

5. **CSS 变量引用** — 使用 themeConfig.cssVars 中定义的变量
   ```json
   { "style": { "color": "var(--primary-color)", "padding": "var(--card-padding)" } }
   ```

### className 命名规范

- 使用 kebab-case（如 `bg-header-card`、`cell-brown`、`job-information-status-info`）
- 建议加业务前缀避免冲突（如 `order-status-badge`、`flight-card-header`）
- 不要使用平台已有的 class 名（如 `card-layout`、`stack-layout`、`form-layout`）

### 输出规范

当转换 HTML 时，如果识别到需要 className 的场景（伪类、子元素选择器、复用样式），需要同时输出：

1. schemaJson 中组件的 `className` 引用
2. 对应的 CSS class 定义，统一写入 `app-config.json` 的 `themeConfig.css` 字段

**不要单独生成 CSS 文件。** 所有 className 对应的 CSS 规则都写入 `app-config.json` 的 `themeConfig.css`。

#### className + themeConfig.css 的完整示例

假设 HTML 中有以下样式需要保留为 className：

```html
<style>
  .flight-result:hover { background-color: #f0f7ff; transform: translateY(-1px); }
  .flight-result .flight-code { font-family: monospace; font-weight: 700; }
  .status-badge { display: inline-block; padding: 2px 8px; border-radius: 4px; font-size: 12px; }
  .status-confirmed { background: #ecfdf5; color: #059669; }
  .status-pending { background: #fffbeb; color: #d97706; }
</style>
```

schemaJson 中的引用：

```json
{
  "component": "Stack",
  "className": "flight-result",
  "componentProps": { "flexDirection": "row" },
  "fields": [
    {
      "component": "Text",
      "className": "status-badge status-confirmed",
      "componentProps": { "content": "HK2" }
    }
  ]
}
```

app-config.json 中的 themeConfig.css：

```json
{
  "themeConfig": {
    "css": ".flight-result:hover {\n  background-color: #f0f7ff;\n  transform: translateY(-1px);\n}\n\n.flight-result.flight-result .flight-code {\n  font-family: monospace;\n  font-weight: 700;\n}\n\n.status-badge {\n  display: inline-block;\n  padding: 2px 8px;\n  border-radius: 4px;\n  font-size: 12px;\n}\n\n.status-confirmed {\n  background: #ecfdf5;\n  color: #059669;\n}\n\n.status-pending {\n  background: #fffbeb;\n  color: #d97706;\n}"
  }
}
```

---

## 八、应忽略的样式

以下样式由组件/平台自动处理，不需要提取到 JSON：

| 样式 | 原因 |
|------|------|
| `display: flex` | Stack 组件自带 |
| `display: grid` | Grid 组件自带 |
| `box-sizing: border-box` | 全局默认 |
| `font-family` | 平台全局字体 |
| `outline`, `cursor` | 交互样式由组件处理 |
| `-webkit-*` 前缀 | 平台 PostCSS 自动添加 |
| `transition`, `animation` | 平台统一动画 |
| `position: relative`（无配合属性时） | 默认行为 |
