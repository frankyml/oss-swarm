# Skill: Marketplaces IOP Domain Memory

> Load this skill when working on ANY task involving the Inditex Marketplaces IOP ecosystem. Provides instant domain context without reading the full vault.

## When to Use

- Any swarm agent task involving marketplaces (Zalando, ASOS, Trendyol, Myntra, Musinsa, etc.)
- Planning, spec writing, code review, or implementation for Marketplaces teams
- When you need to understand IOP architecture, team ownership, or marketplace-specific behavior
- When building or reviewing connectors, publication flows, stock/order logic

## Source

Distilled from `https://github.com/inditex/doc-mkpiopvault` (37 files). Last sync: 2026-05-13.

For the full repo-versioned memory with update protocol, see: `doc-mkpagents/memory/`

---

# ECOSYSTEM OVERVIEW

## Scale (FY2025)

+48M orders | +24M customers | +6M publications | +370M events | 6 teams | ~60 people | 13 active marketplaces | 25 countries | 139 stores

## Active Marketplaces

| Marketplace | Model | Brands | Region | Stack |
|---|---|---|---|---|
| Zalando | FBM (STR/BSK/PB/OY) + FBI (MD) | 5 | Europe | IOP Full |
| ASOS | FBM (AFS Partner) | BSK, PB, STR, OY | EU/USA | IOP Full |
| Trendyol | FBI FULL (Local + International) | MD, BSK, PB, STR, OY, Lefties | Turkey/ME | Hybrid→IOP |
| Cenomi | FBI FULL (Trendyol Int API) | MD, BSK, PB, STR, OY | Saudi Arabia | Hybrid→IOP |
| Tmall | FBI FULL | MD, ZH, ZARA, STR, PB, OY, BSK | China | Legacy |
| Myntra | FBM (Unicommerce OMS) | BSK | India | IOP Full |
| SSG | FBI CLASSIC (MPS legacy) | MD | South Korea | Legacy |
| Musinsa | FBI CLASSIC | MD | South Korea | Legacy |
| Douyin | FBI CLASSIC (E-Label flow) | ZARA | China | Legacy |
| JD | FBI CLASSIC | BSK (opening Q2 2026) | China | Legacy |
| About You | WHOLESALE (paused) | BSK | Europe | Manual/Excel |

## Platform Architecture

```
PACMAN (grouping) → PRPX (adapter) → PRCO (state) → COMCON (dispatch) → Marketplace API
                                                                             ↓
Order Core ← ExternalOrderModifiedEvent ← Marketplace → WCS (via MQ)
                                                             ↓
Stock Core ← PUBLICATION_STATUS + CHANGE_GROUP triggers → Snowflake
```

## Three Coexisting Stacks

1. **Legacy (MPS)** — DB2, run-mode only, target shutdown end FY2026
2. **Hybrid** — IOP product + MPS orders (Trendyol, Cenomi in migration)
3. **IOP FULL** — Target state (Zalando, ASOS, Myntra achieved)

## Key Products

| Product | Role | Status |
|---|---|---|
| IOP (Sales+Stock+Product) | Core platform | Active, expanding |
| SMARTxPACMAN | Replaces SMART+GEMA inside PACMAN | Rollout Apr-Jul 2026 |
| MIND | Data integrity monitoring | Active |
| MKP Toolbox | Operational gap filler | Active |
| GEMA | Publication management UI | Sunset Q2-Q3 2026 |
| SMART | Legacy operational UI | Active for non-migrated |
| MPS | Legacy monolith | Run-mode, shutdown FY2026 |
| PUMBA | Groovy script execution | Active (eCommerce-owned) |

## Key Definitions

- **FBM** (Fulfilled by Marketplace): Marketplace handles logistics
- **FBI** (Fulfilled by Inditex): Inditex ships from own warehouses
- **SOD** (Sell-on-Delivery): Revenue recognized on delivery (all FBI except Zalando)
- **CIR**: Customer-Initiated Return
- **RTO**: Return-to-Origin (courier return)

---

# CURRENT CONTEXT

## Current Focus (as of May 2026)

- **SMARTxPACMAN rollout** (Apr-Jul 2026): GEMA deactivation per brand/marketplace
- **Trendyol IOP migration**: MD/BSK/PB/STR done, OY Q2 2026
- **New openings**: Lefties on Trendyol (done), JD BSK (Q2), ASOS Ireland (Q2), Zalando Luxembourg (Q2)
- **MPS shutdown goal**: Full DB2 decommission by end FY2026
- **Zalando Image Updater**: Story 1 foundation implemented, Stories 2-5 pending

## In-Flight Decisions

