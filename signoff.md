# Day 3 - signoff.md

**Asker:** Mistire Daniel  
**Explainer:** Gersum Asfaw  
**Topic:** Training and post-training mechanics  
**Gap:** Why Q/V-only LoRA can lower training loss without improving held-out rubric behavior

---

## Gap Status: CLOSED

Before this explainer, the confusing part was that the SimPO adapter clearly learned something because training loss fell, but the held-out pass rate did not move. I now understand that this can happen because `q_proj` and `v_proj` LoRA mainly changes attention routing and attended content. It does not directly update key matching, output mixing, or MLP feature transformations.

The gap is closed because I can now explain the result mechanistically: Q/V-only rank-16 LoRA may fit training-example patterns while failing to move the broader constraint-following behavior measured by the rubric. The next defensible experiment is a fixed-rank module ablation: compare `q,v`, `q,k,v,o`, and `q,k,v,o + MLP` while holding data, rank, alpha, training budget, and evaluation constant.
