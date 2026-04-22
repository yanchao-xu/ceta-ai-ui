# CETA 组件索引

> 组件选择阶段的速查表。先读此文件了解所有组件的能力边界，确定用哪些组件后，再读对应的完整文档获取配置细节。

## 布局组件（layout/）

| 组件 | 一句话能力 | 什么时候选它 | 文档 |
|------|-----------|-------------|------|
| Stack | 基于 Flexbox 的弹性布局，控制子组件横向/纵向排列和间距 | 子元素需要横排或竖排、按钮组、页面级容器、导航栏 | Stack.md |
| Grid | 多列栅格布局，支持 colNumber + span 精确控制列宽 | 表单字段并排（2-4列）、页面级精确分栏（12列栅格如 8:4 比例） | Grid.md |
| Card | 带标题的卡片容器，有 5 个独立样式插槽（整体/标题栏/标题文字/描述/内容区） | 有标题的分组区域、`<section>` + 标题、`<fieldset>` + `<legend>` | Card.md |
| Collapse | 折叠面板，子组件在 items[].fields 中 | 可展开/收起的分组、手风琴效果 | Collapse.md |
| Box | 最基础的容器，无特殊布局逻辑，纯靠 style 实现视觉效果 | 渐变色块、圆形徽章、提示信息框、任何需要装饰性背景/边框但不需要标题的容器 | layout/others.md |
| Mobile | 移动端专用布局容器 | 移动端页面 | layout/others.md |

## 展示组件（display/）

| 组件 | 一句话能力 | 什么时候选它 | 文档 |
|------|-----------|-------------|------|
| Text | 显示静态/动态文本，支持链接、前后缀、格式化，可通过 style 做 Badge 效果 | 所有文本内容（标题、段落、标签、带背景色的状态徽章、等宽编号） | Text.md |
| Button | 按钮，支持 19 种 action 类型（submit/link/dialog/request/confirm/download/upload 等） | 所有可点击的操作按钮，action 类型需根据按钮文本和上下文推断 | Button.md |
| Table | 基于 ag-grid 的数据表格，支持列定义、分页、筛选、操作列 | `<table>` 数据列表、列表页核心组件 | Table.md |
| Tabs | 标签页切换，子组件在 items[].fields 中 | 同一区域内切换显示不同内容（不跳转页面） | Tabs.md |
| NavTabs | 导航标签，通过 href 跳转不同页面 | 页面级导航标签栏（点击切换到不同页面 URL） | NavTabs.md |
| Steps | 步骤条/流程进度，支持数据驱动 | 流程步骤指示器、进度展示（不要用多个 Text 手动拼） | Steps.md |
| Status | 带颜色的状态标签，通过 enums 映射值到颜色 | 数据驱动的状态显示（值 → 颜色+文字映射） | Status.md |
| Icon | 图标，支持 material:/oct:/solar:/ant: 前缀和 Emoji | 纯装饰性图标、标题旁图标、状态指示图标 | Icon.md |
| Image | 图片展示 | `<img>` 标签 | Image.md |
| Divider | 分割线 | `<hr>` 或内容区域分隔 | Divider.md |
| List | 数据列表，每项可包含自定义子组件模板 | 重复结构的卡片列表（非表格形式），配合 LinkWrapper 可做可点击列表 | List.md |
| ListSelect | 以列表卡片形式展示选项供用户选择 | 卡片式单选/多选（如选择方案、选择模板） | ListSelect.md |
| EChart | 基于 Apache ECharts 的图表（柱状图/折线图/面积图/饼图/环形图） | 任何图表需求，HTML 中用 div 模拟的柱状图也应转为 EChart | EChart.md |
| Progress | 进度条 | `<progress>` 或进度百分比展示 | Progress.md |
| QRCode | 二维码生成 | 二维码展示 | QRCode.md |
| Audio | 音频播放器，支持标记点 | `<audio>` 标签 | Audio.md |
| Gantt | 基于 DHTMLX 的甘特图 | 项目管理甘特图 | Gantt.md |
| Tree | 树形控件 | 树状层级数据展示 | display/others.md |
| IFrame | 内嵌外部页面 | 嵌入第三方页面 | display/others.md |
| Video | 视频播放 | `<video>` 标签 | display/others.md |
| Carousel | 轮播 | 图片/内容轮播 | display/others.md |
| Tooltip | 悬浮提示 | 鼠标悬停显示提示信息 | display/others.md |
| TodoList | 待办列表 | 待办事项 | display/others.md |


## 输入组件（input/）

