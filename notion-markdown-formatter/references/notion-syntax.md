# Notion 排版边界（MCP 实测）

完整语法不是这里的职责：写入前先读取 MCP 资源 notion://docs/enhanced-markdown-spec，按官方 spec 生成。这里只记官方 spec 没有、但实测会踩坑的垂类知识。

## 实测坑（官方 spec 未写或写得不够明显）

### 写入

- 子块缩进必须 Tab；空行必须独占一行的 empty-block。
- 列表项无内联富文本会渲染为空项。
- Callout 内部用 Notion Markdown，不用 HTML。
- Toggle / Toggle Heading 子内容不缩进就不会包进去。
- 表格单元格只放富文本；fit-page-width 和列宽都可有可无，默认 false。
- Mermaid 节点含括号等特殊字符时用双引号；换行用 br。

### 不能创建（UI 专属）

- Web Bookmark 卡片：MCP 拒绝创建；bookmark 无官方语法。输出裸 URL，用户粘贴转卡片。
- Button、Breadcrumb：Markdown 会被转义成字面文本。
- Tabs：结构可创建，title 属性写入后丢失，标题需用户设置。
- Cell merging：官方不支持通过 Markdown 创建/修改。

### 安全边界

- page 标签：引用已有页面会移动它，删除标签删除子页。纯引用用 mention-page。
- database 标签：同理会移动数据库。
- folder：不能由 Markdown 创建。
