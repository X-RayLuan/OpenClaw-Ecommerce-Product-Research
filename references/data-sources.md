# 数据源调用配方

所有密钥从环境变量读取（见根目录 `.env.example`）。**绝不把真实 key 写进产出文件或提交到 git。**

---

## 1. Amazon / Google Trends / Google AI Overview / WIPO / PACER（MCP）

端点是一个 MCP（streamable HTTP，JSON-RPC 2.0）。`PANGOLIN_MCP_URL` 已含 api_key。
按积点计费——评论(10pt/页)、PACER(12pt/次)最贵，先小批探。

**握手 + 列工具：**
```bash
curl -sS -X POST "$PANGOLIN_MCP_URL" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' \
  | sed 's/^data: //'
```

**调用某工具的通式：**
```bash
curl -sS -X POST "$PANGOLIN_MCP_URL" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"<TOOL>","arguments":{...}}}' \
  | sed 's/^data: //'
```
返回是 SSE，`data:` 行内是 JSON-RPC 结果；`result.content[].text` 里是工具 JSON 字符串，需二次 `json.loads`。

**常用工具与参数：**

| 工具 | 关键参数 | 取数路径 |
|---|---|---|
| `search_amazon` | `keyword`, `site=amz_us`, `format=json`, `page` | `data.json[0].data.results[]`（asin/title/price/star/rating/sponsored） |
| `get_amazon_product` | `asin`, `site` | `data.json[0].data.results[0]`（features/images/bestSellersRankItems/aiReviewsSummary） |
| `get_amazon_reviews` | `asin`, `filterByStar=critical`, `pageCount=1`, `sortBy=recent` | `data.json[0].data.results[]` ⚠️10pt/页 |
| `list_bestsellers` / `list_new_releases` | 类目 | Top 榜单 |
| `keyword_trends` | `keywords[≤5]`, `timeRange=today 12-m`, `region=US` | `data.json.timelineData[]` + `keywordsRankData[]`（上升词，Breakout） |
| `ai_search` | `query`, `mode=overview` | `data.ai_overview` + `json.items[]`（ai_overview / organic / related_searches） |
| `wipo_search` | `source=USID`, `prod` 或 `hol`, `num` | `data.data.hits[]`（IRN/HOL/LCS/RD/STATUS） |
| `pacer_search` | `patentNumber` 或公司名/案号 | 美国诉讼 docket ⚠️12pt/次 |

---

## 2. Reddit / 社媒 / 通用 web 讨论（Tavily）

```bash
# 通用情绪 + 来源
curl -sS -X POST 'https://api.tavily.com/search' -H 'Content-Type: application/json' \
  -d "{\"api_key\":\"$TAVILY_API_KEY\",\"query\":\"<topic> what people love or complain about\",\"search_depth\":\"advanced\",\"include_answer\":true,\"max_results\":8}"

# 仅 Reddit
curl -sS -X POST 'https://api.tavily.com/search' -H 'Content-Type: application/json' \
  -d "{\"api_key\":\"$TAVILY_API_KEY\",\"query\":\"<topic> discussion\",\"include_domains\":[\"reddit.com\"],\"search_depth\":\"advanced\",\"max_results\":8}"
```
读 `answer`（摘要）+ `results[].{title,url,content}`。Walmart/Amazon 评论也常被 Tavily 抓到，是差评原声金矿。
TikTok 选品时也用它抓**话题讨论/达人测评/品类口碑**（`include_domains:["reddit.com","youtube.com","tiktok.com"]`），作趋势近似与评论挖掘的补充源。

---

## 3. 中文电商 / 工厂端竞品 + 神经检索（Exa）

Pangolin 主攻 Amazon 美/欧；**1688、阿里国际站、独立站、中文工厂黄页、小众社区**这些它覆盖不到的源用 Exa 补。
Exa 是神经/语义检索，适合"找同类工厂 / 找某品类货源 / 找竞品独立站"这种模糊意图。`type=auto` 自动在 neural/keyword 间择优。

