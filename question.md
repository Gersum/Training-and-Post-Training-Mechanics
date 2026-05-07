# Day 3 Question - Training and Post-Training Mechanics

**Asker:** Mistire Daniel  
**Explainer:** Gersum Asfaw  
**Topic:** Training and post-training mechanics - LoRA module selection and rank  
**Date:** Day 3, Week 12

## The Question

My Week 11 SimPO run (`methodology_rationale.md`) targeted only `q_proj` and `v_proj` for LoRA adaptation: `r=16`, `alpha=32`. The training log (`ablations/training_run.json`) shows training loss fell from `0.97` to `0.37` over `36` steps, so the model clearly updated its weights. But the held-out pass rate was `14.6%` for both the trained adapter and the untrained baseline (`Delta A = 0.000`, `p=0.585`, `model_card.md` evaluation table).

What does LoRA actually adapt when it updates `q_proj` and `v_proj` specifically, and is there a mechanism-level reason why those two matrices might allow training loss to fall while leaving output behavior on a multi-constraint rubric task unchanged?

Specifically: when LoRA inserts low-rank matrices `Delta W = BA` into `q_proj` and `v_proj`, what changes about how the transformer processes inputs? Why does "low rank" work at all - what makes a rank-16 update meaningful rather than noise? And what determines which behaviors a LoRA adapter can express versus what it cannot, depending on which modules are targeted? Would including `k_proj`, `o_proj`, or the FFN layers have changed what the adapter could learn for a constraint-following task?

## Connection to Existing Work

- **`week-11/tenacious-bench/methodology_rationale.md`** - The LoRA config table says "Target modules: q_proj, v_proj - Attention layers only; smaller adapter footprint." I wrote this without being able to defend it. A senior engineer asking "why not k_proj, o_proj, or the MLP layers?" would expose the gap immediately.
- **`week-11/tenacious-bench/model_card.md` line 56** - The "Target modules" row in the training configuration table has no rationale column. The honest interpretation section attributes the flat Delta A to "scale asymmetry" and "role confusion," but never examines whether module selection was itself a binding constraint.
- **`week-11/tenacious-bench/ablations/training_run.json`** - The concrete observable is the split between optimization and behavior: training loss fell, but held-out pass rate did not move.

## Why This Matters Beyond My Work

Any FDE fine-tuning a LoRA adapter chooses target modules and rank, often by copying a starter notebook default. The decision determines what the adapter can express. A module selection that is reasonable for a generation-quality task may be too narrow for a constraint-following or scoring task. Understanding the mechanism is the only way to defend or challenge the default and to diagnose flat results without attributing everything to backbone size.

## Four-Property Check

| Property | Status |
|---|---|
| Diagnostic | Names the specific mechanism gap: what updating `q_proj` and `v_proj` at rank 16 changes, and whether that is sufficient for rubric-compliance behavior |
| Grounded | Tied to `methodology_rationale.md`, `model_card.md`, and `ablations/training_run.json` |
| Generalizable | Every FDE adapter fine-tune involves module and rank decisions |
| Resolvable | A colleague can close this in one focused explainer with the LoRA paper, an attention-matrix diagram, and one module-set ablation design |
