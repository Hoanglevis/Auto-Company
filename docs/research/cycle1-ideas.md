# Cycle 1 — 产品机会 Brief（research-thompson）

> 日期：2026-10-30 | 范围：可在 Cloudflare 上 1–2 周交付、全自助销售、低责任的开发者工具
> **重要信息盲区**：本轮 WebSearch / WebFetch 权限被拒，未做实时检索。下列链接、价格、营收数字均来自
> 训练知识（截至 2026 年中），标注为 `likely`（多源记忆一致）或 `speculative`（单源/可能已变）。
> **下一步必须**：在定价/立项前由任一有 web 权限的 agent 逐条核实"待核实"项。

## 结论先行

1. **Idea A — RenderKit：Screenshot + HTML/URL→PDF API**（snapog 转型，复用其 Workers+R2+API key 骨架）
2. **Idea B — CronSentry：GitHub Actions scheduled workflow 免埋点监控**（GitHub App + Workers Cron + D1）

两者都是"供给侧差异化"的打法：不做需求聚合，而是在已被证明有人付费的品类中，找大玩家不屑做的切口。
不建议把 snapog 继续作为纯 OG-image 产品推进——@vercel/og 免费，且 Satori 已商品化，没有付费楔子。

---

## Idea A — RenderKit：Screenshot & PDF Rendering API

**一句话**：一个 API key，`/screenshot` 与 `/pdf` 两个 endpoint，按渲染次数计费，跑在 Cloudflare Browser Rendering 上，结果缓存到 R2。

**目标客户**：做 SaaS 的独立开发者/小团队，需要生成链接预览图、发票/报表 PDF、归档页面快照，不想自己运维 headless Chrome（Puppeteer 在 Lambda/容器里的内存与冷启动是公认痛点）。

**需求证据**
| 证据 | 内容 | 可信度 |
|---|---|---|
| ScreenshotOne 公开 build-in-public | 单人创始人 Dmytro Krasun 公开披露 MRR，2024 年已达约 $20k+ MRR 量级（X/博客 https://screenshotone.com/blog/ ） | likely，数字待核实 |
| 品类内多家长期存活的付费竞品 | Urlbox、ApiFlash、ScreenshotAPI.net、PDFShift、DocRaptor、Api2Pdf 均存活 5 年以上 → 证明持续付费需求 | likely |
| "Puppeteer on Lambda/serverless" 痛点 | Stack Overflow / Reddit r/webdev 大量 chromium 体积、超时、内存问题帖 | likely（未逐条引用） |

**竞品与定价**（待核实）
| 竞品 | 定价 | 备注 |
|---|---|---|
| ScreenshotOne https://screenshotone.com/pricing/ | Free 100 次；约 $17/mo 2k 次；约 $79/mo 10k 次 | 功能最全（广告/cookie banner 屏蔽） |
| ApiFlash https://apiflash.com/pricing | Free 100；约 $7/mo 1k；约 $35/mo 10k | 低价锚点 |
| DocRaptor https://docraptor.com/signup | 约 $15/mo 125 docs 起 | PDF 质量高（Prince 引擎），但单价贵 |
| PDFShift https://pdfshift.io/pricing/ | 约 $9/mo 起（按 credits） | 纯 PDF |

**楔子 / 差异化**
- **一个 key 两种输出**：多数竞品只做 screenshot 或只做 PDF，小团队往往要买两家。
- **成本结构优势**：Browser Rendering 按 browser-hour 计费（Workers Paid 含部分免费额度，之后约 $0.09/hr，待核实）。截图约 2–4s，即 ~1000 次/hr，边际成本 < $0.0001/次 → 可做到比 DocRaptor 低一个数量级的 PDF 单价。
- **PDF 模板模式**：`POST /pdf` 传 HTML + JSON data（Handlebars），覆盖"发票/收据"这一最常见用例，DocRaptor 级的贵价 PDF 用户会被价格吸引。
- snapog 的 API key / 计量 / R2 缓存代码可直接复用，MVP 1 周可期。

**价格**：Free 100 次/月；Starter $9/mo 2,000 次；Pro $29/mo 10,000 次；超额 $0.002/次。年付 2 个月折扣。

**最大风险**
- **差异化弱、红海**（最大风险）：ScreenshotOne 功能深度（广告屏蔽、cookie banner、geo、retina）领先；我们只能靠"PDF+截图合一 + 低价"切入，客户转换成本低，价格战空间有限。
- 滥用：被用于截取违法内容/当爬虫代理 → 需速率限制、禁止内网/私有 IP（SSRF）、ToS 明确责任在调用方。
- 平台依赖：Browser Rendering 的并发上限与定价可能变动（`speculative`）。

