# Financial AI Agent Landscape & Domain Research

**Purpose:** Research report for the A-share dividend-investing agent project (the system described in `README.md` V1.1).
**Audience:** The developer (non-financial background), to be shared with the financial partner where useful.
**Date:** 2026-10-26
**Method:** Parallel web research (arXiv abstracts/APIs, paper pages, GitHub) plus direct verification of repo stats via the GitHub API and of contested paper titles via the arXiv API. All star counts below were fetched from the GitHub API on 2026-10-26.

---

## Part 1 — Decoding the spec (for a non-financial developer)

### 1.1 The strategy in one paragraph

The system is a **fundamental value-investing workflow dressed as software**. It finds companies that (a) have a proven multi-year record of paying ordinary cash dividends out of real earnings and cash flow, (b) are currently cheap enough that the *future* dividend stream yields an acceptable after-tax return with a margin of safety, and (c) pass a veto-layer of hard disqualifiers (fraud, broken business model, etc.). It then stages entry over 3–6 tranches (re-checking fundamentals before each one), monitors announcements for deterioration, and computes the resulting after-tax cash flow against household spending. It explicitly never auto-trades. The philosophy is Graham/Buffett-style value investing applied to dividend payers — the "quality dividend growth" school — with unusually strict bookkeeping around taxes and data provenance.

### 1.2 Core concept glossary

| Concept | Plain-English meaning | Why the spec cares |
|---|---|---|
| **DPS** (dividend per share) | Cash paid per share per period | The atomic unit of the whole system; everything is normalized to it |
| **Ordinary vs special dividend** | Ordinary = recurring policy dividend; special = one-off windfalls (e.g., asset sales) | Special dividends must never be extrapolated into "sustainable DPS" (Ch. 4) |
| **Ex-dividend / record / payment dates** | 除权除息日: price drops by dividend; 股权登记日: who's entitled; 派息日: cash arrives | The system tracks the full lifecycle to model when cash actually lands |
| **Payout ratio** (支付率) | Dividends ÷ net profit | Signals whether dividends are affordable; 30–70% is the spec's unproven "reasonable" band |
| **Free cash flow** (FCF) | Operating cash flow − capital expenditure | Dividends should be paid from FCF, not borrowing; "coverage" = FCF ÷ dividends |
| **Net debt / EBITDA** | Leverage relative to pre-interest earnings | Debt-funded dividends are a red flag (Ch. 4.2) — but invalid for banks, whose deposits are business, not debt |
| **ROE** | Net income ÷ shareholders' equity | Long-run profitability; for banks, PB valuation is judged against ROE |
| **PE / PB / EV-EBITDA / FCF yield** | Price vs earnings / book value / enterprise cash earnings / FCF | Different industries need different lenses (PE for utilities, EV/EBITDA for energy/telecom, PB-ROE for banks) |
| **Valuation percentile** (分位) | Where today's PE/PB ranks in its own 5–10 yr history | The spec's "cheapness" signal, with an explicit formula and edge cases (negative PE excluded, direction-aware) |
| **DCF / intrinsic value** | Discounting future cash flows to today | Backs the "conservative intrinsic value" used for the safety-margin price |
| **Margin of safety** (安全边际) | Buy at a discount to estimated intrinsic value | The buffer against estimation error; `P_build = min(yield-price, safety-margin price)` |
| **Normalized earnings** | Earnings with one-offs and cycle effects removed | Prevents extrapolating a boom-year profit (or a disaster year) into DPS forecasts |
| **Three yields** | TTM (backward), forward sustainable (forecast), cost-basis (your own entry price) | Only the forward one justifies *new* buys; cost-basis yield is comfort, not valuation |
| **Tranching** (分批建仓) | Buying a planned position in staged slices | Reduces timing risk; each tranche re-runs the full quality check |
| **Value trap** (价值陷阱) | A high yield caused by the *price* falling on bad fundamentals | Why a 9.5% yield can be a sell signal, not a buy (the "D" row in Ch. 11.2) |
| **Holding-period dividend tax** | A-share: 20% (<1 mo), 10% (1 mo–1 yr), 0% (>1 yr); HK Connect: 20% | The tax is *reconciled on sale* (财税〔2015〕101号), so cash received ≠ final after-tax; the system must track potential back-tax per lot |
| **Point-in-time data** | Data as the market could see it on a given date (using announcement dates, pre-restatement values) | Without it, backtests cheat by "knowing the future" — see §2.3, the single biggest trap in this field |
| **Survivorship bias** | Backtesting only companies that still exist | Delisted/bankrupt names must be included or returns are fiction |
| **Look-ahead bias** (前视偏差) | Signals that use data not yet published at decision time | The #1 way agent backtests fabricate alpha |
| **Restatement** (财报重述) | Companies revise past statements | The spec demands restatements create new versions, never overwrite history |
| **TWR vs MWR** | Time-weighted (manager skill) vs money-weighted (investor outcome) returns | Needed because dividends + deposits + withdrawals otherwise get double-counted |
| **复权 (qfq/hfq)** | Price adjusted for dividends/splits (forward/backward) | A-share prices must be adjusted to be comparable across ex-dividend dates — and the adjustment must be applied point-in-time, not globally |

