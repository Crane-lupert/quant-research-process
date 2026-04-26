# Schedule 13D Activist Strategy — Prior-Art Survey (Case Study)

> Methodology case study from a prior-art survey conducted before scoping
> a 13D-based signal pipeline. Prior-art filtering is the standard first
> step before committing to any new research project, to avoid building
> heavily-scooped territory.
>
> Companion project (status update): the activist-intent classifier this
> survey informed (Project Y) was killed on 2026-04-26 per pre-registered
> data-validity contracts. The labeler feedback that triggered this prior-art
> survey survives as `docs/principles.md §6.7 (LLM-derived Signal 5
> principles)` in the workflow archive; the abandonment is documented in
> `retrospectives/2026-04-26-data-validity-contract-failure-killed.md`. This
> survey, conducted before the data-validity collapse was visible, retains
> its independent value as a prior-art map of the 13D activist landscape.

**Survey date**: 2026-04
**Trigger**: A domain-expert labeler started annotating a 30-item oracle set
for a 13D Item 4 intent classifier and produced 5 substantive critiques
(filer-type stratification, boilerplate vs substantive intent, narrative
agent vs classifier framing, 5-class > binary, ML-jargon collision with
domain-expert audiences). The survey was triggered by the "narrative agent"
critique — was that idea novel enough to spin off as a separate project?

## §1. Academic landscape post-2015

The activism-CAR literature has matured beyond the canonical
Brav-Jiang-Partnoy-Thomas (2008) and Bebchuk-Brav-Jiang (2015) papers.
The literature is now both deeper and more skeptical.

The canonical announcement-window finding (~6-7% CAR around 13D filing)
replicates in newer samples but with material decay and heterogeneity. Brav,
Jiang, and others have documented that filing-day abnormal returns dropped
from ~16% in 2001 to ~3% by 2006 as the strategy industrialized. **The 2025
Bajzik et al. meta-analysis (1,973 estimates from 67 studies, 1983-2021),
after correcting for selective reporting, finds true activism-induced
shareholder value of only 0-1.5%** — substantially below conventional
wisdom — and identifies stronger effects in confrontational campaigns and
sale-oriented activism. DeHaan-Larcker-McClure (2018) report that
pre-to-post activism returns are insignificantly different from zero in
their sample, raising serious questions about the persistence of the
headline finding.

Most directly relevant follow-ups for the labeler's edge ideas:

- **Krishnan-Partnoy-Thomas (2016) "Second Wave of Hedge Fund Activism"** —
  Top-tier activists with demonstrated clout / expertise produce materially
  better outcomes; activists with *more* interventions actually produce
  *lower* returns. Direct support for filer-type stratification.
- **Brav-Dasgupta-Mathews "Wolf Pack Activism" (RFS 2022)** — Wolf-pack
  presence associated with +6 percentage points campaign success rate and
  +8.3% buy-and-hold abnormal return. Pre-filing turnover at trigger date
  is ~325% of normal volume. Clustered activism = elevated alpha.
- **Coffee & Palia (2016) "The Wolf at the Door"** — wolf-pack formation
  around 13D filings, regulatory implications.
- **Corum-Malenko-Malenko "Activism and Takeovers" (RFS 2022)** —
  settlement vs hostile distinctions. Settlement likelihood rose from ~3%
  (2000) to ~21% (2013).
- **Bliss-Molk-Partnoy "Negative Activism" (Berkeley working paper 2019)**
  — activist short campaigns produce >20% target underperformance over 4
  years; informationally distinct from long activism.
- **Greenwood & Schor "Investor Activism and Takeovers"** — most 13D
  announcement returns concentrated in cases that ultimately lead to
  takeover.
- **Bebchuk-Brav-Jackson-Jiang "Pre-Disclosure Accumulations"** —
  documents the 10-day window leakage pattern that the SEC ultimately
  cited when shortening the window to 5 days in October 2023 (effective
  February 2024).

What is **not well-covered academically**: rigorous NLP/LLM classification
of Item 4 intent text linked to CARs. The closest published finance work
uses earlier-generation textual methods (Loughran-McDonald sentiment
dictionaries) on 10-Ks and 8-Ks, not on 13D Item 4 specifically. There are
LLM-on-SEC-filings papers (e.g., scalable-framework-for-10-K and the
SECQUE benchmark on arXiv) but none I could verify that classify 13D
Item 4 into a hostile/strategic/passive scheme and tie it to returns.
**This is a real academic gap.**

## §2. Commercial landscape

