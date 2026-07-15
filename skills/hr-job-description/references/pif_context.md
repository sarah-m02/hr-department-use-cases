# PIF Context Reference

Consulted by the `hr-job-description` skill for every JD it produces. Ensures Role
Overview, What We Offer, and division inference are grounded in PIF specifics —
never generic. Update this file rather than the skill body when PIF facts change.

Sources: PIF's own website (pif.gov.sa), Wikipedia, Saudipedia, vision2030.ai,
and public LinkedIn / careers-site language. All facts here are from public
sources; nothing that requires legal or comms review is included.

---

## 1. Fund identity

- **Name**: Public Investment Fund (PIF), Saudi Arabia
- **Established**: 1971; mandate expanded in 2015 to become the primary vehicle
  for Saudi Arabia's economic diversification
- **Mission** (public wording): "Invest in companies, real estate, and financial
  assets on behalf of the Kingdom of Saudi Arabia's government"
- **Scale** (2026): ~USD 900B AUM; ~1,000+ employees (headcount has grown
  rapidly — never quote a specific figure in a JD)
- **Headquarters**: PIF Tower, Riyadh
- **International offices**: London, New York City, San Francisco (plus growing
  presence via portfolio companies globally)

## 2. Governance (background — do NOT restate in JDs)

- Chairman: HRH Crown Prince Mohammed bin Salman
- Governor: HE Yasir Al-Rumayyan
- Structure: Board of Directors → Executive Management → committees
- Oversight: Council of Economic and Development Affairs

## 3. Investment pools (PIF's own six-pool structure)

Each pool is a natural home for investment roles. The MCQ that asks the user
which division a role sits in should use these names.

| # | Pool name (public) | What sits inside |
|---|---|---|
| 1 | **Saudi Equity Holdings** | Long-term domestic listed and unlisted holdings in national champions (Aramco, STC, Saudi National Bank, Riyad Bank, Almarai) |
| 2 | **Saudi Sector Development** | Newly created / repositioned companies anchoring priority industries (Lucid Motors Saudi, SEVEN, Saudi Coffee Company, Hayat Biotech, Saudi Arabia Railways) |
| 3 | **Saudi Real Estate and Infrastructure** | ROSHN, Diriyah Company, Red Sea Global, Saudi Real Estate Refinance Company |
| 4 | **Saudi Giga-Projects** | NEOM, Red Sea, Qiddiya, Diriyah, other transformational programmes |
| 5 | **International Strategic Investments** | Controlling / anchor stakes in foreign companies (Newcastle United, Heathrow, LIV Golf platform, etc.) |
| 6 | **International Diversified Investments** | Listed equities, fixed income, hedge funds, alternative assets |
| 7 | **Treasury Portfolio** (non-investment) | Custodian of the fund's liquidity — treasury and financial-markets execution |

## 4. Horizontal / functional units (where non-investment roles sit)

