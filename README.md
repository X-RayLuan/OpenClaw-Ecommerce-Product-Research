# Ecommerce / 外贸工厂 Product Research

> 🇨🇳 一个可复用的跨境电商选品研究 skill：以 Agentify AI 选品研究员身份，为外贸工厂/跨境卖家做选品。**平台自适应**——搜索电商（Amazon/Walmart/独立站）走"满足已知需求"打法，兴趣电商（TikTok Shop/短视频）走"创造未知需求"打法。一条流程跑通：时间窗口 → 找场景 → 真实销量阈值 → 评论挖改进点 → 净利测算 → 钩子文案测试 → 小批量实测，外加 USPTO/WIPO 侵权排查 + AI 出图。对外报告只标原始来源、不暴露上游数据厂商。节庆软装 POD（港恋：门挂/门帘/糖果袜/庭院旗/花环/装饰画/抱枕）是其中一个示例客户，不是唯一范围。

> 🇬🇧 A reusable cross-border ecommerce product-research skill. As an Agentify AI research analyst, it does product selection for factories / cross-border sellers, **platform-aware**: search commerce (Amazon/Walmart/own-site) vs interest commerce (TikTok Shop/short video). One flow end to end: timing window → find the scenario → real-sales thresholds → review-mining → net-margin math → hook-copy test → small-batch validation, plus USPTO/WIPO IP screening + AI product images. Reports cite raw sources only. Festival-decor POD is one example client, not the whole scope.

## 用法 / Usage

把本仓库放进 skills 目录（如 `~/.claude/skills/openclaw-ecommerce-product-research/`），
在对话里说"选品 / 选品报告 / TikTok 选品 / Amazon 选品 / 工厂选品 / product research"即可触发。

Drop this repo into your skills directory and trigger it with "product research", "选品",
"TikTok 选品", "Amazon 选品", "工厂选品", or "选品报告".

## 配置 / Configuration

```bash
cp .env.example .env   # 填入三个密钥；.env 已 gitignore，不会进版本库
```

| 变量 / Var | 用途 / Purpose |
|---|---|
| `PANGOLIN_MCP_URL` | Amazon 商品/榜单/评论 + Google Trends/AI Overview + WIPO/PACER |
| `TAVILY_API_KEY` | Reddit / TikTok 话题 / 社媒 / web 讨论 |
| `EXA_API_KEY` | 神经检索：1688/阿里国际站/独立站/源头工厂货源 |
| `ECHOTIK_API_USERNAME` / `ECHOTIK_API_PASSWORD` | EchoTik Open API：TikTok Shop 商品/小店/达人/视频销量 |
| `KIE_API_KEY` | kie.ai gpt-image-2 出图 / image generation |

> TikTok 真实销量已接入 **EchoTik Open API**（HTTP Basic Auth，~$0.001/次），配方见 `references/data-sources.md §4`。

## 结构 / Structure

- `SKILL.md` — 核心心法（搜索 vs 兴趣电商）、平台自适应选品流程、逐品输出字段、硬性约束
- `references/data-sources.md` — 各数据源的 curl 调用配方与取数路径（含 TikTok 阈值）
- `references/report-template.md` — 报告模板

## 脱敏铁律 / Source masking

对外/客户报告**只标原始来源**（Amazon、Google、Reddit、Walmart、Pinterest、USPTO），
**不出现上游数据厂商名**。 Client-facing reports cite raw sources only, never the upstream data vendor.

## 安全 / Security

仓库内**不含任何真实 API key**；所有密钥走 `.env`（已 gitignore）。
No real API keys in the repo — all secrets live in `.env` (gitignored).

## 更新记录 / Changelog

### 2026-06-26
- **EN** Generalized from "Amazon POD festival decor" to a **platform-aware** cross-border / factory selection skill: search-commerce (Amazon/独立站) vs interest-commerce (TikTok). 港恋 POD is now just one example client.
  **中文** 从"Amazon POD 节庆软装"泛化为**平台自适应**的跨境/工厂选品 skill：搜索电商(Amazon/独立站) vs 兴趣电商(TikTok)；港恋降为示例客户。
- **EN** Wired in **EchoTik Open API** for real TikTok Shop sales (HTTP Basic Auth) — `product/list` server-side thresholds, `product/detail`/`product/trend`/`product/review`, shop & creator endpoints. Verified live.
  **中文** 接入 **EchoTik Open API**（HTTP Basic Auth）做 TikTok 真实销量——`product/list` 服务端阈值过滤 + 详情/趋势/评论/小店/达人接口，已实测。
- **EN** **Threshold calibration**: the article's "views/sales > 1000" was inverted — selection wants high conversion (low ratio) + content reach. Replaced with 7d-sales-share>40% (launch signal) + low play/sale ratio (conversion) + high total views/videos (reach).
  **中文** **阈值校准**：文章"播放/销量>1000"口径反了——选品要高转化(比值低)+内容爆发力。改为 7天销量占比>40%(起量) + 低播放销量比(转化) + 高总播放/视频数(爆发力)。
- **EN** Step-by-step flow expanded to 8 platform-aware steps (timing → scenario → real sales → reviews → net margin → IP → hooks → small-batch validation) + 微创新 principle.
  **中文** 流程扩成 8 步平台自适应（时间窗口→找场景→真实销量→评论→净利→IP→钩子→小批量验证）+ 微创新原则。
- **EN** Batch image-gen recipe added to `data-sources.md §5`: submit-all → Python collector → insert by URL (no long foreground bash poll loops, which hit execution time limits).
  **中文** `data-sources.md §5` 新增批量出图配方：submit-all → Python 收集器 → 按 URL 插入（别用前台长 bash 轮询，会撞执行时限）。
