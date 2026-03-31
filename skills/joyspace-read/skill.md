---
name: joyspace-read
description: 阅读并理解JoySpace文档内容，提取结构化信息并生成总结
triggers:
  - joyspace-read
  - js-understanding
  - 阅读joyspace
  - 理解joyspace
  - joyspace文档
---

# JoySpace 文档阅读与理解

通过 Chrome DevTools MCP 导航到 JoySpace 文档页面，提取全部文本内容并生成结构化总结。

## 前置条件

- Chrome DevTools MCP 已配置并连接
- 用户已在浏览器中登录京东账号

## 使用方式

```
/joyspace-read <JoySpace文档URL>
```

## 操作流程

### 步骤 1：导航到文档

```
navigate_page → url: <JoySpace文档URL>
```

如果跳转到 `authme.jd.com` 登录页，提示用户先登录。

> **MCP 页面关闭处理**：如果 MCP 报错 "The selected page has been closed"，不要反复尝试 navigate，直接用 `new_page` 重新打开目标 URL。

### 步骤 2：用文本 snapshot 提取内容（非截图）

**核心原则：使用 `take_snapshot` 读取文本，不要用 `take_screenshot` 截图来读内容。**

snapshot 会返回页面的 a11y 树，包含所有可见文本、表格结构、链接等，效率远高于截图。

```
take_snapshot  （不需要 verbose，默认即可）
```

### 步骤 3：滚动加载剩余内容

JoySpace 文档使用虚拟渲染（`virtual-render-observer`），一次 snapshot 可能无法获取全部内容。需要滚动文档容器后再次取 snapshot：

```javascript
// 滚动文档容器到底部
const container = document.querySelector('.doc-page-container');
if (container) {
  container.scrollTop = container.scrollHeight;
}
```

然后再次 `take_snapshot` 获取后续内容。

如果文档很长，可以分段滚动：

```javascript
// 每次滚动一屏
const container = document.querySelector('.doc-page-container');
container.scrollTop += container.clientHeight;
```

### 步骤 3.5：1:1 保留原文

> **核心原则：必须先完整保留原文内容，再进行任何总结或改写。**

将所有 snapshot 提取到的文本视为"原文"。后续无论是总结、改写还是多轮修改，都必须严格对照原文逐条核实数字、专有名词和状态描述（如"调研中"vs"已完成"）。**不要在自己的上一版总结上迭代修改**，每次修改都回到原文核对。

### 步骤 4：合并与理解内容

将多次 snapshot 的文本内容合并，去重后整理为结构化总结：

1. **提取文档元信息**：标题、标签、最后修改时间、创建人、字数
2. **识别文档结构**：根据标题层级（一、二、三… / 1.1、1.2…）划分章节
3. **提取表格数据**：snapshot 中表格的行列关系通过 `generic` 嵌套节点体现
4. **识别引用文档**：底部 tabpanel "本文引用(N)" 中列出的关联文档

### 步骤 5：输出总结

按以下格式输出：

```
## 文档总结：<标题>

**元信息：** 来源/创建人/修改时间/字数

### 各章节摘要
（按文档结构逐章总结）

### 核心结论
（一句话概括文档的核心意图）
```

## 关键注意事项

1. **优先用 snapshot，不用截图**：snapshot 返回结构化文本，可直接阅读；截图需要视觉解析，效率低且可能漏信息
2. **虚拟渲染问题**：JoySpace 的 `.doc-page-container` 使用虚拟渲染，未滚动到的内容可能不在 DOM 中，需要滚动后重新 snapshot
3. **表格识别**：snapshot 中表格表现为嵌套的 `generic` 节点，表头行和数据行分别是不同的 `generic` 子节点
4. **代码块内容**：代码块中的内容按行编号显示为 StaticText
5. **snapshot 不要加 verbose**：verbose 模式输出过大（100KB+），普通模式足够提取所有文本内容
