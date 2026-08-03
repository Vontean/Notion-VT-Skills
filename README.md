# Notion-VT-Skills

个人收藏的 Notion 相关 Claude Code Skills 集合。

## 已收录

### notion-markdown-formatter

将普通 Markdown 文本转换为 **Notion-flavored Markdown**（标准 Markdown + XML 块标签的混合格式），可直接用于 Notion MCP 工具（create-pages / update-page）。

**能力概览：**

- Callout 提示框（`<callout>`）、Toggle 折叠（`<details>`）、分栏布局（`<columns>`）
- 完整表格格式（颜色、列宽、表头行列）
- 块级 / 内联颜色标记
- 页面 icon（emoji）+ Unsplash 主题封面图自动匹配
- 从 Notion 存储格式逆向转换为可读文本

> 📌 **说明：** 本 skill 基于 YouMind 平台原作者发布的 *Notion Markdown Formatter* 进行二次开发，在保留原有 9 步文档优化流程的基础上，扩展了 Notion 原生 XML 格式支持、完整表格/分栏/颜色等高级块能力，并新增了页面 icon 与封面图的自动匹配。
