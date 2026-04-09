# Tailwind CSS → CETA Style 解析速查表

> 本文件提供 Tailwind CSS utility class 到 CSS 属性值的速查表。
> 强制逐元素解析流程定义在 `ceta-html-analyzer/SKILL.md` 的 Step 1.3 中，适用于所有样式来源。
> 本文件仅在 HTML 使用 Tailwind CSS 时作为速查参考。
>
> **颜色值：** Tailwind 颜色体系是公开标准知识，AI 已知全部颜色值。
> 本文件不列举所有颜色，只提供 class 模式 → CSS 属性的映射规则。
> 如果不确定某个颜色值，标注 `"_unsure": "class-name"` 并告知用户，不要猜。

---

## 解析规则速查

### 颜色类 class

| class 模式 | CSS 属性 | 说明 |
|-----------|---------|------|
| `bg-{color}-{shade}` | `background-color` | 背景色 |
| `text-{color}-{shade}` | `color` | 文字色 |
| `border-{color}-{shade}` | `border-color` | 边框色 |
| `ring-{color}-{shade}` | `box-shadow`（ring） | 环形阴影色 |
| `from-{color}-{shade}` | 渐变起始色 | 配合 `bg-gradient-to-*` |
| `to-{color}-{shade}` | 渐变结束色 | 配合 `bg-gradient-to-*` |
| `via-{color}-{shade}` | 渐变中间色 | 可选 |

颜色值查 Tailwind 官方色板（AI 已知），不要猜。
如果不确定某个颜色值，标注 `"_unsure": "class-name"` 并告知用户。

### 间距类 class

| class | CSS 值 | 说明 |
|-------|--------|------|
| `p-{n}` | `padding: {n*4}px` | 全方向内边距 |
| `px-{n}` | `padding-left/right: {n*4}px` | 水平内边距 |
| `py-{n}` | `padding-top/bottom: {n*4}px` | 垂直内边距 |
| `pt/pr/pb/pl-{n}` | 单方向 padding | |
| `m-{n}` | `margin: {n*4}px` | 全方向外边距 |
| `mx/my/mt/mr/mb/ml-{n}` | 单方向 margin | |
| `gap-{n}` | `gap: {n*4}px` | flex/grid 间距 |
| `space-x-{n}` | 子元素水平间距 | 映射为 gap |
| `space-y-{n}` | 子元素垂直间距 | 映射为 gap |

特殊值：`p-0.5` = 2px, `p-1` = 4px, `p-1.5` = 6px, `p-2` = 8px, `p-2.5` = 10px, `p-3` = 12px, `p-3.5` = 14px, `p-4` = 16px, `p-5` = 20px, `p-6` = 24px, `p-8` = 32px, `p-10` = 40px, `p-12` = 48px, `p-16` = 64px

### 圆角类 class

| class | CSS 值 |
|-------|--------|
| `rounded-none` | 0 |
| `rounded-sm` | 2px |
| `rounded` | 4px |
| `rounded-md` | 6px |
| `rounded-lg` | 8px |
| `rounded-xl` | 12px |
| `rounded-2xl` | 16px |
| `rounded-3xl` | 24px |
| `rounded-full` | 9999px（或 "50%"） |

### 字体类 class

| class | CSS 值 |
|-------|--------|
| `text-xs` | fontSize: 12 |
| `text-sm` | fontSize: 14 |
| `text-base` | fontSize: 16 |
| `text-lg` | fontSize: 18 |
| `text-xl` | fontSize: 20 |
| `text-2xl` | fontSize: 24 |
| `text-3xl` | fontSize: 30 |
| `font-thin` | fontWeight: 100 |
| `font-light` | fontWeight: 300 |
| `font-normal` | fontWeight: 400 |
| `font-medium` | fontWeight: 500 |
| `font-semibold` | fontWeight: 600 |
| `font-bold` | fontWeight: 700 |
| `font-extrabold` | fontWeight: 800 |

