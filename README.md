# JoySpace MCP Skills for Claude Code

通过 Chrome DevTools MCP 操作京东 JoySpace 协作文档平台的 Claude Code Skills 集合。

## Skills 列表

| Skill | 触发命令 | 功能 |
|-------|---------|------|
| **joyspace-create** | `/joyspace-create <标题> [内容]` | 创建新文档并写入标题和正文 |
| **joyspace-read** | `/joyspace-read <URL>` | 阅读文档内容并生成结构化总结 |
| **joyspace-pdf** | `/joyspace-pdf <URL>` | 将文档导出为 PDF |

## 前置条件

1. 安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
2. 配置 [Chrome DevTools MCP Server](https://github.com/anthropics/anthropic-quickstarts/tree/main/chrome-devtools-mcp)
3. 在浏览器中登录京东账号（JoySpace 需要认证）

## 安装方式

将 `skills/` 目录下的文件夹复制到你的 Claude Code skills 目录中：

```bash
# 复制所有 JoySpace skills
cp -r skills/* ~/.claude/skills/
```

或者只安装需要的 skill：

```bash
# 只安装文档创建
cp -r skills/joyspace-create ~/.claude/skills/

# 只安装文档阅读
cp -r skills/joyspace-read ~/.claude/skills/

# 只安装 PDF 导出
cp -r skills/joyspace-pdf ~/.claude/skills/
```

## 使用示例

### 创建文档

```
/joyspace-create 周会纪要 记录本周讨论的三个议题
```

### 阅读文档

```
/joyspace-read https://joyspace.jd.com/pages/xxxxx
```

### 导出 PDF

```
/joyspace-pdf https://joyspace.jd.com/pages/xxxxx
```

## 技术要点

- JoySpace 使用 **Slate.js** 编辑器，采用虚拟光标机制（`contentEditable="false"`），需要通过 JS dispatchEvent 定位光标
- 长内容写入使用 **ClipboardEvent paste** 一次性粘贴，避免逐行输入导致的内容乱序
- 文档使用虚拟渲染，阅读长文档需要滚动后多次取 snapshot
- 新建文档必须通过首页 UI 流程（首页 → 新建 → 文档 → 空白文档），直接构造 URL 会报错

## License

MIT
