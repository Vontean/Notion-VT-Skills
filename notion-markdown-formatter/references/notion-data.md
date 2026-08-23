# Notion 数据块边界（MCP 实测）

数据库、图表、视图不是页面 Markdown，是工具调用。具体参数读 create_database / create_view schema；View DSL 读 notion://docs/view-dsl-spec。这里只记决策和坑。

## 什么时候用数据块

- 数据持续变化、需要多人维护、筛选排序：Database。
- 数量、分布、趋势需要可视化：Chart view。
- 同一份数据放页面并切换表格/看板/画廊/时间线：create_view 的 parent_page_id + data_source_id 生成 inline linked view。

## 不能做的事

- 页面内容里写 database 标签不能创建真实数据块；读取时看到的 database 块是只读表示。
- 不要直接写 data source URL 在页面 Markdown 里“嵌入”，要用 create_view。
- 不要修改系统只读属性。

## 常用坑

- 创建数据前先 fetch 数据库，确认 data_source 和属性名。
- create_view 的 parent_page_id 会把 linked view 追加到页面末尾。
- Chart 必须先 GROUP BY 再配置 CHART；不分组不会生成有意义的图。
- query_data_sources 在这 workspace 有使用限制。
- 数据块适合持续更新的数据，不适合一次性文章排版；文章排版优先用 Markdown 块。