- SSG Go/No Go still pending
- AfterSales ownership scope not resolved (NOT owned by Marketplaces Sales)
- MD on Zalando closure in progress

## Open Risks

- Over-selling from stock recalculation lag (FBM model)
- Manual refunds on Zalando/Trendyol are recurring, not edge cases
- Musinsa contractual exclusivity negotiations ongoing
- About You strategically paused (wholesale only, no API)

---

# MARKETPLACE DETAILS

## Zalando
- **Model**: FBM (STR/BSK/PB/OY) + FBI (MD). Manual refunds supported.
- **KPI**: +20% sales FY2026. Luxembourg opening Q2. Switzerland NO-EAN Q3.
- MD closure in progress.

## ASOS
- **Model**: FBM (AFS Partner). OAuth2. 3 warehouses (UK, DE, USA). Max 4 images (2116x2700px grey bg).
- No activate/deactivate API. ~1 day manual approval. GEMA as mgmt tool.
- **KPI**: +20% sales FY2026. Ireland Q2, Poland Q3/Q4.

## Trendyol
- **Model**: FBI FULL — Local (trl) + International (tri), separate seller IDs.
- Package-centric order model. GMT+3 timestamps (except createdDate=GMT).
- Migrating to getShipmentPackagesStream. 5%/day penalty for delivery delay.
- Lefties opened Apr 2026. OY migration Q2 2026.

## Cenomi
- FBI FULL via Trendyol International API. Saudi Arabia. Technically identical to Trendyol Int.

## Tmall
- FBI FULL. Alibaba ecosystem. Full brand portfolio. Legacy/MPS stack.

## Myntra
- FBM via Unicommerce OMS. BSK only. India. IOP Full.
- Dual integration (Myntra API for product, Unicommerce for orders/returns).
- No test env for Myntra API. Rate limit 50 rps. 4-5h listing lag.

## Musinsa
- FBI CLASSIC. MD only. South Korea. Legacy.
- REST API (bizest.musinsa.com), form-data + XML responses, 25 req/s.
- 3-day shipping SLA. 35% return ratio. 5%/day delay penalty, 10% OOS.

## SSG
- FBI CLASSIC (MPS legacy). MD only. South Korea. Go/No Go pending. Oracle DB.

## Douyin
- FBI CLASSIC (E-Label flow for PIPL compliance). ZARA only. China/ByteDance.

## JD
- FBI CLASSIC. BSK opening Q2 2026. China. JD SDK (Java) + REST API.
- Only PRO env (2 stores: test + official). 300K req/day limit. Migrating to API V2.

## About You
- WHOLESALE (paused). BSK. No API, weekly Excel. Strategically paused.

---

# PRODUCTS & SYSTEMS

## IOP Sales (Order Core + Return Core)
- Receives `ExternalOrderModifiedEvent` from connectors → WCS via MQ (`MARKETPLACE_ORDER_REQUEST`)
- Order ID: `{mkp}_{country}_{brand}_{code}`
- Returns: CIR (via `reversePickupCode`) + RTO (via `shipmentCode`)
- Jira: MPSSALES

## IOP Product (PRPX → PRCO → COMCON)
- PRPX: Inditex model → IOP model | PRCO: publication state source of truth | COMCON: downstream dispatch
- Jira: MPPRODUCT

## IOP Stock (MPSTCORE)
- Triggers: PUBLICATION_STATUS, CHANGE_GROUP, order, return
- FBM: marketplace reserves | FBI: IOP pushes
- Jira: MPSTCORE

## SMARTxPACMAN
- Replaces SMART+GEMA. Rollout Apr-Jul 2026. KPI: -90% onboarding time.
- GEMA deactivation: BSK Myntra (Apr 8) → BSK Trendyol+Cenomi (Apr 14) → PB Trendyol+Cenomi (Apr 21) → Zalando/ASOS (Q2-Q3)
- Jira: MPSMART

## MIND
- Data integrity monitoring. KPI: +70% error reduction FY2026.

## GitHub Repo Patterns
- Sales: `mic-marketplaces{mkp}connector` | Stock: `mic-mkpstock{mkp}connector`
- SMART: `mic-mkpsmart*` | Legacy: `app-mps*` | Libs: `lib-*`

---

# PEOPLE & TEAMS

## System Owners

