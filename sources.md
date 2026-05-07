# Day 3 - sources.md

**Question:** How does LoRA rank mathematically constrain adapter capacity, and what minimal experiment on Gersum's existing stack would justify a specific rank as a production decision?

---

## Source 1 - LoRA: Low-Rank Adaptation of Large Language Models

**Citation:** Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. "LoRA: Low-Rank Adaptation of Large Language Models." arXiv:2106.09685, 2021.  
**URL:** https://arxiv.org/abs/2106.09685  
**Type:** Original research paper  
**Why it is load-bearing:** This paper defines LoRA's core mechanism: freezing base weights and learning low-rank update matrices. It supports the explainer's central claim that rank controls the dimensional bottleneck of the learned update, not just the amount of training.

## Source 2 - Hugging Face PEFT LoRA Documentation

**Citation:** Hugging Face PEFT documentation, LoRA / `LoraConfig`.  
**URL:** https://huggingface.co/docs/peft/package_reference/lora  
**Type:** Authoritative implementation documentation  
**Why it is load-bearing:** This source documents the practical configuration fields used in production LoRA training, including `r`, `lora_alpha`, `target_modules`, `rank_pattern`, and adapter behavior. It grounds the explainer's experiment design in the actual knobs a practitioner would set.

## Hands-On Pattern Used

**Pattern:** Controlled LoRA rank sweep on a fixed benchmark.  
**What it demonstrates:** Hold base model, data split, target modules, training steps, decoding, and benchmark constant while varying only `r`. This isolates whether quality and deployment tradeoffs are caused by rank-capacity changes rather than unrelated training changes.
