# VT Skills

个人使用的 Writing 与 Notion Skills。前者负责把想法聊清楚、组织成文，后者负责把已经成立的文章交给 Notion 呈现。

## 已收录

### vt-writing

作为 reflective co-author，通过深聊、共同判断与编辑完成 sensemaking，把尚未成形的想法发展成作者认得出的文章。平台排版在文章成立后交给对应的 renderer。

### notion-markdown-formatter

将普通 Markdown 文本转换为 **Notion-flavored Markdown**（标准 Markdown + XML 块标签的混合格式），可直接用于 Notion MCP 工具（create-pages / update-page）。

它目前支持：

- Callout（`<callout>`）、Toggle（`<details>`）、columns（`<columns>`）
- 完整表格格式（颜色、列宽、表头行列）
- 块级 / 内联颜色标记
- 页面 icon（emoji）+ Unsplash 主题封面图自动匹配
- 从 Notion 存储格式逆向转换为可读文本

本 Skill 基于 YouMind 平台发布的 *Notion Markdown Formatter* 二次开发，补充了 Notion 原生 XML 格式、完整表格、分栏、颜色、页面 icon 和封面图。
