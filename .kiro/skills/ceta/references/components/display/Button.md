# Button 按钮 — 完整配置参考

> 此文件展示 Button 组件在 CETA 平台中的完整 JSON 配置，包括所有 action 类型的用法。
> 基于 `icp-doc-website/docs/components/button.md` 和 `ButtonElement.js` 源码整理。

---

## 基础结构

```json
{
  "id": "Button_1",
  "type": "Button",
  "componentProps": {
    "type": "primary",
    "size": "middle",
    "content": "按钮文字",
    "icon": "icon-name",
    "action": {
      "type": "submit"
    }
  }
}
```

### componentProps 属性

| 属性            | 类型                                                                          | 默认值      | 说明                                   |
| --------------- | ----------------------------------------------------------------------------- | ----------- | -------------------------------------- |
| type            | `'default'` \| `'primary'` \| `'ghost'` \| `'dashed'` \| `'link'` \| `'text'` | `'default'` | 按钮外观类型（对应 Ant Design Button） |
| size            | `'small'` \| `'middle'` \| `'large'`                                          | `'middle'`  | 按钮尺寸                               |
| content         | string                                                                        |             | 按钮文字内容，支持 Variable Pattern    |
| icon            | string                                                                        |             | 按钮图标名称（命名规则见 Icon.md 图标命名规则，如 `"material:search"`, `"oct:home"`, `"ant:api-outlined"`） |
| clickUseCapture | boolean                                                                       | `false`     | 是否在捕获阶段触发点击事件             |

### action 通用属性

| 属性                     | 类型         | 说明                                                        |
| ------------------------ | ------------ | ----------------------------------------------------------- |
| type                     | string       | 动作类型（见下方各类型详解）                                |
| successAction            | ButtonAction | 动作成功后执行的下一个动作，可递归嵌套                      |
| suppressSuccessLinkDelay | boolean      | 默认 successAction 中的 link 有 2s 延迟，设为 true 取消延迟 |
| successLinkDelayTime     | number       | 自定义 successAction 中 link 延迟时间(ms)，默认 2000        |

## 样式规则

Button 的视觉样式放在 `style`（组件根层），不是 `componentProps.style`：

| 样式目标 | 放在哪里 | 示例 |
|---------|---------|------|
| 背景色/渐变色 | `style.background` 或 `style.backgroundColor` | 支持 `linear-gradient` |
| 文字颜色 | `style.color` | |
| 圆角 | `style.borderRadius` | |
| 边框 | `style.borderColor` | |
| 字重 | `style.fontWeight` | |
| 宽度 | `style.width` | 如 `"100%"` 全宽按钮 |
| 内边距 | `style.padding` | 如 `"12px 32px"` |

### 渐变色按钮

```json
{
  "component": "Button",
  "componentProps": {
    "content": "搜索航班",
    "type": "primary",
    "icon": "material:search-two-tone"
  },
  "style": {
    "background": "linear-gradient(135deg, #00A8E8 0%, #00C9A7 100%)",
    "color": "white",
    "fontWeight": 600,
    "padding": "12px 32px",
    "borderRadius": 12
  }
}
```

### 全宽主色按钮

```json
{
  "component": "Button",
  "componentProps": {
    "type": "primary",
    "content": "下一步",
    "action": { "type": "link", "href": "/next-page" }
  },
  "style": {
    "width": "100%",
    "backgroundColor": "#003366",
    "borderColor": "#003366",
    "borderRadius": 6,
    "fontWeight": 600
  }
}
```

### 危险按钮（红色边框）

```json
{
  "component": "Button",
  "componentProps": {
    "content": "终止",
    "type": "default",
    "action": { "type": "confirm", "title": "确认终止？" }
  },
  "style": {
    "borderRadius": 6,
    "color": "#ff4d4f",
    "borderColor": "#ff4d4f"
  }
}
```

### 渐变色按钮（用 className）