### 1.3 What the spec actually demands computationally

Stripped of finance language, the spec is a precise description of these components:

1. **Data ingestion layer** — securities master (with entity-verification, Ch. 2.1), dividend event tables, financial statements (restatement-aware), prices + adjustment factors, announcement feed (15-minute SLA, Ch. 8.2), tax rule tables (versioned, Ch. 10).
2. **Deterministic calculators** — payout, coverage, normalized-DPS scenarios, three yields, percentiles, DCF, and the five price points (P_target, P_margin, P_build, P_add, P_overvalued). All formula-defined in the spec; none requires intelligence, all require correctness.
3. **State machines** — research labels (qualified/watch/not-qualified/insufficient), alert lifecycle (new → verified → in-progress → resolved/closed), tranche plan states, veto layer.
4. **Scoring engine** — 8 dimensions, 0–100 standardized, 80/20 quality/price split, missing data = partial score + "cannot fully judge," veto as a separate layer (Ch. 11).
5. **Portfolio & cash-flow engine** — weight caps, cash pool (现金水池), expense coverage, tax-lot accounting with holding-period reconciliation.
6. **Event-driven backtester** — point-in-time, look-ahead-safe, costs/taxes/fees included, compared against benchmarks, training/validation/out-of-sample separation (Ch. 15).
7. **Audit substrate** — rule versions, decision snapshots, "published versions must not be silently rewritten" (Ch. 13), old/new value + reason + timestamp on every manual change (Ch. 12.3).

Notice: **everything above is deterministic**. The LLM/agent layer is nowhere in this list — the spec's logic is rules and formulas. The agent's natural home is the *research* work (reading announcements, drafting evidence, proposing DPS scenarios), with all numbers computed by the deterministic layer. This matches the 2026 academic consensus on safe financial-agent design (§2.6).

### 1.4 Bilingual glossary for talking with your partner

| English (this report) | 中文 (her spec) |
|---|---|
| dividend yield | 股息率 |
| payout ratio | 支付率 |
| free cash flow | 自由现金流 |
| margin of safety | 安全边际 |
| tranched position building | 分批建仓 |
| intrinsic value | 内在价值 |
| value trap | 价值陷阱 |
| ex-dividend / record date | 除权除息日 / 股权登记日 |
| normalized earnings | 正常化利润 |
| survivorship bias | 幸存者偏差 |
| look-ahead bias | 前视偏差 |
| point-in-time data | 时点口径 / 公告日可回溯数据 |
| restatement | 财报重述 |
| cash pool | 现金水池 |
| veto layer | 否决层 |
| forward sustainable yield | 可持续预期股息率 |

### 1.5 Questions worth asking your partner before architecture work

1. **Data budget.** A-share data costs money (§3.4): Tushare's useful tier ≈ ¥200–1,500/yr; commercial alternatives (JQData, Wind) are ¥12k+/yr. Who pays, and what's the ceiling?
2. **Is the 15-minute alert SLA (Ch. 8.2) a hard requirement?** It drives whether you need a push feed or polling is acceptable.
3. **Who is the user?** Just the two of you, or a product for external retail investors? External users radically change compliance exposure (投资咨询/荐股 regulation in China) and mean you should get a legal review before launch.
4. **Single-user vs multi-user** — Ch. 12.3 sketches role-splitting; the data model changes if it's real.
5. **Her stance on LLM-generated conclusions appearing in the product.** Her spec is written around human review; the plan to "eventually automate all decisions" needs her explicit staged buy-in (research assistant → proposal agent → rule executor → fully autonomous), tied to backtest evidence rather than a date.

