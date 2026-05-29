---
name: dcf-valuation
description: Performs discounted cash flow (DCF) valuation analysis to estimate intrinsic value per share. Triggers when user asks for fair value, intrinsic value, DCF, valuation, "what is X worth", price target, undervalued/overvalued analysis, or wants to compare current price to fundamental value.
---

# DCF Valuation Skill

## Workflow Checklist

Copy and track progress:
```
DCF Analysis Progress:
- [ ] Step 0: Detect market (US vs KR) and pick the matching path
- [ ] Step 1: Gather financial data
- [ ] Step 2: Calculate FCF growth rate
- [ ] Step 3: Estimate discount rate (WACC)
- [ ] Step 4: Project future cash flows (Years 1-5 + Terminal)
- [ ] Step 5: Calculate present value and fair value per share
- [ ] Step 6: Run sensitivity analysis
- [ ] Step 7: Validate results
- [ ] Step 8: Present results with caveats
```

## Step 0: Detect Market

Decide which path to follow before gathering data:

- **KR path** — if the ticker is a 6-digit number (e.g. `005930`, `035420`) or the company is identified by a Korean name (삼성전자, 네이버). Korean listings report under **K-IFRS** and trade in **KRW**.
- **US path** — if the ticker is an ASCII symbol (`AAPL`, `MSFT`). Reports under US GAAP, trades in USD.

Each step below states **US default** and, where it differs, a **🇰🇷 KR override**. Follow the override only on the KR path; otherwise use the US default unchanged.

## Step 1: Gather Financial Data

**US path** — call the `get_financials` tool with these queries:

> **🇰🇷 KR override:** Call `get_financials_kr` instead (it routes to DART 사업/반기/분기보고서). One natural-language query is enough, e.g. `"005930 최근 5년 연결 재무제표 현금흐름·손익·재무상태표"`. Then use `get_market_data` for the current price if it returns Korean tickers; otherwise fall back to `web_search`/`get_foreign_ownership_kr` for the latest close.
>
> DART account labels (`account_nm`) vary by company and year — **never exact-match**. 영업수익 vs 매출액, 당기순이익(손실) suffixes, etc. Match on substrings / the standardized `account_id` where present, and reconcile across periods.

### 1.1 Cash Flow History
**Query:** `"[TICKER] annual cash flow statements for the last 5 years"`

**Extract:** `free_cash_flow`, `net_cash_flow_from_operations`, `capital_expenditure`

**Fallback:** If `free_cash_flow` missing, calculate: `net_cash_flow_from_operations - capital_expenditure`

### 1.2 Financial Metrics
**Query:** `"[TICKER] financial metrics snapshot"`

**Extract:** `market_cap`, `enterprise_value`, `free_cash_flow_growth`, `revenue_growth`, `return_on_invested_capital`, `debt_to_equity`, `free_cash_flow_per_share`

### 1.3 Balance Sheet
**Query:** `"[TICKER] latest balance sheet"`

**Extract:** `total_debt`, `cash_and_equivalents`, `current_investments`, `outstanding_shares`

**Fallback:** If `current_investments` missing, use 0

### 1.4 Current Price
Call the `get_market_data` tool:

**Query:** `"[TICKER] price snapshot"`

**Extract:** `price`

### 1.5 Company Facts
Call the `get_financials` tool:

**Query:** `"[TICKER] company facts"`

**Extract:** `sector`, `industry`, `market_cap`

**Use:** Determine appropriate WACC range from [sector-wacc.md](sector-wacc.md)

> **🇰🇷 KR override:** `get_financials_kr` does not return a US-style `sector` field. Infer the sector from the company's main business (반도체, 자동차, 2차전지, 바이오, 금융, 통신, 유틸리티, 소비재 …) and read its WACC range from [sector-wacc-kr.md](sector-wacc-kr.md) instead.

## Step 2: Calculate FCF Growth Rate

Calculate 5-year FCF CAGR from cash flow history.

**Cross-validate with:** `free_cash_flow_growth` (YoY), `revenue_growth`

**Growth rate selection:**
- Stable FCF history → Use CAGR with 10-20% haircut
- **Cap at 15%** (sustained higher growth is rare)

## Step 3: Estimate Discount Rate (WACC)