当需要 hover 效果时用 className：

```json
{
  "component": "Button",
  "className": "gradient-btn",
  "componentProps": { "type": "primary", "content": "创建新PNR" }
}
```

themeConfig.css：
```css
.gradient-btn {
  background: linear-gradient(135deg, #00b4d8 0%, #0077b6 100%) !important;
  border: none !important;
}
.gradient-btn:hover {
  background: linear-gradient(135deg, #0096c7 0%, #005f8d 100%) !important;
}
```

---

## Action 类型详解

---

### 1. submit — 提交表单

提交整个表单数据到服务端（调用 `formApi.submit()`）。表单 readonly 时按钮自动隐藏，表单 disabled 时按钮自动禁用。

```json
{
  "id": "SubmitBtn",
  "type": "Button",
  "componentProps": {
    "type": "primary",
    "content": "提交",
    "action": {
      "type": "submit",
      "msg": "提交成功"
    }
  }
}
```

| 属性 | 类型   | 说明                                                                                    |
| ---- | ------ | --------------------------------------------------------------------------------------- |
| msg  | string | 提交成功后的提示信息，支持 Variable Pattern。设为 `null` 不显示提示。默认显示"提交成功" |

提交成功后跳转：

```json
{
  "action": {
    "type": "submit",
    "msg": "提交成功",
    "successAction": {
      "type": "link",
      "href": "/list"
    }
  }
}
```

---

### 2. save — 保存表单（草稿）

与 submit 类似，但以 `saveOnly: true` 模式提交，通常用于保存草稿。表单 readonly 时按钮自动隐藏。

```json
{
  "id": "SaveBtn",
  "type": "Button",
  "componentProps": {
    "content": "保存草稿",
    "action": {
      "type": "save",
      "msg": "保存成功"
    }
  }
}
```

| 属性 | 类型   | 说明                                                            |
| ---- | ------ | --------------------------------------------------------------- |
| msg  | string | 保存成功后的提示信息，支持 Variable Pattern。默认显示"保存成功" |

---

### 3. cancel — 取消

取消当前表单操作（调用 `formApi.cancel()`），通常用于关闭弹窗或返回上一页。

```json
{
  "id": "CancelBtn",
  "type": "Button",
  "componentProps": {
    "content": "取消",
    "action": {
      "type": "cancel"
    }
  }
}
```

带自定义返回状态和数据：

```json
{
  "action": {
    "type": "cancel",
    "state": "cancelled",
    "response": {
      "reason": "{{formData.cancelReason}}"
    }
  }
}
```

| 属性     | 类型   | 说明                                    |
| -------- | ------ | --------------------------------------- |
| state    | string | 取消时传递的状态标识                    |
| response | object | 取消时传递的数据，支持 Variable Pattern |

---

### 4. refresh — 刷新表单

刷新当前表单数据（调用 `formApi.refresh()`），如果按钮在表格内，同时刷新表格。

```json
{
  "id": "RefreshBtn",
  "type": "Button",
  "componentProps": {
    "content": "刷新",
    "icon": "ReloadOutlined",
    "action": {
      "type": "refresh"
    }
  }
}
```

---

### 5. refreshPage — 刷新整个页面

调用 `window.location.reload()` 强制刷新整个浏览器页面（有 1 秒延迟）。

```json
{
  "id": "RefreshPageBtn",
  "type": "Button",
  "componentProps": {
    "content": "刷新页面",
    "action": {
      "type": "refreshPage"
    }
  }
}
```

---

### 6. refreshTable — 刷新表格

刷新指定表格或当前上下文中的表格数据。

```json
{
  "id": "RefreshTableBtn",
  "type": "Button",
  "componentProps": {
    "content": "刷新表格",
    "action": {
      "type": "refreshTable"
    }
  }
}
```

刷新指定表格（通过 tableIds）：

