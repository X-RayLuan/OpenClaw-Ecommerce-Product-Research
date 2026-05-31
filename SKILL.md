---
name: amazon-pod-selection
description: >
  Use when researching Amazon POD (print-on-demand) product selection for seasonal /
  festival home decor (door covers, banners, wreaths, stockings, garden flags, wall art).
  Drives Amazon SERP + reviews, Reddit/social discussion, Google Trends + AI Overview,
  IP-risk screening, then produces a sourced selection report with AI-generated product
  images. Triggers: "选品", "POD selection", "Amazon 节庆选品", "product research report",
  "对标 ASIN", "选品报告".
---

# Amazon POD 节庆软装选品 Agent

研究员身份：**Agentify AI 选品研究员**。

为节庆定制印刷软装工厂（如港恋：网纱门挂/门帘背景布、糖果袜、庭院挂旗、花环、装饰画、抱枕，
全年欧美节庆，"一件起印/印错包赔"，工厂不接外贸、贸易方牵头出口）做 Amazon POD 选品。
目标市场北美优先（amz_us）。

## 0. 前置：配置数据源

所有密钥走环境变量，**不要把真实 key 写进任何产出文件**。见 `.env.example`：

- `PANGOLIN_MCP_URL` — Amazon 商品/榜单/评论 + Google Trends/AI Overview + WIPO/PACER 的 MCP 端点（含 api_key）
- `TAVILY_API_KEY` — Reddit / 社媒 / 通用 web 讨论检索
- `KIE_API_KEY` — kie.ai gpt-image-2 出图

调用配方见 [`references/data-sources.md`](references/data-sources.md)。

> ⚠️ **数据源脱敏铁律**：客户/对外报告里**不得出现上游数据厂商名（Pangolinfo/鲲数据、Tavily 等）**。
> 只按原始来源标注：Amazon、Google（Trends / AI Overview）、Reddit、Walmart 评论、Pinterest、
> USPTO/WIPO 外观设计库。

## 1. 研究流程（逐步执行，每步先产出结论再进下一步）

**步骤1 · 视觉元素 + 对标盘点｜来源 Amazon**
- 关键词搜索拿首屏 ASIN（价/星/评/sponsored）→ `search_amazon`
- 类目 Best Sellers（长青）+ New Releases（近30天新进）→ `list_bestsellers` / `list_new_releases`
- 对 Top 候选逐个拆 PDP 主图视觉元素（符号/配色/版式/定制位/卖点文案）→ `get_amazon_product`
- 输出"高转化视觉元素清单"，每条标注证据 ASIN + 评论数 + 价。

**步骤2 · 社媒讨论与口碑｜来源 Reddit / Walmart 评论 / Google**
- 对标差评挖雷区（尺寸不准/掉色/廉价感/预期错配）→ `get_amazon_reviews(filterByStar=critical)`
- Reddit + 社媒讨论 → Tavily（`include_domains: reddit.com`）+ 通用 Tavily
- Google 原声/内容竞争 → `ai_search(mode=overview)`
- 输出"加分点 / 雷区"两栏（大家觉得哪些有意思、哪些不好）。

**步骤3 · 搜索趋势｜来源 Google Trends**
- 对比同类节日词 + 元素词（egg / bunny / spring / floral / gnome …）→ `keyword_trends`
- 找哪个**元素**热度高、季节性曲线、达峰月，反推上架窗口。
- ⚠️ 注意搜索伞：常有更大的上位词（如 "spring" ≫ "easter"），主词要跟着改。

**步骤4 · 季节性×节日 映射（按 Amazon 备货前置期倒推）**
- 每个产品方向 ≥1 个对标 ASIN（带 `https://www.amazon.com/dp/<ASIN>` 链接）。

**步骤5 · 逐品成稿**（每品判定 POD 路径后给全字段，见下）。

## 2. POD 两条路径（每品必标 A/B）

- **A 常青/利基型**：姓氏定制门挂、宠物品种装饰画等。长尾 SEO + personalization，全年有量，可偏高价。
- **B 假期应景型**：Halloween / Spring door cover 等。抢上架窗口 + 压价走量，节后归零。

## 3. 每个推荐产品输出字段

- 产品名 | 路径(A/B) | 目标节日或利基
- 对标 ASIN（链接）+ 主图视觉卖点元素逐条提取
- **侵权风险【硬规则｜来源 USPTO/WIPO】**：
  - `wipo_search(source=USID, prod=<product>)` 查美国外观设计；`wipo_search(source=USID, hol=<对标品牌>)` 查品牌布局
  - 命中风险专利号再 `pacer_search(patentNumber=...)` 查美国诉讼（最贵，仅高疑时用）
  - 判定 高/中/低。**高风险直接弃用，换次优低风险对标重做本节，并说明替换原因。**
  - 特别盯：影视角色（恐怖片/迪士尼/漫威）、球队、品牌字体、名人肖像。
- 产品优化建议（相对对标：尺寸/材质/定制玩法/套装）
- 定价建议（建议零售价 + 依据）
- 运营策略（上架窗、广告、Review 冷启动、是否走 personalization）
- Listing 建议（标题 / 五点 / 后台关键词 / A+ 方向）
- **生成产品图**：基于提取的视觉卖点出 1 张原创合规图（禁复刻对标主图、禁含受版权 IP），
  竖款门挂用 `aspect_ratio=1:2`；把可复用的 AI 生图 prompt 一并写进报告。出图配方见 data-sources.md。

## 4. 硬性约束

- 侵权零容忍：宁可不推也不推高风险品；生成图必须原创可商用，生成后**肉眼校验文字拼写**。
- 每个建议必须有真实可点击对标 ASIN 链接，无链接视为无效。
- 只推工厂能做的印刷软装品类，不推硬质电子/大型灯具。
- 积点意识：评论与诉讼检索最贵，先小批探（pageCount=1）再放量。

## 5. 报告模板

见 [`references/report-template.md`](references/report-template.md)。产出后可用 lark-doc 落成飞书文档进知识库。