**Use the `sector` from company facts** to select the appropriate base WACC range from [sector-wacc.md](sector-wacc.md).

**Default assumptions (US):**
- Risk-free rate: 4%
- Equity risk premium: 5-6%
- Cost of debt: 5-6% pre-tax (~4% after-tax at 30% tax rate)

Calculate WACC using `debt_to_equity` for capital structure weights.

> **🇰🇷 KR override — use Korean market inputs:**
> - Risk-free rate: **~3%** (10Y Korean Treasury Bond / 국고채), not 4%
> - Equity risk premium: 5-7% (apply a Korea/governance "코리아 디스카운트" lean toward the high end for chaebol-affiliated or cross-held names)
> - Cost of debt: pre-tax market rate, **after-tax at ~22% corporate tax** (K-IFRS effective rate; marginal up to ~24-26% incl. local income surtax) — **not 30%**
> - Base WACC range from [sector-wacc-kr.md](sector-wacc-kr.md)

**Reasonableness check:** WACC should be 2-4% below `return_on_invested_capital` for value-creating companies.

**Sector adjustments:** Apply adjustment factors from [sector-wacc.md](sector-wacc.md) (US) or [sector-wacc-kr.md](sector-wacc-kr.md) (KR) based on company-specific characteristics.

## Step 4: Project Future Cash Flows

**Years 1-5:** Apply growth rate with 5% annual decay (multiply growth rate by 0.95, 0.90, 0.85, 0.80 for years 2-5). This reflects competitive dynamics.

**Terminal value:** Use Gordon Growth Model with 2.5% terminal growth (GDP proxy).

> **🇰🇷 KR override:** Use **~2.0%** terminal growth (Korea's lower potential GDP growth). The Step 6 sensitivity grid should center on this — vary terminal growth across **1.5% / 2.0% / 2.5%** instead of the US 2.0/2.5/3.0.

## Step 5: Calculate Present Value

Discount all FCFs → sum for Enterprise Value → subtract Net Debt → divide by `outstanding_shares` for fair value per share.

## Step 6: Sensitivity Analysis

Create 3×3 matrix: WACC (base ±1%) vs terminal growth (2.0%, 2.5%, 3.0%).

> **🇰🇷 KR override:** terminal-growth axis = **1.5% / 2.0% / 2.5%** (centered on Korea's ~2.0% terminal growth).

## Step 7: Validate Results

Before presenting, verify these sanity checks:

1. **EV comparison**: Calculated EV should be within 30% of reported `enterprise_value`
   - If off by >30%, revisit WACC or growth assumptions

2. **Terminal value ratio**: Terminal value should be 50-80% of total EV for mature companies
   - If >90%, growth rate may be too high
   - If <40%, near-term projections may be aggressive

3. **Per-share cross-check**: Compare to `free_cash_flow_per_share × 15-25` as rough sanity check

If validation fails, reconsider assumptions before presenting results.

## Step 8: Output Format

Present a structured summary including:
1. **Valuation Summary**: Current price vs. fair value, upside/downside percentage
2. **Key Inputs Table**: All assumptions with their sources
3. **Projected FCF Table**: 5-year projections with present values
4. **Sensitivity Matrix**: 3×3 grid varying WACC (±1%) and terminal growth (2.0%, 2.5%, 3.0%)
5. **Caveats**: Standard DCF limitations plus company-specific risks

> **🇰🇷 KR override:**
> - Report all values in **KRW** (intrinsic value per share, market cap). Run the per-share sanity check in KRW.
> - Sensitivity matrix terminal-growth axis: **1.5% / 2.0% / 2.5%**.
> - Add a short **"세후 실현수익률 주의"** caption: the DCF fair value is the company's intrinsic (pre-investor-tax) value. When an investor *realizes* the return, Korean securities transaction tax (증권거래세, as of 2026 ~0.20% of sale proceeds: KOSPI 0.05% 거래세 + 0.15% 농어촌특별세, KOSDAQ 0.20%) and dividend income tax (배당소득세 — 15.4% withholding for residents; ~22% or the tax-treaty rate for foreign holders) reduce the net realized return. These do **not** change the intrinsic value computed above — they are an investor-level adjustment on top of it.