```bash
# 搜索（可选 includeDomains 锁定平台；contents.text 直接回正文）
curl -sS --max-time 60 -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" -H 'Content-Type: application/json' \
  -d '{"query":"<意图，如：义乌 丝巾 围巾 工厂 / silk scarf supplier>","numResults":10,"type":"auto",
       "includeDomains":["1688.com","alibaba.com"],
       "contents":{"text":{"maxCharacters":1500}}}'

# 取全文（拿到 URL 后回填完整正文，maxCharacters 调大）
curl -sS --max-time 90 -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" -H 'Content-Type: application/json' \
  -d '{"urls":["<url1>","<url2>"],"text":{"maxCharacters":2600}}'
```
读 `results[].{title,url,text}`。`type` 可选 `neural`(语义)/`keyword`(精确词)/`auto`。
**脱敏**：对外报告里 Exa 抓到的内容按**原始来源**标注（1688 / 阿里国际站 / 某独立站），不出现 "Exa"。

> 适用：步骤1 找对标工厂/货源、找竞品独立站；步骤2 找中文社区/小红书风格讨论（Tavily 偏英文社媒，中文源 Exa 更准）。

---

## 4. TikTok 真实销量数据（EchoTik Open API）— ✅ 已接入

兴趣电商（TikTok Shop）真实销量走 **EchoTik Open API**（Pangolin 只覆盖 Amazon，不含 TikTok）。

- Base URL：`https://open.echotik.live/api/v3/echotik/`
- 鉴权：**HTTP Basic Auth**，把 `base64(username:password)` 放进 `Authorization: Basic <...>` 头。
- 计费：~$0.001/次，pay-as-you-go；先小批探（`page_size` 小）再放量。
- 脱敏：对外报告只标 **"TikTok"**，不出现 "EchoTik"。

**鉴权头一次性生成：**
```bash
ECHOTIK_AUTH=$(printf '%s:%s' "$ECHOTIK_API_USERNAME" "$ECHOTIK_API_PASSWORD" | base64)
```

**核心：选品候选清单（offline T+1 库，服务端直接按阈值过滤）→ `product/list`**
```bash
curl -sS --max-time 40 -G 'https://open.echotik.live/api/v3/echotik/product/list' \
  --data-urlencode 'region=US' \
  --data-urlencode 'min_spu_avg_price=15' --data-urlencode 'max_spu_avg_price=40' \
  --data-urlencode 'max_total_sale_cnt=5000' \
  --data-urlencode 'sales_trend_flag=1' \
  --data-urlencode 'product_sort_field=4' --data-urlencode 'sort_type=1' \
  --data-urlencode 'page_num=1' --data-urlencode 'page_size=10' \
  -H "Authorization: Basic $ECHOTIK_AUTH" | python3 -m json.tool
```
读 `data[]`。每条含：`product_id / product_name / spu_avg_price / total_sale_cnt / total_sale_7d_cnt /
total_views_cnt / total_views_7d_cnt / product_rating / review_count / seller_id / from_flag(1本土/2跨境)`。

**机会窗口阈值 → 参数映射（实测校准，命中即候选）：**

| 选品信号 | EchoTik 实现 / 判读 |
|---|---|
| 起量阶段（正在爆发的新品） | `sales_trend_flag=1`(上升) + `product_sort_field=4`(按 total_sale_7d_cnt) `sort_type=1`(降序)；**`total_sale_7d_cnt / total_sale_cnt > 40%` = 刚爆发**（比"增长>50%"更可操作） |
| 总销量 < 5000（没被大卖盯上） | `max_total_sale_cnt=5000` |
| 售价 $15–40（最佳价格带） | `min_spu_avg_price=15` `max_spu_avg_price=40` |
| 内容爆发力（需求侧，**要高**） | `total_views_cnt > ~100k` 且 `total_video_cnt > ~50` / `total_ifl_cnt > ~30` → 有人愿意拍、能起量 |
| 转化效率（盈利侧，**要低**） | `total_views_cnt / total_sale_cnt` **越低越好**；实测 **7天口径 `total_views_7d_cnt/total_sale_7d_cnt < ~50` = 转化强**，几百以上=有流量没转化，慎入 |

> ⚠️ **口径校准**：文章原文"播放/销量 > 1000"是**反的**——高比值=有流量没转化的差信号。实测一个转化极强的爆品(250周年款)总比值才 152、7天 24。选品真正要的是**转化高(比值低) + 内容爆发力(总播放/视频数高)** 两者兼备。深查单品用 `product/detail?product_ids=` 取全量 `total_views_*`/`total_sale_*` 字段自行计算。

