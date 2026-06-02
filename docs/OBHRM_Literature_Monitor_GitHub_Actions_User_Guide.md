# OBHRM Literature Monitor GitHub Actions User Guide

本指南面向第一次使用本项目的老师和同学。使用者不需要安装 Codex，不需要安装 Python，也不需要在自己的电脑上配置运行环境；只需要有一个 GitHub 账号，并在网页上运行 GitHub Actions。

## 1. 这个工具会做什么

本工具会根据你输入的关键词、时间窗口和期刊列表，在目标 OBHRM/HCI/preprint 来源中检索符合条件的新文献，并生成：

- 一个公开网页形式的 HTML literature report
- 一个 Markdown report
- 一个 CSV 文献数据表
- 一个 `obhrm_scan_trace.csv` 抓取过程审计表
- 如果配置了 Lark webhook，还会向 Lark 群发送简短摘要

它不会自动登录学校系统，不会绕过付费墙，不会下载 PDF。报告中只提供 DOI 链接，全文下载仍需要读者自己使用学校账号或合法权限完成。

## 2. 第一次使用前需要做什么

1. 打开项目仓库：`https://github.com/qlq20011120/Auto-literature-scrapling`
2. 点击右上角 `Fork`，把仓库复制到自己的 GitHub 账号下。
3. 进入自己 fork 后的仓库。
4. 点击 `Settings` -> `Pages`。
5. 如果页面要求选择部署方式，请选择 `GitHub Actions`。

说明：普通用户通常不能直接在别人的仓库里点击 `Run workflow`。fork 到自己的账号后，就可以在自己的仓库里自由运行。

## 3. 如何运行一次文献检索

1. 在自己 fork 后的仓库页面，点击顶部的 `Actions`。
2. 在左侧选择 `Generate OBHRM Literature Report`。
3. 点击右侧或上方的 `Run workflow`。
4. 按照下面说明填写表单。
5. 点击绿色的 `Run workflow` 按钮。
6. 等待 workflow 运行完成。

GitHub 页面上的进度条是 GitHub Actions 自带功能。每个步骤完成后会自动变成绿色。

## 4. 表单应该怎么填

### 4.1 Keywords

最多可以输入五个关键词，每个输入框只填一个关键词。

示例：

```text
keyword_1: Presenteeism
keyword_2:
keyword_3:
keyword_4:
keyword_5:
```

空白输入框会被忽略。上面的例子只会按照 `Presenteeism` 检索，不会自动加入其他关键词。

如果需要多个关键词：

```text
keyword_1: Organizational Behavior
keyword_2: Asia
keyword_3: Asian
keyword_4: Engagement
keyword_5:
```

### 4.2 Keyword Matching Mode

`match_mode` 有两个选项：

- `any`: 或逻辑。文章只要命中任意一个关键词就会被保留。
- `all`: 并逻辑。文章必须同时命中所有非空关键词才会被保留。

如果你输入了 `Organizational Behavior`、`Asia`、`Asian`、`Engagement`，并选择 `all`，那么只有同时包含这四个概念的文章才会进入报告。

### 4.3 Timezone

选择时间窗口所使用的时区：

- `Asia/Tokyo`: 日本时间
- `America/Chicago`: 美国中部时间，GitHub 会按日期自动处理 CST/CDT
- `Asia/Shanghai`: 北京时间

### 4.4 Start Date / Start Clock

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

### 4.5 End Date / End Clock

这是检索窗口的结束时间，不包含这个时间点。

推荐格式：

```text
end_date: 2026/05/25
end_clock: 00:00
```

结束时间必须晚于开始时间。如果结束时间早于或等于开始时间，workflow 会失败并显示错误提示。

### 4.6 Journal List Checkboxes

可以同时勾选多个期刊列表。系统会自动取并集并去重。

可选列表包括：

- `all-whitelist`: 最宽泛的完整白名单
- `abs-4-and-4-star`: ABS/AJG 2024 的 4 和 4* 来源
- `abs-4-star`: ABS/AJG 2024 的 4* 来源，默认勾选
- `ft50`: FT50 来源
- `utd24`: UTD24 来源

