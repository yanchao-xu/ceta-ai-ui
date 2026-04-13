# EChart 图表

展示型组件，基于 Apache ECharts 渲染各类图表。支持柱状图、折线图、面积图、饼图等。

## 核心概念

EChart 有两种使用模式：

| 模式       | category 值        | 数据来源                      | 适用场景                      |
| ---------- | ------------------ | ----------------------------- | ----------------------------- |
| 数据源驱动 | `"axis"` / `"pie"` | `dataSource` 从后端 API 获取  | 动态数据（列表页、仪表盘）    |
| 静态配置   | `"axis"` / `"pie"` | `echartOption` 中直接写死数据 | 静态报表、HTML 转换、原型展示 |

**关键发现（从真实模板总结）：** 即使是静态数据，也使用 `category: "axis"` 或 `category: "pie"`，配合 `echartOption` 中完整的 ECharts option 即可。不需要 `dataSource`，不需要 `category: "generic"`。

## 样式规则

EChart 的高度有两个位置可以设置，**必须至少设置一个**，否则图表不显示：

| 位置                          | 说明                 | 示例                         |
| ----------------------------- | -------------------- | ---------------------------- |
| `componentProps.style.height` | 组件内部高度（推荐） | `"style": { "height": 300 }` |
| `style.height`                | 组件外层高度         | `"style": { "height": 300 }` |

真实模板中两种写法都有，推荐 `componentProps.style.height`。

---

## 模式一：柱状图（Bar Chart）

### 基础柱状图（静态数据）

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 300 },
    "echartOption": {
      "color": ["#63b2ee"],
      "title": { "text": "Sales by Category", "left": "center" },
      "tooltip": { "trigger": "axis", "axisPointer": { "type": "shadow" } },
      "xAxis": {
        "type": "value",
        "splitLine": { "lineStyle": { "type": [8, 8] } }
      },
      "yAxis": { "type": "category", "data": ["M", "N", "P", "Q"] },
      "series": [
        {
          "name": "Sales",
          "type": "bar",
          "data": [5000, 3000, 2000, 4000],
          "itemStyle": { "borderRadius": 10 }
        }
      ]
    },
    "category": "axis",
    "series": { "type": "bar", "stack": "stack" }
  },
  "style": { "height": 260 }
}
```

### 分组柱状图（多 series + 圆角）

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 356 },
    "echartOption": {
      "tooltip": { "trigger": "axis", "axisPointer": { "type": "shadow" } },
      "legend": { "right": "0", "top": "0" },
      "grid": {
        "top": "32px",
        "left": "8px",
        "right": "16px",
        "bottom": "8px",
        "containLabel": true
      },
      "xAxis": { "type": "category", "boundaryGap": [0, 0.01] },
      "yAxis": { "type": "value" },
      "series": [
        {
          "name": "2022",
          "type": "bar",
          "barWidth": "20%",
          "data": [18203, 23489, 29034, 104970, 131744, 630230],
          "itemStyle": { "borderRadius": [20, 20, 0, 0] }
        },
        {
          "name": "2023",
          "type": "bar",
          "barWidth": "20%",
          "barGap": "30%",
          "data": [19325, 23438, 31000, 121594, 134141, 681807],
          "itemStyle": { "borderRadius": [20, 20, 0, 0] }
        }
      ]
    },
    "category": "axis",
    "series": { "type": "bar", "stack": "stack" }
  }
}
```

