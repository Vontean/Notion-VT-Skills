---
name: notion-markdown-formatter
description: Converts basic Markdown text into Notion-flavored Markdown — a hybrid format mixing standard Markdown with XML tags for Callout boxes, Toggle lists, Tables with colors, Columns, and all supported Notion block types. 将普通 Markdown 转换为 Notion 风格排版（含 XML 块标签）：Callout 提示框、Toggle 折叠、分栏、颜色标记、富文本对齐 Notion 实际存储格式。适用于把文档整理进 Notion MCP 工具的场景。
---

## 核心认知：Notion 使用的是混合格式，不是纯 Markdown

Notion MCP 工具和 `body.storage` 使用的是 **Notion-flavored Markdown**——标准 Markdown + XML 标签的混合体。单纯用 Markdown 写不出 Callout、Toggle、分栏等 Notion 核心特性。

**格式分层规则：**

| 层级 | 表达方式 | 用途 |
|------|---------|------|
| 简单块 | `# ## ### ####` `-` `1.` `- [ ]` `> ` `---` | 标题、列表、待办、引用、分隔线 |
| 富文本 | `**` `*` `~~` `` ` `` `[text](url)` `[^URL]` `` $`eq`$ `` | 粗斜删代、链接、引用、内联公式 |
| 高级块 | `<callout>` `<details>` `<columns>` `<table>` `<synced_block>` 等 | Callout、Toggle、分栏、完整表格 |
| 媒体块 | `![](url)` `<video>` `<audio>` `<file>` `<pdf>` `<embed>` | 图片、视频、音频、文件 |
| 属性 | `{color="..."}` `{toggle="true"}` | 块颜色、Toggle 标题、块级参数 |
| 内联标记 | `<span underline>` `<span color>` `<br>` | 下划线、内联颜色、内联换行 |

**关键规则：**
- 子块缩进**必须用 Tab**（`\t`），不是空格
- 需要转义的字符：`\ * ~ ` $ [ ] < > { } | ^`
- 不支持 heading 5/6 — 自动折成 heading 4
- 空行会被 Notion 自动吃掉；如需显式空行，用 `<empty-block/>`
- 代码块内**不要**转义特殊字符，内容原样输出

---

## 处理流程

### Step 1: 读取并分析原始内容

1. 如果用户用 `@quote` 引用文件，用 `read` 工具读取
2. 如果用户直接粘贴文本，直接用
3. 分析内容结构和语义意图：
   - 识别标题层级（H1-H4，H5/H6 折为 H4）
   - 识别列表、代码块、引用块
   - **判断哪些内容适合转为 Notion 高级块**（见 Step 2-7）
   - 识别长段落或详细说明（可折叠）
   - 识别链接并判断目标类型（Notion 内部页面 / 外部网站，见 Step 6 链接可视化决策）
4. 确保完整获取后再处理

### Step 2: 优化标题层级

按 Notion 规范整理：
1. 确保文档有明确的 H1 主标题
2. H2 为主章节，H3 为子章节
3. 不跳级（H1 → H3 直接跳不行）
4. 段落组缺标题就补
5. 标题简洁清晰，反映内容要点
6. **标题颜色**：行末加 `{color="Color"}` 属性
7. **Toggle 标题**：行末加 `{toggle="true" color?="Color"}`，子内容用 Tab 缩进

### Step 3: 识别并转换 Callout 提示

**Notion 原生 Callout 格式（不是 Markdown 引用！）：**
```xml
<callout icon="💡" color="blue_bg">
  正文内容（支持多行、子块）
</callout>
```

**场景对应：**

| 场景 | icon | color |
|------|------|-------|
| **💡 提示/技巧** | `"💡"` | `"blue_bg"` |
| **⚠️ 警告/注意** | `"⚠️"` | `"yellow_bg"` |
| **✅ 重点/核心** | `"✅"` | `"green_bg"` |
| **📌 补充说明** | `"📌"` | `"gray_bg"` |
| **❓ 问题/思考** | `"❓"` | `"purple_bg"` |
| **🎯 目标/结果** | `"🎯"` | `"red_bg"` |
| **💻 代码相关** | `"💻"` | `"brown_bg"` |

**callout 必须使用 `_bg` 背景色变体**渲染彩色气泡；纯色名（如 `blue`）只作用于文字，背景保持默认浅灰，视觉效果远弱于背景色版。

判断原文关键信息适合转为 Callout 则转。Callout 内可包含子块（Tab 缩进）。

