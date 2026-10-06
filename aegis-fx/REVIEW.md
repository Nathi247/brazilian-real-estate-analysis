# AEGIS-FX V3 + 30 Additions — Review

*Evidence behind [`PLAN.md`](PLAN.md). Date: 2026-10-06.*

## How this review was done

The V3 master spec and ChatGPT's 30 proposed additions were reviewed from five independent angles:

1. **Quant / statistics:** edge, overfitting, sample size.
2. **Data:** point-in-time availability and leakage.
3. **Risk / execution:** sizing, costs, human-in-the-loop.
4. **Architecture / LLM:** buildability, Claude API, testing, deployment.
5. **Red team:** how the project most likely fails.

Each critic's output went to a separate **adversarial verifier**. The verifier tried to refute every finding, re-ran every calculation (all reproduced, with corrections noted below), and fact-checked about 100 external claims using package metadata, source code and web search. Several primary sites (mql5.com, cftc.gov, federalreserve.gov, bls.gov, docs.tradingeconomics.com) were blocked by the network policy. Claims that rest on search snippets are marked as such.

Many numbers come from simulations under stated assumptions: random-walk prices, assumed cost levels, an assumed human latency distribution, and assumed signal rates. Treat them as orders of magnitude to be replaced by measurements in Stage 1, not as facts about your broker or strategy.

---

## Verdict by lens

| Lens | Verdict (condensed) |
|---|---|
| Quant | Not yet sound. There is no strategy to falsify. §44 can't be met on EURUSD alone. The objectives are inconsistent. The additions add about 70–90 thresholds, and searching 100–1,000 noise variants yields +0.17 to +0.32R in-sample, the size of a plausible real edge. Fix: a pre-registered rule, pooled research, calibrated expectancy, a one-shot holdout with power analysis. |
| Data | The plumbing is right, but there is no data-availability model. The 90 threshold can't be validated as written. MT5 history defaults can cap M5 at about 1.3 years. The bar spread field is a per-bar minimum. Broker time ≠ UTC. Calendars revise "previous". COT can be about 50 days late (2025 shutdown). LLMs know the future of any pre-cutoff date. Fix: a bitemporal PIT store, a backtestable deterministic core, forward collectors from day 1. |
| Risk / execution | The controls are right. The step between the alert and the trade is missing: order type, tolerance, expiry, management, attribution. On a 12-pip stop at net 2.2R, only 0.9 pip of adverse entry is tolerable. The limits are mis-calibrated for this frequency, and the 3-loss rule fires on noise. Fix: an execution contract, two ledgers, a read-only watchdog, an investor-password login. |
| Architecture / LLM | The doctrine is right. The LLM sits inside the decision path, so the decision of record can't be backtested. Phase order builds the live stack before any evidence. CrewAI 1.15.23 can't express Opus 5.5's request contract. Reason codes and states are inconsistent. Fix: one pure `decide()` core with two drivers, and the LLM as a post-approval reviewer. |
| Red team | A strong safety spec around a strategy that doesn't exist. Likely failure modes: zero qualifying signals in a leakage-free backtest, too few signals to test anything, or a spuriously good backtest from researcher degrees of freedom. First OOS evidence comes after about 22–33 weeks (V3 order) versus about 7–10 weeks research-first. Fix: reverse the build order; freeze 2–3 concrete hypotheses with baselines. |

---

## What V3 gets right (preserve it)

- **LLM and safety doctrine:**
  - §2.1 and §2.2: deterministic Python owns every number and the LLM owns none.
  - §54: fail-closed defaults.
- **Data handling:**
  - §5: per-record provenance and timestamps.
  - §6: an explicit forming vs completed candle.
  - §56: processing keyed to completed-candle timestamps.
  - §41: the leakage list covers news, releases, sentiment and account state, not just prices.
- **Gates and risk rules:**
  - §35: a structured gate result, never a Boolean.
  - §61: the boundary-style test matrix.
  - §22: round down and recompute risk.
  - §23: no risk increase for higher scores.
  - §26: net R:R.
  - §31: manual kill-switch reset.
  - §33 and §34: signal validity and duplicates as first-class data.
