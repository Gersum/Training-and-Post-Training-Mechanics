# Day 3 Explainer for Mistire - What Q/V LoRA Can and Cannot Change

**Written by:** Gersum Asfaw  
**For:** Mistire Daniel  
**Topic:** Training and post-training mechanics - LoRA module selection and rank

## The Question

Your SimPO run updated only `q_proj` and `v_proj` with LoRA (`r=16`, `alpha=32`). Training loss fell from `0.97` to `0.37`, but held-out pass rate stayed flat at `14.6%` against the untrained baseline. The real question is whether Q/V-only LoRA can learn something real while still failing to change output behavior on a multi-constraint rubric task.

## Short Answer

Yes. There is a mechanism-level reason this can happen.

LoRA on `q_proj` and `v_proj` changes **attention routing** and **the content copied through attention**. It does not directly update the key projection, the attention output projection, or the feed-forward network layers where many token-level feature transformations and decision boundaries are expressed. That means a Q/V adapter can reduce training loss by learning prompt-specific associations while still being too narrow to change held-out rubric behavior such as constraint following, refusal, evidence discipline, or scoring consistency.

## What Q, K, V, and O Do

Inside attention, each token representation is projected into:

```text
Q = X W_q
K = X W_k
V = X W_v
Attention(X) = softmax(Q K^T / sqrt(d)) V W_o
```

`q_proj` changes what each token asks for. `k_proj` changes what each token advertises as matchable. `v_proj` changes what information gets carried forward when a token is attended to. `o_proj` mixes the attended result back into the model stream.

So updating only `q_proj` and `v_proj` means the adapter can change:

- which previous tokens a position tends to attend to;
- what content is copied from attended tokens.

But it leaves unchanged:

- the key-side matching surface (`k_proj`);
- the output mixing after attention (`o_proj`);
- the MLP/FFN transformations that often store task features and nonlinear decision rules.

## What Low Rank Means Here

LoRA freezes the original matrix `W` and adds a low-rank update:

```text
W' = W + Delta W
Delta W = B A
```

With rank `r=16`, the update has at most 16 independent directions per targeted matrix. This is meaningful because downstream adaptation often does not need a full dense update; the LoRA paper shows that useful task adaptation can be low-dimensional. But low rank is still a bottleneck. It can express a compact correction to Q/V behavior, not an arbitrary rewrite of the model.

For your run, `r=16` could be enough to make the training prompts more likely under the SimPO objective while still not enough, or not in the right modules, to change the behaviors that the held-out rubric measures.

## Why Loss Can Fall While Held-Out Behavior Stays Flat

Training loss measures whether the adapter is fitting the training objective. Held-out pass rate measures whether the adapted model changes externally visible behavior under a rubric.

Those are related, but not identical.

A Q/V-only adapter may learn to attend more strongly to phrasing patterns in the preference data. That can lower loss. But a multi-constraint task asks for more than local pattern pickup. It may require the model to:

- preserve multiple constraints at once;
- reject unsupported claims;
- balance rubric dimensions;
- transform evidence into a calibrated judgment;
- maintain instruction hierarchy across the full response.

Those behaviors may depend on distributed circuits across attention and MLP layers. If only Q/V changed, the model may process training examples more efficiently without moving the final decision boundary enough to improve held-out pass rate.

## Would K, O, or MLP Targets Matter?

Potentially, yes.

Adding `k_proj` lets the adapter change both sides of attention matching: what tokens ask for and what tokens match. Adding `o_proj` lets it alter how attended information is mixed back into the residual stream. Adding MLP/FFN layers gives the adapter access to nonlinear feature transformations, which may matter more for rubric compliance than attention routing alone.

That does not mean "always target everything." It means Q/V-only should be treated as a footprint-saving hypothesis, not a defended default.

## Minimal Experiment

Run a module-set ablation with rank and training budget fixed:

```text
A: q_proj, v_proj
B: q_proj, k_proj, v_proj, o_proj
C: q_proj, k_proj, v_proj, o_proj + FFN/MLP projections
```

Keep `r=16`, `alpha=32`, data split, training steps, decoding, and evaluation fixed. Compare:

- training loss curve;
- held-out pass rate;
- per-dimension rubric scores;
- failure types that remain unchanged.

If all settings lower loss but only B or C improves held-out pass rate, module selection was binding. If none improve pass rate, the issue is more likely data, objective, model scale, or rubric/task mismatch.

## Bottom Line

Q/V LoRA changes how the model routes attention and what content attention carries forward. That can be a real update and still fail to change multi-constraint behavior. For a rubric-compliance task, module selection is part of the hypothesis: Q/V-only may be too narrow if the needed behavior lives in key matching, output mixing, or MLP feature transformations. The next defensible step is not guessing a bigger rank; it is a controlled module-target ablation.

## Sources

1. Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models." arXiv:2106.09685.  
   https://arxiv.org/abs/2106.09685

2. Hugging Face PEFT LoRA documentation, especially `r` and `target_modules`.  
   https://huggingface.co/docs/peft/package_reference/lora
