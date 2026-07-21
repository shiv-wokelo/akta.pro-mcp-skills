---
name: competitive-intelligence-tabs-dashboard
description: >
  Builds a tab-based HTML competitive intelligence dashboard for any company
  using Wokelo MCP for live news + enrichment data. Renders 4 tabs (Competitor
  Moves · Insights · All Signals · Logs) plus a Competitor Profiles link.

  ALWAYS USE for: "build a competitive intelligence dashboard for [company]",
  "create a CI dashboard for [company]", "competitor moves dashboard for
  [company]", "signal-based competitive intel for [company]", "akta.pro
  competitive dashboard for [company]".

  Differs from `competitor-intel-dashboard` (8-section sidebar layout). This
  skill uses TABS: Competitor Moves (default, mini sentiment charts with
  color-coded signal dots), Insights (Key Signals, Vulnerabilities, Considerations),
  All Signals (3 white-bg charts + curated feed), Logs (full raw news table).

  akta.pro palette (cream #f4f2ed, teal #2dd4a6), Playfair Display + DM Sans +
  DM Mono, Chart.js 4.4.0.

  Template: /mnt/skills/user/competitive-intelligence-tabs-dashboard/template.html
---

# Competitive Intelligence Tabs Dashboard

## Step 1 — Read The Reference Template

**ALWAYS do this first.** The file at `/mnt/skills/user/competitive-intelligence-tabs-dashboard/template.html` is the working Baseten dashboard (~200KB). It encodes 30+ iterations of design refinement, dot color logic, tab layouts, LOG_FULL hoisting fixes, all-signals white-background scoping, insights tab structure, modal wiring, mini-chart bucketing — none of which is worth rebuilding from scratch.

**Workflow:** copy template.html → pull fresh Wokelo data for the new target → replace the data arrays → keep the CSS + JS structure intact. Don't rewrite the DOM shape or the render functions.

---

## Defaults

| Parameter | Value |
|---|---|
| Time window | **90 days** back from today |
| Articles per company | **500** on news_signals |
| Competitors | **5** unless specified |
| Branding | AKTA.PRO \| BY WOKELO AI |
| Output path | `/mnt/user-data/outputs/[company_slug]_competitive_intelligence.html` |
| Fonts | Playfair Display (headers) · DM Sans (body) · DM Mono (labels) |
| Chart lib | Chart.js 4.4.0 via `https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js` |
| Logos | Clearbit `https://logo.clearbit.com/[domain]` with initials fallback |
| Favicons | Google `https://www.google.com/s2/favicons?domain=[publisher]&sz=32` |

---

## Design Tokens

```css
:root {
  --bg: #f4f2ed; --bg-card: #ffffff; --bg-dark: #0d0d0d;
  --acc: #2dd4a6; --acc-lt: rgba(45,212,166,0.08);
  --border: #e0ddd7; --border2: #c8c4bc;
  --t1: #0d0d0d; --t2: #444; --t3: #888; --t4: #aaa;
  --amber: #d4a017; --amber-lt: rgba(212,160,23,0.10);
  --red: #c0392b;
}
```

**MOVE_CATS palette** (for Competitor Moves category sub-blocks — the colored left rails on signal cards):
- `product` = `#2dd4a6` (teal) — Product Launches
- `gtm` = `#7a5c99` (purple) — GTM / Marketing
- `partnership` = `#2a5f8f` (blue) — Partnerships
- `hiring` = `#d4a017` (amber) — Hiring / Leadership
- `legal` = `#c0392b` (red) — Legal / Risk
- `capital` = `#e67e22` (orange) — Capital / M&A

**LOG_FULL category palette** (for mini-chart dots + heatmap + Logs table category chips):
- `capital` = `#d4a017` (amber) — Capital / M&A
- `product` = `#2dd4a6` (teal)
- `partnership` = `#2a5f8f` (blue)
- `legal` = `#c0392b` (red)
- `leadership` = `#7a5c99` (purple)
- `other` = `#95a5a6` (grey)

*Note the two split palettes*: MOVES uses 6 keys `product / gtm / partnership / hiring / legal / capital`. LOG_FULL uses `product / partnership / capital / legal / leadership / other`. The mini-chart dots on Moves cards color by LOG_FULL category (dominant per week), so the dot legend under the tab controls uses the LOG_FULL palette — not the MOVES palette. Both are shown in the template.

**Per-company palette** — target always teal, others rotate:
```javascript
const COMPANIES = ['[TARGET]','[C1]','[C2]','[C3]','[C4]','[C5]'];
const COLORS = {
  '[TARGET]':'#2dd4a6', '[C1]':'#2a5f8f', '[C2]':'#d4a017',
  '[C3]':'#c0392b',    '[C4]':'#7a5c99', '[C5]':'#e67e22'
};
const LOGO_DOMAIN = { '[TARGET]':'target-domain.com', ... };
```

---

## Execution — 5 Phases

### Phase 1 — Resolve Companies
- Resolve target slug via search
- Identify 5 competitors (from user list, or synthesize from same-market overlap)
- Resolve each competitor slug

### Phase 2 — Pull News (parallel)
For target + each competitor: `news_signals(company: slug, limit: 500, start_date: today-90d, end_date: today)`. Filter: keep articles where the company name appears in title or lede. Discard passing mentions.

### Phase 3 — Pull Enrichment (parallel)
For each: firmographics + headcount + funding + social_media. Pull website_traffic if the plan supports it.

### Phase 4 — Analytical Processing
For every kept article assign: `cat` (product/partnership/capital/legal/leadership/other) · `sent` (pos/neu/neg) · `imp` (hi/md) · `lens` (one-sentence "why this matters to target").

Compute the 18 data structures listed in "Required Data" below.

### Phase 5 — Build HTML
Copy template.html. Replace data arrays. Adjust textual references from "Baseten" to the new target name. Save. Present.

---

## Tab Structure (fixed IDs, do not rename)

- **`t3` Competitor Moves** — DEFAULT ACTIVE tab (first pill in nav row)
- **`t5` Insights** — 2nd pill
- **`t2` All Signals** — 3rd pill (has embedded 3-chart row + curated feed)
- **`t7` Logs** — 4th pill (full raw akta.pro log table)
- **`t4` Competitor Profiles** — NOT a pill; accessed via top-right `<button class="nav-link" data-tab="t4">Competitor Profiles →</button>`

Only one `.tab-content.active` at load (`t3`).

---

## Competitor Moves Tab (`t3`) — Detailed Structure

This is the flagship view. Below is the exact DOM + behavior contract.

### 1. Controls block (`.cm-controls`)

Three sub-blocks stacked:

**Timeline slider** — single teal thumb, 13 tick positions (one per WEEK bucket). Label above: `Timeline · Apr 22 → Jul 21` (updates as thumb moves). The `#ts-single` input is `<input type="range" min="0" max="12" step="1">` — dragging it changes `mvStart` which filters MOVES to items with `dt >= DATE_TICKS[mvStart]`.

**Type filter row** — `.fbar` with chip buttons: `All / Capital & M&A / Product / Partnership / GTM / Hiring / Legal`. Multi-select behavior — clicking `All` clears others; clicking a specific chip toggles into `selMvCats` Set.

**Player filter row** — same pattern, chips per company. Target company is highlighted teal when selected.

### 2. Signal-type dot legend (`.mv-dot-legend`)

Six inline color-swatch chips below controls, labeled `CHART DOTS`. Explains the colored dots on the mini charts:

```html
<div class="mv-dot-legend">
  <span class="mv-dot-lbl">Chart dots</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#d4a017"></span>Capital / M&A</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#2dd4a6"></span>Product</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#2a5f8f"></span>Partnership</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#c0392b"></span>Legal / Risk</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#7a5c99"></span>Leadership</span>
  <span class="mv-dot-item"><span class="mv-dot" style="background:#aaa"></span>Other</span>
</div>
```

Dot colors here match LOG_FULL cat keys (leadership vs hiring), NOT MOVE_CATS. Intentional — dots come from LOG_FULL bucketing, not from MOVES.

### 3. Grid `.moves-grid` — one column per player

CSS: `grid-template-columns: repeat(6, 1fr)` by default; JS rewrites this dynamically to `repeat(N, 1fr)` when player filter narrows the set.

Each column (`.mv-col`) is a card containing 3 stacked pieces:

#### (a) Header row `.mv-head`
- Company logo (48–56px, Clearbit + fallback badge)
- Company name (`.mv-co`, teal + bold if target — apply `.tgt` class)
- Role tag (`.mv-tag`) — `Target` or `Competitor`
- Total signal count in current time window — `.mv-total`: `12 signals in range`

#### (b) Mini sentiment chart `.mv-mini-chart`

- 78px tall canvas `#mvc-{i}` where `i` is column index (0..N-1)
- Chart.js `line` type with `fill:true, tension:0.4`
- `borderColor` = company brand color; `backgroundColor` = `COLORS[co] + '22'` (13% alpha wash)
- 13 data points, one per week from `SENTIMENT_7D[co]` — sentiment % positive values
- **Uniform thin dots** — `pointRadius:2.5` for every week that has ≥1 signal in LOG_FULL for that player; `pointRadius:0` for empty weeks
- **Dot color = dominant LOG_FULL cat that week** — bucket LOG_FULL entries into 13 weekly buckets per player, count per-category, pick argmax. Dot border: `#fff` 1px.
- Tooltip on hover: `74% positive · 8 signals · mostly Product`
- Both axes hidden (`display:false`); no grid lines. Pure sparkline.

#### (c) Signal sub-cards — one block per MOVE_CATS category

**This is the key structure.** Under each player column, iterate `MOVE_CATS` (6 categories):

```html
<div class="mv-cat" style="--rail:#2a5f8f">
  <div class="mv-cat-lbl">
    Partnerships
    <span style="color:#2a5f8f;margin-left:auto;font-weight:700;">3</span>
  </div>
  <div class="mv-item mv-clickable"
       data-moveco="Baseten"
       data-movecat="partnership"
       data-moveidx="0">
    <span class="dt">Jun 4 · HI</span>
    <span class="tt">NVIDIA cited Baseten as Blackwell inference customer (5x DeepSeek V4 cost cut)</span>
  </div>
  <!-- ... more signal sub-cards ... -->
</div>
```

Each **signal sub-card** (`.mv-item.mv-clickable`) has:
- A **colored left rail** matching the category (via `--rail` CSS variable set on parent `.mv-cat`, cascading to `border-left-color`)
- Date + impact label on top line (`Jun 4 · HI`)
- Move title/description below
- **Hover:** background fades white, `translateX(2px)`, subtle shadow — signals interactivity
- **Data attributes for click routing:** `data-moveco` (company), `data-movecat` (MOVE_CATS key), `data-moveidx` (index in `MOVES[co][catKey]`)

Empty categories render: `<div class="mv-item empty">no signals</div>` (italic grey, no border).

Category headers show the count of signals in that category (in the category color) right-aligned within the label.

### 4. Click → Source Modal (`.src-modal`)

Every `.mv-clickable` sub-card wires a click handler calling:

```javascript
openMoveModal(co, MOVES[co][catKey][idx], catKey);
```

`openMoveModal` looks up the underlying ARTICLES record via `findArticleForMove(co, move)`:

```javascript
function findArticleForMove(co, move){
  const arts = ARTICLES.filter(a => a.co === co);
  // First: same-day match
  const dayMatch = arts.find(a => a.dt === move.dt);
  if (dayMatch) return dayMatch;
  // Fallback: fuzzy title match on first 4 words
  const key = move.t.split(' ').slice(0,4).join(' ').toLowerCase();
  return arts.find(a => a.title.toLowerCase().includes(key));
}
```

Modal body renders:
- Category color left rail (4px)
- Header row: logo · company name · date · impact badge (`HIGH IMPACT` red / `MEDIUM` amber) · category chip in its color
- Move title — Playfair Display 19px
- Full body text from `art.body` (paragraph, 14px, line-height 1.65)
- **Target Lens box** — teal-bordered, `.nc-lens` class with `.nc-lbl` label "Baseten Lens" + `art.lens` text
- Source link — `Source: [Publisher] ↗` → opens `art.source.url` in new tab

If no ARTICLES record matches (returns undefined), fallback message: *"Underlying source article not curated — see the full Signal Log in the Logs tab for the raw article."*

Modal close: click backdrop → removes `.open` class. Backdrop has `backdrop-filter: blur(4px)`. Modal content: 1080px max-width, 90vh max-height.

### Recap — Competitor Moves DOM

```
#t3
├── .cm-controls (timeline slider + type filter + player filter)
├── .mv-dot-legend (6 color swatches, LOG_FULL palette)
└── .moves-grid
    ├── .mv-col (player 1)
    │   ├── .mv-head (logo + name + tag + total)
    │   ├── .mv-mini-chart (canvas #mvc-0, 78px, dots colored by dominant cat)
    │   └── .mv-cat × 6 (one per MOVE_CATS)
    │       ├── .mv-cat-lbl (label + count)
    │       └── .mv-item.mv-clickable × N  ← CLICK opens openMoveModal
    ├── .mv-col (player 2) ...
    └── ... (up to 6 columns)
```

---

## Insights Tab (`t5`)

Five stacked blocks (no chart at top — the pulse chart was removed):

1. **Key Signals** — `.theme-grid` with `.tc.tc-click` cards. Each: title + description + 3-leader rows with BLACK company text + TEAL progress bars + `→ View N underlying sources` CTA. Click → source drill modal. Company names are NOT color-coded per player (they're plain black). Bars are always teal.
2. **Where the Market is Cracking Open** — `#vuln-body` table. Competitor rows only. Columns: `Competitor | Vulnerability Signal | [Target] Opening`.
3. **What [Target] Should Consider** — `#consider-body` action table. First column is a bordered card with teal-left-accent showing `Action N` label + action text. Second column is triggers with logos. Third column is rationale. Row is clickable → source modal.
4. **What Happens Next** — `.watch-grid` — 3-column grid of `.wn-card` per competitor. Each: logo + name + current sentiment line + strategic summary + watchlist items.
5. **90-Day Watchlist** — `.watchlist-card` (LIGHT bg, not dark) with 5 numbered high-stakes questions. Each: teal number · bold question · muted description · tag row (company chip · HIGH/MEDIUM · theme tag · publisher).

---

## All Signals Tab (`t2`)

Charts at top, curated feed below. **Chart cards use WHITE background** (`#t2 .chart-card { background: var(--bg-card); }` scoped override), unlike default dark chart cards.

1. **3-chart row** — grid `repeat(3, 1fr)`, gap 14px:
   - Signal Activity Over Time (canvas `#c1-activity-c`) — stacked area, one series per player
   - Sentiment Trend (canvas `#c1-sentiment-c`) — multi-line 7-day rolling % positive per player
   - Signal Heatmap (`#c1-heat-c`) — HTML `.heat-table` with category × company matrix, cell opacity ∝ signal count. Zero cells: `background:#f0ede8; color:#999;` (light, not black).
   - **No grid lines on charts** — `grid:{display:false}` on both x and y.

2. **Curated feed** — filter chip row + search box + `#ng` news-grid. Renders ARTICLES with the required `lens` field.

---

## Logs Tab (`t7`)

Full 200+ row akta.pro log with publisher favicons.

- Header: **"Full log <em>from akta.pro</em>"**
- Multi-select Player filter chip row (`#t7 .chip[data-t6co]`)
- Multi-select Category filter chip row (`#t7 .chip[data-t6cat]`)
- Search input (`#t6-search`)
- `.log-table` with columns: Date · Player · Category · [color chip] · Headline · Publisher

Filter selectors scoped `#t7`, NOT `#t2`. Getting this wrong breaks filters silently.

---

## Competitor Profiles (`t4`, nav-link only)

`.lb-table` — 9 columns, sorted by funding descending. Target row gets `.tgt-row` (teal 5% tint).

Columns:
1. Company — logo + name + `founded · HQ` + stage stacked
2. Description — from `PROFILE_DATA[co].desc`
3. Valuation — bold numeric text
4. Total Funding — teal gradient bar with black text label
5. Revenue (est.) — text
6. Headcount — blue gradient bar
7. Hiring Growth 1y — amber gradient bar
8. Web Traffic monthly — purple gradient bar
9. LinkedIn Followers — orange gradient bar

Bar text: `.lb-bar-txt-black` — black bold with white text-shadow for readability on any bar color.

Rendered from `LB` array (funding in $M, press count, momentum) + `FIRMO` (stage, valuation label) + `PROFILE_DATA` (description, revenue text, traffic). All three must stay in sync — the `LB` row's `funding` field drives the bar width; `FIRMO[co].val` drives the valuation text; `PROFILE_DATA[co].desc` drives the description cell.

**Bar scale**: `maxes = {funding:2385, emp:768, growth_1y:306, traffic:2100000, followers:91348}`. If any player exceeds these, update `maxes` so bars don't overflow.

---

## Required Data Structures

Populate all with fresh Wokelo data. See template.html for exact shapes.

| Variable | Shape | Purpose |
|---|---|---|
| `COMPANIES` | `['Target','C1',...,'C5']` — target first | Iteration order everywhere |
| `COLORS` | `{co: '#hex'}` | Brand color per player |
| `LOGO_DOMAIN` | `{co: 'domain.com'}` | Clearbit lookups |
| `FIRMO` | `[{co, isTarget, founded, hq, stage, val, funding, rounds, emp_li, emp_range, li_follow, growth_6m, growth_1y, website, focus}]` | Firmographic table + tab 4 |
| `WEEKS` | 13-string array | X-axis labels for time-series |
| `DATE_TICKS` | 13-int array of week-start dt values | Timeline slider filter thresholds |
| `DATE_TICK_LBL` | 13-string array | Ticks under timeline slider |
| `SENTIMENT_7D` | `{co: [13 nums 0-100]}` | Sentiment trend line + mini-chart line |
| `ACTIVITY_BY_PLAYER` | `{co: [13 signal counts]}` | Stacked area chart |
| `HEATMAP` | `[{co, capital, product, partnership, legal, leadership, other}]` | Heatmap grid |
| `MOVE_CATS` | 6 categories with color | Competitor Moves category headers |
| `MOVES` | `{co: {product:[{dt,d,t,imp}], gtm, partnership, hiring, legal, capital}}` | Sub-cards in Moves columns |
| `ARTICLES` | `[{co, dt, date, cat, sent, imp, title, url, pub, body, lens, source:{label,url}}]` | Curated feed + modal drill-through source. `lens` REQUIRED. |
| `THEMES` | `[{title, count, desc, leaders:[{n,v}], srcIdx:[ARTICLES indices]}]` | Key Signals cards |
| `VULNS` | `[{co, tags, signal, opening}]` — competitors only | Cracking Open table |
| `CONSIDERATIONS` | `[{action, triggers:[{co,note}], rationale, srcIdx}]` | Action table |
| `WATCH_NEXT` | `[{co, isTarget, sent, summary, items}]` | 6 per-competitor cards |
| `WATCHLIST` | `[{q, desc, co, imp, theme, pub}]` — 5 items | Numbered watchlist card |
| `LOG_FULL` | `[{co, dt, date, cat, sent, imp, title, url, pub}]` — 200+ entries | Logs tab table + mini-chart dot bucketing |
| `LB` | `[{co, funding($M), press, pos_mom, li_follow, emp, growth_1y, ...}]` | Profile table sort + bar widths |
| `PROFILE_DATA` | `{co: {desc, revenue, web_traffic, web_traffic_lbl, web_traffic_growth}}` | Profile table narrative cells |

---

## Critical JS Ordering Rule

`LOG_FULL` is a large array (200+ entries) defined AFTER `renderMoves()` is declared. The initial `renderMoves()` call — which bucketizes LOG_FULL into 13 weekly buckets for the mini-chart dots — MUST be deferred to the very end of the `<script>` block, AFTER LOG_FULL is defined:

```javascript
// Near renderMoves definition — event wiring only, NO initial call yet:
document.getElementById('ts-single').addEventListener('input', ...);
document.querySelectorAll('#t3 .chip[data-mvcat]').forEach(...);
// Initial renderMoves() is deferred to end of script

// ... later ...

const LOG_FULL = [ /* 200+ entries */ ];
const LOG = LOG_FULL.map(...);
// ... log renderer + filter wiring + renderLog() call ...

// At the very bottom of </script>:
renderMoves(); // now safe — LOG_FULL is defined
```

Getting this wrong causes `ReferenceError: LOG_FULL is not defined` on page load and no Moves column renders.

---

## Quality Gates

Before saving, verify:

- [ ] All `[BRACKET]` placeholders removed from visible text
- [ ] Tab pills: `t3` (active), `t5`, `t2`, `t7` present in that order
- [ ] `t4` accessed only via `.nav-link` in top-right nav row (not a pill)
- [ ] `LOG_FULL` defined BEFORE final `renderMoves()` invocation
- [ ] Every `.mv-item` in Competitor Moves has all three data attributes (`data-moveco`, `data-movecat`, `data-moveidx`) — otherwise click routing breaks
- [ ] Every entry in `ARTICLES` has a populated `.lens` field starting `[Target] Lens: …`
- [ ] `findArticleForMove()` succeeds for at least 60% of MOVES entries (day-match or title fuzzy-match). Sparser means ARTICLES doesn't cover MOVES well — users see "not curated" fallback too often.
- [ ] `VULNS` array contains competitors only, no target
- [ ] Target is `COMPANIES[0]` and gets `#2dd4a6` in `COLORS`
- [ ] `LB[co].funding` (in $M) matches `FIRMO[co].funding` (in $) / 1e6 for the same company — mismatch causes profile bar to lie
- [ ] `maxes.funding` in profile-table render ≥ max LB funding value (otherwise bar overflows 100%)
- [ ] Mini charts use dominant LOG_FULL category color per week — hovering a dot: tooltip should read `mostly [Category]`
- [ ] Signal-type dot legend `.mv-dot-legend` present above `.moves-grid` with 6 swatches
- [ ] `#t2 .chart-card { background: var(--bg-card); }` CSS override present (else All Signals charts are dark)
- [ ] `grid:{display:false}` on x and y for `#c1-activity-c` and `#c1-sentiment-c`
- [ ] Signal Heatmap zero cells: `#f0ede8` light bg (not `#1a1a1a` dark)
- [ ] Logs tab title reads exactly `Full log <em>from akta.pro</em>`
- [ ] Log filter listeners scoped `#t7` (not `#t2`)
- [ ] `node --check <extracted-script>` passes (0 exit code, no output)
- [ ] All Chart.js callsites have null-checks on `getElementById` (dead references from prior tabs must not throw)

---

## Edge Cases

- **Thin news (<30 articles for a competitor)**: Note "Limited coverage — N articles" in the WATCH_NEXT card. Populate MOVES sparingly from Claude knowledge with disclosure. Never fabricate.
- **Zero news in the window (competitor dark)**: Populate a WATCHLIST entry flagging the quiet period — that itself is a signal (typically precedes an announcement).
- **Competitor slug unresolvable**: Skip news pull; populate FIRMO + posture from Claude knowledge; mark row `Est.`
- **Private target with no valuation**: Use `Private` in the val column; use headcount × industry benchmark for revenue estimate.
- **Non-Baseten target**: Global find/replace `Baseten` → new target name across the entire template. Update `COMPANIES[0]`, `COLORS[NewTarget]`, `LOGO_DOMAIN[NewTarget]`. Update the `[Target] Lens:` prefix on all articles. Update the "Baseten Lens" text in `openMoveModal` and in the curated feed lens box (search for `.nc-lens` and `Baseten Lens`).

---

## Output & Present

```
/mnt/user-data/outputs/[target_slug]_competitive_intelligence.html
```

Then `present_files` on the path so the user gets a downloadable link.

---

## Reference

Full working example: `/mnt/skills/user/competitive-intelligence-tabs-dashboard/template.html`
(Baseten vs Together AI · Fireworks AI · Modal Labs · Runpod · Anyscale · Cloud AI Inference vertical · 90-day window · Fireworks $17.5B Series D confirmed, IREN $2.8B contracts, etc.)