- **Governance:**
  - §39 and §59: versioning and a change-control chain.
  - §45: N, CI, period and regime on every claim.
  - §68: no false certification.
  - §62: modes, none of which can auto-execute.

---

## Consolidated findings

Severity follows the verifier's correction where it differed from the critic's.

### Critical

| ID | Finding | Key evidence | Plan |
|---|---|---|---|
| C1 | **No falsifiable strategy.** No trigger, entry, stop formula, target algorithm, time stop or management is defined. | §65's 31 outputs and §66's 18 phases contain no "strategy definition" or "pre-registration". Driftless null: P(TP at bR before SL at 1R) = 1/(1+b), i.e. 33.3% at 2R. Net expectancy under the null = −cost ≈ −0.04 to −0.10R. | §5 |
| C2 | **The 90 threshold can't be validated.** The components computable from price and spread sum to 70. Reaching 90 needs ≥ 20 of 30 fundamental + sentiment points, for which V3 names no PIT source. | Maximum 70 without fundamental and sentiment, 80 without fundamental, 90 without sentiment (perfect elsewhere). *Correction:* policy rates, 2-year yields, ALFRED first-release values and COT (aligned to its Friday release) **are** PIT-reproducible. The non-reproducible parts are consensus surprises, retail positioning and LLM outputs. | §6, §7.2 |
| C3 | **The LLM sits inside the decision path.** V3 §1 and the upgraded architecture put it before confluence; Additions #1/#7/#10/#20 make its output a decision input. | Opus 5.5 training cutoff: June 2026, so about 2–4 months of clean data exist today. Detecting a 0.5R veto effect needs ≥ 68 vetoed trades (1.7–13.6 years). Running it on every M15 cycle for a 10-year backtest would cost about $21k–$143k (illustrative upper bound). | §8 |
| C4 | **Sample size vs selectivity.** EURUSD-only with 19 ANDed gates. | Estimated 5–25 qualifying signals a year (the critic's own mid-scenario product is about 6). 500 historical signals → 20–100 years. 80% power, one-sided 5%, SD 1.47R: +0.15R needs 594 effective trades, +0.25R needs 214. A 3-year EURUSD training window of about 75 trades supports estimating 1–2 parameters. | §5, §10.2 |
| C5 | **Data snooping from the additions.** | Roughly 100 extra tunables (judgment count). Best-of-K under the null at N = 200: +0.24R / +0.32R for K = 100 / 1,000 (independent variants), +0.17 / +0.23R (ρ = 0.5). A 4,096-config null grid: median in-sample +0.15R, out-of-sample −0.07R. | §3 (budget), §10.5 |
| C6 | **No execution contract.** Order type, tolerance, expiry and latency are undefined. | d_max = (T_net − 2·S_net)/3. On a 12-pip stop, d_max is 0.9 pip at net 2.2R and 2.2 pips at 2.5R. Under an *assumed* human delay (median 5 min, p90 30 min), about 24–37% of market executions on tight setups would breach net 2.0R. | §7.3 |

### High

| ID | Finding | Key evidence | Plan |
|---|---|---|---|
| H1 | Build order puts backtesting at P14. | About 22–33 part-time weeks to the first OOS result vs about 7–10 research-first (assumed hours). | §11 |
| H2 | Objectives inconsistent and implausible. | WR 60% at 2R = +0.80R/trade, annual Sharpe ≈ 2.7 at 25 trades a year. WR 68% at 3R = +1.72R, Sharpe ≈ 4.6. FP < 20–30% contradicts a 32–40% loss rate. A real +0.10–0.25R edge at 2R means a WR of about 37–42%. | §10.1 |
| H3 | **Economic value is small even if the edge is real.** | E × trades × risk = 0.10R × 10 × 0.25% = 0.25%/yr; 0.25R × 25 × 0.25% = 1.56%/yr. | §10.4, §15 |
| H4 | Trade management undefined. | Break-even, partials, trailing, time stop, Friday and news handling all change WR, average R and holding time, and so swap and event exposure. | §7.3 |
| H5 | No attribution of manual trades to signals; outcomes are selection-biased by human skips and gates. | Manual MT5 trades have magic 0. V3 §38 labels executed trades only, so ablation and veto value can't be estimated. | §7.5, §10.5 |
| H6 | No check of what the human actually placed. | A 0.18 → 1.80 lot typo turns 0.25% risk into about 2.41% at $10k, which is above the 1.5% daily limit in one trade. | §7.5 |
| H7 | "No auto-execution" is a convention, not a control. | The process holds a trading-capable login and ingests external text. An MT5 investor password is read-only at the broker. | §7.5 |
| H8 | Limits mis-calibrated for this frequency. | At 0.25%: daily 1.5% = 6R, weekly 3% = 12R, max DD 8% = 32R. A zero-edge strategy hits −8% within 200 trades only about 19% of the time (77% at 0.50% risk). | §7.6 |
| H9 | The 3-loss rule fires on noise; no recovery condition. | P(≥ 1 run of 3 in 20 trades): 93% at WR 40%, 79% at 50%, 56% at 60%. Expected wait: 9.1 / 14.0 / 24.4 trades. A literal backtest stalls after about 18 trades unless a simulated recovery policy exists. | §7.6 |
| H10 | No point-in-time knowledge time in the data model. | COT as of Tuesday, released Friday 15:30 ET. H.10 published Mondays 16:15 ET. ECB yields next TARGET day 12:00 CET. US macro values are revised (hence ALFRED). | §6.3 |
| H11 | MT5 history and time traps. | Default "Max bars in chart" about 100k → M5 about 1.33 yr, M15 about 4 yr. The Python docs say UTC, but users observe broker wall-clock time (e.g. +3h). Calendar archives use the server's current DST for past events. 2026 US/EU DST-mismatch windows: 8–29 Mar and 25 Oct–1 Nov. | §6.1, §6.4 |
| H12 | Spread baseline undefined. | The bar `spread` field is the per-bar minimum (MQL5 Book, forum tests). The rollover spike inflates a raw 24h mean. A 0.0 raw-account spread breaks naive rules. | §6.4 |
| H13 | CrewAI vs Opus 5.5. | 1.15.23: thinking `Literal['enabled','disabled']` (both 400 on Opus 5.5). No effort passthrough. Forced `tool_choice` when an agent has one tool (400). Deprecated `output_format` + beta header. No refusal handling. Default client timeout `None`. 31 mandatory dependencies. Telemetry on by default. The `[anthropic]` extra pins `anthropic~=0.73`. | §8 |
| H14 | §21 "low temperature" isn't implementable. | Opus 5.5 rejects sampling parameters with HTTP 400; SDK 1.x removed them from signatures (TypeError). | §8 |
| H15 | Reason codes and state machine inconsistent. | §35 has 19 conjuncts and §36 has 18 codes. Mismatches: §20/§36, §33/§36, the `DRAWdown` typo. No daily/weekly conjunct in §35. No codes for position size or the kill switch. §57 mixes lifecycle and per-cycle states; a restart could clear the kill switch. | §4.3, §4.4 |
| H16 | Tests miss what protects a gate. | `NaN > limit` and `NaN <= limit` are both False, so a NaN passes. `floor(x/0.01)` gets 1,076 of 10,000 lot values wrong. No property, mutation or parity tests. | §12 |
| H17 | Deployment unspecified. | MetaTrader5 5.0.6231 ships only win_amd64 wheels. Windows standalone NTP poll defaults to 7 days. Monitoring from inside a dead process is silence. | §11 Stage 4 |
| H18 | Look-ahead in derived features and regimes. | Full-sample ATR percentiles and smoothed regimes leak future distributions. | §6.3 |
| H19 | Gap and news risk beyond the planned stop. | Realised loss = (S+g)/S R. A 10-pip overshoot on a 12-pip stop = 1.83R. EURUSD opened about 200 pips higher after the April 2017 French election. | §7.4 |
| H20 | Time semantics: server vs UTC vs local. | MT5 order expiry is entered in server time. A 2–3h error can trade straight through NFP/CPI. | §6.4, §9 |

### Medium

| Finding | Plan |
|---|---|
| The additive score double counts: RSI14 vs (close−EMA20)/ATR ≈ 0.99 analytically on the same timeframe. Weights are invented. | §7.2 |
| Analogue engine (#3): sparse, noisy, leak-prone. Null tail at k = 30, p = 1/3: P(WR ≥ 50%) = 4.3%, about 8.7 spurious "strong analogue" readings a year. | §13 |
| Regime cells (#4/#5) too small: 12 cells × 42 trades gives ±0.43R. In #4's own example only Strong vs Range is significant (z = 3.06). | §13 |
| Walk-forward lacks purging, embargo and a lockbox, and ignores the designer's look-ahead (2020–2026 is already known to you and to the LLMs). | §10.3 |
| Costs in R scale as 1/stop size: 1.0 pip = 0.17R on 6 pips, 0.07R on 15 pips. | §5 (stop floor) |
| Clustered or overlapping trades inflate N: design effect 1 + (m−1)ρ. The six majors are worth about 2.4–4.4 independent pairs. | §10.2 |
| Winner's curse: at N = 300 and true +0.10R, a passed holdout overstates E by about 95%. | §7.6 |
| Calibration and shrinkage need small-sample methods (Platt, Beta-binomial with m = 30–50). | §13 |
| Backtest/live feature parity (train/serve skew) isn't specified. | §6.3 |
| No historical swap source; triple-swap Wednesday. | §7.4 |
| Margin check undefined. At 1:30 with 10-pip stops, 1.0% open risk ≈ 295% margin level. | §7.4 |
| Portfolio/correlation engine is over-built for EURUSD-only. A 60-day correlation at a true 0.70 has a 95% CI of [0.54, 0.81]. | §7.7 |
| Telegram retry can duplicate an alert (no idempotency key); edits don't re-notify. | §9 |
| Deposits, withdrawals and broker credit corrupt drawdown. | §7.6 |
| Prompt injection via headline text; the §20 schema carries status and confidence fields that invite authority. | §8 |
| pandas-ta can't be installed with numpy 2.5 (numba 0.61.2 requires numpy < 2.3). | §7.1 |
| §3 says "Python 3.11+", but numpy 2.5.3 needs ≥ 3.12. | §14 |
| False precision in ChatGPT's examples. "184 similar, 63.6%" has a Wilson CI of [56.4%, 70.2%], and its +0.48R implies an average win of 1.33R, below the 2R floor (under binary exits). | §9, §13 |

---

## Selected fact checks

| Claim | Status | Note |
|---|---|---|
| MetaTrader5 Python package is Windows-only | TRUE | PyPI 5.0.6231 (2026-09-27): win_amd64 wheels only, cp36–cp314 |
| MT5 bar times are UTC | PARTLY | Docs say UTC; users observe broker server time (e.g. +3h on MetaQuotes-Demo). Measure it. |
| Bar `spread` = per-bar minimum (points) | TRUE (secondary) | MQL5 Book on iSpread plus forum tests; not officially documented |
| Default Max bars = 100,000; restart required | TRUE (snippets) | Readable via `terminal_info().maxbars` |
| COT: Tuesday positions, Friday 15:30 ET release | TRUE | 2025 shutdown: none 1 Oct–12 Nov; resumed 19 Nov; backlog cleared 29 Dec 2025 |
| Dukascopy EURUSD ticks from 2003 | TRUE (secondary) | Terms of use not verified |
| ECB 2-year yield curve daily since 6 Sep 2004 | TRUE | |
| DFEDTARU/L start 16 Dec 2008 | TRUE | Splice DFEDTAR before |
| DXY EUR weight 57.6% | TRUE | DXY is mostly EURUSD; use a USD-ex-EUR index from broker symbols |
| Myfxbook API: 100 req/24h free, snapshot only | TRUE | |
| Free calendars revise "previous" / TE field semantics | PARTLY | TE "Forecast" = consensus confirmed; Previous/Revised mapping needs checking on a known NFP revision |
| MQL5 calendar forecast provenance | UNKNOWN | |
| CFTC TFF history from 2006 | UNKNOWN | Consistent with general knowledge; cftc.gov blocked |
| Opus 5.5: sampling params 400; thinking can't be disabled; forced tool_choice 400; $4/$20 per MTok | TRUE | Anthropic SDK reference |
| CrewAI 1.15.23 limitations (above) | TRUE | Read from the wheel source and metadata |
| pandas-ta incompatible with numpy 2.5 | TRUE | numba 0.61.2 requires numpy < 2.3 |
| LLMs used on pre-cutoff history carry look-ahead | TRUE | Memorisation literature; size varies by task |
| MT5 investor password = read-only | TRUE | Broker help centres |
| MT5 mobile supports Stop Limit and a Date/Time expiry | TRUE | |
| MT5 expiry is entered in server time | PARTLY | High likelihood; verify on your terminal |
| Mobile order dialog has a comment field | UNKNOWN | So matching doesn't depend on comments |
| Simple FX technical-rule profits faded after the 1990s | TRUE | Olson (2004); Neely, Weller & Ulrich (2009) |
| ESMA retail leverage cap 1:30 on majors | TRUE | Your broker's jurisdiction may differ |
| Telegram limits about 1 msg/s per chat | TRUE | Bot API FAQ |
| Claude 3.5 Sonnet (named in the PDF) | Retired | 2025-10-28 |

---

## The ChatGPT additions in one line

They have the right instinct: evidence ≠ probability, ablation, calibration, cost stress, champion/challenger, replay. But most of them **add thresholds, which is the opposite of #29**, and several make LLM output a decision input that can't be backtested. PLAN §13 gives each one KEEP / MODIFY / DEFER / DROP with a reason. Across lenses: #2, #6, #18, #21, #22, #27, #28 and #29 were kept by everyone; #10 and #16 were dropped or deferred by most; #3, #5, #13, #19, #23 and #24 were deferred (or dropped) by most.

---

## The one-page PDF ("AEGIS-FX V2/V3 Technical Specification")

Reviewed earlier in this session; summary:

- **Rendering bugs:**
  - `$\mathbb{E}$` prints literally (and raises `SyntaxWarning: invalid escape sequence '\m'`);
  - `<code>` is ignored;
  - the metadata title and author are "(anonymous)";
  - there is no `repeatRows`, footer or version;
  - unused imports.
- **It contradicts V3:**
  - "INSTITUTIONAL-GRADE" (§68);
  - "0.25–0.50% dynamic by tier" (§23);
  - "≥85%" / "85-89%" written as percentages (§17);
  - targets presented as achievable (§43);
  - "300+ H4 bars" for EMA200, where the measured error is about 3.6 pips median and 9.4 at p95 (500 bars → 0.5 / 1.4, 1,000 → 0);
  - a hard-coded, retired "Claude 3.5 Sonnet";
  - a Tier-3 definition that depends on LLM agreement.
- **Arithmetic:** the expectancy objective never binds given the WR/RR targets, and the FP target contradicts the WR target. The V1 column claims a net R:R floor although V1 had no cost model.

If you regenerate it, generate it from `PLAN.md` instead of hard-coding the content in Python.

---

## Limitations of this review

- **Assumptions, not measurements:**
  - signal frequencies are funnel estimates;
  - latency effects use an assumed delay distribution and a Brownian price model;
  - cost levels are typical retail values.

  All of them are measured in Stage 1 or Stage 6.
- **Search-snippet evidence:** where primary documentation was blocked, the fact rests on snippets (marked above).
- **Judgment counts:** the tunable counts (about 100–240) depend on the counting method; the direction of the conclusion doesn't.
- **Illustrative strategy:** the H1 rule in PLAN §5 is a reasonable starting draft, not a recommendation that it works. Its job is to be falsifiable.
