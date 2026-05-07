# Day 3 Thread - Why Q/V LoRA Can Lower Loss Without Changing Behavior

1. Mistire's LoRA run has the kind of result that looks confusing at first: training loss fell from `0.97` to `0.37`, but held-out pass rate stayed flat at `14.6%`. The adapter learned something, but the rubric behavior did not move.

2. The key is module selection. Updating `q_proj` changes what each token asks to attend to. Updating `v_proj` changes what content gets carried forward from attended tokens. That affects attention routing and attended content.

3. But Q/V-only LoRA does not directly change `k_proj`, `o_proj`, or the MLP layers. So it may not move the parts of the model most responsible for key matching, output mixing, nonlinear feature transforms, or final rubric-compliance behavior.

4. Low rank is meaningful because LoRA learns `Delta W = BA`: a compact update with limited directions. Rank 16 can express useful local corrections, but it is still constrained by both rank and target module.

5. That explains the split: training loss can fall if Q/V updates fit patterns in the training examples, while held-out pass rate stays flat if the task needs broader changes for constraint following, refusal, evidence discipline, or scoring consistency.

6. The next experiment is not just "increase rank." Compare module sets at fixed rank: `q,v` vs `q,k,v,o` vs `q,k,v,o + MLP`. If wider targets improve held-out rubric scores, module selection was binding. If not, look at data, objective, or model scale.
