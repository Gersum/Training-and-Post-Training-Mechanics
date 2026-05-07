# Day 3 - grounding_commit.md

## Grounding Commit Target

**Portfolio artifact:** Mistire's Week 11 `tenacious-bench` methodology rationale and model card.

## Evidence From the Question

The existing Week 11 artifacts already contain the core observable:

- `methodology_rationale.md` records the LoRA target modules as `q_proj` and `v_proj`.
- The configuration used `r=16` and `alpha=32`.
- `ablations/training_run.json` shows training loss fell from `0.97` to `0.37` over `36` steps.
- `model_card.md` reports held-out pass rate stayed flat at `14.6%` for both trained adapter and untrained baseline.
- The reported aggregate effect was `Delta A = 0.000`, `p=0.585`.

## Grounding Edit to Make

Add a "LoRA Module Selection Rationale" section to `methodology_rationale.md` and a matching limitation note to `model_card.md`.

The methodology section should say:

```text
The first SimPO adapter targeted q_proj and v_proj only. Mechanistically, this allowed the adapter to alter attention routing and attended value content, while leaving key projection, attention output mixing, and FFN/MLP transformations unchanged. This was a smaller-footprint starting point, not evidence that q/v is sufficient for multi-constraint rubric compliance.
```

The model-card limitation should say:

```text
The flat held-out result (14.6% baseline vs 14.6% trained; Delta A = 0.000) should not be attributed only to backbone scale or role confusion. Module selection may also be binding: a q_proj/v_proj-only LoRA adapter can reduce training loss while failing to move constraint-following behavior if the needed changes live in k_proj, o_proj, or MLP layers.
```

## Follow-Up Experiment

Run a fixed-rank module ablation:

```text
Config A: q_proj, v_proj
Config B: q_proj, k_proj, v_proj, o_proj
Config C: q_proj, k_proj, v_proj, o_proj + MLP/FFN projections
```

Hold constant:

- `r=16`
- `alpha=32`
- data split
- training steps
- SimPO objective
- decoding settings
- held-out evaluation set

Measure:

- training loss drop
- held-out pass rate
- per-dimension rubric scores
- failure modes that remain unchanged
- adapter parameter count / serving footprint

## Why This Change Is Grounded in Today's Learning

Today's explainer clarified that LoRA is not just "fine-tuning, but smaller." It changes only the matrices you target. For Q/V-only adaptation, the adapter can change attention queries and values, but it does not directly change key matching, output projection, or MLP transformations. That mechanism gives Mistire a defensible explanation for why training loss fell while held-out rubric compliance stayed flat, and it points to the next concrete experiment.