| Product | Vendor | Offering | Coverage gap |
|---|---|---|---|
| **13D Monitor** | Ken Squire (since 2006) | Curated reports on ~2K 13Ds + amendments/yr; tracks 150-200 active campaigns; subscriber list = top IBs, law firms, L/S funds | Human-written reports; no real-time programmatic feed; no API |
| **13D Activist Fund** | mutual fund family (since ~2012) | Replicates activist 13D targets via mutual fund wrapper | Mutual fund wrapper, high expense ratio; no real-time signal exposure |
| **WhaleWisdom** | independent vendor | 13F/13D/13G alerts (free email tier; paid API) | Free tier latency; no NLP on Item 4 |
| **sec-api.io** | independent vendor | Real-time WebSocket stream (~300ms post-EDGAR), structured JSON 13D/G, Item 4 text extracted | Provides text but no intent classification |
| **Finnhub** | independent vendor | Real-time SEC filing API including 13D | Generic feed; no activist taxonomy |
| **SharkRepellent / SharkWatch50** | enterprise terminal vendor | Curated activist campaign DB; SharkWatch50 = top-50 activist index | Backward-looking; not real-time push |
| **Activist Insight (Insightia)** | enterprise vendor | Comprehensive activism database, campaign tracking | Data product, not signal generator |
| **Bloomberg / Refinitiv** | terminal vendors | Raw 13D pointers in news feed; activism tracker page | Raw filings + curated campaign list, but no LLM-parsed Item 4 |
| **Audit Analytics** | enterprise vendor | 13D event analysis & datasets | Research-oriented, not real-time |
| **EDGAR PDS direct** | SEC dissemination feed | Push of all filings as accepted (~2 min from submission) | Raw — no parsing, no taxonomy |

**Key observation**: 13D Monitor is the obvious incumbent for *human-written
narrative*. WhaleWisdom + sec-api.io are the obvious low-cost programmatic
sources. **No public commercial product I could verify offers an
LLM-classified Item 4 intent feed with hostile/strategic/boilerplate
stratification** — vendors expose the text, but classification is left to
the buyer.

## §3. Per-idea edge assessment (the 5 labeler-suggested ideas)

| # | Idea | Verdict | Justification |
|---|---|---|---|
| 1 | **Filer-type stratification** (activist HF vs corporate filer) | **Partly done** | Krishnan-Partnoy-Thomas (2016) and SharkWatch50 already operationalize "top activists." 13D Monitor manually filters this. Academically validated; commercially served by enterprise vendors at high prices. CIK-tagging of "activist vs passive" is table stakes — required floor, not edge. |
| 2 | **Intent classification on Item 4 text** (5-class) | **Niche edge** ★ | Clearest gap. Academic literature distinguishes hostile vs settlement *qualitatively* but no published work does 5-class LLM classification of Item 4 → CAR. No commercial product I verified does this either. The "may engage management" boilerplate vs explicit demands distinction is genuinely valuable signal. **Most defensible idea on the list.** |
| 3 | **Confidence-weighted + disagreement-inverse-weighted ensemble** | **Likely doomed (as primary edge)** | Standard ML hygiene, not alpha source. Useful as a second-order improvement to (2) but doesn't generate edge by itself. Treat as implementation detail of (2), not a separate idea. |
| 4 | **Real-time narrative-summary push agent** | **Heavily scooped (raw delivery); partly open (synthesis quality)** | 13D Monitor (human reports), WhaleWisdom (alerts), sec-api.io (parsed JSON), Bloomberg (terminal alerts) all exist. **However** — no one I verified delivers a structured "Filer X, 6.2% stake, governance, 3 directors, prior 6 campaigns avg 14% CAR" payload in <2 min programmatically. The synthesis layer (filer history + classified intent + base-rate CAR) is open. But this is a *product* edge, not an *alpha* edge — competing on UX with 13D Monitor's $5k/yr report subscribers, not generating PnL. |
| 5 | **13G × ETF inflow concurrency** for short-horizon return | **Likely doomed** | Index inclusion effects have demonstrably **decayed over the last decade** (BIS WP 952, Todorov JMP). 13G filings are passive disclosures with multi-week lag; the timing precision needed for ETF flow concurrency just isn't there. ETF flow data itself is delayed/expensive. Mechanism is plausible but the data-and-timing combination is hostile. |

## §4. Additional edges (deferred from public discussion)

Beyond the labeler's five ideas, the survey identified several
additional edge directions in the published academic and commercial
gaps. The specific candidates are deferred from this public document —
naming them here would compress the build window for any party
(including the survey author) who chooses to develop them.

