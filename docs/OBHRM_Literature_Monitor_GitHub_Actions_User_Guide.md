# OBHRM Literature Monitor GitHub Actions User Guide

本指南面向第一次使用本项目的老师和同学。你不需要安装 Codex，不需要安装 Python，也不需要在自己的电脑上配置运行环境。只要有 GitHub 账号，就可以在网页上生成一份 literature report。

## 1. 这个工具会做什么

你在 GitHub Actions 页面输入：

- 最多 5 个关键词或概念
- 关键词匹配方式：`any` 或 `all`
- 时间窗口和时区
- 一个或多个期刊/平台列表

系统会自动在目标来源中检索文献，并生成：

- 公开网页形式的 HTML report
- Markdown report
- CSV 文献数据表
- `obhrm_scan_trace.csv` 抓取过程审计表
- `obhrm_keyword_trends.json` 关键词年度趋势数据

如果仓库配置了 Lark webhook，系统还会把简短摘要推送到 Lark 群。

重要边界：本工具不会自动登录学校系统，不会绕过付费墙，不会下载 PDF，也不会处理 CAPTCHA。报告只提供公开元数据和 DOI 链接。读者如果想看全文，需要使用自己的学校账号或其他合法权限手动访问。

## 2. 第一次使用前

1. 打开项目仓库：https://github.com/qlq20011120/Auto-literature-scrapling
2. 点击右上角 `Fork`，把仓库复制到自己的 GitHub 账号下。
3. 进入自己 fork 后的仓库。
4. 点击 `Settings` -> `Pages`。
5. 如果页面要求选择部署方式，请选择 `GitHub Actions`。

为什么要 fork：普通用户通常不能直接在别人的仓库里点击 `Run workflow`。fork 到自己的账号后，就可以在自己的仓库里运行 workflow，并生成自己的公开报告网页。

## 3. 如何运行一次检索

1. 进入自己 fork 后的仓库页面。
2. 点击顶部的 `Actions`。
3. 在左侧选择 `Generate OBHRM Literature Report`。
4. 点击 `Run workflow`。
5. 按照下面说明填写表单。
6. 点击绿色 `Run workflow` 按钮。
7. 等待 workflow 运行完成。

GitHub 页面里的进度条是 GitHub Actions 自带功能。每一步完成后会自动变成绿色。

## 4. 表单填写说明

### 4.1 Keywords

最多可以输入 5 个关键词或概念，每个输入框只填一个。

示例：

```text
keyword_1: Presenteeism
keyword_2:
keyword_3:
keyword_4:
keyword_5:
```

空白输入框会被忽略。上面的例子只会按照 `Presenteeism` 检索，不会自动保留之前出现过的 `AI`、`LLM` 或其他关键词。

如果要输入多个关键词：

```text
keyword_1: "Business History"
keyword_2: Asia
keyword_3: Engagement
keyword_4:
keyword_5:
```

### 4.2 引号短语检索

如果你希望一个多词概念按固定顺序一起出现，可以用英文双引号包起来：

```text
"Business History"
```

系统会把外层引号去掉，并把它当作一个完整短语处理。也就是说，它会匹配 `Business History in Asia`，但不会把 `the business school and the end of history` 当成 `Business History`。

当前支持的关键词语法很克制：

- 支持普通关键词，例如 `Asia`
- 支持多词短语，例如 `Business History`
- 支持带英文双引号的短语，例如 `"Business History"`
- 支持 `any` / `all` 匹配模式

暂时不支持 Web of Science advanced search 里的完整 query builder 语法，例如复杂括号、通配符、字段限定符等。

### 4.3 Keyword Matching Mode

`match_mode` 有两个选项：

- `any`：或逻辑。文章命中任意一个非空关键词就会保留。
- `all`：并逻辑。文章必须同时命中所有非空关键词才会保留。

示例：

```text
keyword_1: "Business History"
keyword_2: Asia
keyword_3: Engagement
match_mode: all
```

这表示文章需要同时命中 `Business History`、`Asia` 和 `Engagement`。

### 4.4 系统到底检索文章的哪些位置

生产 workflow 使用 `openalex-source` 策略：

