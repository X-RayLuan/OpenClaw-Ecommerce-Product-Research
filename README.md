# amazon-pod-selection

一个 Claude Code / Agent **skill**：把"Amazon POD 节庆软装选品"做成可复用的研究流程。

研究员身份：**Agentify AI 选品研究员**。面向节庆定制印刷软装（门挂/门帘/糖果袜/挂旗/花环/装饰画/抱枕），
目标市场北美优先。一条命令跑通：Amazon 对标 → 社媒/Reddit 口碑 → Google Trends 元素热度 →
IP 侵权排查 → 逐品报告 + AI 出图。

## 用法

把本仓库放进你的 skills 目录（如 `~/.claude/skills/amazon-pod-selection/`），
然后在对话里说"选品 / POD selection / Amazon 节庆选品 / 选品报告"即可触发。

## 配置

```bash
cp .env.example .env   # 填入三个密钥；.env 已 gitignore，不会进版本库
```

| 变量 | 用途 |
|---|---|
| `PANGOLIN_MCP_URL` | Amazon 商品/榜单/评论 + Google Trends/AI Overview + WIPO/PACER |
| `TAVILY_API_KEY` | Reddit / 社媒 / web 讨论 |
| `KIE_API_KEY` | kie.ai gpt-image-2 出图 |

## 结构

- `SKILL.md` — 研究流程、POD 两条路径、逐品输出字段、硬性约束
- `references/data-sources.md` — 三个数据源的 curl 调用配方与取数路径
- `references/report-template.md` — 报告模板

## 脱敏铁律

对外/客户报告**只标原始来源**（Amazon、Google、Reddit、Walmart、Pinterest、USPTO），
**不出现上游数据厂商名**。

## 安全

仓库内**不含任何真实 API key**。所有密钥走 `.env`（已 gitignore）。