### Step 4: 优化长内容 — Toggle 折叠

**Notion 原生 Toggle 格式：**
```xml
<details color?="Color">
<summary>折叠标题</summary>
  折叠内容（Tab 缩进）
  可以包含多个子块
</details>
```

**Toggle 标题（折叠式标题）：**
```
## 章节名称 {toggle="true" color?="Color"}
  折叠内容（Tab 缩进）
```

**适用场景：**
1. 详细技术步骤或操作说明
2. 补充材料和参考信息
3. 示例代码和详细解释
4. FAQ（每条问答一个折叠）
5. 长清单或详细检查表

将适合折叠的内容转为 Toggle，保持主文档精炼。

### Step 5: 优化列表和表格

**列表优化：**
- 无序列表 `-` 表达并列关系
- 有序列表 `1. 2. 3.` 表达步骤或优先级
- 待办列表 `- [ ]` / `- [x]` 表达任务
- 层级用 Tab 缩进保持清晰
- **列表项必须有内联富文本**，否则会渲染为空项
- 列表项也可加 `{color="Color"}` 属性

**表格优化 — 使用 Notion 完整表格格式：**

```xml
<table fit-page-width="true" header-row="true">
  <colgroup>
    <col color="gray">
    <col>
  </colgroup>
  <tr color="blue_bg">
    <td>列标题</td>
    <td>列标题</td>
  </tr>
  <tr>
    <td>内容</td>
    <td color="green">带颜色单元格</td>
  </tr>
</table>
```

**表格规则：**
- `fit-page-width`: 是否填充满页宽
- `header-row`: 首行是否为表头
- `header-column`: 首列是否为表头
- 颜色优先级：单元格 > 行 > 列
- 单元格**只能**包含富文本，不能嵌套其他块类型
- 可以使用状态 emoji：✅ 🔄 ⏳ ❌ ⚠️ 🔵 🟢 🟡 🔴

### Step 6: 优化代码块、公式和引用

**代码块：**
````
```python
# 示例
print('Hello')
```
````
- 必须设正确的语言标识（`mermaid` / `python` / `javascript` / `bash` 等）
- 代码块内容是原样文字，**不要转义**
- Mermaid 图表：使用 `<br>` 换行，节点文字有特殊字符时包在双引号内

**块公式（居中显示）：**
```
$$
E = mc^2
$$
```

**内联公式：**
```
$`E=mc^2`$
```

**引用块优化：**
- 单行引用：`> 引用文字`
- 多行引用用 `<br>` 连接，不要用普通换行（会导致多个独立引用块）
- 如适合 Callout 视觉风格则转为 `<callout>`

**链接可视化决策（按目标类型选择渲染方式，不要一律用下划线文本链接）：**

| 目标类型 | 判断依据 | 渲染方式 | Notion 渲染效果 |
|---------|---------|---------|----------------|
| Notion 页面 | URL 含 `app.notion.com/p/` 或 `notion.so` | `<mention-page url="..."/>` | 带页面图标 + 解析标题的可点击引用（`/link to page`） |
| Notion 数据库 | URL 为数据库页 | `<mention-database url="..."/>` | 数据库引用 |
| Notion 数据源 | collection 链接 | `<mention-data-source url="..."/>` | 数据源引用 |
| Notion 用户 | 用户主页链接 | `<mention-user url="..."/>` | @用户 引用 |
| 具体日期/时间 | 明确的时间点 | `<mention-date start="2026-08-03"/>` | 日期 mention |
| 外部网站 | 其他域名 | `[描述性文字](url)` | 下划线文本链接 |

**关键规则：**
1. **Notion 内部链接优先用 mention 标签**（`<mention-page>` 等），不要降级为 `[text](url)` 文本链接。mention 在 Notion 里渲染为带实体图标 + 解析标题的引用块，视觉远比下划线链接强。
2. `<mention-page url="..."/>` 的自闭合写法即可，标题内文会被 Notion 忽略并自动解析实际页面名（实测带标题写法也会被规范化为自闭合）。
3. **外部链接无法在 MCP 格式中生成卡片**：实测 `<bookmark>` 会被解析成 `<unknown>` 块，裸 URL 会降级为文本链接。MCP 的 markdown 格式不支持创建 bookmark（网页书签）或 link_preview（链接预览）卡片——那是用户在 Notion UI 里粘贴 URL 时才触发的行为。因此外部链接一律用 `[描述性文字](url)`，不要尝试其他语法。
4. 描述性文字：`[查看官方文档](url)` 而非 `[点击这里](url)`