| System | Owners |
|---|---|
| PACMAN | Adrian Otero, Belén Portela, Mar Rodríguez |
| PRCO/COMCON | Samuel Serrano, Dzmitryi Petrakou |
| WCS | Estela Souto, Begoña Testa, Clara Fernández, Antonio Lorenzo |
| GEMA | Joaquín Giménez, Estela Souto, Begoña Testa |
| MPS | Estela Souto, Begoña Testa, Doramas Ocampo |
| IOP Sales | Doramas Ocampo, Juan Trenado, Mariano Marasco |
| MPSTCORE | Joaquín Giménez, Mariano Marasco, Miguel Fernández |
| MPSMART | Adrian Otero, Belén Portela, Mar Rodríguez, Guillermo Rivero |

- Engineering Manager: Fernando Muñoz (all Marketplaces development)
- 6 teams, ~60 people

## Providers (FY2026)
PARADIGMA (IOP Sales & Stock, SMARTxPACMAN) | VIEWNEXT (MPS Legacy) | EPAM (Publication) | TECNOLOGIAS PLEXUS (Content) | MERLIN (Quality) | GFT (IOP SMART)

## Jira Streams
MPS (legacy) | MPSSALES (orders) | MPSTCORE (stock) | MPSMART (grouping) | MPPRODUCT (publication)

---

# FY2026 KPIs & ROADMAP

| KPI | Target |
|---|---|
| SMARTxPACMAN onboarding | -90% |
| Events visibility | 90% <10min |
| MIND error reduction | +70% |
| Zalando sales | +20% |
| ASOS sales | +20% |
| IOP DB2 shutdown | 100% |

## Openings: Lefties/Trendyol (done), JD BSK (Q2), ASOS Ireland (Q2), Zalando Luxembourg (Q2), Zalando CH NO-EAN (Q3), ASOS Poland (Q3/Q4)
## Closures: The Iconic (Nov 2025), CPD China (Q1 2026), Zalando MD (in progress)
## Growth FY2025: PB +55% | BSK +54% | STR +54% | OY +44% | MD +36% | Zara +4%

---

# CAPABILITIES (Cross-Cutting)

Six transversal capabilities span all marketplaces. Each has its own lifecycle, systems, and risks.

## Publication

Turns grouped product data into marketplace-visible catalog state.

- **Flow**: Product change → PRPX (adapter) → PRCO (state) → COMCON (dispatch) → Marketplace API
- **Systems**: PRCO, COMCON, PRPX, PACMAN, SMARTxPACMAN
- **Prerequisite**: Valid marketplace group (see Grouping)
- **Marketplace notes**: Zalando multicountry (`one-store-fits-all` vs `all-store`), Trendyol category/attribute mapping, TMALL first IOP FULL
- **Risks**: Wrong state per store/country, rollout drift, coupling with pricing/grouping/stock

## Grouping

Maps Inditex product structure (PACMAN groups) to marketplace-level groups per store. Strategic prerequisite for publication and stock.

- **Flow**: PACMAN base group → SMARTxPACMAN generates marketplace groups → `GroupChange` event → Stock Core (recalc) + Product Core (publication eligibility)
- **Systems**: PACMAN, SMARTxPACMAN, SMART (legacy), Product Core, Stock Core
- **Legacy→New**: SMART (manual, frequent "GROUPS ISSUES") → SMARTxPACMAN (automatic from PACMAN, eliminates inconsistencies). Brand-by-brand migration.
- **Risks**: Stale group state, campaign/store mismatch, twinned/sibling propagation gaps

## Orders

Full lifecycle from marketplace purchase to WCS fulfillment.

- **Flow**: Marketplace → connector → `ExternalOrderModifiedEvent` → Order Core → WCS via MQ (`MARKETPLACE_ORDER_REQUEST`)
- **Order ID format**: `{mkp}_{country}_{brand}_{code}` (e.g., `myn_IN_BSK_12345678`)
- **FBM vs FBI**: FBM = marketplace ships, IOP receives status. FBI = IOP pushes stock, WCS ships.
- **Status map**: PAID → SHIPPED → DELIVERED → CANCELLED (marketplace-specific mappings per connector)
- **Marketplace notes**: Zalando FBM+FBI split, Trendyol FBI via PUMBA, Myntra via Unicommerce OMS, ASOS still MPS legacy (IOP migration Q2 2026)
- **Risks**: Order not in WCS (MIND workaround), state divergence, MPS→IOP migration orphans

## Returns

Cross-cutting: marketplace flows, return-request lifecycle, refunds, WCS integration.

- **Types**: CIR (Customer-Initiated, uses `reversePickupCode`) + RTO (Return-to-Origin, uses `shipmentCode`)
- **Flow**: Marketplace return event → connector → IOP Return Core → `ExternalReturnModifiedEvent` → WCS → Stock Core (stock restored)
- **Manual refunds**: Recurring pattern (not edge case) when marketplace lacks return acknowledgement API, or RMA CLO without associated return
- **Marketplace notes**: Zalando FBI fulfillment-aware returns, Trendyol migrating return creation to internal order core, Myntra dual search (CIR+RTO, no pagination)
- **Risks**: Incorrect status mapping, manual refund divergence, RMA in WCS without IOP return (key MIND alert)