```json
{
  "action": {
    "type": "refreshTable",
    "tableIds": ["Table_1", "Table_2"]
  }
}
```

| 属性     | 类型     | 说明                                                 |
| -------- | -------- | ---------------------------------------------------- |
| tableIds | string[] | 要刷新的表格组件 id 数组。不传则刷新当前上下文的表格 |

---

### 7. refreshData — 刷新指定字段数据

刷新表单中指定字段的数据（调用各字段的 `fieldApi.refresh()`）。

```json
{
  "id": "RefreshDataBtn",
  "type": "Button",
  "componentProps": {
    "content": "刷新数据",
    "action": {
      "type": "refreshData",
      "fieldIds": ["Select_1", "Table_1"]
    }
  }
}
```

| 属性     | 类型     | 说明                     |
| -------- | -------- | ------------------------ |
| fieldIds | string[] | 要刷新的字段组件 id 数组 |

---

### 8. dialog — 打开弹窗

在弹窗中打开一个表单或页面。这是最常用的复杂交互类型之一。

```json
{
  "id": "OpenDialogBtn",
  "type": "Button",
  "componentProps": {
    "type": "primary",
    "content": "新建",
    "action": {
      "type": "dialog",
      "msg": "操作成功",
      "openFormProps": {
        "pbcToken": "user-management",
        "formEntityToken": "user-form",
        "formEntityDataId": "{{currentData.id}}"
      },
      "openDialogProps": {
        "title": "新建用户",
        "size": "md",
        "centered": true
      },
      "successAction": {
        "type": "refreshTable"
      }
    }
  }
}
```

#### openFormProps — 弹窗内表单/页面属性

| 属性              | 类型    | 说明                                       |
| ----------------- | ------- | ------------------------------------------ |
| pbcToken          | string  | PBC 标识                                   |
| formEntityToken   | string  | 表单实体标识                               |
| formEntityDataId  | string  | 数据 ID（编辑模式），支持 Variable Pattern |
| retrieveUrl       | string  | 获取数据的 URL                             |
| createUrl         | string  | 创建数据的 URL                             |
| updateUrl         | string  | 更新数据的 URL                             |
| disabled          | boolean | 是否禁用表单（只读查看）                   |
| readonly          | boolean | 是否只读                                   |
| context           | object  | 传递给弹窗表单的上下文数据                 |
| FormRendererProps | object  | 传递给 FormRenderer 的额外属性             |

#### openDialogProps — 弹窗外观属性

| 属性                       | 类型                                                   | 默认值   | 说明                                                   |
| -------------------------- | ------------------------------------------------------ | -------- | ------------------------------------------------------ |
| title                      | string                                                 |          | 弹窗标题。不设置时自动使用打开页面的 title             |
| size                       | `'xs'` \| `'sm'` \| `'md'` \| `'lg'` \| `'fullscreen'` | `'sm'`   | 弹窗预设宽度                                           |
| width                      | number \| string                                       |          | 自定义弹窗宽度（优先于 size）                          |
| centered                   | boolean                                                | `true`   | 是否居中显示                                           |
| hideFooter                 | boolean                                                | `false`  | 是否隐藏弹窗底部按钮栏                                 |
| okText                     | string                                                 | `'确定'` | 确定按钮文字                                           |
| cancelText                 | string                                                 | `'取消'` | 取消按钮文字                                           |
| okProps                    | object                                                 |          | 确定按钮的额外 props（如 `{ icon: "CheckOutlined" }`） |
| cancelProps                | object                                                 |          | 取消按钮的额外 props                                   |
| disableUseOpenContentTitle | boolean                                                | `false`  | 禁止自动使用打开内容的 title                           |

查看详情（只读弹窗）：

```json
{
  "action": {
    "type": "dialog",
    "openFormProps": {
      "formEntityToken": "user-form",
      "formEntityDataId": "{{currentData.id}}",
      "disabled": true
    },
    "openDialogProps": {
      "title": "查看详情",
      "size": "lg",
      "hideFooter": true
    }
  }
}
```