示例：如果同时勾选 `abs-4-star` 和 `ft50`，系统会搜索这两个列表合并后的所有来源，而不是只搜索其中一个。

如果一个列表都不勾选，workflow 会失败并提示至少选择一个列表。

### 4.7 Output Label

可以留空。留空时系统会自动生成报告文件夹名称。

如果你希望 URL 或 artifacts 文件夹更容易识别，可以填写一个简短英文标签，例如：

```text
presenteeism-week
```

### 4.8 Public Site URL

通常留空。

留空时，系统会自动使用当前 fork 仓库的 GitHub Pages 地址：

```text
https://<github-user>.github.io/<repo-name>/
```

只有当你自己维护 Netlify 或自定义域名时，才需要填写这个字段。

### 4.9 Push Lark

如果仓库中已经配置了 Lark webhook secrets，可以保持开启。否则即使开启，系统也会跳过 Lark 推送，不影响报告生成。

## 5. 如何查看结果

workflow 运行完成后，有三种查看方式。

### 5.1 查看公开网页报告

打开刚刚完成的 workflow run，在页面中找到 summary。里面会显示：

```text
Public report: https://<github-user>.github.io/<repo-name>/reports/<run-folder>/
Public index: https://<github-user>.github.io/<repo-name>/
```

点击 `Public report` 可以看到本次漂亮的 HTML 周报。

点击 `Public index` 可以看到这个仓库已经生成过的报告列表。

### 5.2 下载 artifacts

在 workflow run 页面底部找到 `Artifacts`，下载 `obhrm-report-...`。

压缩包通常包含：

- `obhrm_daily_report.md`
- `obhrm_daily_report.html`
- `obhrm_daily_records.csv`
- `obhrm_scan_trace.csv`
- `run.log`

### 5.3 查看 Lark 摘要

如果配置了 Lark 推送，Lark 群会收到一条简短摘要。摘要只包含：

- 关键词
- 时间窗口
- 每个期刊或平台命中的文章数量
- 公开网页报告链接

Lark 摘要不会包含完整文献信息，完整内容请打开 HTML report。

## 6. 抓取是否完整

当前生产 workflow 使用 `openalex-source` 策略：

1. 先把每一本期刊或平台解析成 OpenAlex source id。
2. 再逐一搜索每个 source、每个关键词、每个时间窗口。
3. 使用 OpenAlex cursor pagination 持续翻页，直到 OpenAlex 返回没有下一页。

这意味着生产 workflow 默认不是只从前 2000 条里挑结果。它会按 source/concept/window 组合尽量完整遍历。

审计文件 `obhrm_scan_trace.csv` 可以用来检查每个 source/concept 的遍历情况。重点看：

- `api_total_count`: OpenAlex 认为该查询一共有多少条
- `fetched_count`: 实际抓取了多少条
- `pages_fetched`: 实际翻了多少页
- `status`: 是否 `complete`

如果 `status` 不是 `complete`，说明该 source/concept 可能没有完整抓完，需要查看 `run.log` 和 `query_url`。

## 7. 常见问题

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
- 增加更具体的关键词

### 为什么 abstract 特别长？

报告会尽量保留公开元数据 API 返回的 abstract 信息。目前不会自动截断 abstract，因为截断可能损失科研信息。如果只想快速浏览，可以先看标题、期刊、DOI 和 matched concepts，再决定是否展开阅读全文。

## 8. 推荐填写模板

### 精准检索模板

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

### 多关键词并逻辑模板

```text
keyword_1: Organizational Behavior
keyword_2: Asia
keyword_3: Asian
keyword_4: Engagement
keyword_5:
match_mode: all
timezone: Asia/Tokyo
start_date: 2026/05/18
start_clock: 00:00
end_date: 2026/05/25
end_clock: 00:00
journal lists: abs-4-star + ft50
```