## Stock

Strongest transversal capability — connects publication, grouping, fulfillment, and integrity.

- **Triggers**: `PUBLICATION_STATUS` change, `CHANGE_GROUP` event, order created, return completed
- **FBM**: Marketplace manages reservation, IOP receives updates. **FBI**: IOP calculates and pushes stock.
- **Systems**: MPSTCORE (calculation+integrity), IOP Sales (notifications), PUMBA (reconciliation), Snowflake (traceability)
- **Risks**: Over-selling from recalculation lag (especially FBM), marketplace-specific reservation semantics, manual ops lack traceability

## Support Dashboards

Operational capability for detecting and resolving discrepancies between marketplace and Inditex systems.

- **Not passive**: Dashboards drive operational queues and workaround procedures
- **Discrepancy types**: Product state mismatch, order not in WCS, return without RMA, stock divergence, group inconsistency
- **Systems**: IOP, MPS (legacy), WCS, Snowflake, PUMBA, MIND (automated detection), MKP Toolbox (workaround execution)
- **Coverage**: TMALL, Douyin, Trendyol, Musinsa, WeChat, Cenomi, Zalando, Myntra — each has its own dashboard docs
- **Risks**: Manual triage for automatable categories, category drift as systems evolve

## Capability × Marketplace Matrix

| Marketplace | Publication | Grouping | Orders | Returns | Stock | Support |
|---|---|---|---|---|---|---|
| TMALL | Medium | High | — | Medium | High | High |
| Zalando | High | Medium | — | High | High | High |
| Trendyol | High | Medium | — | High | High | High |
| JD | High | Low | — | Medium | High | Medium |
| Musinsa | Medium | Low | — | Medium | Medium | High |
| Douyin | Medium | Low | — | Low | Low | High |
| WeChat | Low | Low | — | Low | Low | High |
| Cenomi | Medium | Low | — | Medium | Medium | High |

## System Clusters

- **Legacy/Support**: MPS, WCS, dashboards, workaround procedures
- **Sales**: IOP Sales, marketplace connectors, XML/MQ/REST flows
- **Stock**: MPSTCORE, stock integrity, recalculation, notification
- **Grouping**: PACMAN, SMARTxPACMAN, MPSMART, `GroupChange`
- **Publication**: PRCO, COMCON, rollout/release services, Janus/PIPE

---

# AGENT-SPECIFIC GUIDANCE

## For Researchers
- Vault path: `02_Marketplaces/{name}.md`, `03_Products/{name}.md`, `05_Capabilities/capability-{name}.md`
- Shell escaping with `gh api` + `base64 -d` fails in subagents — use main session
- Always cross-reference vault with Jira for current status
- Trendyol has dual variants (Local/Int) — always clarify which

## For Coders
- **Tech stack**: Java (Spring Boot) connectors, React (AMIGA Web) frontends, PUMBA (Groovy) scripts
- **Pitfalls**: Trendyol GMT+3 (except createdDate=GMT), Myntra no test env, JD only PRO env, Musinsa form-data+XML
- **Events**: `ExternalOrderModifiedEvent`, `PUBLICATION_STATUS`, `CHANGE_GROUP`
- **MQ**: `MARKETPLACE_ORDER_REQUEST` (to WCS)

## For Reviewers
- Stock changes need race condition analysis (over-selling risk)
- Same marketplace can have FBM + FBI brands (Zalando)
- Manual refunds are RECURRING (not edge cases) on Zalando + Trendyol
- No new DB2 dependencies. Respect rate limits. Document timezone assumptions.

## For Spec Writers
- Domain language: "Publication" (not listing), "Grouping" (not bundling), "Store" = brand+country instance
- Always state which stack: Legacy / Hybrid / IOP Full
- AfterSales NOT owned by Marketplaces Sales. PUMBA owned by eCommerce.
- Reference vault: `See: doc-mkpiopvault/02_Marketplaces/{name}.md`

## For QA Guardians
- Mandatory: No DB2 deps, rate limits respected, order ID format, timezone docs, FBM/FBI branching, manual refunds tested, GEMA timeline considered
- Fragile areas: Stock lag, Trendyol dual-variant, Myntra 4-5h lag, Hybrid coexistence, group change propagation
- Validation: MIND dashboards, Support dashboards, Snowflake reconciliation