编辑后刷新表格：

```json
{
  "action": {
    "type": "dialog",
    "msg": "保存成功",
    "openFormProps": {
      "formEntityToken": "user-form",
      "formEntityDataId": "{{currentData.id}}"
    },
    "openDialogProps": {
      "title": "编辑用户",
      "size": "md"
    },
    "successAction": {
      "type": "refreshTable"
    }
  }
}
```

---

### 9. request — 发送 HTTP 请求

向服务端发送自定义 HTTP 请求。

```json
{
  "id": "RequestBtn",
  "type": "Button",
  "componentProps": {
    "type": "primary",
    "content": "审批通过",
    "action": {
      "type": "request",
      "method": "post",
      "url": "/api/approve/{{currentData.id}}",
      "requestData": {
        "status": "approved",
        "comment": "{{formData.comment}}"
      },
      "msg": "审批成功",
      "successAction": {
        "type": "refreshTable"
      }
    }
  }
}
```

| 属性                  | 类型                                         | 说明                                                               |
| --------------------- | -------------------------------------------- | ------------------------------------------------------------------ |
| method                | `'get'` \| `'post'` \| `'put'` \| `'delete'` | HTTP 方法                                                          |
| url                   | string                                       | 请求地址，支持 Variable Pattern                                    |
| requestData           | object                                       | 请求数据。每个值支持 Variable Pattern。不传则默认发送整个 formData |
| msg                   | string                                       | 请求成功后的提示信息，支持 Variable Pattern                        |
| transformDataRequest  | string                                       | 请求前的数据转换表达式（EvalWorker）                               |
| transformDataResponse | string                                       | 响应后的数据转换表达式（EvalWorker）                               |

批量删除（结合表格选中行）：

```json
{
  "action": {
    "type": "confirm",
    "title": "确认删除",
    "content": "确定要删除选中的数据吗？",
    "successAction": {
      "type": "request",
      "method": "post",
      "url": "/api/batch-delete",
      "requestData": {
        "ids": "{{context.tableContext.selectedIds}}"
      },
      "msg": "删除成功",
      "successAction": {
        "type": "refreshTable"
      }
    }
  }
}
```

---

### 10. setFormData — 设置表单字段值

直接修改表单中指定字段的值。

```json
{
  "id": "SetDataBtn",
  "type": "Button",
  "componentProps": {
    "content": "填充默认值",
    "action": {
      "type": "setFormData",
      "formData": {
        "status": "active",
        "name": "{{currentData.userName}}",
        "department": "{{context.defaultDepartment}}"
      }
    }
  }
}
```

| 属性     | 类型   | 说明                                                              |
| -------- | ------ | ----------------------------------------------------------------- |
| formData | object | 键为字段的 keyPath，值为要设置的数据。每个值支持 Variable Pattern |

---

### 11. download — 下载文件

通过浏览器下载文件。

```json
{
  "id": "DownloadBtn",
  "type": "Button",
  "componentProps": {
    "content": "下载报表",
    "icon": "DownloadOutlined",
    "action": {
      "type": "download",
      "url": "/api/export/{{currentData.id}}",
      "filename": "报表.xlsx"
    }
  }
}
```

| 属性     | 类型   | 说明                                  |
| -------- | ------ | ------------------------------------- |
| url      | string | 下载文件的地址，支持 Variable Pattern |
| filename | string | 下载保存的文件名                      |
| msg      | string | 下载成功后的提示信息                  |

---

### 12. upload — 上传文件

弹出文件选择框，选择文件后上传到服务端。

```json
{
  "id": "UploadBtn",
  "type": "Button",
  "componentProps": {
    "content": "导入数据",
    "icon": "UploadOutlined",
    "action": {
      "type": "upload",
      "url": "/api/import",
      "accept": ".xlsx,.xls,.csv",
      "uploadFileKey": "file",
      "msg": "导入成功",
      "uploadingMsg": "正在导入...",
      "successAction": {
        "type": "refreshTable"
      }
    }
  }
}
```