1. 先把所选期刊/平台解析为 OpenAlex source。
2. 对每个 source、每个关键词、每个时间窗口进行 OpenAlex 检索。
3. OpenAlex 候选检索主要依赖公开元数据中的 title/abstract search 能力。
4. 拿到候选结果后，系统会在本地再次检查文章的 `title`、`abstract`、`keywords`。
5. 最终报告和 CSV 中的 `matched fields` 会显示到底是哪些字段命中了关键词。

因此，最稳妥的理解是：系统会尽量在标题、摘要和关键词元数据中寻找匹配，但它依赖公开数据库提供的元数据质量。如果某篇文章没有公开摘要或关键词，系统只能基于可见字段判断。

### 4.5 Timezone

选择时间窗口使用的时区：

- `Asia/Tokyo`：日本时间
- `America/Chicago`：美国中部时间，GitHub 会按日期自动处理 CST/CDT
- `Asia/Shanghai`：北京时间

### 4.6 Start Date / Start Clock

这是检索窗口的开始时间，包含这个时间点。

推荐格式：

```text
start_date: 2026/05/18
start_clock: 00:00
```

也可以写成：

```text
start_date: 26/05/18
start_clock: 8:00
```

### 4.7 End Date / End Clock

这是检索窗口的结束时间，不包含这个时间点。

推荐格式：

```text
end_date: 2026/05/25
end_clock: 00:00
```

结束时间必须晚于开始时间。如果结束时间早于或等于开始时间，workflow 会失败，并显示明确的错误提示。

### 4.8 Journal List Checkboxes

可以同时勾选多个期刊/平台列表。系统会自动取并集并去重。

可选列表包括：

- `all-whitelist`：完整白名单，范围最广
- `abs-4-and-4-star`：ABS/AJG 2024 中 4 和 4* 来源
- `abs-4-star`：ABS/AJG 2024 中 4* 来源，默认勾选
- `ft50`：FT50 来源
- `utd24`：UTD24 来源

如果同时勾选 `abs-4-star` 和 `ft50`，系统会搜索两个列表合并后的所有来源。

注意：这些列表之间本来就有大量重叠，所以多选之后 target sources 数量不一定会简单相加。例如，如果 `abs-4-star` 的来源已经大多包含在你勾选的其他列表中，那么同时勾选多个列表后，最终 target sources 数量可能变化很小，甚至保持不变。这不是 bug，而是去重后的并集结果。

如果一个列表都不勾选，workflow 会失败并提示至少选择一个列表。

### 4.9 Output Label

可以留空。留空时系统会自动生成报告文件夹名称。

如果希望 URL 或 artifacts 文件夹更容易识别，可以填写一个简短英文标签，例如：

```text
business-history-asia
```

### 4.10 Public Site URL

通常留空。

留空时，系统会自动使用当前 fork 仓库的 GitHub Pages 地址：

```text
https://<github-user>.github.io/<repo-name>/
```

只有当你自己维护 Netlify 或自定义域名时，才需要填写这一项。

### 4.11 Push Lark

如果仓库已经配置 Lark webhook secrets，可以保持开启。否则即使开启，系统也会跳过 Lark 推送，不影响报告生成。

## 5. 如何查看结果

workflow 运行完成后，有三种查看方式。

### 5.1 查看公开网页报告

打开刚刚完成的 workflow run，在页面中找到 summary。里面会显示：

```text
Public report: https://<github-user>.github.io/<repo-name>/reports/<run-folder>/
Public index: https://<github-user>.github.io/<repo-name>/
```

点击 `Public report` 可以看到本次 HTML report。

点击 `Public index` 可以看到这个仓库已经生成过的报告列表。列表右侧会显示运行者 GitHub 账号和运行时间；旧报告没有这些元数据时，会显示 `legacy report`。

### 5.2 下载 artifacts

在 workflow run 页面底部找到 `Artifacts`，下载 `obhrm-report-...`。

压缩包通常包含：

- `obhrm_daily_report.md`
- `obhrm_daily_report.html`
- `obhrm_daily_records.csv`
- `obhrm_scan_trace.csv`
- `obhrm_keyword_trends.json`
- `run.log`

### 5.3 查看 Lark 摘要