---

## Idea B — CronSentry：GitHub Actions 定时任务监控

**一句话**：安装一个 GitHub App，零代码改动，自动监控所有 `on: schedule` workflow——漏跑、延迟、失败、因 60 天不活跃被自动禁用时，立即发 email/Slack/Discord 告警。

**目标客户**：把 GitHub Actions 当免费 cron 用的开发者与小团队（数据同步、备份、爬取自家数据、报表、依赖更新）。

**需求证据**
| 证据 | 内容 | 可信度 |
|---|---|---|
| GitHub 官方文档 | 公共仓库 60 天无活动后 scheduled workflow 自动禁用；高负载时 schedule 事件可能延迟或被丢弃（https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule ） | likely（文档行为确认，路径待核实） |
| GitHub Community 讨论 | 大量 "scheduled workflow not running / delayed by hours" 帖（https://github.com/orgs/community/discussions ，搜 "schedule cron delayed"） | likely |
| 品类付费证明 | Healthchecks.io（单人公司，创始人在博客公开经营数据，https://blog.healthchecks.io/ ）、Cronitor、Dead Man's Snitch 均以"cron 心跳监控"长期盈利 | likely，营收数字待核实 |

**竞品与定价**（待核实）
| 竞品 | 定价 | 局限 |
|---|---|---|
| Healthchecks.io https://healthchecks.io/pricing/ | Free 20 checks；约 $20/mo 100 checks；约 $80/mo 1000 | 需在每个 job 里手动加 ping（curl） |
| Cronitor https://cronitor.io/pricing | Free 少量 monitor；之后按 monitor 计费约 $2/monitor/mo | 同上，需埋点 |
| Dead Man's Snitch https://deadmanssnitch.com/plans | 约 $5/mo 起 | 同上，老牌、无 GitHub 原生集成 |

**楔子 / 差异化**
- **零埋点**：通用心跳监控都要求用户改 workflow 加 ping；我们通过 GitHub App 读取 workflow 文件里的 cron 表达式 + `workflow_run` webhook，自动推算"应在何时跑"，未按时出现即告警。安装即生效，这是 Healthchecks 结构上不会做的 niche（它要保持通用性）。
- **独有检测**：60 天禁用前 7 天预警（监控 repo 活动）+ 一键 re-enable；这是 GitHub 特有、通用工具无法覆盖的痛点。
- **分发**：GitHub Marketplace 支持付费 App，自带计费与发现渠道 → 无需人工销售；也可走 Lemon Squeezy。
- Cloudflare 契合度高：Workers 接 webhook，Cron Triggers 每分钟扫描"逾期"，D1 存 schedule 状态，无需浏览器渲染，成本近零。

**价格**：Free 3 个 repo；Pro $9/mo 无限 repo（个人）；Team $29/mo 组织级 + Slack + 历史 90 天。

**最大风险**
- **市场天花板**（最大风险）：用户是"用免费 cron 的人"，付费意愿天然偏低；SOM 可能只有数百付费用户量级 → 估计上限 $2–5k MRR（`speculative`）。需早期以 Free→Pro 转化率验证。
- 平台风险：GitHub 若原生增加 schedule 失败告警即被 Sherlock。
- 权限信任：需 `actions:read` / metadata 权限，安全敏感用户会犹豫 → 最小权限 + 公开源码可缓解。

---

## 对比与建议（分析，非事实）

| 维度 | A RenderKit | B CronSentry |
|---|---|---|
| 付费需求证据 | 强（多家盈利竞品） | 中（品类强，GitHub 细分未验证） |
| 差异化 | 弱–中 | 强（零埋点，结构性 niche） |
| 构建难度 | 低（复用 snapog） | 中（GitHub App OAuth + cron 解析） |
| 营收天花板 | 高 | 中低 |
| 自助销售可行性 | 高 | 高（Marketplace） |

**建议**：先做 A（复用代码最快上线、品类已证明能赚钱，主要验证"能否拿到首个付费用户"）；B 作为并行低成本实验——先上一个 landing page + waitlist 测需求，再决定是否构建。
两者共用一套 billing（Lemon Squeezy MoR）与 API key 基础设施。

## 信息盲区与获取方式
1. 所有竞品当前定价 → 直接抓取各 pricing 页面核实（下一个有 web 权限的 cycle 第一件事）。
2. Cloudflare Browser Rendering 当前计费与并发限制 → developers.cloudflare.com/browser-rendering/platform/pricing/。
3. ScreenshotOne / Healthchecks 最新营收 → 创始人 X 账号与博客。
4. B 的真实付费意愿 → landing page + "Pro 预购"按钮，2 周内 >20 个 waitlist 才构建。
