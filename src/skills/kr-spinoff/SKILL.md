---
name: kr-spinoff-analysis
description: Analyzes Korean corporate split events (물적분할·인적분할) and their impact on existing parent-company shareholders — holding-company discount, double-counting, loss of a core growth business, and 쪼개기 상장 (subsidiary IPO) dilution. Triggers when a 6-digit Korean ticker or Korean company name is mentioned together with 물적분할, 인적분할, 분할, 자회사 상장, 쪼개기 상장, spin-off, or "did the split hurt shareholders".
---

# Korean Split (물적분할 / 인적분할) Analysis Skill

The goal is **not** to list filings. It is to answer: *was this split good or bad for the
**existing parent-company shareholders**, and why?* Lead with that verdict.

## Background — two split types (decide which applies)

- **물적분할 (physical division)**: the parent spins a business unit into a **100%-owned new
  subsidiary**. The parent's shareholders own the subsidiary only *indirectly*. The classic risk
  is **쪼개기 상장** — the subsidiary later IPOs, raising capital by selling new shares to outside
  investors, which **dilutes** the parent's stake and exposes parent holders to the **holding-company
  discount** and **double-counting** (the same business valued twice, but the parent trades at a
  discount to the sum of parts). Generally **negative** for existing parent shareholders.
- **인적분할 (personal/horizontal division)**: shares of the new entity are distributed to existing
  shareholders **pro-rata**. Holders directly own both pieces, so it is broadly **value-neutral**
  at the moment of split (value can still move on subsequent re-rating).

The canonical cautionary case is **LG화학 → LG에너지솔루션** (2020 물적분할, 2022 IPO): LG화학
shareholders saw their indirect claim on the battery business diluted, and LG화학 re-rated to a
holding-company-like discount. Use this as the reference pattern.

## Workflow Checklist

```
KR Split Analysis Progress:
- [ ] Step 1: Resolve target and find the split event(s)
- [ ] Step 2: Classify the split (물적분할 vs 인적분할) and build the timeline
- [ ] Step 3: Map the post-split ownership / control structure
- [ ] Step 4: Assess the value impact on existing parent shareholders
- [ ] Step 5: (Optional) Quantify with financials
- [ ] Step 6: Present verdict + caveats
```

## Step 1: Find the Split Event(s)

Call `get_filings_kr` for the target ticker. Split disclosures are filed as **주요사항보고서**, so
use `filing_type: "material"`. Splits are often years in the past, so pass a wide window via
`start_date` (e.g. go back 5-10 years) — the default range is only ~1 year.

Scan `report_nm` for: `분할`, `물적분할`, `인적분할`, `회사분할`, `분할합병`, and the subsidiary
IPO (`증권신고서` / `발행공시`, `filing_type: "issuance"`, if a 쪼개기 상장 is suspected). Note the
receipt date (`rcept_dt`) and the new entity name.

If the user already named the event (e.g. "LG화학 물적분할"), confirm it in the filings rather than
searching blind, and supplement with `web_search` for board-resolution / approval dates if the
filing record is thin.

## Step 2: Classify and Build the Timeline

State explicitly whether it is **물적분할** or **인적분할** — this drives the entire verdict.
Build a short timeline: board resolution → shareholder meeting (분할 승인) → 분할기일 (effective
date) → subsidiary IPO date (if any).

## Step 3: Map the Post-Split Ownership Structure (supporting)

Use `get_large_holders_kr` (5%룰 대량보유) and `get_insider_trades_kr` (임원·주요주주) on the
**parent** to confirm the parent's stake in the new entity and the controlling-family / holding-company
chain above it. For broader group context (재벌 그룹 구조, 지주사 위치) use `web_search` — full group
mapping is out of scope; only pull what's needed to explain the value impact.

## Step 4: Assess the Value Impact on Existing Parent Shareholders

This is the core. For a **물적분할 + IPO** pattern, evaluate:
- **Dilution of indirect claim**: how much of the subsidiary did the parent give up at IPO?
- **Holding-company / 코리아 디스카운트**: does the parent now trade below sum-of-parts?
- **Double-counting**: is the crown-jewel business being valued in both the listed subsidiary and the parent?
- **Loss of the growth engine**: did the parent's best-growing segment leave the directly-held entity?
- **Use of IPO proceeds**: do they flow to the subsidiary (parent holders don't directly benefit) or the parent?

For **인적분할**, note it is broadly neutral at split, then look at any subsequent re-rating.

## Step 5: (Optional) Quantify with Financials

Use `get_financials_kr` to compare the parent's consolidated vs separate (연결 vs 별도) figures and,
where available, segment contribution before and after the split — to show how much earnings power
left the directly-held entity.

> DART account labels (`account_nm`) vary by company/year — match on substrings / `account_id`, never exact strings.

## Step 6: Output Format

Present:
1. **Verdict** (1-2 sentences): net good / neutral / bad for existing parent shareholders, and the single biggest reason.
2. **Event summary**: split type, timeline, entities involved.
3. **Ownership impact**: dilution and control-chain effect (with figures where available).
4. **Value impact**: discount / double-counting / lost-growth analysis from Step 4.
5. **Caveats**: 코리아 디스카운트 is partly structural; minority-shareholder protection rules have
   been tightening (e.g. 2022+ regulatory pushback on 쪼개기 상장) — note if relevant to timing.