### 堆叠柱状图

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 300 },
    "echartOption": {
      "color": ["#f8cb7f", "#7cd6cf"],
      "title": { "text": "Sales Forecast", "left": "center" },
      "tooltip": { "trigger": "axis", "axisPointer": { "type": "shadow" } },
      "legend": { "data": ["X", "Y"], "top": "0", "right": "0" },
      "grid": {
        "left": "3%",
        "right": "4%",
        "bottom": "3%",
        "containLabel": true
      },
      "xAxis": {
        "type": "category",
        "data": ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"]
      },
      "yAxis": {
        "type": "value",
        "axisLabel": { "formatter": "{value}K" },
        "splitLine": { "lineStyle": { "type": [8, 8] } }
      },
      "series": [
        {
          "name": "X",
          "data": [5, 4, 3, 5, 10, 6, 7],
          "type": "bar",
          "stack": "x",
          "barWidth": "30%"
        },
        {
          "name": "Y",
          "data": [10, 22, 28, 43, 49, 32, 21],
          "type": "bar",
          "stack": "x",
          "barWidth": "30%",
          "itemStyle": { "borderRadius": [10, 10, 0, 0] }
        }
      ]
    },
    "category": "axis",
    "series": { "type": "bar", "stack": "stack" }
  }
}
```

### 简洁柱状图（隐藏 X 轴，适合嵌入卡片）

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 205 },
    "echartOption": {
      "tooltip": { "trigger": "axis", "axisPointer": { "type": "shadow" } },
      "grid": {
        "left": 0,
        "right": 0,
        "bottom": 0,
        "top": "8%",
        "containLabel": true
      },
      "xAxis": [
        {
          "type": "category",
          "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
          "axisTick": { "alignWithLabel": true },
          "show": false
        }
      ],
      "yAxis": [
        {
          "type": "value",
          "min": 0,
          "max": 90,
          "interval": 30,
          "splitLine": { "lineStyle": { "type": [8, 8] } }
        }
      ],
      "series": [
        {
          "name": "Amount",
          "type": "bar",
          "barWidth": "30%",
          "data": [32, 55, 45, 72, 55, 40, 62],
          "itemStyle": {
            "normal": { "color": "#1B8FFF", "barBorderRadius": 8 }
          }
        }
      ]
    },
    "category": "axis",
    "series": { "type": "bar", "stack": "stack" }
  }
}
```

### 每根柱子不同颜色

用 series.data 中的 `itemStyle` 单独设置每根柱子的颜色：

```json
"series": [{
  "type": "bar", "barWidth": 40,
  "data": [
    { "value": 2670, "itemStyle": { "color": "#ff6b35" } },
    { "value": 135, "itemStyle": { "color": "#000000" } },
    { "value": 6, "itemStyle": { "color": "#ff6b35" } },
    { "value": 0, "itemStyle": { "color": "#000000" } }
  ]
}]
```

---

## 模式二：折线图 / 面积图（Line / Area Chart）

### 双色面积图

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 300 },
    "echartOption": {
      "color": ["#f89588", "#63b2ee"],
      "title": {
        "text": "Financial Overview",
        "subtext": "FY2024",
        "left": "center"
      },
      "tooltip": {
        "trigger": "axis",
        "axisPointer": {
          "type": "cross",
          "label": { "backgroundColor": "#6a7985" }
        }
      },
      "legend": {
        "data": ["Expense", "Income"],
        "bottom": "0",
        "left": "center"
      },
      "xAxis": {
        "type": "category",
        "boundaryGap": false,
        "data": ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
      },
      "yAxis": {
        "type": "value",
        "axisLabel": { "formatter": "{value}K" },
        "splitLine": { "lineStyle": { "type": [8, 8] } }
      },
      "series": [
        {
          "name": "Expense",
          "type": "line",
          "areaStyle": { "color": "rgba(248, 149, 136, 0.8)" },
          "lineStyle": { "color": "#f89588", "width": 2 },
          "data": [88, 92, 120, 98, 120, 136]
        },
        {
          "name": "Income",
          "type": "line",
          "areaStyle": { "color": "rgba(99, 178, 238, 0.3)" },
          "lineStyle": { "color": "#63b2ee", "width": 2 },
          "data": [56, 64, 100, 124, 150, 200]
        }
      ]
    },
    "category": "axis",
    "series": { "type": "line", "stack": "stack" }
  }
}
```

### 堆叠面积图（smooth + label）

```json
"series": [{
  "name": "Line 4", "type": "line", "stack": "Total", "smooth": true,
  "label": { "show": true, "position": "top" },
  "areaStyle": { "opacity": 0.2 },
  "emphasis": { "focus": "series" },
  "data": [220, 402, 231, 134, 190, 230, 120]
}]
```

### 迷你折线图（Sparkline，嵌入卡片）

隐藏坐标轴，只显示趋势线，适合放在指标卡片旁边：

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 62, "width": 80 },
    "echartOption": {
      "grid": {
        "containLabel": false,
        "left": 0,
        "right": 0,
        "bottom": 0,
        "top": "2%"
      },
      "xAxis": {
        "data": ["Jan", "Feb", "Mar", "Apr", "May", "Jun"],
        "show": false,
        "type": "category"
      },
      "yAxis": { "show": false, "type": "value" },
      "series": [
        {
          "data": [0, 3, 2, 5, 4, 8],
          "symbol": "none",
          "itemStyle": {
            "normal": { "lineStyle": { "width": 3 }, "color": "#7cd6cf" }
          },
          "type": "line"
        }
      ]
    },
    "category": "axis",
    "series": { "stack": "stack", "type": "line" }
  }
}
```

