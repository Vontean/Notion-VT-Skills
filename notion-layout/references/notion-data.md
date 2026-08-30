# Notion 数据块设计调用

数据库、图表、视图不是页面 Markdown，是 Notion 工具调用。这里回答：什么时候用数据块、用哪种 view、怎么创建。具体参数见 create_database / create_view schema；View DSL 见 notion://docs/view-dsl-spec。

## 什么时候用数据块

- 数据持续变化、需要多人维护、筛选排序：Database。
- 数量、分布、趋势需要可视化：Chart view。
- 同一份数据放页面并切换展示方式：inline Database view（create_view 的 parent_page_id + data_source_id）。
- 一次性文章或静态内容：不用数据块，用 Markdown 块，避免引入维护成本。

## 数据块 vs 表格：怎么选

- 数据只展示一次、不变：表格（Markdown）。
- 数据以后会增删改、需要筛选/排序/视图：Database。
- 需要图表：Database + Chart view。

## 创建流程

1. 创建 Database：create_database，指定 schema 和标题，拿到 data_source_id。
2. 添加数据：create_pages 到 data_source_id。
3. 数据可视化或嵌入页面：create_view。

## View 类型：什么时候用哪种

| 视图 | 什么时候用 |
|---|---|
| table | 默认，数据行视图 |
| board | 按状态/分组看板 |
| gallery | 卡片式展示，适合封面内容 |
| list | 简化列表 |
| calendar | 日期属性驱动的日历 |
| timeline | 开始/结束日期的时间线 |
| chart | 数量、分布、趋势可视化 |
| map | 位置/地点属性 |
| form | 收集提交 |
| dashboard | 组合多个视图/图表概览 |

## Chart 怎么配

Chart 必须先 GROUP BY 一个属性，再配置图表类型和聚合。

- 类型：column、bar、line、donut、number
- 聚合：count、sum、average、min、max（可 ON 一个属性）
- 颜色：gray、blue、green、purple、orange、red、auto、colorful
- 可加：HEIGHT、SORT、STACK BY、CAPTION

示例思路：

GROUP BY 一个分类属性; CHART 类型 AGGREGATE 聚合 COLOR 颜色

## inline linked view：放在页面哪里

create_view 的 parent_page_id 把 linked view **追加到页面末尾**，引用已有 data_source_id。适合：

- 文末放“任务/参考资料”数据库。
- 页面某处放一个筛选过的数据快照。

## 常见坑

- 页面内容里写 database 标签不能创建真实数据块；读取时看到的 database 块是只读表示。
- 不要直接写 data source URL 嵌进页面 Markdown，要用 create_view。
- 创建前先 fetch 数据库，确认 data_source 和属性名。
- 不要修改系统只读属性。
- query_data_sources 在这 workspace 有使用限制。
