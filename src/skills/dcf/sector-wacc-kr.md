# Sector WACC Adjustments — Korea (KOSPI/KOSDAQ)

Use these as starting points for Korean listings, then adjust for company-specific factors.
These ranges already assume Korean market inputs — a ~3% risk-free rate (10Y 국고채) and a
5-7% equity risk premium — so do not re-apply a US risk-free rate on top.

## Determining Company Sector

`get_financials_kr` does not return a standardized `sector` field. Infer the sector from the
company's main business (line of business, KRX industry classification) and match it below.

## WACC by Sector

| Sector (KR) | Typical WACC Range | Notes |
|-------------|-------------------|-------|
| 반도체 / Semiconductors | 8-11% | High cyclicality; capex-heavy (삼성전자, SK하이닉스) |
| IT·인터넷·소프트웨어 | 9-12% | Growth-stage; assess maturity (네이버, 카카오) |
| 자동차·부품 / Autos | 8-10% | Cyclical, FX-sensitive exports (현대차, 기아) |
| 2차전지 / Batteries | 9-12% | Capex-heavy growth, demand uncertainty (LG에너지솔루션, 삼성SDI) |
| 바이오·제약 / Bio·Pharma | 10-13% | Pipeline and regulatory risk; high for pre-profit |
| 금융 / Financials | 8-10% | Leverage inherent to model (은행·증권·보험) |
| 통신 / Telecom | 7-9% | Defensive, regulated, dividend-heavy (SKT, KT) |
| 유틸리티 / Utilities | 6-8% | Regulated tariffs; rate-sensitive (한국전력) |
| 화학·소재 / Chemicals·Materials | 8-10% | Cyclical, commodity-linked |
| 철강·조선 / Steel·Shipbuilding | 9-11% | Deep cyclicality, global demand exposure |
| 소비재·유통 / Consumer·Retail | 8-10% | Domestic demand; staples lower, discretionary higher |
| 건설 / Construction | 9-11% | Project and real-estate cycle exposure |
| 지주사 / Holding companies | 9-12% | Apply holding-company / 코리아 디스카운트 (see below) |

## Adjustment Factors

Add to base WACC:
- **High debt (D/E > 1.5)**: +1-2%
- **Small cap (< ₩1조 / ~$700M market cap)**: +1-2%
- **Governance / 코리아 디스카운트** (chaebol-affiliated, opaque related-party dealings): +0.5-1.5%
- **Circular shareholding / 순환출자 exposure**: +0.5-1%
- **Holding-company structure (지주사, double-counting risk)**: +1-2%
- **Concentrated customer or single-buyer dependence**: +0.5-1%

Subtract from base WACC:
- **Market leader with durable moat**: -0.5-1%
- **Recurring / subscription revenue model**: -0.5-1%
- **Investment-grade credit rating (AA- and above, KR scale)**: -0.5%
- **Strong, consistent dividend payer with low payout volatility**: -0.5%

## Reasonableness Checks

- WACC should typically be 2-4% below ROIC for value-creating companies.
- If calculated WACC > ROIC, the company may be destroying value.
- Cross-check against KOSPI sector peers where available.
- Cost of debt is taken after-tax at a **~22% corporate tax rate** (K-IFRS effective; not the US 30%).