- **Strategy and Insights**
- **Finance**
- **Risk and Compliance** (three-lines-of-defence model)
- **Legal**
- **Human Capital**
- **Corporate Communications**
- **ESG and Sustainability**
- **Capital Markets Programme** (debt / sukuk issuance)
- **Local Content and Industrial Procurement**
- **Internal Audit**
- **Technology** (including PIF's HUMAIN AI initiative)
- **Portfolio Management** (oversight of portfolio companies)
- **PIF Academy** (internal training and development)

## 5. Division-inference hints (used by the MCQ)

Given a role title, the skill should propose the two-to-three most likely
divisions in the MCQ, plus "Other divisions." **No option gets a "Recommended"
label** — ordering is signal enough. The user knows their own role.

| Role keyword contains… | Top-guess division | Second guess | Third guess |
|---|---|---|---|
| "Real Estate," "Property," "Infrastructure," "Urban" | Saudi Real Estate and Infrastructure | Saudi Giga-Projects | Saudi Sector Development |
| "Investment Analyst," "Portfolio Manager," "PE," "M&A" (no geography cue) | Saudi Sector Development | Saudi Equity Holdings | International Strategic Investments |
| "Giga-Project," "NEOM," "Red Sea," "Qiddiya," "Diriyah" | Saudi Giga-Projects | Saudi Real Estate and Infrastructure | Saudi Sector Development |
| "Global," "International," "Foreign" (investment role) | International Strategic Investments | International Diversified Investments | Saudi Sector Development |
| "Public Markets," "Listed," "Public Equities" | Saudi Equity Holdings | International Diversified Investments | Treasury Portfolio |
| "Treasury," "Liquidity," "FX" | Treasury Portfolio | Finance | Capital Markets Programme |
| "Risk," "Compliance," "Audit" | Risk and Compliance | Internal Audit | Legal |
| "Legal," "Counsel," "Regulatory" | Legal | Risk and Compliance | Corporate Communications |
| "HR," "Talent," "People," "Recruit," "Reward" | Human Capital | PIF Academy | Strategy and Insights |
| "Comms," "PR," "Brand," "Marketing" | Corporate Communications | Strategy and Insights | Human Capital |
| "Technology," "Engineering," "Data," "AI," "Product" | Technology | Strategy and Insights | Portfolio Management |
| "ESG," "Sustainability," "Green" | ESG and Sustainability | Strategy and Insights | Risk and Compliance |
| "Strategy," "Insight," "Research," "Planning" | Strategy and Insights | Portfolio Management | Corporate Communications |
| "Portfolio Company," "Value Creation," "Ops" | Portfolio Management | Strategy and Insights | Saudi Sector Development |
| "Finance," "Controller," "FP&A," "Accounting" | Finance | Treasury Portfolio | Risk and Compliance |
| "Debt," "Sukuk," "Capital Markets," "IR" | Capital Markets Programme | Finance | Treasury Portfolio |
| "Procurement," "Local Content," "Supply Chain" | Local Content and Industrial Procurement | Finance | Strategy and Insights |

If none of the keyword rules fire, present the four most common pools /
functions (Saudi Sector Development · Corporate Communications · Human Capital ·
Strategy and Insights) plus "Other divisions." Do not guess wildly.

## 6. Culture and EVP (tone to use in Role Overview and What We Offer)

**Tone words PIF uses about itself** (drawn from public LinkedIn and careers
copy — safe to echo):

- "High-performance"
- "Bold thinking"
- "Purposeful challenge"
- "Rigour and pace"
- "Long-term value creation"
- "Shaping the economic transformation of Saudi Arabia" (generic enough not
  to trip the Vision 2030 legal-review rule)
- "Global mandate, national mission"
- "Multicultural, meritocratic"

**Programs and offerings that are safe to mention:**

- PIF Academy (internal training and development)
- Graduate Development Program (12-month rotation for early-career hires)
- Portfolio Management Development Program
- International exposure (offices in London, New York, San Francisco)
- Portfolio-company secondments

**Do NOT invent or mention:**

- Specific comp figures or ranges
- Bonus structure
- Vacation days
- Housing / education allowance amounts
- Any Vision 2030 target percentages or dates
- Named national programs (Saudization / Nitaqat / etc.)
- Named individuals (governor, chair) in the JD body

## 7. Talent profile PIF hires

Useful as a mental model for calibrating must-have vs. nice-to-have:

- **Investment roles**: mix of international top-tier PE / IB / SWF experience
  and Saudi nationals returning from international institutions. CFA common at
  Analyst / Senior Analyst; MBA common at Associate+; management-consulting
  background welcome
- **Corporate function roles**: Big-4 or top-tier consulting background common;
  bilingual capability an advantage but not mandatory below MD level
- **Technology roles**: hyperscaler / big-tech / top-tier SaaS backgrounds;
  research or PhD common for HUMAIN / AI roles
- **Giga-project / real estate**: infrastructure funds, top-tier developers,
  megaproject delivery experience

## 8. Language, location, and defaults

- **Location default**: Riyadh, on-site, unless the trigger specifies otherwise
- **Language of the JD**: English. **Do not mention Arabic** as required,
  preferred, or optional — Arabic handling is under separate testing and the
  JD must stay silent on it for now
- **Reports-to**: infer where possible from level (Analyst → Senior Analyst /
  VP; VP → MD / Head of Pool; MD → Governor's Office / Investment Committee)

## 9. Per-division "flavor" snippets

Short, safe-to-echo language for the Role Overview when we know the division.
These are starting points — the skill should paraphrase, not paste verbatim.

- **Saudi Equity Holdings**: "long-horizon steward of Saudi Arabia's most
  strategic listed and unlisted holdings"
- **Saudi Sector Development**: "building the companies that will anchor
  entirely new sectors of the Saudi economy"
- **Saudi Real Estate and Infrastructure**: "delivering the physical backbone
  of the Kingdom's next chapter — housing, urban platforms, and infrastructure
  at national scale"
- **Saudi Giga-Projects**: "transformational programmes that are redrawing the
  map of the Kingdom"
- **International Strategic Investments**: "anchor stakes in globally strategic
  businesses that connect PIF to the world's most important growth themes"
- **International Diversified Investments**: "institutional portfolio
  management across public and alternative markets, at sovereign scale"
- **Treasury Portfolio**: "custodian of PIF's liquidity, executing treasury
  and financial-markets activity at sovereign scale"
- **Technology**: "PIF's Technology function is building the digital and AI
  foundations of the fund and its portfolio, including the HUMAIN initiative"
- **Human Capital**: "shaping the workforce that will deliver PIF's mandate,
  from Analyst intake through executive succession"
- **Strategy and Insights**: "the internal brain trust setting direction and
  stress-testing PIF's biggest capital calls"
- **Corporate Communications**: "the voice of PIF to Saudi society, portfolio
  companies, and the international investor community"
- **Risk and Compliance**: "second-line assurance across every capital
  deployment PIF makes — from Analyst-authored memos to giga-project boards"
- **Legal**: "counsel across the full spectrum of PIF activity, from domestic
  M&A to sovereign-scale international transactions"
- **Finance**: "the fund's financial spine — reporting, planning, and control
  at sovereign scale"
- **ESG and Sustainability**: "embedding ESG into every capital decision and
  portfolio relationship PIF manages"
- **Capital Markets Programme**: "PIF's issuance platform — debt, sukuk, and
  investor relations for one of the world's most-watched sovereign issuers"
- **Portfolio Management**: "the interface between PIF and its portfolio
  companies — value creation, governance, and performance oversight"
- **Local Content and Industrial Procurement**: "shifting PIF's spend and its
  portfolio companies' spend into the Saudi supply chain, at scale"
- **Internal Audit**: "third-line assurance covering PIF's control environment
  end to end"
- **PIF Academy**: "PIF's internal training engine — from Graduate Development
  Program cohorts to executive succession programmes"

## 10. Hard rules the skill MUST obey when consulting this file

1. Never emit any of the "Do NOT invent or mention" items in §6.
2. Never name a specific person by name (governor, chair, etc.) in the JD body.
3. Never quote a specific AUM figure, headcount figure, or portfolio company
   name unless the role is explicitly attached to that portfolio company.
4. Never assert Arabic as a required, preferred, or optional language — this
   is a testing gap and the JD must stay silent on it.
5. Never label any MCQ option as "Recommended" — ordering is signal enough.
6. When in doubt about a fact, drop it rather than guess. Every claim in a JD
   should be defensible against a hiring-manager review.
