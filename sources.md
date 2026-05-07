# Day 3 - sources.md

**Question:** What does LoRA adapt when it targets `q_proj` and `v_proj`, and why might Q/V-only updates lower training loss while leaving held-out rubric behavior unchanged?

---

## Source 1 - LoRA: Low-Rank Adaptation of Large Language Models

**Citation:** Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. "LoRA: Low-Rank Adaptation of Large Language Models." arXiv:2106.09685, 2021.  
**URL:** https://arxiv.org/abs/2106.09685  
**Type:** Original research paper  
**Why it is load-bearing:** This paper defines LoRA as a frozen-weight model plus low-rank trainable update matrices. It supports the explainer's mechanism claim that `Delta W = BA` constrains what each targeted projection can learn, and that low-rank adaptation can work because downstream updates are often low-dimensional.

## Source 2 - Hugging Face PEFT LoRA Documentation

**Citation:** Hugging Face PEFT documentation, LoRA / `LoraConfig`.  
**URL:** https://huggingface.co/docs/peft/package_reference/lora  
**Type:** Authoritative implementation documentation  
**Why it is load-bearing:** This source documents `r`, `lora_alpha`, and `target_modules`, including examples that target modules such as `q_proj` and `v_proj`. It grounds the explainer's claim that module selection is an explicit adapter design choice, not a property LoRA decides automatically.

## Companion Source - Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning

**Citation:** Aghajanyan, A., Gupta, S., & Zettlemoyer, L. "Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning." ACL-IJCNLP 2021.  
**URL:** https://aclanthology.org/2021.acl-long.568/  
**Type:** Primary research paper  
**Why it is load-bearing for Mistire's explainer to Gersum:** This paper supports the claim that downstream fine-tuning can often be effective in a lower-dimensional subspace than the full parameter space, which helps explain why small LoRA ranks can be meaningful rather than noise.

## Hands-On Pattern Used

**Pattern:** Module-target ablation at fixed rank.  
**What it demonstrates:** Keep rank, alpha, data, training budget, and evaluation fixed while changing only the LoRA target modules: `q,v` vs `q,k,v,o` vs `q,k,v,o + MLP`. This isolates whether the flat held-out result is plausibly caused by module selection rather than rank, data, or objective alone.
