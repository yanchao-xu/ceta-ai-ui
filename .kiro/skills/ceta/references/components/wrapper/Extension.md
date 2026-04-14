# Extension 扩展组件

外部扩展组件容器，支持在 CETA 表单/页面中嵌入自定义实现的组件。

## 三种模式

| 模式 | extensionType | 说明 |
|------|--------------|------|
| 远端通用 | `remote-common` | JS/CSS 文件自行托管，通过 URL 加载 |
| 远端通用（CETA 托管）| `remote-common-ceta-hosted` | JS/CSS 由 CETA 托管，支持发布到组件库、充当表单字段 |
| 本地 React | `local-react` | 项目导出后本地实现，通过 extensionId 注册 |

> 推荐使用 **CETA 托管** 模式，功能最完整。

## schemaJson

### 远端通用模式

```json
{
  "component": "Extension",
  "componentProps": {
    "extensionType": "remote-common",
    "extensionJsUrl": "https://example.com/foo.js",
    "extensionCssUrl": "https://example.com/foo.css",
    "extensionParams": {
      "userId": ":context.userProfile.id"
    }
  }
}
```

### 远端通用模式 - CETA 托管（推荐）

JS/CSS 文件由 CETA 托管，通过表单设计器 GUI 上传。不建议直接修改 JSON 配置。

```json
{
  "component": "Extension",
  "componentProps": {
    "extensionType": "remote-common-ceta-hosted",
    "extensionJs": {
      "filename": "foo.js"
    },
    "extensionCss": {
      "filename": "foo.css"
    },
    "extensionParams": {
      "userId": ":context.userProfile.id"
    }
  }
}
```

### 本地 React 模式

```json
{
  "component": "Extension",
  "componentProps": {
    "extensionType": "local-react",
    "extensionId": "com.example.custom-extension",
    "extensionParams": {
      "userId": ":context.userProfile.id"
    }
  }
}
```

项目导出后本地注册：

```javascript
import { setup } from '@icp/settings';
import MyCustomExtension from './MyCustomExtension';

setup({
  extensionComponents: {
    'com.example.custom-extension': MyCustomExtension
  }
});
```

## componentProps

| 属性 | 类型 | 默认值 | 支持 Variable Pattern | 说明 |
|------|------|--------|:--------------------:|------|
| className | string | | ✔️ | Root div 的 className |
| style | object | | | Root div 的 style |
| extensionType | `'local-react'` \| `'remote-common'` \| `'remote-common-ceta-hosted'` | | | 扩展模式 |
| extensionId | string | | | 本地 React 模式：注册时的 id |
| extensionJsUrl | string | | | 远端通用模式：JS 文件地址 |
| extensionCssUrl | string | | | 远端通用模式：CSS 文件地址 |
| extensionJs | `{ filename: string }` | | | CETA 托管模式：JS 文件对象 |
| extensionCss | `{ filename: string }` | | | CETA 托管模式：CSS 文件对象 |
| extensionParams | object | | ✔️ | 传入扩展的参数对象 |

## JS 挂载函数（远端通用模式）

远端通用模式的 JS 文件需 default 导出一个 mount 函数：

```javascript
export default function mount(element, { params, formApi, messageApi, restApi, i18nApi, routerApi }) {
  // params 即 extensionParams 解析后的值
  element.innerText = 'Hello World';

  return function unmount() {
    // 清理工作
  };
}
```

mount 函数也可返回一个 Handle 对象（而非单独的 unmount 函数）：

```typescript
interface Handle<T> {
  unmount?: () => void;
  updateParams?(params: object): void;
  updateValue?(value: T): void;       // 充当字段时使用
  validate?(value: T): boolean;        // 充当字段时使用
}
```

## 充当表单字段

CETA 托管模式的 Extension 可配置为表单字段。配置后：
- 自动为表单添加该字段，无需手动添加
- mount 函数额外获得 `fieldApi` 参数

### fieldApi 接口

```typescript
interface FieldApi<T> {
  setValue: (value: T) => void;      // 设置字段值
  setTouched: () => void;            // 标记字段已操作，触发校验
  id: string;                        // 字段 id
  name: string;                      // 字段 name
  defaultValue: T;                   // 字段默认值
}
```

### 暴露给外部的方法

充当字段时，mount 返回的 Handle 对象可额外定义：

- `updateValue(value)`: 供外部调用修改内部值（表单联动、重置场景）
- `validate(value)`: 验证字段值。返回 `true` 通过；返回 `false` 显示通用失败提示；抛异常则显示异常信息作为失败提示

## JS API

通过 `formApi.getFieldApi()` 获取：

| Name | Type | Description |
|------|------|-------------|
| node | HTML Element | 组件的根 DOM 元素 |

## 脚手架

远端通用模式可通过脚手架创建工程：

```bash
npx --registry https://nexus.dev.bizops.com.cn/repository/npm-group/ create-icp-extension@latest
```

创建后项目中包含 AI 工具系统提示词文件 `<project>/icp-extension.md`。

开发阶段可将本地 dev server 地址填入 `extensionJsUrl`，启动项目进行在线 debug。

## 使用场景

- 嵌入自定义业务组件（图表、编辑器等）
- 集成第三方库或微前端模块
- 实现 CETA 内置组件无法覆盖的复杂交互
- 充当自定义表单字段（CETA 托管模式）
