# Notion-VT-Skills

Notion 排版 Skill 的 MVP 仓库。主线只维护“把已成文的文字排版成更好的 Notion 页面”。

## 收录

### notion-markdown-formatter

把一篇已经成立的 Markdown 整理成**骨架清晰、视觉有层次、叙事有节奏**的 Notion 页面，输出 Notion-flavored Markdown，可直接交给 Notion MCP（create-pages / update-page）。

它专注于三件事：

- **骨架**：章节层级与顺序，让每个部分承担清晰的位置。
- **层次**：Callout、Toggle、分栏、表格、分隔线等块的选择，只保留支撑阅读路径的强调。
- **节奏**：段落长度、观点分组与停顿，让阅读速度跟随内容张力。

Notion 平台能力按两类拆成 Reference，仅在生成时读取，只记 MCP 官方资源读不到的垂类边界：

- **notion-syntax.md**：页面内容排版的决策与实测坑（UI 专属能力、安全边界）。
- **notion-data.md**：Database、Chart、View 的边界与坑。
- 完整语法以 Notion MCP 资源为准（enhanced-markdown-spec / view-dsl-spec）。