---

## 模式三：饼图 / 环形图（Pie / Doughnut Chart）

### 环形图（Doughnut）

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 180 },
    "echartOption": {
      "tooltip": { "trigger": "item" },
      "series": [
        {
          "name": "Access From",
          "type": "pie",
          "radius": ["65%", "85%"],
          "avoidLabelOverlap": false,
          "itemStyle": {
            "borderRadius": 4,
            "borderColor": "#fff",
            "borderWidth": 2
          },
          "label": { "show": false },
          "emphasis": { "label": { "show": false } },
          "labelLine": { "show": false },
          "data": [
            { "value": 1048, "name": "Search Engine" },
            { "value": 735, "name": "Direct" },
            { "value": 580, "name": "Email" },
            { "value": 484, "name": "Union Ads" },
            { "value": 300, "name": "Video Ads" }
          ]
        }
      ]
    },
    "category": "pie",
    "series": {
      "type": "pie",
      "radius": ["40%", "70%"],
      "itemStyle": {
        "borderRadius": 4,
        "borderColor": "#fff",
        "borderWidth": 2
      }
    }
  },
  "style": { "border": "1px solid #ccc" }
}
```

### 带中心文字的环形图

用 `graphic` 在环形中心显示汇总数字：

```json
{
  "component": "EChart",
  "componentProps": {
    "style": { "height": 300 },
    "echartOption": {
      "color": ["#63b2ee", "#f89588", "#76da91", "#f8cb7f", "#7cd6cf"],
      "tooltip": { "trigger": "item", "formatter": "{a} <br/>{b}: {c} ({d}%)" },
      "title": { "text": "Profit", "left": "left" },
      "legend": {
        "orient": "horizontal",
        "bottom": "0",
        "data": ["A", "B", "C", "D", "E"]
      },
      "series": [
        {
          "name": "Data",
          "type": "pie",
          "radius": ["40%", "60%"],
          "center": ["50%", "50%"],
          "data": [
            { "value": 335, "name": "A" },
            { "value": 310, "name": "B" },
            { "value": 234, "name": "C" },
            { "value": 135, "name": "D" },
            { "value": 648, "name": "E" }
          ],
          "label": { "position": "outside", "formatter": "{b}: {d}%" },
          "labelLine": { "show": true },
          "itemStyle": {
            "borderWidth": 4,
            "borderColor": "#fff",
            "borderRadius": 4
          }
        }
      ],
      "graphic": {
        "type": "text",
        "left": "center",
        "top": "middle",
        "style": {
          "text": "$500,000",
          "textAlign": "center",
          "fontSize": 20,
          "fontWeight": "bold",
          "fill": "#000"
        }
      }
    },
    "category": "pie",
    "series": {
      "type": "pie",
      "radius": ["40%", "70%"],
      "itemStyle": {
        "borderRadius": 4,
        "borderColor": "#fff",
        "borderWidth": 2
      }
    }
  }
}
```

---

## componentProps 完整属性表

| 属性                  | 类型                               | 默认值      | 说明                                                                                               |
| --------------------- | ---------------------------------- | ----------- | -------------------------------------------------------------------------------------------------- |
| category              | `'axis'` \| `'pie'` \| `'generic'` |             | axis=轴坐标系（柱状图/折线图），pie=饼图，generic=直接用数据源结果作为 option                      |
| chartSettings         | object                             |             | axis 模式配 xAxisField/yAxisField，pie 模式配 legendField/valueField（仅数据源驱动时需要）         |
| echartOption          | object                             |             | **核心属性** — 完整的 ECharts option，支持 title/tooltip/legend/grid/xAxis/yAxis/series/graphic 等 |
| series                | object                             |             | 顶层 series 配置（type/stack 等），与 echartOption.series 合并                                     |
| style                 | object                             |             | **组件内部样式，必须设置 height**，如 `{ "height": 300 }`                                          |
| dataSource            | `{ token, pbcToken, listUrl }`     |             | 数据源（动态数据时使用）                                                                           |
| dataUrl               | string                             |             | 通过 URL 获取数据                                                                                  |
| dataFilters           | array                              |             | 固定 filter 条件                                                                                   |
| sortModel             | `Array<{ colId, sort, sortType }>` |             | 排序方式                                                                                           |
| dataResponseKeyPath   | string                             | `'results'` | 数据在 response 中的路径                                                                           |
| transformDataResponse | string                             |             | 数据转换表达式                                                                                     |
| translateDataResponse | boolean                            | `true`      | 是否翻译请求结果                                                                                   |
| debounceTime          | number                             | `200`       | 请求 debounce time                                                                                 |
| httpMethod            | `'get'` \| `'post'`                | `'post'`    | HTTP 方法                                                                                          |
| selectColId           | `Array<string>`                    |             | 加载的数据字段                                                                                     |
| className             | string                             |             | 组件的 className                                                                                   |

## echartOption 常用配置速查

| 配置项    | 说明       | 常用值                                                          |
| --------- | ---------- | --------------------------------------------------------------- |
| `color`   | 全局调色盘 | `["#63b2ee", "#f89588", "#76da91", "#f8cb7f", "#7cd6cf"]`       |
| `title`   | 标题       | `{ text, subtext, left, textStyle, subtextStyle }`              |
| `tooltip` | 提示框     | axis 图用 `trigger: "axis"`，pie 图用 `trigger: "item"`         |
| `legend`  | 图例       | `{ data, orient, bottom/top/right/left }`                       |
| `grid`    | 绘图区域   | `{ left, right, top, bottom, containLabel: true }`              |
| `xAxis`   | X 轴       | `{ type: "category"/"value", data, show, axisTick, axisLabel }` |
| `yAxis`   | Y 轴       | `{ type: "value", axisLabel: { formatter }, splitLine }`        |
| `series`  | 数据系列   | 见各图表类型                                                    |
| `graphic` | 自定义图形 | 用于在饼图中心显示文字                                          |

## series 类型速查

| series.type | 图表类型    | 关键属性                                                         |
| ----------- | ----------- | ---------------------------------------------------------------- |
| `"bar"`     | 柱状图      | `barWidth`, `stack`, `itemStyle.borderRadius`, `itemStyle.color` |
| `"line"`    | 折线图      | `smooth`, `symbol`, `lineStyle`, `areaStyle`（加了就变面积图）   |
| `"pie"`     | 饼图/环形图 | `radius`（单值=饼图，数组=环形），`center`, `label`, `labelLine` |

## 虚线分割线技巧

模板中大量使用虚线分割线：

```json
"splitLine": { "lineStyle": { "type": [8, 8] } }
```

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name  | Type         | Description       |
| ----- | ------------ | ----------------- |
| node  | HTML Element | 组件的根 DOM 元素 |
| chart | object       | EChart 实例       |

## HTML 识别

- CETA 原生：`.echart-element`
- 通用 HTML：`<canvas>` 图表容器、CSS class 含 `chart`、HTML 中的柱状图/折线图/饼图视觉结构

### HTML 图表 → EChart 的转换规则

当 HTML 中出现以下结构时，应优先使用 EChart 组件而非手动用 Box/Stack 拼：

| HTML 特征                               | 转换为                       |
| --------------------------------------- | ---------------------------- |
| 多个 `<div>` 用 height 百分比模拟柱状图 | EChart `series.type: "bar"`  |
| `<canvas>` 图表容器                     | EChart（分析图表类型）       |
| CSS class 含 `chart-bar`、`chart-grid`  | EChart 柱状图                |
| 饼图/环形图 SVG 或 Canvas               | EChart `series.type: "pie"`  |
| 折线趋势图                              | EChart `series.type: "line"` |

转换时从 HTML 中提取：

1. 数据值（从文本内容、height 百分比、data 属性等）
2. 类别标签（X 轴标签）
3. 颜色（从 CSS class 或 inline style）
4. 标题文本

## 参考模板

真实项目的 EChart 使用示例见 `tempaltes/echart/` 目录：

- `echart1.json` — 柱状图、迷你折线图、面积图、堆叠柱状图、饼图
- `echart2.json` — 面积图、环形图、分组柱状图
- `echart3.json` — 简洁柱状图、卡片内嵌图表
- `tiantain.json` — 环形图（Doughnut）独立示例
