# Tutorial 1: Basics

Learn the core concepts of LLM_PPOpt and run your first pipeline.

## Learning Objectives

By the end of this tutorial, you will:

- Understand the 6-stage pipeline architecture
- Run a basic pipeline execution
- Interpret the output files

## Pipeline Overview

LLM_PPOpt generates synthetic training data through 6 stages:

```
1. Persona Generation    → Create diverse user profiles
2. System Prompt         → Generate personalized prompts
3. Query Generation      → Assign and adapt queries
4. Interaction           → Simulate conversations
5. Distractor            → Add realistic noise
6. Training Data Export  → Output structured samples
```

## Step 1: Configuration

First, review the main configuration file:

```yaml
# config.yaml (key sections)

api:
  provider: "openai"
  model: "gpt-4o-mini"

persona_generation:
  num_personas: 10           # Start small

query_generation:
  selection:
    queries_per_persona: 3   # 3 queries per persona

interaction_generation:
  min_turns: 2
  max_turns: 4

distractor:
  enabled: true
  activation_probability: 0.25
```

## Step 2: First Run

Run the pipeline with a small number of personas:

```bash
python run.py --num-personas 5
```

Expected output:

```
================================================================================
Starting Enhanced Persona Generation Pipeline
Mode: INCREMENTAL (will skip existing outputs)
================================================================================

[Stage 1] Generating Personas...
Generating personas: 100%|████████████████████| 5/5 [00:15<00:00]
[OK] Generated 5 personas

[Stage 2] Generating Queries - batch 1...
[OK] Generated queries for 5 personas

[Stage 3] Generating Interactions - batch 1...
Generating interactions: 100%|████████████████████| 15/15 [01:30<00:00]
[OK] Generated 15 interactions (batch 1)

[Stage 4] Distractor Application - SKIPPED (already applied in Stage 3)

[Stage 5] Collecting and Exporting Training Data...
[OK] Exported training data

================================================================================
Pipeline Complete
================================================================================

PIPELINE SUMMARY:
  Personas Generated: 5
  Queries Generated: 15
  Interactions Generated: 15
  Training Samples: 15

TOKEN USAGE SUMMARY:
  Total API Calls: 85
  Total Input Tokens: 42,500
  Total Output Tokens: 25,000
  Total Tokens: 67,500
```

## Step 3: Explore Output

After running, check the output directory:

```
output/
├── personas/
│   ├── persona_20260206_001.json
│   ├── persona_20260206_002.json
│   └── ...
├── queries/
│   ├── persona_20260206_001.json
│   └── ...
├── interactions/
│   ├── interaction_persona_001_*.json
│   ├── index.json
│   └── ...
└── training_data/
    ├── train_samples_20260206_*.json
    ├── statistics.json
    └── token_usage_*.json
```

### View a Persona

```bash
cat output/personas/persona_20260206_001.json
```

```json
{
  "persona_id": "persona_20260206_001",
  "features": {
    "age_band": "25_34",
    "role": "engineer",
    "communication_style": "casual",
    "response_length": "medium"
  },
  "system_prompt": "You are helping a 25-34 year old engineer...",
  "metadata": {
    "created_at": "2026-02-06T10:30:00"
  }
}
```

### View an Interaction

```bash
cat output/interactions/interaction_persona_001_*.json
```

Key fields:

- `messages`: The conversation turns
- `num_turns`: Number of exchanges
- `metadata.distractor_applied`: Whether noise was added

### View Training Data

```bash
cat output/training_data/train_samples_*.json | head -50
```

Key fields:

- `prompt_trajectory`: All user prompts in order
- `full_conversation`: Complete dialogue
- `noisy_initial_queries`: Noise variations

## Step 4: Programmatic Usage

You can also run the pipeline from Python:

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

# Initialize
pipeline = EnhancedPersonaGenerationPipeline("config.yaml")

# Run
result = pipeline.run(num_personas=5)

# Access results
print(f"Personas: {len(result['personas'])}")
print(f"Interactions: {len(result['interactions'])}")

# Inspect first persona
persona = result['personas'][0]
print(f"ID: {persona['persona_id']}")
print(f"Features: {persona['features']}")
```

## Key Concepts

### Incremental Mode

By default, the pipeline skips existing outputs:

```yaml
experiment:
  incremental: true
```

Run again with more personas:

```bash
python run.py --num-personas 10
# Only generates 5 new personas (5 already exist)
```

### Stage Control

Run specific stages:

```bash
# Only persona generation
python run.py --stage persona --num-personas 20

# Only training data export
python run.py --stage training
```

### Skip Features

```bash
# Skip noise injection
python run.py --skip-distractor

# Skip query style transfer
python run.py --skip-query-transfer
```

## Exercises

1. **Run with 10 personas** and compare token usage
2. **Disable distractor** and observe the difference in outputs
3. **Check statistics.json** to understand data distribution

## Next Steps

Continue to [Tutorial 2: Persona Generation](tutorial_02_persona_generation.md) to learn about customizing personas.
