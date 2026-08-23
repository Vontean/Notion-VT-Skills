# Notion 语法参考

本文件只在需要生成或阅读 Notion-flavored Markdown 时读取。它是已知可用的语法与实测约束，不是排版规则。

## 混合格式

Notion MCP 与 body.storage 使用标准 Markdown 与 XML 标签的混合格式。普通 Markdown 写不出 Callout、Toggle、分栏等块。

| 层级 | 表达方式 | 用途 |
|------|---------|------|
| 简单块 | `# ## ### ####`、`-`、`1.`、`- [ ]`、`> `、`---` | 标题、列表、待办、引用、分隔线 |
| 富文本 | 粗体、斜体、删除线、行内代码、链接、内联公式 | 强调、术语、引用与公式 |
| 高级块 | `<callout>`、`<details>`、`<columns>`、`<table>`、`<synced_block>` 等 | Callout、Toggle、分栏、完整表格 |
| 媒体块 | `![](url)`、`<video>`、`<audio>`、`<file>`、`<pdf>`、`<embed>` | 图片、视频、音频、文件 |
| 属性 | `{color="..."}`、`{toggle="true"}` | 块颜色、Toggle 标题、块级参数 |
| 内联标记 | `<span underline>`、`<span color>`、`<br>` | 下划线、内联颜色、内联换行 |

硬约束：

- 子块缩进必须用 Tab（`\t`），不是空格。
- 需要转义：反斜杠、星号、波浪号、反引号、美元符、方括号、尖括号、花括号、竖线、脱字符。
- 不支持 H5/H6，统一折成 H4。
- 空行会被 Notion 自动吃掉；需要显式空行时用 `<empty-block/>`。
- 代码块内不要转义特殊字符，内容原样输出。

## 标题

- 行末加 `{color="Color"}` 上色。
- Toggle 标题：行末加 `{toggle="true" color?="Color"}`，子内容用 Tab 缩进。

## Callout

```xml
<callout icon="💡" color="blue_bg">
  正文内容（支持多行、子块）
</callout>
```

彩色背景使用 `_bg` 变体；纯色名只改变文字颜色，背景仍是默认样式。Callout 内可包含子块（Tab 缩进）。

| 场景 | icon | color |
|------|------|------|
| 💡 提示/技巧 | `"💡"` | `"blue_bg"` |
| ⚠️ 警告/注意 | `"⚠️"` | `"yellow_bg"` |
| ✅ 重点/核心 | `"✅"` | `"green_bg"` |
| 📌 补充说明 | `"📌"` | `"gray_bg"` |
| ❓ 问题/思考 | `"❓"` | `"purple_bg"` |
| 🎯 目标/结果 | `"🎯"` | `"red_bg"` |
| 💻 代码相关 | `"💻"` | `"brown_bg"` |

## Toggle

```xml
<details color?="Color">
<summary>折叠标题</summary>
  折叠内容（Tab 缩进）
  可以包含多个子块
</details>
```

适用：详细技术步骤、补充材料、示例代码、FAQ、长清单、检查表。

## 列表

- 无序列表 `-` 表达并列关系。
- 有序列表 `1. 2. 3.` 表达步骤或优先级。
- 待办列表 `- [ ]` / `- [x]` 表达任务。
- 列表项必须有内联富文本，否则渲染为空项。列表项可加 `{color="Color"}`。

## 表格

```xml
<table fit-page-width="true" header-row="true">
  <colgroup>
    <col width="100">
    <col width="480">
    <col width="140">
  </colgroup>
  <tr color="blue_bg">
    <td>列标题</td>
    <td>列标题</td>
    <td>列标题</td>
  </tr>
  <tr>
    <td>内容</td>
    <td>内容</td>
    <td color="green">带颜色单元格</td>
  </tr>
</table>
```

规则：

