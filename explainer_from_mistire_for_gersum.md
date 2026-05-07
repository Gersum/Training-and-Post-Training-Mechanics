# LoRA Rank is a Production Decision, Not a Hyperparameter

**Written by:** Mistire Daniel  
**For:** Gersum Asfaw's Day 3 question

---

Gersum's Week 11 model card reports a LoRA rank `r`, lists quality outcomes on `tenacious-bench`, and names the rank in the training configuration table. It cannot defend that number. If a client engineer asks "why `r=X` and not `r=X/2`?" the honest answer is: we tried it and it worked. That is not a production decision. That is a benchmark outcome without a mechanism.

This post closes the gap: what rank mathematically constrains, why that creates a specific under/over-capacity tradeoff, and what the minimal experiment is to justify a rank choice as defensible for the Tenacious deployment.

## What Rank Actually Controls

LoRA adapts a weight matrix `W0` by adding `Delta W = BA`, where `B` is `d x r` and `A` is `r x k`. The rank `r` is the number of independent directions in weight space the adapter can update.

A concrete way to see this: `Delta W = BA` is a sum of `r` rank-1 outer products:

```text
Delta W = b1 a1^T + b2 a2^T + ... + br ar^T
```

Each term shifts the weight matrix along one direction. `r=1` means you can only move the matrix along a single axis: very constrained. `r=64` means you can combine 64 independent directions: much more expressive, but also much more prone to fitting noise when your dataset is small.

This is not an abstract concern. The number of trainable parameters scales linearly with `r`. For a Qwen 0.5B layer with hidden dimension 896, one LoRA module at `r=16` adds approximately:

```text
2 * (896 * 16) = 28,672 parameters
```

Doubling to `r=32` doubles that. Across layers and targeted modules, the difference between `r=8` and `r=64` is roughly 8x in LoRA adapter parameters for those targets.

## The Tradeoff: Under-Capacity vs Over-Capacity

**Under-capacity (rank too low):** The adapter cannot span the subspace of weight-space changes needed to learn the target behavior. Training loss may plateau early or fail to decrease. Held-out performance can stay weak because the adapter has learned a distorted but still-underfit approximation.

**Over-capacity (rank too high):** The adapter has enough expressivity to memorize training examples rather than generalizing the underlying pattern. Training loss falls fast and deep. Held-out performance degrades or stays flat because what was learned is the specific token patterns in training pairs, not the generalizable behavior.

The key insight from Aghajanyan et al. (2021) is that fine-tuning tasks can have low intrinsic dimensionality: useful task changes may live in a much smaller subspace than the full weight space. That explains why small-rank adapters can work at all. The question is not "bigger rank is better" but "what is the intrinsic dimension of this specific task?"

## The Minimal Experiment to Justify a Production Rank

You do not need a full hyperparameter sweep. You need a rank ablation on the existing stack, evaluated on the `tenacious-bench` dev partition rather than the sealed held-out set:

```python
ranks_to_test = [1, 4, 8, 16, 32]
```

For each rank, train with identical configuration: same epochs, learning rate, seed, module set, and training data. Evaluate with the existing `scoring_evaluator.py`. Record three numbers per rank:

| Metric | Why it matters |
| --- | --- |
| Dev pass rate (%) | Quality signal: is more rank actually helping? |
| Adapter parameter count (M) | Size cost: proportional to `r` |
| Inference latency delta (ms p95) | Runtime cost: measured on the existing eval loop |

Plot pass rate vs rank. The right production rank is the smallest `r` at which the curve has flattened. Everything to the right pays size and possible runtime cost for no meaningful quality gain.

The resulting plot converts "I used `r=16`" into a defensible statement: "I used `r=16` because it sits at the quality plateau, and larger ranks do not produce enough gain to justify the additional adapter footprint."

## What Defensible for Production Looks Like

A rank is defensible for production when you can make three statements with evidence:

1. **Quality claim:** Ranks above `r=X` show no meaningful improvement on the `tenacious-bench` dev set.
2. **Cost claim:** Each rank increase changes adapter size and measured p95 latency by a known amount on the target hardware.
3. **Pareto claim:** `r=X` sits at the quality plateau with the lowest cost footprint.

Without these three statements, a rank in a model card is a benchmark number. With them, it is a production decision a client engineer can interrogate and a future FDE can revisit when the training data changes.

## Applied to the Tenacious Conversion Engine

Gersum's conversion engine has a LoRA component contributing to quality and cost outcomes across the `enrich -> compose -> qualify -> book -> sync` pipeline. The rank ablation above produces:

- a revised model card section that names the quality plateau and cites the experiment;
- a deployment recommendation that can quote per-call latency delta at the chosen rank;
- a retraining trigger: if the training data grows substantially, re-run the ablation because the task's intrinsic dimension may shift.

The rank you trained with may turn out to be the right answer. The experiment makes that finding defensible rather than accidental.

## Pointers

- Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models."  
  https://arxiv.org/abs/2106.09685
- Aghajanyan et al., "Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning."  
  https://aclanthology.org/2021.acl-long.568/
- Hugging Face PEFT LoRA documentation.  
  https://huggingface.co/docs/peft/package_reference/lora