| 属性          | 类型   | 说明                                                     |
| ------------- | ------ | -------------------------------------------------------- |
| url           | string | 上传地址，支持 Variable Pattern                          |
| accept        | string | 接受的文件类型（如 `.xlsx,.csv`），支持 Variable Pattern |
| uploadFileKey | string | 上传文件在请求中的字段名（必填）                         |
| msg           | string | 上传成功后的提示信息。设为 `null` 不显示                 |
| uploadingMsg  | string | 上传中的提示信息。设为 `null` 不显示                     |

---

### 13. link — 页面跳转

导航到指定页面，支持站内路由跳转和外部链接。

```json
{
  "id": "LinkBtn",
  "type": "Button",
  "componentProps": {
    "type": "link",
    "content": "查看详情",
    "action": {
      "type": "link",
      "href": "/detail/{{currentData.id}}"
    }
  }
}
```

新标签页打开：

```json
{
  "action": {
    "type": "link",
    "href": "https://example.com/doc",
    "target": "_blank"
  }
}
```

替换当前历史记录（不可后退）：

```json
{
  "action": {
    "type": "link",
    "href": "/dashboard",
    "replace": true
  }
}
```

| 属性                          | 类型    | 默认值  | 说明                                                      |
| ----------------------------- | ------- | ------- | --------------------------------------------------------- |
| href                          | string  |         | 跳转地址，支持 Variable Pattern 和 ConditionalProperty    |
| target                        | string  |         | HTML a 标签的 target 属性（如 `_blank` 新标签页打开）     |
| replace                       | boolean | `false` | 是否替换当前路由历史（React Router replace）              |
| hrefIsSiteBased               | boolean | `false` | 是否为站点级跳转（跨 PBC）。详见下方 hrefIsSiteBased 说明 |
| suppressBasePath              | boolean | `false` | 是否跳过 React Router 的 base path 拼接                   |
| suppressInheritIncludeDeleted | boolean | `false` | 是否禁止自动继承当前页面 URL 中的 `include_deleted` 参数  |

#### hrefIsSiteBased 跨 PBC 跳转说明

`hrefIsSiteBased` 控制链接是在当前 PBC 内部跳转还是跨 PBC 跳转到其他模块的页面/表单。

- **`hrefIsSiteBased: false`（默认）** — PBC 内部跳转，`href` 不需要带 PBC token 前缀，平台会自动拼接当前 PBC 的 basename。
  - 跳转到 PBC 内的 Page：`"href": "/page/pagetoken"`
  - 跳转到 PBC 内的 Form：`"href": "/form/formtoken/layout/layouttoken"`

- **`hrefIsSiteBased: true`** — 跨 PBC 跳转，`href` 必须包含目标 PBC token 作为路径前缀，平台不会自动拼接 basename。
  - 跳转到其他 PBC 的 Page：`"href": "/pbctoken/page/pagetoken"`
  - 跳转到其他 PBC 的 Form：`"href": "/pbctoken/form/formtoken/layout/layouttoken"`

跨 PBC 跳转示例：

```json
{
  "action": {
    "type": "link",
    "hrefIsSiteBased": true,
    "href": "/order-management/page/order-list"
  }
}
```

PBC 内部跳转示例（默认行为，可省略 hrefIsSiteBased）：

```json
{
  "action": {
    "type": "link",
    "href": "/page/order-list"
  }
}
```

---

### 14. globalMethod — 调用全局方法

调用挂载在 `window` 对象上的全局方法，传入表单数据。

```json
{
  "id": "GlobalMethodBtn",
  "type": "Button",
  "componentProps": {
    "content": "调用自定义方法",
    "action": {
      "type": "globalMethod",
      "globalMethod": "myCustomHandler"
    }
  }
}
```