其它常用过滤：`category_id`/`category_l2_id`/`category_l3_id`（先用下方分类接口取 id）、`is_hot=1`(爆款)、
`min_first_crawl_dt`(yyyyMMdd，找新近上架)、`min/max_total_views_cnt`、`min/max_product_commission_rate`、`free_shipping`。

**其它配套接口（同 base + Basic Auth，GET）：**

| 用途 | 接口 | 关键参数 / 取数 |
|---|---|---|
| 类目 id 解析 | `product/category/list`（一级/二级/三级分类） | 先拿 `category_id` 再喂 `product/list` |
| 单品历史趋势（最多 180 天） | `product/trend` | `product_id`,`start_date`,`end_date`（yyyy-MM-dd）→ 每日 `total_sale_1d_cnt`/`spu_avg_price` 画增长曲线 |
| 评论挖改进点（Step 4） | `product/review/list` | `product_id` → 评论原声 → 痛点/加分点 |
| 关联达人/视频/直播 | `product/sale-creator/list`·`product/video/list`·`product/live/list` | 找谁在带、用什么钩子 |
| 小店盘竞品 | `shop/list`·`shop/trend`·`shop/detail`·`shop/product/list` | 对标小店销量趋势 |
| 关键词/类目大盘 | `search/general` | 类目趋势、爆发词 |

> 凡是阈值无 EchoTik 实据支撑的方向，仍按"待销量数据二次确认"标注，不得当成已验证爆款。
> EchoTik 取不到的中文源/独立站竞品继续用 Exa（§3），社媒口碑用 Tavily（§2）。

---

## 5. 出图（kie.ai gpt-image-2）

```bash
# 提交任务（竖款门挂用 aspect_ratio=1:2）
TASK=$(curl -sS -X POST 'https://api.kie.ai/api/v1/jobs/createTask' \
  -H "Authorization: Bearer $KIE_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"gpt-image-2-text-to-image","input":{"prompt":"<PROMPT>","aspect_ratio":"1:2","resolution":"1K"}}' \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['data']['taskId'])")

# 轮询取图（state=success 后从 resultJson.resultUrls 取）
curl -sS "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=$TASK" \
  -H "Authorization: Bearer $KIE_API_KEY" \
  | python3 -c "import sys,json;d=json.load(sys.stdin)['data'];import json as j;print(d['state']);print(j.loads(d.get('resultJson') or '{}').get('resultUrls'))"
```
约 6 积点/张、15–90s。**出图后务必肉眼校验文字拼写**（gpt-image 偶尔画错字）。生图 prompt 要求：原创、无受版权 IP、e-commerce hero shot、门框实景。

### 批量出图（多张时必读 · 踩坑固化）

要出多张图（如报告每模块配一张）时，**别用前台长 bash 轮询循环**——会撞执行时限（实测 2–5 分钟被 kill），且 bash 循环里逐个 `curl` 易整体卡死。正确姿势 **submit-all → Python 收集器一次性收 URL → 按 URL 插入**：

1. **先全部提交**，把 `模块|taskId` 写进一个文件（提交很快，秒返回 taskId）。
2. **用 Python（urllib，每次调用带 `timeout=15`）批量轮询**取 `resultUrls`，不要用 `while read + curl` 的 bash 循环：
   ```python
   import os,json,urllib.request
   key=os.environ['KIE_API_KEY']
   for mod,tid in [l.split('|') for l in open('/tmp/tasks.txt').read().split()]:
       d=json.load(urllib.request.urlopen(urllib.request.Request(
           f"https://api.kie.ai/api/v1/jobs/recordInfo?taskId={tid}",
           headers={"Authorization":f"Bearer {key}"}),timeout=15))['data']
       if d['state']=='success':
           print(mod, json.loads(d['resultJson'])['resultUrls'][0])
   ```
3. **插入直接用图 URL**（`<img href="<kie_url>">`），飞书会自动 re-host 成永久素材，**无需先下载**（除非要肉眼校验文字才下载 Read）。
4. 单张同步出图仍可用上面的 curl 法；只有"多张"才必须走批量收集器。
