---
name: joyspace-pdf
description: 导出JoySpace文档为PDF。当用户要求将joyspace.jd.com的文档保存、导出、下载为PDF时使用此skill。
triggers:
  - joyspace-pdf
  - js2pdf
  - joyspace导出pdf
  - joyspace pdf
  - 导出pdf
  - 导出为pdf
  - 保存为pdf
  - 下载pdf
  - export pdf
  - joyspace export
---

# JoySpace 文档导出 PDF

将京东 JoySpace 在线协作文档导出为 PDF 文件。需要通过 Chrome DevTools MCP 操作浏览器完成。

## 前置条件

- Chrome DevTools MCP 已配置并连接
- 用户已在浏览器中登录京东账号（JoySpace 需要认证）

## 使用方式

```
/joyspace-pdf <JoySpace文档URL>
```

示例：
```
/joyspace-pdf https://joyspace.jd.com/pages/5eSbiFIqAS5XWF0rk7mV
```

## 操作流程

### 步骤 1：导航到文档页面

```
navigate_page → url: <JoySpace文档URL>
```

如果跳转到登录页面（authme.jd.com），提示用户先在浏览器中登录，登录完成后重新导航。

### 步骤 2：等待页面加载完成

确认页面 URL 为 joyspace.jd.com（而非 authme.jd.com），表示已成功进入文档。

### 步骤 3：打开更多操作菜单

通过 JavaScript 找到并点击页面右上角的 "..." 更多按钮：

```javascript
// 关键：该按钮的 class 为 "moreBtn ant-dropdown-trigger"
const moreBtn = document.querySelector('.moreBtn.ant-dropdown-trigger');
moreBtn.click();
```

> 注意：这个按钮在 a11y snapshot 中不容易直接识别，需要通过 JS 查找 class 名 `moreBtn` 来定位。

### 步骤 4：展开"导出为PDF"子菜单

菜单弹出后，在 snapshot 中找到：
- `menuitem "导出为PDF"` 下的 `button "导出为PDF"` (expandable, haspopup="menu")

使用 **hover** 操作（而非 click）来展开子菜单：

```
hover → uid of "导出为PDF" button
```

### 步骤 5：选择导出选项

子菜单展开后会出现两个选项：
- **"仅导出正文"** — 只导出文档正文内容
- **"导出正文与评论"** — 导出正文和评论

默认选择 **"仅导出正文"**，点击对应 menuitem。

### 步骤 6：确认导出成功

页面顶部会出现绿色提示：**"导出成功，敏感内容请勿外传"**

PDF 文件会自动下载到浏览器的默认下载目录（通常是 `~/Downloads`）。

## 关键注意事项

1. **"..." 按钮定位**：不要通过 snapshot 的 uid 盲目点击，要用 JS 查询 `.moreBtn.ant-dropdown-trigger`
2. **子菜单展开**：必须用 `hover` 而非 `click` 来展开"导出为PDF"的子菜单
3. **登录状态**：JoySpace 需要京东内网认证，如果未登录会跳转到 authme.jd.com
4. **导出选项**：默认选"仅导出正文"，除非用户明确要求包含评论
