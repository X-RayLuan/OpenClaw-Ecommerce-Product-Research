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

---

## 3. 出图（kie.ai gpt-image-2）

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
