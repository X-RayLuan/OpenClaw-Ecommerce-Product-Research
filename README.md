# OpenClaw Ecommerce Product Research

> 🇨🇳 一个可复用的电商选品研究 skill：以 Agentify AI 选品研究员身份，为节庆/季节性家居软装（门挂、门帘、糖果袜、庭院旗、花环、装饰画、抱枕）做 Amazon POD 选品。一条流程跑通：Amazon 对标 ASIN → Reddit/社媒口碑 → Google Trends 元素热度 → USPTO/WIPO 侵权排查 → 逐品报告 + AI 出图。对外报告只标原始来源、不暴露上游数据厂商。

> 🇬🇧 A reusable ecommerce product-research skill. As an Agentify AI research analyst, it runs Amazon POD selection for seasonal/festival home decor (door covers, banners, stockings, garden flags, wreaths, wall art, pillows). One flow end to end: Amazon benchmark ASINs → Reddit/social sentiment → Google Trends element heat → USPTO/WIPO IP screening → per-product report + AI product images. Reports cite raw sources only.

## 用法 / Usage

把本仓库放进 skills 目录（如 `~/.claude/skills/openclaw-ecommerce-product-research/`），
在对话里说"选品 / POD selection / product research / 选品报告"即可触发。

Drop this repo into your skills directory and trigger it with "product research", "选品",
"POD selection", or "选品报告".

## 配置 / Configuration

```bash
cp .env.example .env   # 填入三个密钥；.env 已 gitignore，不会进版本库
```

| 变量 / Var | 用途 / Purpose |
|---|---|
| `EXA_API_KEY` | **核心**：神经检索，锁 amazon.com 拿对标 PDP + 1688/阿里国际站/独立站货源 |
| `TAVILY_API_KEY` | **核心**：Reddit / 社媒 / web 讨论与口碑 |
| `KIE_API_KEY` | 出图：kie.ai gpt-image-2 / image generation |
| `PANGOLIN_MCP_URL` | 〔可选〕结构化 Amazon 商品/榜单/评论 + Google Trends/AI Overview + WIPO/PACER；没配用 Exa+Tavily 平替 |

> 默认仅需 `EXA_API_KEY` + `TAVILY_API_KEY` 即可跑通全流程；`PANGOLIN_MCP_URL` 为可选增强。
> Default flow runs on `EXA_API_KEY` + `TAVILY_API_KEY` alone; `PANGOLIN_MCP_URL` is an optional enhancement.

## 结构 / Structure

- `SKILL.md` — 研究流程、POD 两条路径、逐品输出字段、硬性约束
- `references/data-sources.md` — 三个数据源的 curl 调用配方与取数路径
- `references/report-template.md` — 报告模板

## 脱敏铁律 / Source masking

对外/客户报告**只标原始来源**（Amazon、Google、Reddit、Walmart、Pinterest、USPTO），
**不出现上游数据厂商名**。 Client-facing reports cite raw sources only, never the upstream data vendor.

## 安全 / Security

仓库内**不含任何真实 API key**；所有密钥走 `.env`（已 gitignore）。
No real API keys in the repo — all secrets live in `.env` (gitignored).
