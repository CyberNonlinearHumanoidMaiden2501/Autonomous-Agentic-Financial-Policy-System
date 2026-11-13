# Data Sources Map

> Every external data type the system consumes (per the spec's data dictionary, README Ch. 13), with redundant sources, paywall status, and verification level.
>
> Related: [How the System Works](../docs/how-the-system-works.md) · [Financial AI Agent Landscape](financial-ai-agent-landscape.md)
>
> **Compiled 2026-11-13.** "Live-verified" means tested from this machine with real API calls on that date; "vendor page" means the vendor's own pricing page/document was fetched and read.

## Legend

| Mark | Meaning |
|---|---|
| ✅ | Free |
| 🟡 | Freemium — free tier exists |
| 💰 | Paid |
| **V** | Vendor-official pricing (read from the vendor's own page) |
| **L** | Live-verified (real API call / HTTP test, 2026-11-13) |
| **T** | Third-party reported — ballpark only |
| **N** | No public price list — sales-led |

**Bold** source name = recommended primary per type.

---

## 1. Securities master & industry classification (证券与行业)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **Tushare Pro** ([tushare.pro](https://tushare.pro)) | 🟡 ¥200/yr | V | `stock_basic`, ST/name-change history, SW industry codes |
| AkShare | ✅ | L | Code lists, Eastmoney/SW industry boards |
| BaoStock ([baostock.com](http://baostock.com)) | ✅ | L | Codes, industry, listing status |
| SSE / SZSE official sites | ✅ | L | Authoritative listed-company registers |
| Shenwan classification ([swsresearch.com](https://www.swsresearch.com/institute_sw/allIndex/downloadCenter/industryType)) | ✅ | T | 31/134/346-level industry code files, free download |
| JQData (聚宽) | 💰 | N | Trial free (~1M rows/day); paid pricing sales-led; ⚠️ non-mainland IPs blocked |
| Wind / iFinD / Choice | 💰 | T | Terminal pricing below |

## 2. Quotes & valuation ratios (行情与估值)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **Tushare Pro** | 🟡 ¥200/yr | V | Daily bars + `daily_basic` (PE/PB/PS/股息率) + `adj_factor`; free tier = unadjusted daily only (8,000 calls/day) |
| AkShare | ✅ | L | OHLCV + qfq/hfq via Eastmoney/Sina/THS; scraped, no SLA |
| BaoStock | ✅ | L | K-lines 1990–present, own 复权 method |
| Futu OpenD ([openapi.futunn.com](https://openapi.futunn.com)) | ✅ | T | Free gateway; quote quota scales with account assets; A-share LV1 for mainland clients |
| 理杏仁 ([lixinger.com](https://www.lixinger.com)) | 💰 ¥518/yr | V | **Valuation percentile specialty**: PE/PB/EV-EBITDA with 1/3/5/10-yr bands (Ch. 5.2 exactly); price table effective 2026-03-04: 周32/月70/季168/年518/二年928/三年1248 |
| JQData | 💰 | N | Mainland-only |
| Wind | 💰 ~¥26,800–39,800/seat/yr | T | Procurement records |
| iFinD | 💰 ~¥12,800/seat/yr | T | Industry-association notice (中评协办〔2026〕24号); no public price page (404 on /purchase) |
| Choice | 💰 | N | Personal-edition price not public; procurement records only |

## 3. Financial statements & indicators (财务数据)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **Tushare Pro** | 🟡 ¥200/yr | V | Income/BS/CF + `fina_indicator`; **point-in-time capable via `ann_date` + `update_flag`** — the decisive feature |
| AkShare | ✅ | L | Statements from Eastmoney/Sina; no restatement awareness |
| BaoStock | ✅ | L | Full statements; no PIT support |
| CNINFO ([cninfo.com.cn](http://www.cninfo.com.cn)) | ✅ | L | Official reports as PDFs — ground truth, needs parsing |
| JQData | 💰 | N | Restatement-aware statement metadata; mainland-only |
| adata ([1nchaos/adata](https://github.com/1nchaos/adata)) | ✅ | T | ⚠️ Core indicators only — the three statements are "暂未提供" |
| Wind / iFinD / Choice | 💰 | T | Institutional grade |

## 4. Dividend events & annual DPS series (分红事件/年度分红序列)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **Tushare Pro** | 🟡 ¥200/yr | V | Full lifecycle: 预案/登记/除息/派息 dates, pre/post-tax, 预案→实施 status, from 2000 — richest table found |
| Eastmoney datacenter ([data.eastmoney.com/yjfp](https://data.eastmoney.com/yjfp/)) | ✅ | L | Keyless JSON API (`RPT_SHAREBONUS_DET`), live-verified returning data |
| AkShare | ✅ | L | `stock_fhps_em` / `stock_fhps_detail_*` / CNINFO endpoints |
| BaoStock | ✅ | L | `query_dividend_data()` incl. pre/post-tax DPS; ⚠️ post-2017 coverage ambiguous — test empirically before relying on it |
| CNINFO | ✅ | L | Per-company 分红方案/实施公告 — primary *event* source |
| SSE dividends page ([sse.com.cn](https://www.sse.com.cn/market/stockdata/dividends/)) | ✅ | L | Aggregated 分红/送股 stats, free JSON API |
| SZSE | — | L | ❌ No dedicated dividend page (404 confirmed) — use its disclosure search or the sources above |
| 理杏仁 | 💰 ¥518/yr | V | 股息率 series with percentile context |

## 5. Announcements & filings, full text (公告/报告)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **CNINFO 巨潮** | ✅ | L | Keyless POST API — **live-verified 2026-11-13** (returned 8,784 real 2026 "分红" announcements); PDFs downloadable; anti-crawl throttling, pageSize 30 |
| SSE / SZSE disclosure search | ✅ | L | Official per-exchange search |
| AkShare (`stock_notice_report`) | ✅ | L | CNINFO/Eastmoney wrapper |
| 见微数据 ([jianweidata.com](https://www.jianweidata.com)) | 🟡 | L | "免费注册即可使用" confirmed on site; full-text search, 7-day trial; paid tiers not public (N) |
| Wind / iFinD | 💰 | T | Structured filing data + search |

> None of these provide point-in-time ("what was public on date X") queries — **build your own announcement archive** (research doc §2.4).

## 6. News (新闻)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **新浪财经 7x24** ([finance.sina.com.cn/7x24](https://finance.sina.com.cn/7x24/)) | ✅ | L | Keyless JSON feed — **live-verified 2026-11-13** returning current items |
| 财联社 telegraph ([cls.cn](https://www.cls.cn/telegraph)) | ✅ web | L | Free web feed live; ⚠️ unofficial JSON endpoint **confirmed dead** (returns HTML); official API commercial (~¥1/call, T) |
| Eastmoney news | ✅ | L | Via AkShare |
| 华尔街见闻 ([wallstreetcn.com](https://wallstreetcn.com/)) | 🟡 | L | Basic feed free; VIP pages 404/500 from here; ~¥9,888/yr Alpha figure is 2025 third-party (T) |

## 7. Research reports (研报)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **东方财富研报中心** ([data.eastmoney.com/report](https://data.eastmoney.com/report/)) | ✅ | T | Free summaries + PDFs, keyless API (page fetch timed out here; API documented) |
| 慧博投研 ([hibor.com.cn](http://www.hibor.com.cn)) | 🟡 | T | 1 report/day free; paid tiers ~¥328–5,980/yr — third-party only, no public price list |
| Wind / iFinD / Choice | 💰 | T | Included in terminal fees |
| Broker public channels | ✅ | T | Free selections from 券商官网/公众号 |

⚠️ Thinnest redundancy of all types — free full 研报 is genuinely scarce in China.

## 8. Macro, rates, FX (宏观/利率/汇率)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **PBOC** ([pbc.gov.cn](https://www.pbc.gov.cn)) | ✅ | T | LPR, benchmark rates, FX parity, monetary stats — web pages, no API |
| **Chinamoney** ([chinamoney.com.cn](https://www.chinamoney.com.cn/chinese/sddsint/)) | ✅ | T | Treasury yield history with Excel export, LPR charts |
| Chinabond ([chinabond.com.cn](https://www.chinabond.com.cn)) | ✅ | T | Yield curves free since 2026-01; free registration for history download |
| NBS ([data.stats.gov.cn](https://data.stats.gov.cn/)) | ✅ | T | CPI/PMI/GDP, keyless JSON API documented; overseas IPs may 403 |
| SAFE ([safe.gov.cn](https://www.safe.gov.cn)) | ✅ | T | RMB central parity, date-filtered |
| Tushare macro interfaces | 🟡 | V | Shibor, treasury yields via API (points tier) |
| Wind EDB | 💰 | T | Institutional macro database |

## 9. Industry policy & concession data (行业政策/特许经营)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **NDRC** ([ndrc.gov.cn](https://www.ndrc.gov.cn)) | ✅ | T | Electricity/gas pricing policy documents |
| NEA ([nea.gov.cn](https://www.nea.gov.cn)) | ✅ | T | Energy statistics and regulation |
| MOT ([xxgk.mot.gov.cn](https://xxgk.mot.gov.cn)) | ✅ | T | Annual 全国收费公路统计公报 — toll revenue, mileage, debt |
| Company filings via CNINFO | ✅ | L | Concession terms and expiry — the per-company facts live here |
| Wind / iFinD industry databases | 💰 | T | Structured policy/industry metrics |

## 10. Tax & legal rules (税务与法规)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **国家税务总局法规库** ([fgk.chinatax.gov.cn](https://fgk.chinatax.gov.cn/)) | ✅ | T | Full-text regulations searchable by validity — where 财税〔2015〕101号 lives |
| 财政部 MOF ([mof.gov.cn](http://www.mof.gov.cn)) | ✅ | T | Fiscal policy documents |
| CSRC 法规数据库 ([csrc.gov.cn](http://www.csrc.gov.cn)) | ✅ | T | Free since 2019-07: laws, rules, enforcement decisions |
| 北大法宝 ([pkulaw.com](https://www.pkulaw.com)) | 💰 | T | Ballpark ~¥3,600/yr personal — third-party; purchase page is a JS shell (no public list) |

## 11. Corporate actions beyond dividends (股本变动/配股/回购)

| Source | Cost | Ver. | Notes |
|---|---|---|---|
| **Tushare Pro** | 🟡 ¥200/yr | V | `share_float`, buyback interfaces, rights issues |
| AkShare | ✅ | L | 股本变动/回购 functions |
| CNINFO | ✅ | L | Authoritative per-event notices |
| BaoStock | ✅ | L | Adjustment factors per corporate action |

---

## Internal data — no external source

Five of the spec's datasets are **internal by design** and never leave the system: 预测与模型 (forecasts), 账户与持仓 (user-entered), 组合与计划 (user-entered), 风险与评分 (derived), 规则与审计 (system rules). Their *inputs* must still carry the provenance metadata of the sources above.

## Tushare pricing — officially verified (vendor doc, 2026-11-13)

| 积分 tier | Price | What it unlocks |
|---|---|---|
| 0 | Free | 8,000 calls/day, unadjusted daily bars only |
| 2,000 | ¥200/yr | 100,000 calls/API — financial statements + dividend data |
| 5,000 | ¥500/yr | Regular data unlimited |
| 10,000 | ¥1,000/yr | Adds 特色数据 (earnings forecasts, chip distribution, etc.) |
| 15,000 | ¥1,500/yr | 特色数据 unlimited |
| — separate | 历史分钟 ¥2,000 · 实时分钟 ¥1,000/月 | Minute data is outside the points system |
| — separate | 新闻舆情、公告 interfaces | **Separately permissioned, not included in any tier** |

## Bottom line

- A **complete free tier exists for every data type** — the announcement backbone (CNINFO) and news stream (Sina 7x24) are both live-verified free.
- The only early spend worth making: **Tushare ¥200/yr** (PIT financials + the dividend table; nothing free replicates both). Optional later: 理杏仁 ¥518/yr for percentile views.
- Redundancy genuinely thins out in two places: full research reports (研报) and — for non-mainland deployment — JQData's quality tier (mainland-only).
- Prices marked **T** are ballparks from procurement records or third parties; vendors marked **N** publish no price list and sell through sales teams — get quotes in writing before budgeting around them.
