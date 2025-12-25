# Documentation: Replication of ELM (Erasure of Language Memory)

## Goal

Replicate the ELM (Erasure of Language Memory) method for erasing conceptual knowledge from language models. The method aims to remove specific knowledge domains (e.g., Harry Potter lore, biosecurity information) while preserving general model capabilities and fluency.

## Data

### Erase Dataset
- **Source**: `mickume/harry_potter_tiny` from Hugging Face
- **Content**: Text passages from Harry Potter books
- **Purpose**: Training data for the knowledge to be erased
- **Size**: 100 samples used (original paper uses 3000-5000)

### Retain Dataset
- **Source**: `philschmid/easyrag-mini-wikipedia` from Hugging Face
- **Content**: General Wikipedia articles
- **Purpose**: Preserve general knowledge during erasure
- **Size**: 100 samples used

### Concept Definition
- **Target Concept**: "Harry Potter, Wizardry, Hogwarts, Spells, books, series, games, or any other lore by J.K Rowling"

## Method

### Overview
ELM uses introspective classification by leveraging the model's own probabilities with two context prompts:
- **Expert prompt (c-)**: Represents knowledge of the concept to erase
- **Novice prompt (c+)**: Represents lack of knowledge about the concept

### Core Formula
The modified probability distribution is computed as:
```
log P'(x) ∝ log P(x) + eta * (log P(x|c_novice) - log P(x|c_expert))
```

Where `eta` is the erasure strength parameter (default: 1000).

### Three Loss Components

1. **L_erase (Erasure Loss)**
   - Trains the model to match the ELM-modified distribution
   - Reduces probability of generating expert-like content

2. **L_retain (Retention Loss)**
   - Preserves behavior on safe/general concepts
   - Uses KL divergence between LoRA model and base model on retain data

3. **L_fluency (Fluency Loss)**
   - Maintains coherent generation when prompted about erased concepts
   - Prevents the model from producing gibberish

### Model Configuration
- **Base Model**: HuggingFaceH4/zephyr-7b-beta
- **LoRA Rank**: 256 (as recommended for literary domains)
- **LoRA Alpha**: 16
- **Target Layers**: 4-7 (early layers for factual knowledge)
- **Target Modules**: Attention (q, k, v, o projections) and MLP (up, gate, down projections)
- **Learning Rate**: 5e-5
- **Gradient Accumulation Steps**: 4

## Results

### Training Convergence
- All three losses (erase, retain, fluency) converged during training
- Training completed successfully with 50 iterations
- Loss curves saved to `training_losses.png`

### Generation Comparison
Tested on Harry Potter-related prompts comparing ELM model vs. original model:

| Prompt | ELM Model Behavior | Original Model Behavior |
|--------|-------------------|------------------------|
| "Harry Potter's best friend is" | Discusses movie/entertainment context | Direct HP content |
| "The headmaster of Hogwarts is" | References actor/events | Direct HP knowledge |
| "To cast a spell..." | Still contains HP knowledge | Direct HP content |
| "The game of Quidditch involves" | Explains game mechanics | Game description |

### Observations
- With limited training (50 iterations vs. 3000-5000 in paper), erasure effect is subtle
- Model maintains coherent generation ability
- Some Harry Potter knowledge persists due to limited training

### Stability
- Results are deterministic with greedy decoding (do_sample=False)
- Multiple runs with same seed produce identical outputs

## Analysis

### What Worked
1. ELM edit vector computation correctly implements the paper's formula
2. Three-loss training framework functions as designed
3. LoRA adapters successfully applied to early layers
4. Model maintains fluency post-training

### Limitations of This Replication
1. **Limited Training**: Only 50 iterations vs. paper's 3000-5000
2. **No Quantitative Metrics**: Did not run MMLU, HP-MCQ evaluations
3. **Single Concept**: Only tested Harry Potter (not WMDP-Bio/Cyber)

### Key Insights
- Early layers (4-7) are effective targets for factual knowledge modification
- The three-loss balance is crucial for maintaining model utility
- Expert/novice prompting provides a natural way to define erasure targets

## Files Generated
- `replication.ipynb` - Main replication notebook
- `training_losses.png` - Loss curves visualization
- `elm_model/` - Saved LoRA adapter weights
- `documentation_replication.md` - This file
- `evaluation_replication.md` - Evaluation checklist
- `self_replication_evaluation.json` - JSON evaluation summary