### 布局类 class

| class | CSS 属性 | CETA 映射 |
|-------|---------|----------|
| `flex` | display: flex | → Stack 组件 |
| `flex-row` | flex-direction: row | componentProps.flexDirection: "row" |
| `flex-col` | flex-direction: column | componentProps.flexDirection: "column" |
| `items-center` | align-items: center | componentProps.alignItems: "center" |
| `items-start` | align-items: flex-start | componentProps.alignItems: "flex-start" |
| `items-end` | align-items: flex-end | componentProps.alignItems: "flex-end" |
| `justify-center` | justify-content: center | componentProps.justifyContent: "center" |
| `justify-between` | justify-content: space-between | componentProps.justifyContent: "space-between" |
| `justify-end` | justify-content: flex-end | componentProps.justifyContent: "flex-end" |
| `flex-wrap` | flex-wrap: wrap | componentProps.flexWrap: "wrap" |
| `flex-1` | flex: 1 1 0% | style.flex: 1 |
| `flex-shrink-0` | flex-shrink: 0 | style.flexShrink: 0 |
| `grid` | display: grid | → Grid 组件 |
| `grid-cols-{n}` | grid-template-columns | componentProps.colNumber: n |

### 尺寸类 class

| class | CSS 值 |
|-------|--------|
| `w-full` | width: "100%" |
| `w-{n}` | width: {n*4}px（如 w-12 = 48px） |
| `h-{n}` | height: {n*4}px |
| `min-h-screen` | minHeight: "100vh" |
| `max-w-{size}` | maxWidth（sm=384,md=448,lg=512,xl=576,2xl=672） |

### 边框/阴影类 class

| class | CSS 值 |
|-------|--------|
| `border` | border: 1px solid（需配合 border-color） |
| `border-2` | border: 2px solid |
| `border-t/r/b/l` | 单方向边框 |
| `shadow-sm` | boxShadow: "0 1px 2px rgba(0,0,0,0.05)" |
| `shadow` | boxShadow: "0 1px 3px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06)" |
| `shadow-md` | boxShadow: "0 4px 6px rgba(0,0,0,0.1)" |
| `shadow-lg` | boxShadow: "0 10px 15px rgba(0,0,0,0.1)" |

### 其他常用 class

| class | CSS 值 |
|-------|--------|
| `truncate` | overflow: hidden, textOverflow: ellipsis, whiteSpace: nowrap |
| `overflow-hidden` | overflow: hidden |
| `cursor-pointer` | cursor: pointer |
| `opacity-{n}` | opacity: n/100 |
| `transition` | transition（忽略，CETA 不支持 inline transition） |
| `hidden` | display: none → hidden: true |
| `uppercase` | textTransform: uppercase |
| `lowercase` | textTransform: lowercase |
| `italic` | fontStyle: italic |
| `underline` | textDecoration: underline |
| `line-through` | textDecoration: line-through |
| `text-center` | textAlign: center |
| `text-right` | textAlign: right |
| `whitespace-nowrap` | whiteSpace: nowrap |
| `whitespace-pre-line` | whiteSpace: pre-line |

---

## 不确定时的处理

如果遇到无法确定值的 Tailwind class：

1. **不要猜** — 不要用"差不多"的值替代
2. **标注** — 在 JSON 中加 `"_unknownClass": "class-name"`
3. **告知用户** — 在 analysis.json 或确认摘要中说明哪些 class 未能解析

---

## 伪类和响应式 class 的处理

| class 模式 | 处理方式 |
|-----------|---------|
| `hover:*` | → className + themeConfig.css |
| `focus:*` | → className + themeConfig.css |
| `active:*` | → className + themeConfig.css |
| `sm:/md:/lg:/xl:` | 忽略响应式前缀，取最大断点的值 |
| `dark:*` | 忽略暗色模式（除非用户要求） |
| `group-hover:*` | → className + themeConfig.css |
