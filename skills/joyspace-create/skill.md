---
name: joyspace-create
description: 在JoySpace中创建新文档并输入标题和内容。通过Chrome DevTools MCP操作浏览器完成。
triggers:
  - joyspace-create
  - joyspace新建
  - 创建joyspace
  - 新建文档
  - create joyspace
---

# JoySpace 创建新文档

通过 Chrome DevTools MCP 在京东 JoySpace 平台创建新文档，并输入标题和正文内容。

## 前置条件

- Chrome DevTools MCP 已配置并连接
- 用户已在浏览器中登录京东账号

## 使用方式

```
/joyspace-create <标题> [内容描述]
```

示例：
```
/joyspace-create 论AI时代营销安全的发展
/joyspace-create 周会纪要 记录今天讨论的三个议题
```

## 操作流程

### 步骤 1：导航到 JoySpace 首页

```
navigate_page → url: https://joyspace.jd.com/h/receive
```

> **重要**：不要尝试通过直接 URL（如 `joyspace.jd.com/pages/new?teamId=xxx`）创建文档，会返回 40004 错误。必须从首页通过 UI 操作新建。

如果跳转到 `authme.jd.com` 登录页，提示用户先登录。

> **MCP 页面关闭处理**：如果 MCP 报错 "The selected page has been closed"，不要反复尝试 navigate，直接用 `new_page` 重新打开目标 URL。

### 步骤 2：点击"新建"按钮

在 snapshot 中找到 `StaticText "新建"` 对应的可点击元素，点击后弹出下拉菜单。

```
click → uid of "新建"
```

### 步骤 3：选择"文档"类型

下拉菜单中点击 `menuitem "文档 文档"`：

```
click → uid of "文档" menuitem
```

此时会弹出 **模板选择对话框**（dialog "选择模版开始创作"）。

### 步骤 4：创建空白文档

在模板对话框中，点击 **"新建空白文档"**：

```
click → uid of "新建空白文档"
```

新文档会在 **新标签页** 中打开。

### 步骤 5：切换到新文档标签页

```
list_pages  → 找到新创建的文档页面（URL 格式为 joyspace.jd.com/pages/xxx）
select_page → pageId: <新页面的ID>, bringToFront: true
```

### 步骤 6：输入标题

JoySpace 使用 **Slate.js 编辑器**，采用虚拟光标机制（`contentEditable="false"`），不能通过常规的 click + fill 操作。正确的输入方式：

**6.1** 在 snapshot 中找到 `textbox multiline`（无 placeholder 的那个，非评论框），点击使其获得焦点：

```
click → uid of textbox (focusable multiline，通常在文档内容区域)
```

**6.2** 通过 JavaScript 在标题行位置模拟鼠标事件，将光标定位到标题：

```javascript
() => {
  const titleLine = document.querySelector('.slate-editor .sl-paragraph');
  if (!titleLine) return 'no title line';
  const rect = titleLine.getBoundingClientRect();
  const x = rect.left + rect.width / 2;
  const y = rect.top + rect.height / 2;
  ['mousedown', 'mouseup', 'click'].forEach(type => {
    titleLine.dispatchEvent(new MouseEvent(type, {
      bubbles: true, cancelable: true, view: window,
      clientX: x, clientY: y
    }));
  });
  return 'clicked title';
}
```

**6.3** 使用 `type_text` 输入标题文字：

```
type_text → text: "<标题>"
```

### 步骤 7：输入正文内容

**7.1** 按 `Enter` 从标题行换到正文区域：

```
press_key → key: "Enter"
```

**7.2** 使用 **ClipboardEvent paste** 一次性写入全部正文（推荐方式）：

> **重要**：不要使用逐行 `type_text` + `press_key Enter` 的方式输入长内容。Slate.js 编辑器在快速连续输入时会导致内容乱序。必须使用剪贴板粘贴方式一次性写入。

```javascript
() => {
  const text = `第一段内容...

第二段内容...

第三段内容...`;  // 用 \n\n 分隔段落，\n 换行
  const editor = document.querySelector('.slate-editor');
  if (!editor) return 'no editor';
  const clipboardData = new DataTransfer();
  clipboardData.setData('text/plain', text);
  const pasteEvent = new ClipboardEvent('paste', {
    bubbles: true,
    cancelable: true,
    clipboardData: clipboardData
  });
  editor.dispatchEvent(pasteEvent);
  return 'pasted';
}
```

将正文内容整理为纯文本字符串（段落之间用 `\n\n` 分隔），通过 `evaluate_script` 执行上述代码一次性粘贴到编辑器中。

**备选方式**：如果内容非常短（几句话以内），也可以使用 `type_text` 逐段输入：

```
type_text → text: "第一段内容..."
press_key → key: "Enter"
type_text → text: "第二段内容..."
```

### 步骤 8：确认保存

JoySpace 支持自动保存，输入完成后页面会显示 **"自动保存于 今天 XX:XX"**。无需手动保存。

通过 `take_snapshot` 确认标题和内容已正确写入，然后将文档 URL 返回给用户。

## 关键注意事项

1. **必须从首页新建**：直接构造 URL 创建文档会报错（40004），必须通过首页 → 新建 → 文档 → 空白文档的 UI 流程
2. **新标签页**：新文档在新标签页打开，必须用 `list_pages` + `select_page` 切换过去
3. **Slate.js 虚拟光标**：编辑器 `contentEditable="false"`，不能直接 click 文本节点的 uid，必须先 click `textbox multiline` 获取焦点，再用 JS dispatchEvent 定位光标
4. **长内容必须用剪贴板粘贴**：逐行 `type_text` 在 Slate.js 中输入长内容会导致行乱序，必须使用 `ClipboardEvent paste` 一次性写入
5. **自动列表转换**：以 `1. ` 开头的文本会被编辑器自动转为有序列表，编号会自动递增。如需避免，可调整文本格式（如用"1）"代替"1. "）
5. **登录状态**：如果未登录会跳转到 authme.jd.com，提示用户先登录
6. **内容生成**：如果用户只给了标题或主题，应根据主题自主撰写完整的文章内容后再输入
