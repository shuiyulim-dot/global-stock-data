# global-stock-data

Full-stack market data for US & HK equities — **13 layers · 30+ endpoints · 11 sources · zero-auth · only depends on `requests`**

> 🌐 English (default) · 🇨🇳 中文文档在页面底部，点击 **🇨🇳 中文文档（Chinese）** 展开

> **V2.0 (2026-07-24): official-source-first.** Added CBOE's official options chain (full Greeks + IV + **0DTE** + unusual flow), FINRA market-wide daily short volume, SEC EDGAR filing stream (Form 4 insider / 8-K / 13F) plus a market-wide screener, and Treasury yield curve / CFTC COT / earnings calendar. **Every source is labeled with its compliance tier and the terms quoted verbatim** — "official" does not mean "free to use".

A single self-contained Skill file that turns raw US/HK stock data, scattered across many sources, into a toolkit an AI coding assistant can call directly. You no longer memorize Eastmoney secid prefixes, Yahoo crumb auth flows, or SEC EDGAR CIK mappings — that's all handled.

> **This project distributes code, not market data.** Data is fetched by the user under each source's own terms. For commercial use, rely only on tier-S sources (see below).

> **V1.0.1 fix (2026-06-20 · PR #1):** five functions (quote / Yahoo kline / daily fund flow / search / market list) were missing `params=params`, so requests hit a bare URL and **always came back empty** — all five fixed; also fixed a follow-on `AttributeError` in `market_stock_list` when Eastmoney's `diff` returns a dict (normalized dict→list). Thanks to @APTX4869-maker.

> Compatible with [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/anthropics/openclaw). The Skill file is structured Markdown + embedded Python — any AI coding assistant with context injection can use it.

---

## Architecture

```
US & HK Full-Stack Data · 13-layer architecture · V2.0
│
├── Market Data      Sina(gb_/rt_hk) + Tencent(us/r_hk) + Eastmoney push2     Real-time quotes, 25-78 fields
├── K-line           Sina(back to 1984) + Yahoo chart                         Daily / weekly / monthly / minute
├── Technical Ind.   MA/EMA + MACD + RSI + KDJ + Bollinger                    Pure Python, zero extra deps
├── Fundamentals     Eastmoney datacenter+GMAININDICATOR + Yahoo + SEC XBRL   Statements + key metrics + holdings
├── Fund Flow        Eastmoney push2his                                       Daily main/large/medium/small flow
├── Options          Yahoo crumb                                              Options chain (US only)
├── SEC Filing       EDGAR submissions + XBRL                                 10-K/10-Q/8-K + 503 GAAP metrics (US)
└── Tools            Eastmoney search+list + Yahoo search + SEC CIK           Search + market list + news + ticker↔CIK

━━━ New in V2.0 (official-source-first) ━━━
├── Options·CBOE     cdn.cboe.com official delayed feed   ⭐ full chain + IV + delta/gamma/vega/theta/rho + 0DTE + flow
├── Short Volume     FINRA Reg SHO                        ⭐ market-wide daily short volume (12,112 symbols) + series
├── Filing Stream    EDGAR daily index + full-text        ⭐ Form 4 insider / 8-K / 13F same-day + full text since 2001
├── Market Screener  EDGAR frames                         ⭐ any XBRL tag across the whole market (1,842~5,309 co.) — free
└── Macro / Calendar Treasury + CFTC + Nasdaq             ⭐ yield curve (1M~30Y) + COT + earnings calendar
```

### Compliance tiers (new in V2.0 — read before you use it)

Source terms differ a lot. **"Official" does not mean "free to use".** The tiers below come from reading each source's actual terms of service on 2026-07-24. Quotes are verbatim.

| Tier | Commercial | Redistribute | Sources | Basis (quoted) |
|---|---|---|---|---|
| **S** | ✅ | ✅ | SEC EDGAR / Treasury / CFTC | EDGAR states *"Anyone can access and download this information **for free**"*, *"We **allow scripted access**"*. Hard limit: **10 requests/second**, User-Agent must be declared |
| **B** | ⚠️ verify first | ❌ | FINRA | Files are published for download, but site terms prohibit *"data mining, scraping or harvesting tools (including robots)"* and state *"**non-commercial use**"* |
| **C** | ❌ needs permission | ❌ | CBOE / Nasdaq / Yahoo / Eastmoney / Sina / Tencent | CBOE requires *"**approval in advance**"* + *"**execution of a license agreement**"*; Yahoo states **personal use only** |
| **⛔ Excluded** | — | — | HKEX (CCASS) | Terms explicitly prohibit *"any '**robot**', '**bot**', '**spider**', '**scraper**'..."* and apply *"**whether or not for gain**"* → **this tool ships no scraper for it** |

A working HKEX shareholding layer was built, then removed. Shipping code that violates a source's terms would defeat the purpose of grading sources in the first place.

---

## Quick start

**3 steps, 2 minutes.**

```bash
# 1. Create the skill directory
mkdir -p ~/.claude/skills/global-stock-data

# 2. Download SKILL.md
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md

# 3. Install the dependency
pip install requests
```

⚠️ **Before using the SEC layers, set `SEC_CONTACT` in SKILL.md to your real name and email.** SEC requires a declared User-Agent; requests without one are rejected as an *Undeclared Automated Tool*. The code raises a clear error if you forget.

Launch Claude Code and say "check AAPL's financials" — the skill activates on its own.

> **Codex / OpenClaw users:** paste SKILL.md into your system prompt or project context file. The embedded Python runs as-is.

---

## Endpoints

### Market data (real-time / delayed)

| Endpoint | Data |
|----------|------|
| Sina | US 36 fields (incl. Chinese name / EPS / PE) / HK 25 fields |
| Tencent | US 71 fields / HK 78 fields (incl. PE/PB/market cap/turnover) |
| Eastmoney push2 | US/HK quotes via secid, incl. name / change% / turnover |

### K-line (daily / weekly / monthly / minute)

| Endpoint | Data |
|----------|------|
| Sina | US daily K-line, back to 1984 |
| Yahoo chart | US + HK, v8 API, no crumb, daily/weekly/monthly/minute |

### Technical indicators (computed locally)

| Endpoint | Data |
|----------|------|
| Indicators | MA/EMA + MACD(DIF/DEA/hist) + RSI(6/12/24) + KDJ + Bollinger, pure Python on K-line |

### Fundamentals

| Endpoint | Data |
|----------|------|
| Eastmoney datacenter | US/HK three statements (balance / income / cash flow) |
| Eastmoney GMAININDICATOR | Key metrics (US 49 / HK 75 fields: ROE/ROA/EPS/margins) |
| Yahoo quoteSummary | 23 modules (financials + key stats + analysts + institutional holdings) |
| SEC EDGAR XBRL | 503 GAAP metrics (US only) |

### Fund flow

| Endpoint | Data |
|----------|------|
| Eastmoney push2his | Daily main/large/medium/small net inflow, US + HK |

### Options ⭐ (US only)

| Endpoint | Data |
|----------|------|
| **CBOE official** | Full chain + **IV + delta/gamma/vega/theta/rho**, plus **0DTE filtering** and **unusual-flow** detection (vol/OI > 1 = new positioning) |
| Yahoo (fallback) | Options chain calls + puts, all expiries — **no Greeks** |

### Short interest ⭐ (US only)

| Endpoint | Data |
|----------|------|
| FINRA Reg SHO | Market-wide **daily short volume** (12,112 symbols measured), per-symbol time series, ratio ranking |

### Filing stream ⭐ (US only)

| Endpoint | Data |
|----------|------|
| EDGAR daily index | Same-day **Form 4 insider / 8-K / 13F / 144** filings, one day: 547 / 370 / 261 / 118 |
| EDGAR full-text | Search every filing body back to 2001 |
| EDGAR submissions | 10-K/10-Q/8-K full filing list |
| EDGAR XBRL | Structured financials (revenue / net income / EPS) |

### Market-wide screener ⭐ (US only)

| Endpoint | Data |
|----------|------|
| EDGAR frames | One XBRL tag across the entire market in one call (net income CY2025Q1 = 5,309 companies) — a free screener |

### Macro / calendar ⭐

| Endpoint | Data |
|----------|------|
| Treasury | US yield curve (1M~30Y, daily) |
| CFTC | Commitments of Traders (COT) |
| Nasdaq | Earnings calendar (incl. pre/post-market + EPS estimates) |

### Tools

| Endpoint | Data |
|----------|------|
| Eastmoney search | Stock search (Chinese + English, with market-code mapping) |
| Eastmoney push2 list | Full-market list (US 5925+ / HK 18000+), sort by change% / volume |
| Yahoo search | News by ticker |
| SEC CIK mapping | ticker ↔ CIK (US only) |

### Auth

Every source is **free, no API key**. Yahoo crumb is auto-managed. SEC EDGAR only needs a declared User-Agent (set `SEC_CONTACT`).

---

## What V2.0 adds

| Feature | Notes |
|---------|-------|
| ⭐ **0DTE options flow** | CBOE official feed with full Greeks + IV; `unusual_activity()` flags new positioning via vol/OI > 1. **yfinance has no Greeks; OpenBB's free tier doesn't have this** |
| ⭐ **Market-wide short volume** | FINRA Reg SHO, one file covers the whole market (12,112 symbols), daily short-volume-ratio trend |
| ⭐ **Filing stream** | EDGAR daily index — one measured day: 547 Form 4 insider / 370 8-K / 261 13F |
| ⭐ **Free market screener** | EDGAR frames — one XBRL tag across the whole market (net income CY2025Q1 = 5,309 companies) |
| ⭐ **Compliance tiers** | Every source graded S/B/C with terms quoted — the commercial / redistribution line is spelled out |
| ⭐ **Built-in rate limiting** | Thread-safe throttle; SEC capped at 8 req/s against the official 10 |

### From V1.0 (still here)

| Feature | Notes |
|---------|-------|
| **Zero-auth** | 11 sources, all free, Yahoo crumb auto-managed |
| **Minimal deps** | `requests` only, no third-party data wrappers |
| **US + HK** | Quotes / K-line / statements / fund flow across both markets |
| **Indicators built in** | MA/EMA/MACD/RSI/KDJ/Bollinger, pure Python off the K-line, no extra deps |
| **Full-market list** | Eastmoney push2 pulls US 5925+ / HK 18000+, sorted by change% / volume |
| **Bilingual key metrics** | Eastmoney GMAININDICATOR (Chinese) + Yahoo quoteSummary (English) |
| **Smart code mapping** | Eastmoney secid prefix auto-detect (105/106/107/116), Yahoo `.HK` handled |

---

## Usage examples

Just tell your AI assistant:

| Scenario | Prompt |
|----------|--------|
| US quote | "What's AAPL's price and PE" |
| HK quote | "How's Tencent 00700 doing today" |
| K-line | "Pull TSLA's daily K-line for the past 6 months" |
| Statements | "Show Apple's latest quarterly income statement" |
| Valuation | "BABA's PE/PB/ROE and analyst target price" |
| Institutional holdings | "Which institutions hold NVDA, and what %" |
| Fund flow | "Is money flowing into or out of AAPL lately" |
| 0DTE options flow | "Show NVDA's 0DTE options flow — unusual activity" |
| Short volume | "What's TSLA's short volume ratio trend this week" |
| Insider filings | "Any Form 4 insider filings today for my watchlist" |
| Full-text filings | "Which companies first mentioned 'HBM4' in an 8-K" |
| Market screener | "Rank all filers by R&D expense for CY2025Q1" |
| Macro | "What's the 10Y-2Y Treasury spread right now" |
| SEC filing | "When was Apple's latest 10-K filed" |
| Technical analysis | "AAPL's MACD and RSI — any golden cross?" |
| Batch compare | "Compare valuations of AAPL MSFT GOOGL" |

---

## Data source priority

| Scenario | Primary | Fallback | Notes |
|----------|---------|----------|-------|
| US quotes | Sina `gb_XXXX` | Tencent / Eastmoney push2 | Sina has Chinese name + EPS + PE |
| HK quotes | Tencent `r_hkXXXXX` | Sina / Eastmoney push2 | Tencent has the most fields (78) |
| US K-line | Sina | Yahoo chart | Sina goes back to 1984; Yahoo does multi-period |
| HK K-line | Yahoo chart | — | Sina HK K-line is down |
| Options / Greeks / 0DTE | **CBOE official** ⭐ | Yahoo options | CBOE has Greeks; Yahoo doesn't. US only |
| Short volume | **FINRA Reg SHO** ⭐ | — | Whole market in one file. US only |
| Filing stream | **EDGAR daily index** ⭐ | — | Form 4 / 8-K / 13F same day. US only |
| Market screener | **EDGAR frames** ⭐ | — | Free, whole-market cross-section. US only |
| Yield curve / COT / earnings | **Treasury / CFTC / Nasdaq** ⭐ | — | Macro + event-driven |
| Statements (structured) | Yahoo quoteSummary | — | English, full report structure |
| Key stats | Yahoo quoteSummary | Eastmoney GMAININDICATOR | PE/PB/EV/margins/target |
| Institutional holdings | Yahoo quoteSummary | — | Top 10 institutions + insiders |
| Fund flow | Eastmoney push2his | — | Daily main/large/medium/small |
| SEC filing | EDGAR | — | Official, US only |
| Search / news | Eastmoney search / Yahoo search | — | — |
| Market list | Eastmoney push2 clist | — | Sort by change% / volume |

---

## Data sources

| Source | Tier | Protocol | Auth | Coverage |
|--------|------|----------|------|----------|
| **SEC EDGAR** | **S** | HTTPS | None (real UA) | US filings / XBRL / **filing stream** / **full-text search** / **market screener** |
| **US Treasury** | **S** | HTTPS | None | **Yield curve (1M~30Y)** |
| **CFTC** | **S** | HTTPS | None | **COT reports** |
| **FINRA** | **B** | HTTPS | None | US **market-wide daily short volume** (verify before commercial use) |
| **CBOE** | **C** | HTTPS | None | US **options chain + Greeks + IV + 0DTE** (use needs Cboe's prior approval) |
| **Nasdaq** | **C** | HTTPS | None | US **earnings calendar** (terms unverified) |
| Eastmoney (push2 / push2his / datacenter / search) | C | HTTPS | None | US+HK quotes / fund flow / statements / search |
| Yahoo Finance | C | HTTPS | cookie+crumb (auto) | US+HK all categories (**personal use only**) |
| Sina | C | HTTP | None | US+HK quotes, US K-line |
| Tencent | C | HTTPS | None | US+HK quotes |

**Tiers:** **S** = government data, commercial + redistribution OK · **B** = published files, verify before commercial use · **C** = needs prior approval or terms unverified, personal research only. See "Compliance tiers" above for the quoted basis.

---

## FAQ

**Q: How does this relate to a-stock-data?**
Sister project. a-stock-data covers China A-shares (Shanghai / Shenzhen / Beijing); global-stock-data covers US and HK. Install both skills side by side — they don't conflict.

**Q: Does Yahoo Finance need an API key?**
No. The code fetches cookie + crumb automatically and refreshes on expiry.

**Q: Any limits on SEC EDGAR?**
Yes. SEC requires a declared User-Agent and caps requests at 10/second. Set `SEC_CONTACT` to your real email; the built-in throttle stays under the limit.

**Q: HK options data?**
No. HK options aren't in Yahoo's coverage and need HKEX's paid proprietary feed. The options layer is US only.

**Q: Running from a server in mainland China — can it reach Yahoo / SEC?**
Yahoo and SEC are overseas and may be flaky on a direct connection. Use a proxy, or lean on the Eastmoney / Sina / Tencent sources.

**Q: Can I use it without Claude Code?**
Yes. SKILL.md is Markdown + embedded Python. Codex, OpenClaw, or any AI coding assistant can read it. You can also copy the Python out and run it in your own scripts.

---

## Changelog

See [CHANGELOG.md](./CHANGELOG.md).

---

## Support

If this saved you time, a coffee is appreciated ☕

<p align="center">
  <img src="./assets/wechat-sponsor.jpg" width="240" alt="WeChat sponsor QR">
</p>
<p align="center">
  <a href="https://ifdian.net/a/simonlin">Afdian</a> ·
  <a href="https://buymeacoffee.com/simonlin1212">Buy Me a Coffee</a>
</p>

> Want a data endpoint that isn't here? Open an [Issue](https://github.com/simonlin1212/global-stock-data/issues). Sponsors' issues go first.

---

## Disclaimer

This project provides data-access tools only. It is not investment advice. Investing involves risk.

---

## License

[Apache License 2.0](./LICENSE)

**Author:** Simon Lin · TikTok [@simonlin121212](https://www.tiktok.com/@simonlin121212) · Douyin "Simon林" · WeChat "硅基世纪"

---

<details>
<summary><b>🇨🇳 中文文档（Chinese）</b></summary>

<br>

美股港股全栈数据工具包 — **13 层架构 · 30+ 个端点 · 11 个数据源 · 全部零鉴权 · 仅依赖 `requests`**

> **V2.0（2026-07-24）：官方源优先。** 新增 CBOE 官方期权链（完整希腊字母 + IV + **0DTE** + 异动 flow）、FINRA 全市场每日空头成交量、SEC EDGAR 申报事件流（Form 4 内部人 / 8-K / 13F）与全市场横截面筛选、美债收益率曲线 / CFTC COT / 财报日历。**每个数据源标注了合规级别与条款原文**——因为"官方"不等于"可自由使用"。

一个自包含的 Skill 文件，把分散在多个数据源里的美股/港股原始数据整合成 AI 编程助手直接能用的工具集。你不用再背东财 secid 前缀、Yahoo crumb 鉴权流程、SEC EDGAR 的 CIK 映射——全部封装好了。

> **本工具只分发代码，不分发、不转售任何市场数据。** 数据由使用者自行按各源条款获取。商用请只依赖 S 级源。

> **V1.0.1 修复（2026-06-20 · PR #1）：** 5 个函数（个股行情/Yahoo K线/日级资金流/搜索/全市场列表）漏传 `params=params`，请求实际是裸 URL，**此前始终返回空数据** → 一次性补齐；并修了 `market_stock_list` 在东财 `diff` 返回 dict 时的连带 `AttributeError`。致谢 @APTX4869-maker。

### 架构

```
美股港股全栈数据 · 13 层架构 · V2.0
│
├── 行情层      新浪 + 腾讯 + 东财push2        实时报价 25-78 字段
├── K线层      新浪(回溯至1984) + Yahoo         日/周/月/分钟
├── 技术指标    MA/EMA + MACD + RSI + KDJ + 布林  纯Python，零额外依赖
├── 基本面      东财三表+GMAININDICATOR + Yahoo + SEC XBRL
├── 资金面      东财push2his                    日级主力/大单/中单/小单
├── 期权层      Yahoo crumb                     期权链（仅美股）
├── SEC Filing  EDGAR submissions + XBRL        10-K/10-Q/8-K + 503个GAAP指标
└── 工具层      东财search+列表 + Yahoo + SEC CIK

━━━ V2.0 新增（官方源优先）━━━
├── 期权·CBOE   cdn.cboe.com 官方延时   ⭐ 全链+IV+完整希腊字母+0DTE+异动flow
├── 做空层      FINRA Reg SHO           ⭐ 全市场每日空头成交量(实测12,112只)+时序
├── 申报事件流  EDGAR 每日索引+全文检索 ⭐ Form4内部人/8-K/13F当日 + 2001至今检索
├── 全市场横截面 EDGAR frames           ⭐ 任意XBRL标签一次拿全市场(1,842~5,309家)=免费screener
└── 宏观/日历   Treasury + CFTC + Nasdaq ⭐ 收益率曲线 + COT + 财报日历
```

### 合规分级（取用前必读）

各源条款差异极大，**"官方"不等于"可自由使用"**。以下结论来自 2026-07-24 逐家实读条款原文：

| 级别 | 可商用 | 可再分发 | 源 | 依据（原文摘录） |
|---|---|---|---|---|
| **S** | ✅ | ✅ | SEC EDGAR / Treasury / CFTC | EDGAR 明示 *"for free"*、*"allow scripted access"*；**硬上限 10 请求/秒**，须声明 User-Agent |
| **B** | ⚠️自行确认 | ❌ | FINRA | 数据文件主动发布；但条款禁止 *"data mining, scraping or harvesting tools"*，并声明 *"non-commercial use"* |
| **C** | ❌需授权 | ❌ | CBOE / Nasdaq / Yahoo / 东财 / 新浪 / 腾讯 | Cboe 要求 *"approval in advance"* + *"license agreement"*；Yahoo 写明 personal use only |
| **⛔ 已排除** | — | — | HKEX (CCASS) | 条款明文禁止 robot/bot/spider/scraper，且适用于"不论是否营利" → **本工具不提供该抓取代码** |

一个已跑通的 HKEX 席位持股层被删掉了——发布违反条款的抓取代码，会让"给数据源分级"这件事本身失去意义。

### 快速开始

```bash
mkdir -p ~/.claude/skills/global-stock-data
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md
pip install requests
```

⚠️ **用 SEC 相关层之前，先把 SKILL.md 里的 `SEC_CONTACT` 改成你自己的真实姓名和邮箱**，否则 SEC 会拒绝请求。

启动 Claude Code，说一句「帮我看看 AAPL 的财报」自动激活。Codex / OpenClaw 用户把 SKILL.md 贴入系统 prompt 即可。

### 使用示例

跟你的 AI 助手说这些话就能激活：「AAPL 现在什么价，PE 多少」/「腾讯 00700 今天行情」/「拉 TSLA 最近半年日K线」/「苹果最新一季利润表」/「NVDA 的 0DTE 期权异动」/「TSLA 本周空头成交占比趋势」/「今天有哪些 Form 4 内部人申报」/「哪些公司在 8-K 里首次提到 HBM4」/「按 CY2025Q1 研发费用给全市场排名」/「现在 10Y-2Y 美债利差多少」/「AAPL 的 MACD 和 RSI 有没有金叉」。

### FAQ

**和 a-stock-data 什么关系？** 姊妹项目。a-stock-data 覆盖 A 股，global-stock-data 覆盖美股港股，两个 Skill 可同时装、互不冲突。

**Yahoo 要 API Key 吗？** 不要，代码自动管理 cookie + crumb。

**SEC EDGAR 有限制吗？** 有，须声明 User-Agent + 每秒 10 次上限，代码已内置节流；记得改 `SEC_CONTACT`。

**港股期权有吗？** 没有，期权层仅美股。

**国内服务器能访问 Yahoo/SEC 吗？** 境外服务，直连可能不稳，建议走代理或优先用东财/新浪/腾讯。

**不用 Claude Code 能用吗？** 能。SKILL.md 是 Markdown + 内嵌 Python，任何 AI 编程助手都能读，也可以直接把代码复制出来跑。

### 免责声明

本项目仅提供数据获取工具，不构成任何投资建议。股市有风险，投资需谨慎。

### License

[Apache License 2.0](./LICENSE) — 自由使用，注明出处即可。

**作者：** Simon 林 · 抖音「Simon林」 · 公众号「硅基世纪」

</details>
