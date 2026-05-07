# Day 3 - signoff.md

**Asker:** Gersum Asfaw  
**Explainer:** Mistire Daniel  
**Topic:** Training and post-training mechanics  
**Gap:** LoRA rank as a defensible production decision

---

## Gap Status: CLOSED

Before this explainer, I understood LoRA rank as a rough capacity hyperparameter, but I could not explain what it constrained mathematically or how I would defend a chosen `r` in a production review. I now understand that LoRA freezes the base weights and learns a low-rank update `Delta W = BA`, where rank `r` limits the number of independent update directions the adapter can express. That makes rank a capacity and efficiency decision, not just a training setting.

The explainer closed the gap because it gave me a concrete decision rule: run a controlled rank sweep on `tenacious-bench`, hold all other training variables fixed, measure quality by stage plus latency/cost, and choose the smallest rank on the quality plateau that does not regress critical slices. I can now explain how rank connects to under-capacity, over-capacity, adapter size, and deployment tradeoffs in my Week 10/11 conversion-engine work.
