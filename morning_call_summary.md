# Day 3 - morning_call_summary.md

## What Was Ambiguous in the Original Draft

Mistire's original concern started as "my LoRA adapter trained, but held-out pass rate did not improve." That was real, but still too broad. It could have been blamed on model size, SimPO setup, data quality, rank, or evaluation noise. The morning call sharpened the gap toward a specific mechanism: whether targeting only `q_proj` and `v_proj` constrained what the adapter could express.

## How the Question Was Sharpened

Gersum pushed Mistire to separate two facts: training loss fell (`0.97 -> 0.37`), but held-out pass rate stayed flat (`14.6% -> 14.6%`). That distinction made the question diagnostic: the adapter optimized something, but the optimized behavior did not transfer to the multi-constraint rubric. The final question now asks what Q/V LoRA changes inside attention and whether missing `k_proj`, `o_proj`, or MLP targets could explain the flat behavioral result.

The question was also grounded more tightly in artifacts: `methodology_rationale.md` for the Q/V module choice, `ablations/training_run.json` for the loss curve, and `model_card.md` for the flat held-out result. That makes the question answerable in one explainer while still generalizing to any FDE choosing LoRA target modules.

## Final Sharpened Question

What does LoRA adapt when it targets `q_proj` and `v_proj`, and why might that reduce training loss while leaving held-out multi-constraint rubric behavior unchanged?

## Attestation

Confirmed by: Mistire Daniel / Gersum Asfaw  
Date: _______________________