---

## Part 2 — Academic state of the art (2024–2026)

### 2.1 The consensus architecture

Across 2024–2026 papers, financial LLM agents have converged on a four-part design:

- **Role decomposition** — a hierarchy of specialists (analysts by data type, a synthesizer, and a final decision-maker). TradingAgents made the bull/bear debate famous; newer work (e.g., Oxford's fine-grained task decomposition, arXiv 2602.23330) splits roles into finer tasks.
- **Layered memory** — FinMem (arXiv 2311.13743) established recency/relevance/importance retrieval over a hierarchical store; most successors keep this. For a dividend system, memory = accumulated evidence across quarters.
- **Tool-augmented computation** — the LLM orchestrates and reasons; tools compute metrics and fetch data. FinAgent (arXiv 2402.18485) is the reference.
- **Reflection / self-improvement** — FinCon (arXiv 2407.06567) formalized "verbal reinforcement" (textual gradient descent on prompts). The 2026 governance work (§2.4) exists largely because raw reflection is unauditable.

### 2.2 Key systems

**General trading agents**

| Work | Venue | Design | Caveat / takeaway |
|---|---|---|---|
| FinMem (2311.13743) | AAAI-SS 2024 | Layered memory + risk personality | Reference memory architecture; small universe, daily horizon |
| FinAgent (2402.18485) | KDD 2024 | Multimodal (charts/text) + tools + dual reflection | Earliest multimodal agent; short-horizon, crypto-flavored |
| FinCon (2407.06567) | NeurIPS 2024 | Manager–analyst + verbal reinforcement | Self-improving prompts, hard to audit |
| TradingAgents (2412.20138) | arXiv 2024 | 7 roles, bull/bear debate, risk team, PM veto | De-facto topology reference; paper's own footnote concedes its headline Sharpe is an artifact of a calm Q1-2024 window; ~11 LLM calls per decision |
| StockAgent (2407.18957) | ACM TIST | Multi-agent market *simulator* (behavioral study) | Not a strategy; useful for stress-simulation |
| HedgeAgents (2502.13165) | WWW 2025 | 4 agents, 23 tools, 3 memory types, balance-aware | Reported returns widely considered implausible; treat as design ideas only |

**Long-horizon / fundamental / value investing (closest to your spec)**

| Work | Venue | Design | Relevance to you |
|---|---|---|---|
| **QuantMental** (2510.21147) | arXiv 2025 | Hierarchical: macro agent + firm-level fundamental/technical/news/report agents + PPO portfolio agent, on **CSI 300 A-shares**, 2024 out-of-sample | The single most transferable architecture found for fundamental A-share multi-agent work |
| **Columbia screener** (2603.23300) | arXiv 2026 | LLM strategy screening on fundamental characteristics, **annual retraining**, 5–10 yr evals | Long-horizon discipline done right; sobering finding that human analysts *degraded* performance |
| GuruAgents (2510.01664) | arXiv 2025 | Investor personas (Graham/Buffett…) + **deterministic metric pipelines** | Philosophy-as-prompt + deterministic tools beat free-form LLM stock picking in their tests |
| MarketSenseAI 2.0 (2502.00415) + validation (2604.17327) | arXiv 2025/26 | RAG + agents, monthly-thesis cadence | Proven monthly research → conviction-screened portfolio pattern |
| Oxford expert teams (2602.23330) | arXiv 2026 | Fine-grained task decomposition; leakage-controlled backtests on Japanese stocks | Japan ≈ dividend-heavy market; their eval hygiene is worth copying |
| FundaPod (2605.27864) | arXiv 2026 | Personas write memos; disagreements surfaced to a human PM; knowledge-graph evidence links | Directly matches your auditability + human-review requirements |
| MASS (2505.10278) / MEME (2602.11918) | arXiv 2025/26 | PKU: agent-population scaling / investment-argument extraction on SSE-50 & CSI indices | A-share-native; MEME's temporal OOS design is a good leakage check |
| RD-Agent (2505.15155) | Microsoft, 2025 | Autonomous factor-R&D loop on qlib (propose → implement → backtest → feedback) | The pattern for iterating on dividend-sustainability factors automatically |

### 2.3 Evaluation & bias — the part most systems get wrong

This literature exploded in late 2025 and is the most important reading for you:

- **Five canonical failures** (arXiv 2603.27539): look-ahead bias, survivorship bias, backtest overfitting, transaction-cost neglect, regime-shift blindness — "can flip the sign of reported returns."
- **164-paper bias audit** (arXiv 2602.14233): no single bias addressed in >28% of papers; formal proof that survivorship inflates backtested returns.
- **FINSABER** (2505.07078, KDD 2026): two decades, 100+ symbols — previously reported LLM advantages "deteriorate significantly" once survivorship and data-snooping are controlled.
- **Memorization** (2504.14765): masking tickers doesn't remove model recall; pre-cutoff "skill" is unidentifiable from memorized data.
- **FinAgentBench** (2025): with the headline ticker withheld, models still produce strong signals — pure memorization. Real judgment exists only on post-cutoff text.
- **FinanceGym** (2607.27853, Google Cloud AI + UCLA): point-in-time deep-research benchmark with hard cutoff dates; *every* system scores far worse post-cutoff (<45% ceiling).
- **AI-Trader** (2512.10971, HKU): live, contamination-free benchmark across US / crypto / **SSE-50 A-shares** — top models posted negative returns and **no agent beat SSE-50 buy-and-hold**.
- **StockBench** (2510.02209): most agents fail to beat buy-and-hold.
- **FINAL-Bench** (HuggingFace, 2026): metacognition benchmark — agents can say "I might be wrong" (awareness 0.694) but cannot fix errors (recovery 0.302).

**Implication:** your backtest harness is not a phase-2 nicety, it's the credibility backbone of the whole product. The spec's Ch. 15 requirements (point-in-time data, delisted names, train/validation/OOS) are exactly the 2026 consensus, and her partner-written spec is ahead of most published agent papers on this.

### 2.4 Guardrails, governance, auditability

- **SafeFinAgent** (ICLR 2026): inter-agent guardrails + finance-specific hallucination detector (fabricated metrics, phantom regulations) → −67% hallucinations, −94% task performance retained; naive guardrails cost 23–41% performance. Key data point: safety is cheap only if designed in.
- **SHARP** (2605.06822): symbolic rubric optimization so self-improvement stays human-auditable.
- **SAGE-Fin** (2608.09025): typed authority-handoff contracts — agent actions require explicit effect authorization.
- **TrustTrade** (2603.22567): selective consensus + cross-validation against over-trusting retrieved information.
- **TradeArena** (2605.28850): records full trajectories (observations → signals → decisions → fills) as an auditable testbed.

These map almost one-to-one onto your spec's veto layer, alert state machine, and audit requirements — the spec's governance instincts are what 2026 research is formalizing.

### 2.5 Takeaways for this project

1. The role-team topology (researchers → critics → risk → final gate) is settled practice — borrow it, but expect it to be LLM-call expensive; run cheap models on retrieval, expensive ones only on reasoning.
2. Deterministic computation behind tools, LLM for reading/writing/synthesizing — never let the model produce a number it didn't fetch.
3. Long-horizon dividend work is a genuine research gap: nothing published does multi-year dividend sustainability + tax-aware cash flow on A-shares. That's your differentiation, not a reason to worry.
4. Evaluation hygiene is existential; A-share live results (SSE-50) are sobering. Expect the agent to *underperform* naive benchmarks until data and guardrails are right.
5. Design the audit layer before the agent loop — retrofit auditing is how most published systems fail their own claims.

---

## Part 3 — Open-source landscape

All stats verified via GitHub API on 2026-10-26.

### 3.1 Multi-agent LLM frameworks

| Project | Stars | License | Notes for you |
|---|---|---|---|
| [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents) | 115,078 | Apache-2.0 | The canonical analyst→researcher→bull/bear debate→risk team→PM topology; LangGraph-based; data layer swappable. Active. |
| [virattt/ai-hedge-fund](https://github.com/virattt/ai-hedge-fund) | 64,262 | MIT | Simplest code to fork: parallel specialist agents emitting typed signal JSON, PM final gate. No corporate actions, no tax. Active. |
| [hsliuping/TradingAgents-CN](https://github.com/hsliuping/TradingAgents-CN) | 32,506 | "Other" ⚠️ | **Most relevant project found**: TradingAgents topology wired to AkShare/Tushare for A-shares, Chinese output. Review license before commercial use. |
| [The-FinAI/FinCon](https://github.com/The-FinAI/FinCon) | ~70 | MIT | Official FinCon repo; verbal-reinforcement memory pattern for accumulating evidence across quarters. |
| [AI4Finance-Foundation/FinRobot](https://github.com/AI4Finance-Foundation/FinRobot) | 8,310 | Apache-2.0 | Scheduler routing across four agent layers + MCP support; document-analysis layer maps to your CNINFO ingestion. |
| [MingyuJ666/Stockagent](https://github.com/MingyuJ666/Stockagent) | 714 | none | StockAgent simulator (behavioral evaluation, not production). |

### 3.2 Quant research infrastructure

| Project | Stars | License | Notes for you |
|---|---|---|---|
| [microsoft/qlib](https://github.com/microsoft/qlib) | 49,716 | MIT | Microsoft's quant platform: expression-engine factors, model zoo, A-share conventions built in. **Two things to steal even if you don't adopt it**: (1) the point-in-time fundamentals DB design (reports usable only from announcement date), (2) the **NestedExecutor** — multi-level nested decision execution that is the canonical look-ahead-safe event-driven structure. |
| [microsoft/RD-Agent](https://github.com/microsoft/RD-Agent) | 15,060 | MIT | Autonomous factor-R&D loop on qlib; repurpose for iterating dividend-sustainability factor models. |

### 3.3 Research tooling & skills

| Project | Stars | License | Notes for you |
|---|---|---|---|
| [openbq-org/OpenBB](https://github.com/openbq-org/OpenBB) | 74,725 | source-available ⚠️ | Provider-agnostic financial data API (~100 providers); its standardized models for fundamentals/dividends are a good schema reference. Its "OpenBB Agents" live inside the commercial Workspace; the old standalone `openbb-agents` is archived. |
| [quant-sentiment-ai/claude-equity-research](https://github.com/quant-sentiment-ai/claude-equity-research) | 744 | MIT | "Skill-pack" pattern: markdown research protocols + tool bundles per analyst persona — cheap way to build your researcher/bull/bear personas. |
| [agi-now/buffett-skills](https://github.com/agi-now/buffett-skills) | 840 | none ⚠️ | Buffett-framework prompts (moats, owner earnings, margin of safety) — scaffolding for your analyst agent; check license. |
| [astro30/valinvest](https://github.com/astro30/valinvest) | 196 | MIT | Piotroski F-Score / Graham formulas ready-made; dormant but formulas transfer. |
| [UFund-Me/Qbot](https://github.com/UFund-Me/Qbot) | 18,735 | MIT | A-share all-in-one (data download → factors → backtest → live), LLM is a thin assistant layer. |

### 3.4 A-share data sources

| Source | Cost | Dividend coverage | Point-in-time? | Verdict |
|---|---|---|---|---|
| **Tushare Pro** ([waditu/tushare](https://github.com/waditu/tushare), 15,465★, BSD-3-Clause) | points tiers: ~¥200–1,500/yr | Full lifecycle `dividend` table: ann/record/ex/pay dates, pre/post-tax amounts, 预案→实施 status, from 2000 | **Yes — the killer feature**: financial statements carry `ann_date` + `update_flag` (0 = superseded, 1 = latest), so `ann_date <= asof AND update_flag==1` gives genuine restatement-aware, announcement-date-accurate fundamentals | **Recommended primary source** |
| **AkShare** ([akfamily/akshare](https://github.com/akfamily/akshare), 23,081★, MIT) | free | `stock_fhps_em` / `stock_fhps_detail_*` / CNINFO dividend endpoints | No (scraped, current-knowledge only, no SLA) | Great free complement for announcements/calendars |
| **BaoStock** | free | `query_dividend_data()`: full timeline incl. pre/post-tax amounts | No | Useful cross-check; official GitHub repo is gone (mirrors only) — data service still live |
| **JQData** (JoinQuant, closed) | trial free; professional ≈ ¥12k+/yr | Dividend extras thinner than Tushare's table; restatement metadata available | Partial (restatement-aware statements) | Reportedly blocked non-mainland IPs since 2026-08; verify |
| [1nchaos/adata](https://github.com/1nchaos/adata) | free | 5,274★, Apache-2.0: dividend tables + financials | No | Zero-cost backup |
| **CNINFO 巨潮资讯** | free (anti-crawl) | Official disclosure API — the announcement source of truth | You build the PIT archive | Wrappers: [agentladle/mcp-cninfo](https://github.com/agentladle/mcp-cninfo) (MCP server, 6 tools), [rollysys/use_cninfo](https://github.com/rollysys/use_cninfo) (CLI + PDF→Markdown), [linxiaoqi5111-del/finhot](https://github.com/linxiaoqi5111-del/finhot) (Atom-RSS watchlist → clean alert-source pattern) |

**PIT verdict:** point-in-time prices (via adjustment factors) and fundamentals (Tushare `ann_date`/`update_flag`) are solvable now. A point-in-time *announcement text* archive does not exist open-source — you must snapshot CNINFO yourself. That's a small pipeline and worth owning.

### 3.5 Backtesting engines

| Engine | Stars | License | A-share fit |
|---|---|---|---|
| [ricequant/rqalpha](https://github.com/ricequant/rqalpha) | 6,840 | NOASSERTION ⚠️ | **Purpose-built for A-shares.** Native corporate actions (dividends/splits in data bundles, ex-day adjustments). **v6.2.0 (Jun 2026) added the differentiated holding-period dividend tax (20%/10%/0%) — the only open engine found with real tax-aware accounting.** Strongest off-the-shelf fit. |
| qlib NestedExecutor | (49.7k, above) | MIT | The look-ahead-safe nested-decision architecture to imitate; A-share calendars/limit mechanics built in |
| [nautechsystems/nautilus_trader](https://github.com/nautechsystems/nautilus_trader) | 30,863 | LGPL-3.0 | Production-grade Rust engine, native corporate-action events, backtest/live parity — most robust, steepest curve |
| [fasiondog/hikyuu](https://github.com/fasiondog/hikyuu) | 3,579 | Apache-2.0 | A-share-native, 复权 handled internally, C++ speed |
| [vnpy/vnpy](https://github.com/vnpy/vnpy) | 46,068 | MIT | For later: backtest → paper → live in one event engine |
| [mementum/backtrader](https://github.com/mementum/backtrader) | 23,578 | GPL-3.0 ⚠️ | Dormant since 2024; no corporate-action engine; GPL viral |
| [polakowo/vectorbt](https://github.com/polakowo/vectorbt) | 9,541 | fair-code ⚠️ | Vectorized research speed on pre-adjusted data; easy to introduce look-ahead — use for factor sweeps only |
| [stefan-jansen/zipline-reloaded](https://github.com/stefan-jansen/zipline-reloaded) | 1,974 | Apache-2.0 | US-centric; CN bundles abandoned; right conceptual model for PIT 复权 though |

### 3.6 Dividend analytics, tax lots, ledger — and the confirmed gaps

- [ricequant/rqalpha](https://github.com/ricequant/rqalpha) v6.2 — reference implementation of the A-share dividend-tax tiers (even if you don't adopt the engine).
- [abishekprabakar/taxlot-lab](https://github.com/abishekprabakar/taxlot-lab) — only general tax-lot engine found (FIFO/LIFO/HIFO); its lot logic transfers to A-share holding-period reconciliation (ignore its wash-sale rules — that's US-only).
- [ghostfolio/ghostfolio](https://github.com/ghostfolio/ghostfolio) — 9,582★, AGPL ⚠️: portfolio/dividend ledger UX and data-model reference (not A-share-tax-aware).
- [FinMind/FinMind](https://github.com/FinMind/FinMind) — 2,895★, Apache-2.0: Taiwan dividend/statement dataset schema inspiration.

**Gaps confirmed by search (i.e., things you will build as your own IP):**
1. **No open-source dividend-sustainability scoring library** exists (payout-ratio/FCF-coverage/dividend-history scoring). Dividend screeners found were all ≤3-star toys.
2. **No open-source China dividend tax-lot / cash-flow ledger** (holding-period reconciliation per lot).
3. **No open-source point-in-time CNINFO announcement archive.**
4. No A-share dividend-investing multi-agent system — TradingAgents-CN is the closest, and it's momentum/technical-trading flavored.

---

## Part 4 — Gap analysis: spec needs vs what exists

| Spec component (README ch.) | Closest existing | Gap → build/adopt |
|---|---|---|
| 8-dim screening + industry templates (Ch. 2–3) | GuruAgents pattern, valinvest formulas | **Build**: A-share data wiring + industry template engine; LLM assists evidence-writing only |
| Dividend history, 3 yields, payout/coverage (Ch. 4–5) | Tushare/AkShare raw data | **Build**: deterministic calculators; no OSS equivalent |
| Price gates + DCF + safety margin (Ch. 6) | qlib expressions, valinvest | **Build**: incl. finite-concession DCF for highways — unique to your spec |
| Tranching plans + alert state machine (Ch. 6–8) | none | **Build**: deterministic state machines |
| Scoring 80/20 + veto layer (Ch. 11) | none | **Build** |
| Tax-aware cash flow & cash pool (Ch. 9–10) | RQAlpha tax tiers, taxlot-lab lots, ghostfolio UX | **Build**: ledger with per-lot holding-period reconciliation |
| Announcement ingestion, 15-min SLA (Ch. 8.2) | CNINFO scrapers/MCP | **Build**: pipeline + your own PIT archive |
| Look-ahead-safe backtest (Ch. 15) | RQAlpha, qlib NestedExecutor | **Adopt** (likely RQAlpha for tax support; qlib architecture as fallback/reference) |
| Audit trails, rule versioning (Ch. 12–13) | qlib rule mgmt, SAGE-Fin ideas | **Build**: append-only audit substrate |
| LLM research layer (not in spec yet) | TradingAgents(-CN) topology, FundaPod memo pattern, SafeFinAgent guardrails | **Design**: personas + tools-only data access + hallucination checks |

---

## Part 5 — Recommended next steps

1. **Data spike (1–2 weeks).** Buy Tushare's 2,000-point tier (~¥200/yr). Verify the `dividend` table and `ann_date`/`update_flag` PIT queries on 5–10 bank/utility names; reconstruct 10-year ordinary-DPS series and cross-check against AkShare/BaoStock. This de-risks the single most load-bearing external dependency.
2. **Backtest core decision.** Prototype one toy dividend strategy on both RQAlpha (tax support) and qlib's NestedExecutor (PIT fundamentals); pick one. My lean: RQAlpha, because holding-period tax accounting is in your spec's Ch. 10 and RQAlpha is the only engine that has it.
3. **Walking skeleton.** Deterministic calculators (yields, gates, percentile) + one industry template (banks), validated against the Ch. 14 acceptance cases turned into executable tests. No LLM yet.
4. **LLM layer last, guarded.** Researcher/critic/risk personas following TradingAgents-CN's topology, with tool-only data access, evidence-linked outputs (FundaPod-style), and a SafeFinAgent-style hallucination check on every figure. Wire its outputs into the existing alert/audit substrate.
5. **Evaluation harness from day one.** PIT cutoff-date evals (FinanceGym/StockBench pattern), alert recall/precision tests, and a bias checklist (the five failure modes) as a CI gate on any new agent behavior.
6. **Partner questions (§1.5) answered before scope-locking** the MVP.

---

## Appendix — verification notes & caveats

- **Could not verify / likely non-existent as cited:** papers titled "Mori" (value-investing agent), "MemFin" (memory trading agent), or "FamaAgent" — arXiv API title searches return nothing relevant (checked 2026-10-26; a `hughcai/Mori` GitHub repo also returns 404). A "FINAL" agentic-finance benchmark exists only as **FINAL-Bench** on HuggingFace (metacognition benchmark), not as an arXiv paper, and `ucfinlab/final-bench` on GitHub does not exist. Closest real substitutes are listed in §2.2. If your partner or future sources cite these names, treat them as unverified until a real URL is produced.
- **HedgeAgents' reported returns** (≈70% annualized) are widely considered implausible and unreplicated — cited above for design ideas only.
- **Star counts/licenses** are a point-in-time snapshot (GitHub API, 2026-10-26); licenses marked ⚠️ need legal review before commercial reuse (TradingAgents-CN "Other", OpenBB source-available, RQAlpha NOASSERTION, backtrader GPL-3.0, ghostfolio AGPL-3.0, vectorbt fair-code).
- **Vendor details to re-verify before budgeting:** Tushare tier pricing, JQData pricing and the reported non-mainland IP restriction (since 2026-08), CNINFO rate limits.
- Several 2026 arXiv items cited here (FundaPod, MEME, SAGE-Fin, SHARP, TradeArena, FinanceGym, Columbia screener, Oxford expert teams) are preprints without peer review yet.