- 必须 `fit-page-width="true"`，否则表格会 hug 内容、长文本被挤。
- 每列都在 `<colgroup>` 里设 `<col width="N">`。
- 渲染宽度等于各列 width 之和，总和约 720px（Notion 默认内容区宽度）。
- 列宽按信息承载量分配：序号/状态 80~120px，名称/标题 160~240px，描述/正文 300~480px。
- header-row 控制首行是否是表头，header-column 控制首列。
- 颜色优先级：单元格 > 行 > 列。
- 单元格只能包含富文本，不能嵌套其他块。
- 状态 emoji：✅ 🔄 ⏳ ❌ ⚠️ 🔵 🟢 🟡 🔴。

## 代码、公式与引用

代码块设置正确语言标识；内容不转义。Mermaid 用 `<br>` 换行，特殊字符节点包在双引号内。

块公式：
```
$$
E = mc^2
```

内联公式用反引号包裹公式内容，写在普通文本行内；例如 `$E=mc^2$` 的表达可参考富文本说明。

引用块：

- 单行引用 `> 引用文字`。
- 多行引用用 `<br>` 连接，不要用普通换行（会变成多个独立引用块）。

## 链接

| 目标类型 | 判断依据 | 渲染方式 |
|---------|---------|---------|
| Notion 页面 | URL 含 `app.notion.com/p/` 或 `notion.so` | `<mention-page url="..."/>` |
| Notion 数据库 | URL 为数据库页 | `<mention-database url="..."/>` |
| Notion 数据源 | collection 链接 | `<mention-data-source url="..."/>` |
| Notion 用户 | 用户主页链接 | `<mention-user url="..."/>` |
| 具体日期/时间 | 明确时间点 | `<mention-date start="2026-08-03"/>` |
| 外部网站 | 其他域名 | `<bookmark url="URL">标题</bookmark>` |

关键规则：

- 内部链接用 mention，不要降级为 `[text](url)`。mention 会保留实体图标并解析页面标题。
- `<mention-page url="..."/>` 自闭合即可；带标题写法也会被 Notion 规范化为自闭合。
- 外部链接用 Web Bookmark 卡片。MCP 支持 `<bookmark>` 语法：
  - `url` 是目标链接；inner text 是卡片标题。
  - 必须用带 inner text 的完整写法；自闭合会被转义成字面文本。
  - MCP 读取现有 bookmark 时显示 `<unknown url="块锚点" alt="bookmark"/>`，这是只读表示，不代表写入失败。
- 描述性文字：`[查看官方文档](url)`，不要写 `[点击这里](url)`。

## 分栏

```xml
<columns>
  <column ratio="50">
    左栏内容
  </column>
  <column ratio="50">
    右栏内容
  </column>
</columns>
```

`ratio` 为百分比，两栏加起来 100。创建新栏时每栏都设 ratio；省略则均分。

## 颜色与强调

内联颜色：

- 文字色：`<span color="red">红色文字</span>`
- 背景色：`<span color="red_bg">红底文字</span>`
- 可用：gray, brown, orange, yellow, green, blue, purple, pink, red（加 `_bg` 变背景）

块级颜色：行末加 `{color="green"}`。

强调：加粗用于关键词，斜体用于术语或引用，下划线 `<span underline="true">` 少数情况使用。

## 空行与视觉分隔

`<empty-block/>` 独占一行，前后各留一个空行分隔相邻块；不要用裸空行。

- 语义观点之间插入，同一观点内的多段不插。
- 卡片块（Callout、mention-page）作为独立模块，前后各插一个。
- 列表/表格作为整体模块，前后各插一个；内部不插。
- 每个 H2 标题后插一个，让标题与正文分离。

## 逆向转换

Notion body.storage 转回可读 Markdown：

1. 标题、列表、引用等 Markdown 保持不变。
2. `<callout>` 提取 icon 和正文，展示为 `> 💡 **标题**`。
3. `<details>` 提取 summary 作为标题，子内容保留。
4. `<columns>` 拆分各 `<column>` 为独立区块。
5. `<table>` 转为 Markdown 表格。
6. `<mention-page>` 转为 `[[页面名称]]`；`<mention-user>` 转为 `@用户名`；`<mention-date>` 转为日期。
