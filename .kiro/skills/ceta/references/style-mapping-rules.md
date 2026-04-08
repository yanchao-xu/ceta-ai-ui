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

### 优先级规则

- 布局组件（Stack、Grid、Card、Box）：两者最终合并渲染到同一个 `<div>`，`componentProps.style` 覆盖 `style`
- 输入组件（Input、Select 等）：`style` 作用于外层 wrapper，`componentProps.style` 作用于输入控件本身
- **通用做法**：布局组件的视觉样式放 `componentProps.style`，尺寸/间距放 `style`

---

## 二、各布局组件的样式属性

### Stack（Flex 布局）

Stack 的 flex 属性直接写在 `componentProps` 顶层，不在 `style` 里：

```json
{
  "component": "Stack",
  "componentProps": {
    "flexDirection": "row",
    "alignItems": "center",
    "justifyContent": "space-between",
    "gap": 16,
    "flexWrap": "wrap",
    "style": {
      "padding": "16px 24px",
      "backgroundColor": "#f5f5f5",
      "borderRadius": 8,
      "minHeight": 60
    }
  },
  "style": {
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
| `padding` | `componentProps.style.padding` | 内部样式 |
| `background-color` | `componentProps.style.backgroundColor` | 内部样式 |
| 其他视觉样式 | `componentProps.style.*` | 内部样式 |
| `margin` | `style.margin` | 外层间距 |

### Grid（栅格布局）

Grid 的布局参数在 `componentProps` 顶层，视觉样式在 `componentProps.style`：

```json
{
  "component": "Grid",
  "componentProps": {
    "colNumber": 3,
    "columnGap": 16,
    "rowGap": 12,
    "alignItems": "start",
    "justifyContent": "stretch",
    "style": {
      "padding": "12px",
      "backgroundColor": "#fafafa"
    }
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
| 其他视觉样式 | `componentProps.style.*` | 内部样式 |

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

最简单的容器，样式直接放 `componentProps.style`：

```json
{
  "component": "Box",
  "componentProps": {
    "style": {
      "padding": 16,
      "backgroundColor": "#f0f2f5",
      "borderRadius": 4
    }
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
    "gap": 16,
    "style": {
      "backgroundColor": "#f0f2f5",
      "padding": "24px",
      "minHeight": "100vh"
    }
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
    "justifyContent": "flex-end",
    "style": {
      "padding": "12px 0"
    }
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
    "rowGap": 16,
    "style": {
      "padding": "16px 0"
    }
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

## 六、从 HTML 提取样式的流程

### 步骤 1：识别 inline style

```html
<div style="display: flex; gap: 16px; padding: 24px; background-color: #f5f5f5;">
```

提取所有 `style` 属性中的 CSS 键值对。

### 步骤 2：识别 class 暗示的样式

某些 class 暗示了特定样式，不需要显式提取：

| class | 暗示的样式 | 处理方式 |
|-------|----------|---------|
| `.stack-layout` | `display: flex` | 已由 Stack 组件处理 |
| `.grid-layout` | `display: grid` | 已由 Grid 组件处理 |
| `.card-layout` | 卡片样式 | 已由 Card 组件处理 |
| `.ant-btn-primary` | 主按钮样式 | 映射为 `type: "primary"` |

### 步骤 3：分离布局属性和视觉属性

将提取的 CSS 分为两类：

**布局属性**（映射到 `componentProps` 顶层）：
- `flex-direction`, `align-items`, `justify-content`, `gap`, `flex-wrap`（Stack）
- `grid-template-columns`, CSS 变量（Grid）

**视觉属性**（映射到 `componentProps.style`）：
- `background-color`, `color`, `font-size`, `font-weight`
- `padding`, `border`, `border-radius`, `box-shadow`
- `width`, `height`, `min-height`, `max-width`
- `opacity`, `overflow`, `text-align`

**外层间距**（映射到 `style`）：
- `margin`, `margin-top`, `margin-bottom` 等

### 步骤 4：转换属性名和值

- 属性名：kebab-case → camelCase
- 值：纯 px 数字去单位，其他保留字符串

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
2. 对应的 CSS class 定义，写入 `output/{业务名称}/{业务名称}-theme-css.css`

后续由 ceta-app-config SKILL 将 CSS 合并到 `app.config.json` 的 `themeConfig.css` 中。

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