如果配置了 Lark 推送，Lark 群会收到一条简短摘要。摘要只包含：

- 关键词
- 时间窗口
- 每个期刊或平台命中的文章数量
- 公开网页报告链接

完整文献信息请打开 HTML report。

## 6. HTML report 中的新图表怎么理解

HTML report 会出现 `Keyword Trajectories` 区块：

- 每张小卡片对应一个关键词。
- 横轴是年份。
- 纵轴是该关键词在所选来源和时间窗口内的候选文献出现次数。
- 点击小卡片可以放大查看。
- `Combined Keyword Trajectories` 会把多个关键词画在同一张总折线图里，方便比较趋势。

重要说明：趋势图展示的是每个关键词各自的年度候选出现频率。即使 `match_mode` 选择 `all`，趋势图仍然按单个关键词分别统计，用来观察概念热度；最终文章列表才是按 `all` 或 `any` 逻辑过滤后的结果。

## 7. 抓取是否完整

当前生产 workflow 使用 `openalex-source` 策略：

1. 先把每一本期刊或平台解析成 OpenAlex source id。
2. 再逐一搜索每个 source、每个关键词、每个时间窗口。
3. 使用 OpenAlex cursor pagination 持续翻页，直到 OpenAlex 返回没有下一页。

这意味着生产 workflow 默认不是只从前 2000 条里挑结果。它会按 source/concept/window 组合尽量完整遍历。

审计文件 `obhrm_scan_trace.csv` 可以用来检查每个 source/concept 的遍历情况。重点看：

- `api_total_count`：OpenAlex 认为该查询共有多少条
- `fetched_count`：实际抓取了多少条
- `pages_fetched`：实际翻了多少页
- `status`：是否 `complete`
- `query_url`：对应的 OpenAlex 查询链接

如果 `status` 不是 `complete`，说明该 source/concept 可能没有完整抓完，需要查看 `run.log` 和 `query_url`。

## 8. 常见问题

### 为什么我在别人的仓库里看不到 Run workflow？

GitHub 只允许有足够权限的人在某个仓库里运行 workflow。普通老师同学应该先 fork 仓库，然后在自己的 fork 中运行。

### 为什么报告网页打开是 404？

通常是 GitHub Pages 尚未部署完成。请等待一两分钟，再刷新页面。

如果仍然 404，请检查：

- `Settings` -> `Pages` 是否选择了 `GitHub Actions`
- workflow 是否完整运行成功
- summary 里的 URL 是否来自自己的 fork 仓库

### 为什么结果非常多？

可以缩小搜索范围：

- 使用 `abs-4-star` 而不是 `all-whitelist`
- 使用 `all` 而不是 `any`
- 缩短时间窗口
- 增加更具体的关键词或短语
- 使用双引号输入固定短语，例如 `"Business History"`

### 为什么 abstract 特别长？

报告会尽量保留公开元数据 API 返回的 abstract 信息。目前不会自动截断 abstract，因为截断可能损失科研判断所需的信息。如果只想快速浏览，可以先看标题、期刊、DOI、matched concepts 和 matched fields。

## 9. 推荐填写模板

### 精准短语检索

```text
keyword_1: "Business History"
keyword_2: Asia
keyword_3:
keyword_4:
keyword_5:
match_mode: all
timezone: Asia/Tokyo
start_date: 1970/01/01
start_clock: 00:00
end_date: 2026/06/01
end_clock: 00:00
journal lists: abs-4-star + ft50
```

### 单关键词检索

```text
keyword_1: Presenteeism
keyword_2:
keyword_3:
keyword_4:
keyword_5:
match_mode: any
timezone: Asia/Tokyo
start_date: 2026/05/18
start_clock: 00:00
end_date: 2026/05/25
end_clock: 00:00
journal lists: abs-4-star
```

### 多关键词或逻辑检索

```text
keyword_1: AI
keyword_2: LLM
keyword_3: "Large Language Model"
keyword_4:
keyword_5:
match_mode: any
timezone: Asia/Tokyo
start_date: 2026/05/18
start_clock: 00:00
end_date: 2026/05/25
end_clock: 00:00
journal lists: abs-4-star + ft50 + utd24
```
