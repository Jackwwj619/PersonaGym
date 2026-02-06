# Quick Start

This guide will help you get started with PersonaGym in minutes.

## 5-Minute Introduction

Here's a complete workflow from configuration to training data generation:

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

# Initialize pipeline with configuration
pipeline = EnhancedPersonaGenerationPipeline("config.yaml")

# Run the complete 6-stage pipeline
result = pipeline.run(num_personas=5)

# Access results
print(f"Personas: {len(result['personas'])}")
print(f"Interactions: {len(result['interactions'])}")
print(f"Training samples: {result['training_data']['total_samples']}")
```

Or use the command line:

```bash
python run.py --num-personas 5
```

## Core Workflows

### 1. Basic Pipeline (Persona Generation Only)

For generating personas without interactions:

```bash
python run.py --mode basic --num-personas 100
```

```python
from src.pipeline import PersonaGenerationPipeline

pipeline = PersonaGenerationPipeline("config.yaml")
result = pipeline.run(
    num_personas=100,
    generate_prompts=True,
    export_dataset=True
)

print(f"Generated {result['num_personas']} personas")
```

### 2. Enhanced Pipeline (Full Data Generation)

For complete training data with interactions and noise:

```bash
python run.py --mode enhanced --num-personas 10
```

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
result = pipeline.run(num_personas=10)

# Results include all stages
personas = result['personas']           # List of persona specs
queries = result['queries']             # Dict: persona_id -> queries
interactions = result['interactions']   # List of interactions
training_data = result['training_data'] # Export statistics
```

### 3. Stage-by-Stage Execution

Run specific stages independently:

```bash
# Only persona generation
python run.py --stage persona --num-personas 50

# Only query generation (requires existing personas)
python run.py --stage query

# Only interaction generation
python run.py --stage interaction

# Only training data export
python run.py --stage training
```

### 4. Skip Specific Features

```bash
# Skip style transfer in query generation
python run.py --skip-query-transfer --num-personas 5

# Skip noise injection
python run.py --skip-distractor --num-personas 5
```

## Configuration Overview

The main configuration file `config.yaml` controls all pipeline behavior:

```yaml
# API Configuration
api:
  provider: "openai"
  api_key: "${OPENAI_API_KEY}"  # From environment
  model: "gpt-4o-mini"
  inference:
    temperature: 0.7
    max_completion_tokens: 2048

# Persona Generation
persona_generation:
  num_personas: 1000
  sampling_strategy: "random"
  feature_availability_rate: 0.7
  diversity:
    enabled: true
    min_hamming_distance: 3

# Query Generation
query_generation:
  selection:
    queries_per_persona: 5
  style_transfer:
    enabled: true
    transfer_probability: 0.5

# Interaction Generation
interaction_generation:
  min_turns: 2
  max_turns: 5
  max_workers: 20

# Distractor (Noise Injection)
distractor:
  enabled: true
  use_semantic: true
  activation_probability: 0.25

# Experiment Settings
experiment:
  seed: 42
  incremental: true
```

## Understanding Output

### Output Directory Structure

After running the pipeline:

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
    ├── train_samples_20260206_123456.json
    ├── statistics.json
    └── token_usage_20260206_123456.json
```

### Sample Output Formats

**Persona Spec** (`output/personas/persona_*.json`):

```json
{
  "persona_id": "persona_20260206_001",
  "features": {
    "age_band": "25_34",
    "role": "engineer",
    "communication_style": "casual",
    "response_length": "medium",
    "technical_level": "advanced"
  },
  "system_prompt": "You are assisting a 25-34 year old engineer...",
  "metadata": {
    "created_at": "2026-02-06T10:30:00",
    "num_features": 15
  }
}
```

**Training Sample** (`output/training_data/train_samples_*.json`):

```json
{
  "sample_id": "sample_20260206_001",
  "persona_id": "persona_001",
  "persona_features": {...},
  "original_query": "Help me write a Python script",
  "initial_query": "hey can u help me write some python code",
  "prompt_trajectory": [
    "hey can u help me write some python code",
    "make it handle errors better",
    "looks good thanks"
  ],
  "full_conversation": [
    {"role": "user", "content": "hey can u help me..."},
    {"role": "assistant", "content": "Sure! Here's a..."},
    ...
  ],
  "num_turns": 3
}
```

## Common Patterns

### Pattern 1: End-to-End Pipeline

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

# Full pipeline with all stages
pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
result = pipeline.run(num_personas=100)

# Check token usage
if 'training_data' in result:
    print(f"Total samples: {result['training_data']['total_samples']}")
```

### Pattern 2: Incremental Generation

Resume from previous runs (skips existing outputs):

```yaml
# In config.yaml
experiment:
  incremental: true
```

```bash
# First run: generates 50 personas
python run.py --num-personas 50

# Second run: skips existing 50, generates 50 more
python run.py --num-personas 100
```

### Pattern 3: Custom Persona Features

Modify `input/persona.yaml` to customize dimensions:

```yaml
dimensions:
  role:
    name: role
    is_constraint: true
    values:
      - student
      - researcher
      - developer
      - manager
      - custom_role  # Add custom values
```

### Pattern 4: Multi-Provider Model Pool

Use diverse models for assistant responses:

```yaml
# In config.yaml
interaction_generation:
  assistant_model:
    model_pool:
      - provider: openai
        model: gpt-4o-mini
        weight: 0.3
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.3
      - provider: openrouter
        model: google/gemini-2.0-flash-exp
        weight: 0.4
```

## Next Steps

- **Deep Dive**: Check out [Tutorials](tutorials/index.md) for step-by-step guides
- **Configuration**: See [Configuration Guide](user_guide/configuration.md) for all options
- **Persona System**: Learn about [Persona Dimensions](user_guide/persona_system.md)
- **API Reference**: Explore the [API Documentation](api/index.md)

## Troubleshooting

### Empty Responses

If you see "Assistant returned empty response":

1. Check your API key has sufficient credits
2. Verify the model name is correct
3. Try reducing `max_workers` to avoid rate limits

### Slow Generation

For faster generation:

```yaml
experiment:
  batch_size: 10  # Process in batches

interaction_generation:
  max_workers: 10  # Parallel workers
```

### Memory Issues

For large-scale generation:

```bash
# Generate in batches
python run.py --num-personas 100  # First batch
python run.py --num-personas 200  # Incremental adds 100 more
```

For more help, see the [Installation](installation.md) guide's troubleshooting section.