| 属性         | 类型   | 说明                                                   |
| ------------ | ------ | ------------------------------------------------------ |
| globalMethod | string | `window` 上的方法名。调用时传入 `(formData, response)` |

---

### 15. formMethod — 调用表单方法

调用通过 `formApi.getMethods()` 注册的自定义表单方法。支持返回 Promise。

```json
{
  "id": "FormMethodBtn",
  "type": "Button",
  "componentProps": {
    "content": "自定义处理",
    "action": {
      "type": "formMethod",
      "methodName": "handleCustomLogic",
      "successAction": {
        "type": "refresh"
      }
    }
  }
}
```

| 属性       | 类型   | 说明                                                             |
| ---------- | ------ | ---------------------------------------------------------------- |
| methodName | string | 表单方法名。方法接收 `{ response, buttonApi, currentData }` 参数 |

---

### 16. logout — 退出登录

调用系统退出登录接口。

```json
{
  "id": "LogoutBtn",
  "type": "Button",
  "componentProps": {
    "content": "退出登录",
    "icon": "LogoutOutlined",
    "action": {
      "type": "logout"
    }
  }
}
```

---

### 17. confirm — 确认弹窗

弹出确认对话框，用户点击"确定"后执行 successAction。常用于删除等危险操作的二次确认。

```json
{
  "id": "DeleteBtn",
  "type": "Button",
  "componentProps": {
    "type": "primary",
    "content": "删除",
    "action": {
      "type": "confirm",
      "title": "确认删除",
      "content": "确定要删除「{{currentData.name}}」吗？此操作不可撤销。",
      "successAction": {
        "type": "request",
        "method": "delete",
        "url": "/api/users/{{currentData.id}}",
        "msg": "删除成功",
        "successAction": {
          "type": "refreshTable"
        }
      }
    }
  }
}
```

| 属性    | 类型   | 说明                              |
| ------- | ------ | --------------------------------- |
| title   | string | 确认框标题，默认"确认"            |
| content | string | 确认框内容，支持 Variable Pattern |

---

### 18. updateTableRow — 更新表格行数据

在可编辑表格中添加、更新或删除行。按钮必须在 EditableTable 上下文中使用。

添加行：

```json
{
  "id": "AddRowBtn",
  "type": "Button",
  "componentProps": {
    "content": "添加一行",
    "action": {
      "type": "updateTableRow",
      "updateType": "add",
      "rowData": {
        "status": "draft",
        "createTime": "{{context.now}}"
      }
    }
  }
}
```

更新行：

```json
{
  "action": {
    "type": "updateTableRow",
    "updateType": "update",
    "rowData": {
      "id": "{{currentData.id}}",
      "status": "approved"
    }
  }
}
```

删除行：

```json
{
  "action": {
    "type": "updateTableRow",
    "updateType": "remove",
    "rowData": "{{currentData.id}}"
  }
}
```

| 属性       | 类型                                | 说明                                                |
| ---------- | ----------------------------------- | --------------------------------------------------- |
| updateType | `'add'` \| `'update'` \| `'remove'` | 操作类型                                            |
| rowData    | string \| object                    | 行数据，支持 Variable Pattern。remove 时可以只传 id |

---

### 19. scanCode — 扫码

调用设备摄像头扫描二维码/条形码，返回扫描结果文本。适用于移动端场景。

```json
{
  "id": "ScanBtn",
  "type": "Button",
  "componentProps": {
    "content": "扫一扫",
    "icon": "ScanOutlined",
    "action": {
      "type": "scanCode",
      "successAction": {
        "type": "setFormData",
        "formData": {
          "barcode": "{{response}}"
        }
      }
    }
  }
}
```

扫码结果通过 `response` 传递给 `successAction`。

---

## 组合示例

### 表格工具栏（新建 + 批量删除 + 导出）