### Step 7: 增强可读性和视觉层级

**分栏布局：**
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
- `ratio` 为百分比，两栏加起来 100
- 创建新栏时每栏都设 ratio；省略则均分

**段落优化：**
- 段落间留适空格（但避免多余空行，Notion 会自动间距）
- 长段落拆分为多个短段落
- 每段聚焦一个主题

**强调和格式：**
- **加粗**突出关键词
- *斜体*表示术语或引用
- `` `内联代码` `` 标注技术名词
- `<span underline="true">下划线</span>` 用于强调（少数情况）

**内联颜色：**
- 文字色：`<span color="red">红色文字</span>`
- 背景色：`<span color="red_bg">红底文字</span>`
- 可用颜色：gray, brown, orange, yellow, green, blue, purple, pink, red（加 `_bg` 后缀变背景）

**块级颜色：**
行末加 `{color="green"}` 给整个块上色。文本色 / 背景色都支持。

**视觉分隔：**
- 用 `---` 分隔大章节
- 适当留空创造视觉呼吸感

**内联 Emoji 使用：**
- 标题或 Callout 中适度用 emoji 增强可读性
- 避免滥用，保持专业感

### Step 8: 匹配页面 Icon 和封面图

在输出文档内容之前，自动为文档选择合适的 **页面 icon（emoji）** 和 **封面图片 URL**，方便用户在 Notion 中快速定位。

#### 8.1 Emoji Icon 选择

根据文档主题和类型，从以下映射选择最合适的 emoji，如有多个候选选最贴合的那个：

| 文档类型 | 候选 Emoji |
|---------|-----------|
| 技术方案 / 架构设计 | 🏗️ 🔷 🧱 ⚙️ |
| API 文档 / 接口说明 | 🔌 📡 🔗 🧩 |
| 代码 / 开发相关 | 💻 ⌨️ 🛠️ 🚀 |
| 产品需求 / PRD | 📋 🎯 🗂️ ✨ |
| UI/UX 设计 | 🎨 🖌️ 📱 🖼️ |
| 用户研究 / 访谈 | 🧑‍💻 🔍 💬 🗣️ |
| 数据分析 / 报告 | 📊 📈 🔢 🧮 |
| 会议记录 | 📝 🗒️ 🎙️ 👥 |
| 项目规划 / Roadmap | 🗺️ 🏁 📅 🧭 |
| 学习笔记 / 教程 | 📚 ✏️ 🧠 💡 |
| 部署运维 | 🚢 🐳 ☁️ 🔧 |
| 安全审计 | 🔒 🛡️ 🔐 ⚠️ |
| 测试相关 | 🧪 ✅ 🔍 🎯 |
| 写作 / 内容创作 | ✍️ 📖 📰 🖊️ |
| 复盘 / Retro | 🔄 🪞 💭 🌱 |
| 个人成长 / OKR | 🌱 🏆 🎯 🚵 |
| 财务 / 预算 | 💰 🧾 📊 💵 |
| 通用 / 其他 | 📄 ✨ 📌 |

判断逻辑：
1. 先读文档标题和开头几段，判断文档大类
2. 从对应类别的候选 emoji 中选最匹配的
3. 如果文档有明显主题词（如 "API" "数据库" "用户访谈"），优先主题词匹配
4. 不确定时选 ✨ 或 📄 作为安全默认

#### 8.2 封面图搜索

封面图从 Unsplash 获取，流程如下：

**Step 1 — 提取搜索关键词：**
从文档标题和核心内容中提取 1-3 个英文关键词。例如：
- "用户增长策略分析" → `growth strategy`
- "React 组件库设计规范" → `react code design`
- "Q3 产品路线图" → `product roadmap planning`

**Step 2 — 搜索 Unsplash 图片：**
优先用 WebFetch 直接打开 Unsplash 搜索页：
```
https://unsplash.com/s/photos/{keywords}
```
（实测 WebSearch 搜 `site:unsplash.com` 经常返回空结果，直接 WebFetch 搜索页更可靠。）

如 WebFetch 受限不可用，再用 firecrawl_search 兜底：
```
site:unsplash.com {keywords} wallpaper
```

