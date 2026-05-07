# Day 3 - grounding_commit.md

## Grounding Commit Target

**Portfolio artifact:** Week 10/11 `tenacious-conversion-engine` model card, training log, and `tenacious-bench` evaluation package.

## Evidence Found in Week 10/11 Work

The Week 10/11 repository already contains enough evidence to justify the gap, but not enough to defend the rank choice:

- `model_card.md` says a first live Path B run completed on **Google Colab T4** using **ORPO** with a **LoRA adapter** on `Qwen/Qwen2.5-0.5B-Instruct`.
- `training/training_run.log` records `status=completed`, `algorithm=orpo`, `backbone=Qwen/Qwen2.5-0.5B-Instruct`, `train_rows=94`, `eval_rows=11`, `global_step=24`, `epoch=1.0`, `train_runtime_seconds=97.7594`, and `train_loss=1.5820321440696716`.
- `training/training_run.log` also records `adapter_artifact=outputs/path_b_orpo/final_adapter`, so the adapter exists as a named training output.
- The first held-out smoke test in both `model_card.md` and `training/training_run.log` was task `tb-0169` on `tone_style_drift`, with `baseline_score=0.2` and `trained_score=0.2`.
- `model_card.md` therefore recommends: **do not replace the Week 10 generator with this adapter yet**.
- `tenacious_bench_v0_1` contains `105` train tasks, `63` dev tasks, and `42` held-out tasks.
- `training_data/path_b_preference_pairs.jsonl` contains `105` preference pairs derived from the Tenacious-Bench train partition.
- `tenacious_bench_v0_1/contamination_check.json` records `passed=true`, `max_8gram_overlap_count=0`, and `max_token_cosine_similarity=0.0`.
- `tenacious_bench_v0_1/examples/score_results.json` shows three example deterministic scores at `0.8` for `tb-0001`, `tb-0064`, and `tb-0180`.

## Missing Evidence

The local Week 10/11 artifacts do **not** record:

- the LoRA rank `r` used in the completed adapter run,
- `lora_alpha`,
- target modules,
- adapter parameter count,
- a rank sweep,
- quality/latency/cost results by rank,
- stage-level failure slices by rank.

That is the concrete portfolio gap: the model card can say a LoRA adapter was trained and that it did not improve the first held-out smoke test, but it cannot explain whether the result was caused by rank under-capacity, data/objective mismatch, insufficient training, or another setup choice.

## Grounding Edit to Make

Add a "LoRA Rank Evidence" section to the Week 10/11 model card and training notes. The first version should be honest:

```text
LoRA rank evidence status:
- Current adapter: ORPO + LoRA on Qwen/Qwen2.5-0.5B-Instruct
- Training completed: yes
- Train/eval rows: 94 / 11
- Held-out smoke test: tb-0169 tone_style_drift
- Baseline vs trained score: 0.2 -> 0.2
- Deployment recommendation: do not replace Week 10 generator yet
- Rank recorded in artifact: no
- Rank sweep completed: no
- Conclusion: current evidence is insufficient to defend the chosen rank
```

Then add a follow-up rank-sweep table template:

```text
rank | adapter_params | bench_score | tone | grounding | booking | latency_ms | cost_per_task | decision
2    | TBD            | TBD         | TBD  | TBD       | TBD     | TBD        | TBD           | candidate
4    | TBD            | TBD         | TBD  | TBD       | TBD     | TBD        | TBD           | candidate
8    | TBD            | TBD         | TBD  | TBD       | TBD     | TBD        | TBD           | candidate
16   | TBD            | TBD         | TBD  | TBD       | TBD     | TBD        | TBD           | candidate
32   | TBD            | TBD         | TBD  | TBD       | TBD     | TBD        | TBD           | candidate
```

## Why This Change Is Grounded in Today's Learning

Today's explainer clarified that LoRA rank mathematically limits the learned update `Delta W = BA`. Because rank controls adapter capacity, a production model card should not merely say "LoRA was used." It should record the rank, explain why that rank was chosen, and show whether additional rank improved `tenacious-bench` quality enough to justify the extra adapter footprint.

The current Week 10/11 evidence supports a careful conclusion: the adapter training stack works, but the trained adapter is not deployment-ready and the rank choice is not yet defensible. The grounding commit should make that limitation explicit instead of hiding it behind a generic LoRA training summary.
