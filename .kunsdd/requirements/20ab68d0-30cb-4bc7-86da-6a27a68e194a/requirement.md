# Address documentation issues

## **Consistency issues found**

### **Tier 1 — tax-critical: the same ledger has 3 different column maps**

This is the big one and it's exactly where you don't want drift.

**EinAus column letters disagree across three reference docs:**

- `WORKBOOK_FIFO.md` ("confirmed from DEV, June 2026"): `B=FelipeCode`, `D=Anteile`, `E=Kurs R$`, `H=Action`, `I=Kurs €`, `AA=Anlagentyp`, `AC=Is latest`.
- `DATAFLOW.md` (§"Current post-June 2026"): `D=Kurs R$`, `G=Kurs €`, `H=Gebühren €`, `Z=Anlagentyp`.
- `SCHEMAS.md` (§"Key Formulas (Final — Post-Kurs Decoupling, June 2025)"): `E=Action`, `G=Kurs R$ (=D)`, `H=Kurs €`, `Z=Anlagentyp`, `AA=Is latest`.

The root cause: WORKBOOK\_FIFO documents a **FelipeCode column inserted at B**, shifting everything right by one. DATAFLOW and SCHEMAS were never updated for that insertion, so their letters are off-by-one from B onward. SCHEMAS even uses `B` as the eurbrl *date* lookup key (`VLOOKUP(B; eurbrl…)`) while WORKBOOK\_FIFO uses `C` (Datum) — because in SCHEMAS, `B` is still Datum.

**Payouts column order directly contradicts between docs:**

- `WORKBOOK_FIFO.md` + `BTG_WORKFLOW.md` (§5 paste instructions): `E=EurBrl`, `F=Brutto R$`, `G=Quellensteuern R$`, `K=Anlagentyp`, `P=Aktien`.
- `SCHEMAS.md` §Payouts + `B3_INCOME_EVENTS.md` §Payouts Schema Compatibility: `E=Brutto R$`, `F=Quellensteuern R$`, `G=EurBrl`, `K=empty spacer`, no `P`.

These can't both be right, and they drive copy/paste targets for income that feeds German filings. WORKBOOK\_FIFO ("from DEV") is presumably authoritative; SCHEMAS + B3\_INCOME\_EVENTS are stale.

**Staging tab** `BTG_Brokerage_Import` **also has two orderings:** `INGESTION.md` says `B=FelipeCode, C=Datum, D=Anteile, E=Kurs R$`; `SCHEMAS.md` §BTG\_Brokerage\_Import says `B=Datum, C=Anteile, D=Kurs R$, E=FelipeCode`. (Plus the INGESTION table reuses `G`/`H` twice — a broken table, same failure mode as the BTG\_WORKFLOW one you flagged.)

### **Tier 2 — stale status that contradicts a "done" elsewhere**

- **RX-007 is MITIGATED in its own entry but ACTIVE in the RISKS.md summary table** — and it's counted in "Active risks requiring action: 6" and excluded from "Mitigated risks: 5", while ROADMAP §Corporate Actions says it's mitigated. The detail/summary halves of the same file disagree.
- **The "phasing out" decision didn't propagate.** ROADMAP deprecates `btg-import`/`BTG_Brokerage_Import` and `b3-income`/`B3_Income_Events`, but `OPERATIONS.md` still presents Workflow D and Workflow F as current with no deprecation banner; `INDEX.md` still routes parser work to them; `INVENTORY.md` still lists B3 income as "Ready for Phase 1 implementation" (two states behind — ROADMAP says implemented *and* deprecating) and BTG\_Brokerage\_Import as "Production Use".
- `Transaction` **model Literal is stale in INGESTION.md.** It still shows `Literal["Buy","Sell","Dividend","Interest","ComeCotas","Withholding","Transfer"]`, but SCHEMAS and INVENTORY say it was narrowed June 2026 to `Buy|Sell|Adjust` with income types moved to `IncomeEvent`.

### **Tier 3 — removed things still referenced as live**

- **SCHEMAS.md has a duplicate** `IndexFundos` **section.** One (§indexFundos, \~L573) says "REMOVED — June 2026"; a second (§IndexFundos, \~L775) says "Authority Level: Authoritative" and "Expected future consolidation," i.e., describes it as live. (There are likewise two `Kurs` sections.)
- **AGENTS.md still lists** `IndexFundos` **in the Tier 0 list** ("legacy; targeted for consolidation"), though it's removed everywhere else.
- **AGENTS.md Codebase Layout points to** `.kilo/plans/` for archived plans; the real directory is `.kunsdd/`.

### **Tier 4 — counters, anchors, branding (low harm, high churn)**