| 组件 | 一句话能力 | 什么时候选它 | Field type | 文档 |
|------|-----------|-------------|-----------|------|
| Input | 单行文本输入，支持前后缀、大号等宽样式 | `<input type="text/email/tel/search">` | TEXT_BOX | Input.md |
| Textarea | 多行文本输入 | `<textarea>` | TEXTAREA | Textarea.md |
| Password | 密码输入 | `<input type="password">` | PASSWORD | input/others.md |
| NumberPicker | 数字输入，支持精度、步长、货币格式化 | `<input type="number">` | NUMBER | NumberPicker.md |
| Select | 下拉选择，支持静态选项和数据源 | `<select>` 标签 | SELECT | Select.md |
| Radio | 单选按钮组 | 多个 `<input type="radio">` 同 name | RADIO | Radio.md |
| Checkbox | 多选框组 | 多个 `<input type="checkbox">` 组成选项组 | CHECKBOX | Checkbox.md |
| Switch | 开关切换 | 单个 checkbox 且语义为"是否XX" | SWITCH | Switch.md |
| DatePicker | 日期选择，可带时间 | `<input type="date/datetime-local">` | DATE | DatePicker.md |
| DateRangePicker | 日期范围选择 | 日期范围输入 | DATE | input/others.md |
| TimePicker | 时间选择 | `<input type="time">` | TIME | input/others.md |
| Upload | 文件上传，支持图片卡片模式 | `<input type="file">` | UPLOAD | Upload.md |
| ACL | 关联引用选择器，带弹窗表格搜索 | 需要从其他表单实体选择关联数据的字段 | ACL | ACL.md |
| Cascader | 级联选择（省市区等多级联动） | 多级联动的 `<select>` 组合 | CASCADER | Cascader.md |
| TreeSelect | 树形下拉选择 | 树状层级数据选择 | TREE_SELECT | input/others.md |
| EditableTable | 可编辑表格（子表），支持增删行 | 表单内嵌的明细行表格（如订单行项目） | EDITABLE_GRID | EditableTable.md |
| RichText | 富文本编辑器（基于 wangeditor） | `contenteditable` 或富文本编辑区域 | RICH_TEXT | RichText.md |
| Rate | 星级评分 | 评分组件 | RATE | Rate.md |
| ListSelect | 卡片式列表选择（单选/多选） | 以卡片形式展示选项供选择 | — | ListSelect.md |
| Slider | 滑块选择数值 | 滑块输入 | — | input/others.md |
| Formula | 公式字段 | 计算字段 | FORMULA | input/others.md |
| CodeEditor | 代码编辑器 | 代码输入 | — | input/others.md |

## 包装器组件（wrapper/）

| 组件 | 一句话能力 | 什么时候选它 | 文档 |
|------|-----------|-------------|------|
| Data | 数据容器，从 API 获取数据提供给子组件，不渲染可见 UI | 需要为展示组件提供动态数据（如 Steps 的状态、Text 的动态内容） | Data.md |
| LinkWrapper | 链接包装器（别名 Link），给子组件添加点击跳转能力 | `<a>` 包裹复杂子元素（卡片、图片区块）、List 中每项可点击跳转、Dashboard 卡片跳转 | LinkWrapper.md |
| Extension | 外部扩展组件容器，嵌入自定义 JS/React 组件 | CETA 内置组件无法覆盖的复杂交互、集成第三方库 | Extension.md |
| Timer | 定时器 | 定时执行操作 | wrapper/others.md |

## 易混淆组件速查

| 场景 | 正确选择 | 错误选择 | 判断依据 |
|------|---------|---------|---------|
| `<a>` 只包裹纯文本 | Button (action type=link) | LinkWrapper | 内容简单，用 Button 即可 |
| `<a>` 包裹卡片/图片等复杂子元素 | LinkWrapper | Button | 内容复杂，需要包装器 |
| 带背景色的文本标签（Badge） | Text + style | Status / Box | 静态展示用 Text，数据驱动映射用 Status |
| 数据驱动的状态标签（值→颜色映射） | Status | Text | 需要 enums 映射时用 Status |
| 有标题的分组区域 | Card | Box | Card 有标题插槽，Box 没有 |
| 纯装饰性容器（渐变色块、圆形徽章） | Box | Card | 不需要标题，只需要视觉效果 |
| 单个 checkbox 语义为"是否XX" | Switch | Checkbox | 开关语义用 Switch |
| 多个 checkbox 组成选项组 | Checkbox | Switch | 多选语义用 Checkbox |
| 页面级导航标签（点击跳转页面） | NavTabs | Tabs | 跳转页面用 NavTabs |
| 同区域内切换内容（不跳转） | Tabs | NavTabs | 切换内容用 Tabs |
| 步骤/流程进度指示 | Steps | 多个 Text 手动拼 | 永远用 Steps，不要手动拼 |
| HTML 中用 div 模拟的柱状图 | EChart | Box + Stack 手动拼 | 永远用 EChart，不要手动拼 |
| 表单内嵌的可编辑明细行 | EditableTable | Table | EditableTable 是输入组件（存数据），Table 是展示组件 |
| 从其他表单选择关联数据 | ACL | Select | ACL 有弹窗表格搜索，Select 只有下拉 |