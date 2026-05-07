# Week 12 Day 3 - Knowledge Gap (Training and Post-Training Mechanics)

**Name:** Gersum Asfaw  
**Submitting to Explainers:** On Behalf of Mistire Daniel  
**Week 10/11 implementation context:** `tenacious-conversion-engine` (LoRA-trained component) and `tenacious-bench`

## Gap I identified in my Week 10/11 implementation

My Week 10 conversion-engine includes a LoRA-tuned component that contributes to quality and cost outcomes in the pipeline. I chose a rank `r` during training, and I can report outcome metrics, but I cannot defend that choice from first principles or evaluation evidence. If a reviewer asks why I used this rank and not half or double, I cannot answer mechanically or empirically.

In practice, I understand that rank controls adapter capacity, but I cannot yet explain the mechanism-level tradeoff between:

- under-capacity (rank too low: adapter misses task-specific behavior the base model cannot generalize to),
- over-capacity (rank too high: adapter overfits narrow patterns, unstable generalization at inference), and
- cost and latency footprint at inference and deployment (adapter size, merge overhead, serving constraints).

So my gap is this: **my rank choice is empirical but not defensible - I cannot separate what the rank mathematically constrained from what my training data or objective caused, and I cannot produce evidence that would justify `r = X` as a production decision rather than a trial-and-error pick.**

## Final research question (precise and concise)

In conversion-engine LoRA component, I chose rank `r` empirically but cannot defend it from mechanism or evidence. How does LoRA rank mathematically constrain adapter capacity, and what minimal experiment on my existing stack (evaluated on tenacious-bench with quality, latency, and cost metrics) is sufficient to justify a specific rank as a production decision rather than a benchmark-only pick?

## Why this gap matters for my implementation

Right now, if I ship or hand off the conversion-engine with a LoRA adapter, I cannot explain the rank choice to a client, a reviewer, or my own future self in a way that is grounded in mechanism or measurement. Closing this gap changes three things:

- **Workflow side:** I can redesign the LoRA training decision process to produce a rank that is justified by task complexity, not iteration luck.
- **Benchmark side:** I can attribute quality gains (or regressions) in `tenacious-bench` to rank-capacity effects rather than treating all fine-tuning outcomes as one undifferentiated result.
- **Deployment side:** I can make honest, mechanism-grounded efficiency claims in the model card and deployment recommendation sections, rather than reporting outcomes without explaining what drove them.

## Connection to existing work

- `tenacious-conversion-engine` trained component (Week 10/11): LoRA-based adaptation contributes to quality/cost outcomes across `enrich -> compose -> qualify -> book -> sync`.
- Model card and evaluation artifacts: outcomes are reported, but gains are not yet attributed to rank-capacity tradeoffs.
- Deployment recommendation sections: practical performance and efficiency claims lack mechanism-grounded rank justification.
