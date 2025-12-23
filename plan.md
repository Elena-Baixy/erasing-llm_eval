# Plan
## Objective
To develop a principled approach for erasing broad conceptual knowledge from language models by leveraging the model's own introspective classification capabilities to reduce generation probabilities for concept-specific content while preserving broader model capabilities.

## Hypothesis
1. Language models can act as their own critics to evaluate whether text belongs to a particular concept, enabling self-classification as a natural objective for unlearning.
2. Effective concept erasure requires modifying the model to reduce the likelihood of generating text it would classify as containing the target concept, rather than reversing gradients or manipulating representations.
3. Low-rank adapters applied to early model layers enable precise knowledge modification while maintaining broader capabilities.

## Methodology
1. ELM uses introspective classification by leveraging implicit model probabilities with two context prompts: c− representing the concept to erase (expert) and c+ representing an alternative (novice), to modify generation distributions via probability ratios.
2. The method combines three loss terms: Lerase (cross-entropy between ELM model and classifier-modified distribution), Lretain (preserve behavior on safe concepts), and optionally Lfluency (maintain coherent generation for smaller models).
3. Low-rank adapters (LoRA) are trained on early model layers (layers 4-7 for Zephyr-7B, rank 4, η=500) to target factual knowledge localization while avoiding damage to unrelated knowledge.
4. Training data consists of erase datasets (5,000 WMDP-Bio, 1,000 WMDP-Cyber, or 3,000 Harry Potter texts, max 700 chars each) and retain datasets from safe concepts, prepended with expert/novice context prompts.

## Experiments
### WMDP biosecurity and cybersecurity concept erasure
- What varied: Model architecture (Zephyr-7B, Mistral-7B, Llama3-8B, Llama3-8B-Instruct, Qwen2.5-32B, Llama3-70B) and baseline methods (RMU, RepNoise, ELM)
- Metric: Innocence (WMDP-Bio/Cyber MCQ accuracy, lower is better; target ~25% random), Specificity (MMLU, MT-Bench, higher is better), Seamlessness (Reverse perplexity R-PPL, lower is better)
- Main result: ELM achieves near-random performance on WMDP (Bio: 29.7-33.7%, Cyber: 26.6-28.2%) while maintaining MMLU (75.2-78.8%) and MT-Bench (7.1-7.9) scores, with better fluency (R-PPL 4.3-10.9) than baselines RMU and RepNoise.

### Ablation study of loss components
- What varied: Presence/absence of Lerase, Lretain, Lfluency terms and their weights (λ1, λ2, λ3) on Zephyr-7B
- Metric: WMDP-Bio accuracy (innocence), MMLU (specificity), R-PPL (seamlessness)
- Main result: Lerase is crucial for erasure (w/o: 64.8% Bio vs. 29.7% with), Lretain vital for specificity (w/o: 23.6% MMLU vs. 56.6% with), Lfluency essential for coherence (w/o: 29.8 R-PPL vs. 11.0 with).

### Robustness to adversarial attacks
- What varied: Attack method (GCG with 5000 iterations, BEAST) on ELM vs. original models
- Metric: Ability to induce harmful generation post-attack
- Main result: ELM resists GCG even after 5000 steps while original models succumb within 200 iterations. BEAST also fails to extract erased information from ELM.

### Internal representation analysis
- What varied: Method (ELM, RMU, RepNoise) analyzing probing accuracy and activation norms across layers
- Metric: Linear probe accuracy for WMDP concepts across layers, activation norm distribution
- Main result: ELM and RMU achieve near-random probing accuracies across all layers. ELM activation norms return to baseline in middle layers while RMU shows persistent disruption affecting fluency.

### Harry Potter literary domain erasure
- What varied: Method (ELM, RMU, WHP) on Llama-2-7B Chat
- Metric: HP-MCQ accuracy (innocence), MMLU (specificity), R-PPL (seamlessness)
- Main result: ELM achieves 38.3% HP-MCQ (better erasure than WHP 58.6% and RMU 51.0%) while maintaining 45.3% MMLU and 3.4 R-PPL, demonstrating balanced erasure and fluency.

### Hyperparameter analysis
- What varied: LoRA rank, erasure strength η, layer range (early vs. late layers)
- Metric: WMDP erasure efficacy and general benchmark performance
- Main result: Early layers (4-7) more effective than late layers for erasure. No clear trend with LoRA rank; lower ranks perform comparably. Optimal config: rank 4, η=500, layers 4-7.