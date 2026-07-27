# JATF Market Radar + Research Engine v1

## Objective

Extend the proven realtime quote pipeline into a two-layer opportunity-discovery system without turning the Spreadsheet into a permanent raw-data warehouse.

The system must answer four questions before each regular meeting:

1. Where is capital concentrating now?
2. Which themes are strengthening, confirming, weakening or reversing?
3. Which non-held leaders deserve research attention?
4. Has any active holding or research candidate materially changed state?

Radar scores prioritize attention. They are not automatic investment recommendations.

## Architecture

### Layer 1 — Market Data

Keep `Live Market Data` as the visible rolling 5–7 session store. GitHub should retain or reconstruct enough history to calculate:

- 1D return
- 3D return
- 5D return
- 10D return
- 20D return
- volume ratio versus 20D average
- distance from 5D and 20D high
- relative strength versus market benchmark
- data freshness and coverage status

### Layer 2 — Radar Signals

Produce one current-state record per ticker and one aggregate record per theme/sub-sector:

- market
- ticker
- name
- theme
- sub-sector
- universe origin
- latest source session
- generated timestamp
- 1D / 3D / 5D / 10D / 20D returns
- trend score
- relative-strength score
- volume/flow score
- breadth contribution
- risk penalty
- composite radar score
- signal state: New / Strengthening / Confirming / Weakening / Reversing
- research trigger flag and reason

### Layer 3 — Research Pipeline

Only promoted radar signals enter durable research.

Required workflow:

`New -> Screening -> Deep Dive -> Committee Ready -> Promoted / Hold / Rejected / Archived`

Every active research record must distinguish:

- underlying company/theme thesis
- instrument suitability
- current evidence
- missing evidence
- primary sources required
- thesis break
- next verification
- stale-after date
- committee destination

## Spreadsheet Responsibilities

### `02 - Market Radar`

Current active radar state plus a concise material-change log. Do not append a full essay for every theme every day.

### `08 - Theme Ranking`

Machine-generated current theme/sub-sector ranking. One current row per active theme or sub-sector.

### `03 - Research Pipeline`

Durable research state and research questions. One active record per research subject.

### `13 - Opportunity Queue`

Only committee-near candidates. It must not duplicate every Watchlist or Research Pipeline name.

### `04 - Decision Queue`

Executable plans only: trigger, size, funding source, thesis break, deadline and committee recommendation.

## Governance Gates

1. Same-day freshness gate by market.
2. Portfolio and Watchlist coverage gate.
3. Minimum-history gate before multi-day scoring.
4. No research promotion from a single-day price move alone unless a verified catalyst exists.
5. One active radar state per ticker/theme.
6. One active research record per subject.
7. Underlying-first / instrument-second analysis.
8. Scores rank attention; committee evidence determines action.
9. Every signal must carry its source session and generated timestamp.
10. Re-running the workflow must update current state without uncontrolled duplication.

## Delivery Phases

### Phase A — Data Foundation

- Persist at least 20 completed sessions outside the visible Spreadsheet layer, or retain sufficient bounded history for calculations.
- Compute ticker-level returns, volume and benchmark-relative metrics.
- Add freshness, missing-history and coverage validation.
- Upload a bounded ticker-level Radar Signals payload.

### Phase B — Theme Aggregation

- Read ticker-to-theme and sub-sector mapping from the operating database.
- Aggregate breadth, leadership and persistence.
- Populate `08 - Theme Ranking`.
- Identify leaders, participants, laggards and potential funding sources.

### Phase C — Research Triggers

- Create deterministic trigger rules.
- Update `02 - Market Radar` current-state rows.
- Flag candidates for Research Pipeline review without automatically creating investment conclusions.
- Produce meeting-ready change metadata.

### Phase D — Meeting Integration

For each US or HK meeting:

1. Refresh GitHub market data.
2. Validate timestamp, market session and coverage.
3. Read current Theme Ranking and material signal changes.
4. Review active research triggers and unresolved candidates.
5. Review portfolio holdings.
6. Move only committee-ready opportunities into Decision Queue.

## Initial Deterministic Triggers

A candidate may be flagged when one or more of the following occurs:

- Composite score enters the top decile and both 3D and 5D relative strength confirm.
- Theme breadth improves for two consecutive completed sessions.
- Volume ratio exceeds 1.5 while relative strength is positive and a catalyst is verified.
- An existing active theme changes state by two levels.
- A portfolio holding falls into the bottom decile of its strict peer group.
- A leveraged instrument materially diverges from its underlying after timing, FX, NAV and path dependency are considered.

No single trigger is an automatic buy or sell instruction.

## v1 Non-Goals

- No autonomous trading.
- No opaque AI-generated score.
- No full news-ingestion engine before price/flow calculations are stable.
- No permanent raw-history warehouse inside Google Sheets.
- No promotion based only on narrative popularity.

## Acceptance Criteria

- US and HK radar refresh through GitHub.
- Every result includes source session and generated timestamp.
- Rankings are reproducible from stored inputs.
- Missing or stale active holdings fail the workflow.
- Repeated runs update current-state rows without duplicates.
- A regular meeting can identify top strengthening themes, emerging non-held leaders, weakening themes and research triggers in under five minutes.