**Step 3 — 提取图片 URL：**
从搜索页中取第一张图片。Unsplash 图片 URL 格式为：
```
https://images.unsplash.com/photo-{photo_id}?w=1200&h=630&fit=crop
```
- 从页面 meta 标签或 `img src` 中提取 `photo-{id}` 部分
- `w=1200&h=630&fit=crop` 参数生成适合 Notion 封面的 1200×630 裁剪图
- 若搜索页抓取失败（页面结构变化或反爬），降级到 Step 4 的通用封面

**Step 4 — 备选方案（搜索不可用时）：**
如果网络搜索受限，使用以下通用封面 URL（选择与主题最接近的）：

| 主题 | 备用封面 URL |
|------|------------|
| 科技/代码 | `https://images.unsplash.com/photo-1518770660439-4636190af475?w=1200&h=630&fit=crop` |
| 设计/创意 | `https://images.unsplash.com/photo-1558655146-9f40138edfeb?w=1200&h=630&fit=crop` |
| 自然/环境 | `https://images.unsplash.com/photo-1501854140801-50d01698950b?w=1200&h=630&fit=crop` |
| 城市/建筑 | `https://images.unsplash.com/photo-1477959858617-67f85cf4f1df?w=1200&h=630&fit=crop` |
| 抽象/极简 | `https://images.unsplash.com/photo-1557683316-973673baf926?w=1200&h=630&fit=crop` |
| 商业/办公 | `https://images.unsplash.com/photo-1497366216548-37526070297c?w=1200&h=630&fit=crop` |
| 纸张/文档 | `https://images.unsplash.com/photo-1512758017271-d7b84c2113f1?w=1200&h=630&fit=crop` |

#### 8.3 输出格式

在文档内容开头，显式标明 icon 和 cover：

```
<!-- notion-page-icon: 🚀 -->
<!-- notion-page-cover: https://images.unsplash.com/photo-xxx?w=1200&h=630&fit=crop -->
```

后续使用 Notion MCP 工具（create-pages / update-page）时，将 icon 和 cover 作为参数传入。

### Step 9: 生成优化文档

1. 用 Write 工具创建新文档，包含优化后的内容
2. 文档标题格式：`"原标题 - Notion Optimized Version"`
3. 文档最顶部标注 icon 和 cover（Step 8 的产出）：
```
<!-- notion-page-icon: {选定的 emoji} -->
<!-- notion-page-cover: {搜索到的封面图 URL} -->
```
4. 接着是文档说明 Callout：
```xml
<callout icon="📝" color="gray_bg">
  **Document Description**
  This document has been optimized according to Notion formatting specifications, including:
  - ✅ Optimized heading hierarchy
  - ✅ Callout boxes
  - ✅ Toggle collapsible lists
  - ✅ Formatted lists and tables
  - ✅ Enhanced readability
  - 🎨 Page icon & cover image
  This content can be directly used with Notion MCP tools.
</callout>
---
```
5. 之后是完整的 Notion-flavored Markdown 内容
6. 告知用户：
   - 文档已生成，可直接用于 Notion MCP 工具
   - 页面 icon（emoji）和封面图 URL 已标注在文档顶部
   - 使用 notion-create-pages 时，将 `icon` 和 `cover` 参数传入即可自动设置

### Step 10: 提供使用说明

告知用户：
1. 优化后的文档已创建
2. 内容格式为 **Notion-flavored Markdown**（含 XML 标签），可直接作为 Notion MCP 工具（create-pages / update-page）的输入
3. 展示效果说明：
   - `<callout>` → 带颜色背景和图标的气泡提示框
   - `<details>` → 可折叠/展开的内容块
   - `<columns>` → 多栏并排布局
   - `<table>` → 完整表格（含颜色、列宽）
   - `{toggle="true"}` → 可折叠标题
4. 如需调整，随时告知
5. 询问是否需要进一步优化

### 补充：从 Notion 格式逆向转换

如果用户提供的是 Notion body.storage（XML 混合格式），需要转换回可读格式：
1. 标题/列表/引用等 Markdown 部分保持不变
2. `<callout>` → 提取 icon 和正文，展示为 `> 💡 **标题**` 样式
3. `<details>` → 提取 summary 作为标题，子内容保留
4. `<columns>` → 拆分各 `<column>` 为独立区块
5. `<table>` → 转为 Markdown 表格格式
6. `<mention-page>` → 转为 `[[页面名称]]` 链接
7. `<mention-user>` → 转为 `@用户名`
8. `<mention-date>` → 转为日期文字
