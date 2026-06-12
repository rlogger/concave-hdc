# concave-hdc

Distribution-free risk control for streaming high-dimensional
representations, with market microstructure as the application.

Second derivation of the bayes-hdc line. bayes-hdc is the library
(github.com/rlogger/bayes-hdc, PyPI, DOI 10.5281/zenodo.20635099).
This repo is the theory: derivations, proofs, experiments, paper
drafts. No new library gets built here. Anything that ships, ships
as a bayes-hdc module.

The name: the validity chapter runs on test martingales built from
betting functions, and the right bets maximize concave log-wealth
(Kelly). The name is the method.

## Thesis

HDC gives you cheap, robust, online-updatable representations of
fast data. bayes-hdc gave those representations calibrated
probabilities and finite-sample guarantees, but the guarantees
assume exchangeability. Markets are the canonical setting where
that assumption fails and where the cost of a false alarm is
denominated in dollars. Close the gap with theory, then demonstrate
on the hardest honest data available.

Three chapters, in dependency order:

### 1. Capacity (how much fits in d dimensions, with constants)

How many items can a d-dimensional bundle hold before retrieval
fails, as an explicit function of (d, k, noise), not asymptotics?
Thomas-Dasgupta-Rosing (JAIR 2021) settled pieces of the
deterministic case. The probabilistic case is open: bundles of
GaussianHVs, where each item carries its own variance.

Targets:
- finite-d retrieval bounds for MAP bundles of GaussianHVs via
  sub-Gaussian concentration; explicit constants, verified against
  the bayes-hdc Monte Carlo suite
- the variance budget: how posterior uncertainty in the codebook
  trades against capacity
- hierarchical bundling (the chunk-cleanup trick already in
  bayes_hdc/structures.py) as a provable capacity amplifier; the
  empirical table in BENCHMARKS.md says flat dies at T~200 and
  hierarchical holds at T=800; derive why

Tools: sub-Gaussian/sub-exponential concentration, Johnson-
Lindenstrauss, union bounds over codebooks. Vershynin's HDP book is
the reference shelf.

### 2. Anytime validity (guarantees that survive drift)

bayes-hdc's ConformalAnomalyDetector controls FPR at alpha under
exchangeability. Streams break exchangeability. The fix is the
conformal martingale / e-process line: turn conformal p-values into
a test martingale via a betting function, control the anytime FPR
with Ville's inequality, and detect changepoints when the wealth
process explodes.

Targets:
- an OnlineConformalDetector for bayes-hdc: anytime-valid FPR
  control, no fixed test set, safe under optional stopping
- betting-function choice as a concave-utility problem: which bets
  maximize log-wealth growth against specified drift alternatives
  (this is Kelly; this is also where the repo name comes from)
- power analysis: under what drift rates does the martingale detect
  before a fixed-window baseline

Tools: test martingales, Ville's inequality, e-processes and
anytime-valid inference (Ramdas et al.), game-theoretic probability
(Shafer-Vovk, which is formulated as betting against a market;
the metaphor is load-bearing here).

### 3. Application (two decisions a desk actually makes)

No alpha gets published, ever; if it works you trade it. What gets
published and adopted is the infrastructure that protects and sizes
edge. Two concrete deliverables:

**3a. Strategy gating.** The kill-switch problem: a running book's
edge assumes a regime; the regime shifts; when do you de-risk?
Rolling-Sharpe thresholds and CUSUM are either invalid under
continuous monitoring (peeking destroys the nominal false-alarm
rate) or need distributional assumptions. The e-process gate is
anytime-valid: monitor every tick, stop whenever, the false-alarm
budget holds. Evaluate on simple strategy P&L (momentum or pairs on
liquid names): drawdown avoided vs opportunity cost paid, as a
function of the false-alarm budget, against the CUSUM and
rolling-Sharpe baselines.

**3b. Flow-toxicity monitoring.** VPIN is the standard
order-flow-toxicity input to quote-pulling decisions and it carries
no guarantees (see Andersen-Bondarenko's critique). Build the
anytime-valid replacement on LOB features, evaluate on
flash-crash-labeled LOBSTER data: detection latency vs false-alarm
rate, with the rate actually held where the theory says.

Honesty rules carried over from bayes-hdc: tuned baselines, all
splits disclosed, committed results with provenance, losses printed.

## The claim that pushes the field

Detection and sizing are the same object. An e-process is a wealth
process: you test "regime unchanged" by betting against it, and
under log-utility the optimal test is the Kelly-optimal bet. So the
de-risking schedule (how much to scale the book down as evidence
accumulates) is not a separate heuristic bolted onto a detector; it
IS the detector's wealth process. Sequential testing and portfolio
choice are siloed literatures, game-theoretic probability was built
on market metaphors, and almost nobody has taken it back to actual
markets with a decision attached. That bridge, with data, is the
paper.

Where HDC honestly fits: it will not out-predict a transformer. Its
edge is that bind/bundle are O(d) elementwise with no gradients: a
feature map cheap enough for the hot path at wire speed, where big
models cannot run. That is a latency claim, not an intelligence
claim, and it is the only alpha-adjacent sentence this project gets.

## Why this is the quant-fellowship paper

The math is the hiring bar: concentration of measure, martingales,
optional stopping, Kelly betting. Every theorem is an interview
answer. The applications are the two decisions (gate the book, pull
the quotes) a research shop will recognize as theirs, written by
someone who understands why alpha itself never appears in a paper.

## Status and sequencing

Parked until two things clear: the QIQL paper (NeurIPS clock) and
the bayes-hdc launch/JMLR submission. Then:

1. Chapter 2 first. It is the most novel, it ships a usable
   bayes-hdc module, and it stands alone as a workshop paper
   (target: fall 2026, ICAIF or a COPA-adjacent venue).
2. Chapter 1 second; pairs with the module for the full paper.
3. Chapter 3 last; it needs 1 and 2 to mean anything.

## Reading list

Capacity: Thomas, Dasgupta, Rosing (JAIR 2021); Frady, Kleyko,
Sommer (Neural Comp 2018); Vershynin, High-Dimensional Probability.
Validity: Vovk on conformal martingales; Ramdas et al.,
game-theoretic statistics and safe anytime-valid inference (2023
survey); Shafer & Vovk, Game-Theoretic Foundations for Probability
and Finance; Volkhonskiy et al. (COPA 2017) inductive conformal
martingales.
Application: Easley, Lopez de Prado, O'Hara on VPIN and the
Andersen-Bondarenko critique; Zhang, Zohren, Roberts, DeepLOB;
FI-2010 benchmark paper and its known leakage caveats; LOBSTER
documentation; Kelly 1956 and Breiman 1961 for growth-optimal
betting.

## Open questions parked here

- retrieval error exponent for GaussianHV bundles as f(d, k, sigma):
  conjecture sub-Gaussian rate, constants unknown
- does temperature scaling commute with conformal calibration, or
  does recalibrating logits invalidate downstream p-values in the
  streaming case
- minimal drift conditions under which the conformal martingale has
  nontrivial power but the i.i.d. detector's realized FPR blows up
  (this is the motivating figure for the paper)
- whether the hierarchical-bundle capacity proof gives a rate that
  matches the T=800 empirical wall
