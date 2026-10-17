# Salvage notes: nibble

Audit date: 2026-10-17. Old repos were read, not modified. nibble is a typed decision
layer **built on Kev** (`~/Projects/vendor/kev`, Apache-2.0), plus a trainer for small
task-specific deciders. First domain: picking templates and primitives for playdough.

Three old repos tried this space. All three drifted:

- **system-one-mini**: five fixed DistilBERT heads over synthetic "controlled diagnosis" states. It cannot accept a new question.
- **system-a**: fine-tuned Laya to predict a coding agent's next tool call (behavioral cloning). The wrong domain, and nothing to do with picking primitives.
- **system-one-research-review**: an adversarial audit of system-one-mini. It is the most useful of the three, because it lists exactly what went wrong.

Kev already does the part these repos never reached: question-conditioned typed decisions with a fitted temperature and a `--init_from` fine-tune path.

## How to bring anything in

Copy only the excerpt you need into `docs/reference/<name>.md`, with this header:

```text
> Provenance: davidghatley/<repo>@<commit>:<path> (copied 2026-10-17, excerpt)
> Status: reference only. Not a spec. Overridden by anything in this repo.
```

Never copy code wholesale. Kev is a dependency, not something you copy.

## 1. Carry as reference

| What | Source | Why |
|---|---|---|
| Hypothesis ledger, the summary table only | `system-one-research-review@af23648:research_state/HYPOTHESIS_LEDGER.md` (the first table, HYP-001..023) | A ready-made failure checklist for any trainer run. The key entries: HYP-002 (splits reused 99.8% of canonical states because nuisance fields inflated the dedup key), HYP-010 (a fixed-schema parser is not a question-conditioned decider), HYP-015/016 (overconfident; a temperature fitted on one renderer doesn't transfer), HYP-017 (the macro metric double-counted a redundant head), HYP-018 (one renderer per split is pseudo-replication), HYP-022 (the release was not reproducible from the repo). Copy the table only. |
| Retrospective, "What worked" and §2 | `system-one-mini@4e08635:reports/project-retrospective.md` | Shared-state encoding was the real win: 5 decisions in 4.95 ms p50 on the 3060, and 41.9× cheaper than re-encoding at 50 questions. The lesson that hurt: 100% on familiar wording fell to 44.5% on unseen wording, while confidence stayed at 97%. Copy about 30 lines. |
| Evidence table with a baseline that beat the model | `system-a@b4407e5:release/i3/MODEL_CARD.md`, "Evidence" table | The fine-tuned Laya beat TF-IDF on macro-F1 (0.445 vs 0.293) but was much worse on NLL (3.41 vs 0.95), because of near-certain wrong answers. Report NLL and Brier next to accuracy, and always include a lexical baseline. |
| Positive-control-first rule | `system-a@b4407e5:ITERATIONS.md`, Iteration 1 vs 2 | Iteration 1's trainer regressed. Iteration 2 first reproduced a public specialist-vs-base gap (68.0% vs 35.5%), which validated the evaluator, and only then trained. Do this before trusting any nibble trainer result. One paragraph. |
| Evaluation-power rule | `local-mobile-research@29b6284:experiments/comparability/CONTRACT.md` §4 (the minimum-meaningful-difference table), echoed in `docs/RESEARCH.md` on local branch `portfolio-pivot@32e05f2` | With n of about 100–150, only differences of 10–20 points are detectable. Kev's README says the same thing: with 400 records, the gain was inside the noise. Size the held-out prompt set before claiming "at least as well as the big model". |

Test ideas to rebuild (no code copied):

1. Split by a **canonical key over decision-relevant fields only**, so that nuisance fields can't inflate the dedup key.
2. Keep a wording/renderer holdout that is separate from the dev set.
3. Read the test set once per release (Kev already does this with `locked_test`).
4. Always compare against a TF-IDF/logistic baseline and the untouched Kev checkpoint.
5. Report accuracy, NLL, Brier and automation-at-error-budget.

## 2. Lessons

- **5-head DistilBERT vs Kev:** fixed heads were fast and type-safe but learned the template language, not the decision. They could not take new questions or new option sets. Kev puts the questions and options in the input and scores each option with a pointer head, so a new template catalog needs no new head. For nibble's template/primitive picks, where the option set changes as the catalog grows, **only the Kev approach fits**. Keep the old repos' one real win, computing the state once and reusing it across questions; Kev's server already reuses the state cache.
- **Synthetic labels from a generator teach the generator.** All of system-one-mini's data came from a single state machine. Train nibble on real logged decisions from bonfire runs (`decisions.jsonl`), with outcomes attached. Use synthetic data only as a supplement.
- **Behavioral cloning is not the goal** (system-a model card: it predicts the *observed* action, not the best one). Label nibble's records with outcomes (did the game build, was it playable), not just "what the big model picked".
- **Calibration does not travel.** Refit the temperature on nibble's own held-out slice for each domain (Kev's fine-tune path already does this). Don't reuse Kev's default for threshold decisions.
- **Process overhead swallowed the work.** system-a has 31 research memos, frozen predeclarations and "Iteration 3 not authorized" gates, but it shipped a 6-class classifier where 2 classes had no support. Keep one small experiment log; prefer one honest run over gate paperwork.

## 3. Leave behind

- **system-one-mini code** (`system_one/*`, the five-head model, synthetic generator, diversity renderers) and its HF release. The architecture is the wrong one for changing option sets.
- **All system-a code and data:** the Laya trainer (`training/`), `pipelines/trace2decision`, the GLM-trace dataset, the `release/i3` bundle, `apps/reflex`, the 47 MB `artifacts/`, `research/01..31`. Wrong intent (coding next-action), and Laya is superseded by Kev.
- **`system-a@b4407e5:shared/typed_decisions/schema.py`.** Kev's JSONL request/label format is now the contract. Don't maintain a second validator.
- **system-one-research-review's machinery:** `research_state.json`, the schemas and renderers, the `TEAM_*_REPORT.md` files. Only the ledger table carries over.
- **RTX 3060 Laya VRAM measurements.** They are tied to Laya and don't apply to Kev sizes.