- **Hand-maintained counts drift:** `HA_GUI_DESIGN.md` says "21 items" / "all existing 21 items"; HUMAN\_ACTIONS.md actually has 23 (9 open + 14 completed). B3\_INCOME\_EVENTS says "\~63 in-scope events" while the golden test asserts 62.
- **FIFO sheet count:** WORKBOOK\_FIFO header says "18 sheets," the numbered table lists 17 and omits `Zinsen 2025` and `xAssetTypes` (both used elsewhere in the same doc); ROADMAP says "18 (originally 19)."
- **Line-number anchors are already off:** INDEX §Section Quick-Reference points "Payouts → line 628"/"EinAus → 337"; actual is \~646/\~356. INDEX also undercounts WORKBOOK\_FIFO (says 362 lines; it's \~437).
- **Doc-header convention is itself drifting:** AGENTS.md §Documentation Policy mandates a Type/Prerequisites/Size/Updated block in every doc; only \~6 docs have it (INVENTORY, DATAFLOW, SCHEMAS, INGESTION, CRYPTO\_TAX\_REPORT, VORABPAUSCHALE). GLOSSARY, HISTORY, INDEX, WORKBOOK\_FIFO, BTG\_WORKFLOW, HUMAN\_ACTIONS, RISKS, ROADMAP, HA\_GUI\_DESIGN use an italic "*Last updated*" line or nothing.
- `DE_Vorabpauschale` **ownership is muddled:** SCHEMAS lists it as both an AssetIndex column and an AssetTypes column; the Vorabpauschale design settled on `xAssetTypes.DE_Vorabpauschale` as the sole switch, but the AssetIndex schema wasn't cleaned, and the design doc references `TeilfreistellungCategory`/`TeilfreistellungOverridePct` columns that aren't in the AssetTypes schema.
- **FIFO\_Gains protection status is inconsistent:** AGENTS.md §"Before You Write Code" #5 lists it as off-limits/authoritative, but AGENTS.md §Authority Levels says "FIFO\_Gains is… not part of Tier 0"; OPERATIONS lists it under "Do not modify."
- Mixed agent branding (Kun / DeepSeek GUI / Kilo) across AGENTS, OPERATIONS, HA\_GUI\_DESIGN; one typo "Wokflow" (OPERATIONS L528).

## **The patterns underneath (why it drifts)**

Almost everything above is one of four root causes:

1. **The same fact is materialized in multiple docs.** EinAus/Payouts column maps live in 3–4 places; the *correct* one is whichever was edited last. This is \~all of Tier 1.
2. **Detail and a hand-written summary of that detail coexist in one file** (RISKS summary table, HUMAN\_ACTIONS summary, sheet counts, event counts). The summary is never regenerated, so it lags the detail.
3. **Removals aren't propagated.** When a sheet/command/field is killed, there's no mechanism to find every mirror, so residue survives (duplicate IndexFundos, Tier 0 list, deprecated workflows).
4. **Anchors and counts reference volatile data** (line numbers, item counts) instead of stable identifiers.

## **How to tighten it for other agents**

Ranked by leverage. The first two would prevent most of what I found.

**1. One canonical owner per fact; everyone else links.**  
Pick a single home for each high-drift fact and forbid restating it:

- EinAus + Payouts + staging column maps → **only** in `WORKBOOK_FIFO.md` (and the staging-tab schemas in one place). SCHEMAS/DATAFLOW/INGESTION should *link to the anchor*, never re-tabulate. Right now WORKBOOK\_FIFO is the newest and best, which makes SCHEMAS' FIFO-workbook sections pure liability — they're the stale mirrors. Collapsing them removes the entire Tier 1 problem permanently.
- Add a short "Canonical fact owners" table to `INDEX.md` (or AGENTS.md): *"Column maps → WORKBOOK\_FIFO. Risk states → RISKS. Task statuses → ROADMAP/HUMAN\_ACTIONS."* Then an explicit agent rule: **when you change an owned fact, edit only the owner; when you find a mirror, replace it with a link.**

**2. Make summaries/counters generated, not hand-written.**  
You already have the machinery: `domain/ha_parser.py` round-trips HUMAN\_ACTIONS.md and *auto-generates* its summary table. Extend that exact pattern to RISKS.md (the RX-007 detail/summary split is precisely what a renderer prevents), and mark every summary block `<!-- generated: do not hand-edit -->`. Same for the FIFO sheet count and B3 event count — derive, don't transcribe.

**3. Add a** `docs-lint` **CI step.** You already run ruff/mypy/pytest in `.github/workflows/ci.yml`; a Markdown linter converts drift into a failing check, which is the only thing that reliably disciplines *other* agents. Cheap checks with high payoff:

- Every `docs/*.md` has the mandated header block (Type/Prerequisites/Size/Updated). Fixes the policy-not-followed problem and makes it self-enforcing.
- Every `RX-NNN` / `HA-NNN` cross-reference resolves to an existing entry.
- Summary-table row count == detail-section count; each summary row's status == the detail's Lifecycle/Status field (would have caught RX-007).
- A "removed terms" denylist: once `indexFundos`, `_Anlagentyp`, `Kurs_Import`, etc. are retired, any *non-historical* mention fails. (Allow them only under a `## History`/`(REMOVED)` heading.)
- Optional: grep for duplicate `## <SheetName>` headings within a file (would have caught the double IndexFundos/Kurs in SCHEMAS).

**4. Replace line-number anchors with heading slugs.** INDEX §Section Quick-Reference and SCHEMAS' internal "line N" pointers drift on every edit. Use `SCHEMAS.md#einaus` style links; they survive edits and CI can validate they resolve.

**5. A removal/deprecation propagation checklist** in AGENTS.md: "When you remove a sheet/command/model field: (a) grep the term across `docs/`, (b) update or link every hit, (c) leave it only under a REMOVED/History heading, (d) update INVENTORY + ROADMAP + INDEX routing." The indexFundos removal was \~90% propagated — a checklist closes the last 10% that became the contradictions above.

**6. "Updated + one-line changelog" enforcement.** Require bumping `Updated` and appending a changelog line on any content change; docs-lint flags a content diff with no `Updated` bump. HISTORY.md already does running changelogs well — make that the norm and let CI enforce it.