The methodology point preserved: a structured prior-art survey
routinely surfaces edges that the domain expert who triggered the
survey did not list. The "additional candidates per labeler-suggested
candidate" ratio in this case was approximately 1:1 — each idea the
labeler raised triggered a corresponding gap in the published
landscape that the labeler did not name.

This pattern is itself a recommendation for any future research repo
that solicits domain-expert input: schedule a separate prior-art
survey *after* the expert input, not before, so that the survey
benefits from the framing the expert provides while still uncovering
the experts' blind spots.

## §5. Honest "is this worth building" verdict

**The narrative-agent product (idea 4) by itself is duplicating 13D
Monitor with worse distribution.** Squire has owned this niche for 20
years, owns the relationships with the activist hedge funds whose moves
matter, and has a captive subscriber base of investment banks and funds.
Competing with that on "we summarize 13Ds faster" loses.

**A defensible niche does exist** in the gap between (a) commercial
parsing-only feeds (sec-api.io, WhaleWisdom, Bloomberg), which expose
text but no intent classification, and (b) human-curated narrative
services (13D Monitor), which are slow and not programmatic. The
precise composition of a candidate signal pipeline is project-specific
and not detailed here. The SEC's October 2023 acceleration of disclosure
deadlines (5 days for 13D, 2 days for amendments) is a structural
tailwind for fast structured parsing — the window where stale reports
lose to programmatic signal is widening.

**Caveats for a solo dev**:
- The Bajzik 2025 meta-analysis showing true activism alpha of 0-1.5%
  (after publication-bias correction) is the sober anchor. Headline 7%
  CARs are sample-period artifacts; do not assume the 2008-era effect
  size.
- Capacity is small. This works as a personal research / SaaS feed sold
  to ~50-200 buy-side clients; it does not work as a scaled fund product
  unless extended to higher-frequency or international 13D-equivalents
  (UK 3% disclosures, EU TR-1 forms).
- The NLP/LLM step is now cheap enough that the moat is **CIK curation
  + prompt/labeling craft + downstream feature infrastructure**, not
  the model itself. Anyone with $100/mo of API credits can replicate
  the classifier in a weekend; the edge is in the labeled training set
  and the historical conditioned features.
- **Live-trading safety**: any production deployment should treat LLM
  intent classification as advisory only — never auto-route orders on a
  single classifier's output. The 13D Monitor model (human-curated
  reports to traders) exists for a reason.

**Bottom line**: Idea 2 (intent classification) is the only
labeler-suggested item with a genuinely novel framing. The labeler's
narrative-agent framing should be downgraded from headline product to
UX wrapper around the underlying signal pipeline — build the signal
first, decide on distribution second. The specific signal composition
that fills the gap identified in this section is treated as
project-private.

## Methodology lessons that transfer

- **Prior-art filter is mandatory before scoping**: 5 labeler-suggested
  ideas surveyed → 1 turns out to be defensible, 2 are heavily scooped,
  1 is implementation detail, 1 is doomed by data-timing constraints.
  Discovery rate ~20%. Without the survey, all 5 would have entered the
  scoping doc as "ideas to pursue."
- **Sober anchor against headline effects**: a single recent
  meta-analysis (Bajzik 2025) reset the prior on activism alpha from
  ~7% to 0-1.5%. Always pull the most recent meta-analysis or
  publication-bias-corrected estimate before assuming literature effect
  sizes will hold.
- **Regulatory windows shift edge dynamics**: the Oct-2023 SEC rule
  change (5-day 13D, 2-day amendments) is a structural tailwind for
  fast structured parsing. Track regulatory windows when scoping
  data-pipeline projects.
- **Domain-expert labeler ≠ ML-expert critic**: the labeler's surface
  critiques (filename naming, "may engage" boilerplate) mapped cleanly
  to academic findings (Krishnan-Partnoy-Thomas filer-type, Brav et al.
  hostile-vs-soft engagement). Domain feedback often *encodes* known
  literature without citing it; cross-checking against the literature
  validates which intuitions are sharp.

## References

**Academic** — citations available in the original survey log; key
authors cited above. The Bajzik et al. (2025) meta-analysis (Wiley)
and the Brav-Dasgupta-Mathews (2022, RFS) wolf-pack paper are the two
most load-bearing for the verdict in §3.

**Commercial** — products in §2 are publicly listed; pricing details
where stated are best-effort approximations from public materials and
should be re-verified at point of decision.
