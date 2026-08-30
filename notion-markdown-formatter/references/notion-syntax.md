# Notion 内容排版设计调用

输入是一篇已成立的 Markdown。这里回答两个问题：这段内容该用哪个 Notion 块；该块怎么写才好看。完整语法见 MCP 资源 notion://docs/enhanced-markdown-spec。

## 骨架：先决定章节，再决定每章用什么块

- 通读原文，找到主题、开头承诺、各节推进、结尾落点。
- H1 是页面标题；H2 是主章节；只有章节内部确实需要分组才用 H3，不跳级。
- 章节标题写“这一部分让读者得到什么”，不写关键词罗列；没有作用就不加标题。
- 各节顺序保留原文逻辑；只调整明显错位且不改变内容的顺序，并说明。
- 开头保持正文，不把承诺转成 Callout；结尾不追加总结。

## 块选择：每个模块“什么时候用”

一个页面只保留支撑阅读路径的强调；每章最多一个有分量的重点块；不为丰富而堆块。

| 块 | 什么时候用 |
|---|---|
| Callout | 独立信息、打断主线、值得注意 |
| Toggle | 长补充、步骤、FAQ、代码、检查表；主线不折叠 |
| Toggle Heading | 一个标题下带可折叠内容，且内容由标题统领 |
| Columns | 两个以上真正并列或对照的短块 |
| Table | 三个以上对象按相同字段比较 |
| Quote | 引用原文或保留一段完整的话 |
| Code / Mermaid | 代码或关系/流程图示 |
| Block equation | 独立展示的公式 |
| Divider | 需要停顿又不值得新标题 |
| Empty block | 显式分隔模块；块自带间距，不频繁用 |
| Tabs | 内容互相可切换 |
| Synced block | 同一段内容需要在多个地方保持同步 |
| Table of contents | 长文档需要目录导航 |
| Media（视频/音频/文件/PDF/图片） | 需要展示文件或媒体 |
| Database / Chart | 持续变化、需要筛选排序或可视化；见 notion-data.md |

## Callout：图标与颜色

Callout 是“色块卡片”，适合划出区域。每页最多 2-3 个，不要每个重点都套。

| 场景 | icon | color |
|---|---|---|
| 提示/技巧 | 💡 | blue_bg |
| 警告/注意 | ⚠️ | yellow_bg |
| 重点/核心 | ✅ | green_bg |
| 补充说明 | 📌 | gray_bg |
| 问题/思考 | ❓ | purple_bg |
| 目标/结果 | 🎯 | red_bg |
| 代码相关 | 💻 | brown_bg |

Callout 内可放标题、列表、引用、Toggle、子 Callout；子块 Tab 缩进。颜色用 _bg 变体；纯色名只改文字颜色。内部用 Notion Markdown，不用 HTML。

## Toggle 与 Toggle Heading

Toggle 用 details 标签，summary 是标题，内容 Tab 缩进。

Toggle Heading 让标题本身可折叠：标题行尾加折叠属性，子内容 Tab 缩进。子内容不缩进就不会包进 toggle。

## Columns

columns 标签包裹多个 column，每栏设 ratio（百分比，加起来 100）。新创建时可省略；更新已有分栏时每栏都要显式设 ratio。

每个柱内可放多个块（段落、列表、Callout 等），适合并置对照。

## Table：列宽怎么分

表头要明确，单元格只能放富文本，不要用 raw HTML。

列宽按信息承载量分配（总和约 720px，Notion 默认内容区宽度）：

- 序号/状态/操作：80~120px
- 名称/标题：160~240px
- 描述/正文：300~480px

fit-page-width 控制是否填满页面宽度；width 可留空自动分配，也可显式设置。颜色优先级：单元格 > 行 > 列。

## 列表：什么情况用哪种

- 无序列表：并列关系。
- 有序列表：步骤或优先级。
- 待办列表：任务清单。

列表项必须有内联富文本，否则渲染为空项。

## 引用

多行引用用 br 标签，不要用普通换行（会拆成多个块）。行尾可加颜色。

## 代码与 Mermaid

代码块用语言标识；内容原样输出，不转义。

Mermaid 用 mermaid 语言；节点文字含括号等特殊字符时用双引号；换行用 br 标签。

## 公式

行内公式：反引号包裹公式内容，写在普通文本里。

块公式：双美元符包裹，独立展示。

## 链接：按目标类型选呈现方式

不要一律用下划线文本链接。

- Notion 页面：mention-page（带图标、解析标题）
- Notion 数据库：mention-database
- Notion 数据源：mention-data-source
- 用户：mention-user
- 日期/时间：mention-date
- 外部网站：普通 [text](url)

mention 的 url 必须是真实存在的实体；inner text 可选。描述性文字：[查看官方文档](url)，不要写 [点击这里](url)。

## 媒体块

图片、视频、音频、文件、PDF 有对应标签；HTML 附件必须用 embed 标签，不要用 code/file。

## Synced block 与 Table of contents

Synced block：同一段内容多处同步；新建时省略 url，读取时才有 url。

Table of contents：长文档放目录，table_of_contents 自闭合标签。

## 颜色与强调：什么时候用

颜色只在承载语义时用：警告、重点、状态。默认不加。

- 文字色：span 加 color
- 背景色：span 加 color，值用 _bg
- 块级颜色：行尾加 color 属性
- 可用：gray、brown、orange、yellow、green、blue、purple、pink、red

强调：加粗关键词，斜体术语，下划线少数情况。

## 空行与节奏

块自带间距，空行不是默认节奏工具。只在显式分隔模块时用：

- 语义观点之间（一个观点 1-2 段完成，同一观点内不插）
- 卡片块（Callout、mention-page）前后各一个
- 列表/表格整体前后各一个
- 每个 H2 标题后一个

必须独占一行，前后有其他文本；不要用裸空行。

## UI 专属：不要承诺自动创建

- Web Bookmark 卡片：MCP 拒绝创建；输出裸 URL，用户粘贴转卡片。
- Button、Breadcrumb：Markdown 会被转义成字面文本。
- Tabs：结构可创建，title 属性写入后丢失，标题需用户设置。
- Cell merging：不支持通过 Markdown 创建/修改。

## 安全边界

- page 标签：引用已有页面会移动它，删除标签删除子页。纯引用用 mention-page。
- database 标签：同理会移动数据库。
- folder：不能由 Markdown 创建。