```json
{
  "id": "Toolbar",
  "type": "Layout",
  "componentProps": { "style": { "display": "flex", "gap": "8px" } },
  "children": [
    {
      "id": "CreateBtn",
      "type": "Button",
      "componentProps": {
        "type": "primary",
        "content": "新建",
        "icon": "PlusOutlined",
        "action": {
          "type": "dialog",
          "msg": "创建成功",
          "openFormProps": {
            "formEntityToken": "user-form"
          },
          "openDialogProps": {
            "title": "新建用户",
            "size": "md"
          },
          "successAction": {
            "type": "refreshTable"
          }
        }
      }
    },
    {
      "id": "BatchDeleteBtn",
      "type": "Button",
      "componentProps": {
        "content": "批量删除",
        "action": {
          "type": "confirm",
          "title": "确认删除",
          "content": "确定要删除选中的数据吗？",
          "successAction": {
            "type": "request",
            "method": "post",
            "url": "/api/users/batch-delete",
            "requestData": {
              "ids": "{{context.tableContext.selectedIds}}"
            },
            "msg": "删除成功",
            "successAction": {
              "type": "refreshTable"
            }
          }
        }
      }
    },
    {
      "id": "ExportBtn",
      "type": "Button",
      "componentProps": {
        "content": "导出",
        "icon": "DownloadOutlined",
        "action": {
          "type": "download",
          "url": "/api/users/export",
          "filename": "用户列表.xlsx"
        }
      }
    }
  ]
}
```

### 表单底部（提交 + 保存草稿 + 取消）

```json
{
  "id": "FormFooter",
  "type": "Layout",
  "componentProps": {
    "style": { "display": "flex", "gap": "8px", "justifyContent": "flex-end" }
  },
  "children": [
    {
      "id": "SubmitBtn",
      "type": "Button",
      "componentProps": {
        "type": "primary",
        "content": "提交",
        "action": {
          "type": "submit",
          "msg": "提交成功",
          "successAction": {
            "type": "link",
            "href": "/list"
          }
        }
      }
    },
    {
      "id": "SaveDraftBtn",
      "type": "Button",
      "componentProps": {
        "content": "保存草稿",
        "action": {
          "type": "save",
          "msg": "草稿已保存"
        }
      }
    },
    {
      "id": "CancelBtn",
      "type": "Button",
      "componentProps": {
        "content": "取消",
        "action": {
          "type": "cancel"
        }
      }
    }
  ]
}
```

---

## Variable Pattern 说明

action 中许多字段支持 Variable Pattern，即可以使用 `{{...}}` 语法引用动态数据：

| 变量                                    | 说明                                                |
| --------------------------------------- | --------------------------------------------------- |
| `{{formData.fieldName}}`                | 当前表单数据                                        |
| `{{currentData.fieldName}}`             | 当前行数据（表格场景）                              |
| `{{context.key}}`                       | 表单上下文数据                                      |
| `{{params.id}}`                         | URL 路由参数                                        |
| `{{response.key}}`                      | 上一个 action 的返回结果（在 successAction 中使用） |
| `{{context.tableContext.selectedIds}}`  | 表格选中行的 ID 数组                                |
| `{{context.tableContext.selectedRows}}` | 表格选中行的完整数据                                |

---

## ⚠️ 注意事项

- `open` 类型已废弃（@deprecated），请使用 `dialog` 代替
- `submit` 和 `save` 类型的按钮在表单 `readonly` 时自动隐藏，`disabled` 时自动禁用
- `confirm` 类型本身不执行业务操作，需要通过 `successAction` 链接实际操作
- `successAction` 支持递归嵌套，可以实现 confirm → request → refreshTable 这样的链式操作
- `dialog` 类型的 `successAction` 在弹窗关闭后执行，不走通用的 successAction 流程
- `cancel` 和 `refreshPage` 类型不支持 successAction
