# Day 3 - evening_call_summary.md

In the evening call, Mistire confirmed that the explainer answered the actual mechanism gap: what Q/V-only LoRA changes, and why a loss drop can coexist with flat held-out rubric behavior. The discussion sharpened the distinction between optimization evidence (training loss fell) and behavioral evidence (pass rate stayed at `14.6%`).

The main revision was to make the recommended experiment module-specific rather than rank-only: compare `q,v` against `q,k,v,o` and `q,k,v,o + MLP` at fixed `r=16`, `alpha=32`, training budget, and evaluation setup. Mistire signed off that this explains why module selection may be a binding constraint and gives a concrete next ablation.
