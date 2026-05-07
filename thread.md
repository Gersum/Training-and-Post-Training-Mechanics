# Day 3 Thread - LoRA Rank as a Production Decision

1. I used to treat LoRA rank (`r`) like a training knob: try a value, check the score, move on. The gap I closed today is that rank is not just a knob. It is the capacity limit on the adapter update.

2. Mechanically, LoRA freezes the base weight `W` and learns a low-rank update: `W' = W + BA`. If `A` is `r x d_in` and `B` is `d_out x r`, then rank `r` limits how many independent update directions the adapter can learn.

3. That gives the production tradeoff. Rank too low can miss task-specific behavior. Rank high enough captures the useful correction. Rank too high can add capacity without improving held-out workflow quality.

4. For my conversion engine, the defensible choice is not "highest benchmark score." It is the smallest rank that reaches a stable quality plateau on `tenacious-bench` without regressing latency, cost, or critical slices like qualification and booking decisions.

5. The minimal experiment: train ranks `[2, 4, 8, 16, 32]` with the same base model, data split, target modules, steps, and decoding settings. Then compare quality, adapter size, latency, cost, and stage-level failures.

6. The main FDE lesson: a rank choice should be an evidence-backed deployment decision. "We used `r=8`" is not enough. I need to be able to say what capacity it allowed, where quality plateaued, and what higher rank failed to buy.
