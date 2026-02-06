# Configuration

This guide covers all configuration options for LLM_PPOpt.

## Configuration File Structure

The main configuration file `config.yaml` is divided into several sections:

```yaml
api:              # LLM API settings
paths:            # Input/output directories
persona_generation:   # Persona sampling settings
formulation:      # System prompt generation
query_generation: # Query adaptation settings
interaction_generation:  # Dialogue simulation
distractor:       # Noise injection settings
training_data:    # Export settings
experiment:       # Pipeline control
```

## API Configuration

### Single Provider

```yaml
api:
  provider: "openai"          # Provider: openai, anthropic, openrouter
  api_key: "sk-..."           # API key (or use environment variable)
  model: "gpt-4o-mini"        # Model name
  inference:
    temperature: 0.7          # Sampling temperature (0-2)
    max_completion_tokens: 2048  # Max tokens per response
```

### Environment Variable Reference

```yaml
api:
  api_key: "${OPENAI_API_KEY}"  # References environment variable
```

### OpenRouter Configuration

```yaml
api:
  provider: "openrouter"
  model: "anthropic/claude-3.5-haiku"

openrouter:
  api_key: "${OPENROUTER_API_KEY}"
  base_url: "https://openrouter.ai/api/v1"
```

## Paths Configuration

```yaml
paths:
  # Input files
  persona_bank:
    persona: "input/persona.yaml"
  sampling:
    config: "input/sampling_config.yaml"

  # Output directories
  persona_specs:
    output_dir: "output/personas"
    format: "json"
  queries:
    output_dir: "output/queries"
  interactions:
    output_dir: "output/interactions"
  training_data:
    output_dir: "output/training_data"

  # Logs
  logs:
    dir: "logs"
    level: "INFO"  # DEBUG, INFO, WARNING, ERROR
```

## Persona Generation Configuration

```yaml
persona_generation:
  num_personas: 1000          # Number of personas to generate
  sampling_strategy: "random" # Sampling strategy
  feature_availability_rate: 0.7  # % of features per persona

  diversity:
    enabled: true             # Enable diversity constraints
    min_hamming_distance: 3   # Minimum feature difference
    max_retries: 100          # Max retries for unique persona
```

### Sampling Strategies

| Strategy | Description |
|----------|-------------|
| `random` | Random sampling with constraints |
| `stratified` | Balanced distribution across dimensions |
| `weighted` | Custom weights per dimension |

## Formulation Configuration

System prompt generation from persona features:

```yaml
formulation:
  system_prompt_template: "prompts/persona_to_system_prompt.txt"
  max_retries: 3              # Retries on API failure
  retry_delay: 2              # Seconds between retries
  validate_output: true       # Validate generated prompts
  min_prompt_length: 50       # Minimum prompt length
```

## Query Generation Configuration

```yaml
query_generation:
  # Query source
  dataset:
    path: "input/query.jsonl"
    max_queries: null         # null = use all queries

  # Selection per persona
  selection:
    queries_per_persona: 5    # Queries assigned per persona

  # Style transfer (adapt query to persona)
  style_transfer:
    enabled: true
    transfer_probability: 0.5  # Probability of applying transfer
    template: "prompts/query_style_transfer.txt"
```

## Interaction Generation Configuration

```yaml
interaction_generation:
  min_turns: 2                # Minimum conversation turns
  max_turns: 5                # Maximum conversation turns
  max_workers: 20             # Parallel workers
  max_retries: 3              # Retries per interaction
  retry_delay: 2              # Seconds between retries
  max_supplement_rounds: 3    # Extra rounds for failed interactions

  # Assistant model configuration
  assistant_model:
    # Option 1: Single model
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.7
    max_completion_tokens: 1024

    # Option 2: Weighted model pool
    model_pool:
      - provider: openai
        model: gpt-4o-mini
        weight: 0.2
      - provider: openai
        model: gpt-4o
        weight: 0.2
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.3
      - provider: openrouter
        model: google/gemini-2.0-flash-exp
        weight: 0.3

  # User feedback model
  user_model:
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.8
    max_completion_tokens: 256
    system_prompt_template: "prompts/user_feedback.txt"
    feedback_probability: 0.6
```

## Distractor Configuration

Noise injection for model robustness:

```yaml
distractor:
  enabled: true               # Enable noise injection
  use_semantic: true          # Use semantic 3-layer distractor
  activation_probability: 0.25  # % of queries to add noise
  strategy_path: "input/distractor_strategy.yaml"

  # Legacy rule-based strategies (use_semantic: false)
  noise_strategies:
    - name: typo
      probability: 0.3
      intensity: 0.1
    - name: word_swap
      probability: 0.2
      num_swaps: 2
    - name: word_insertion
      probability: 0.2
      insertion_rate: 0.05
```

### Semantic Distractor Layers

When `use_semantic: true`, noise is applied via three layers:

| Layer | Weight | Description |
|-------|--------|-------------|
| `surface_noise` | 50% | Intent/slots preserved, surface form changes |
| `incomplete_info` | 30% | Intent clear, slots missing/vague |
| `semantic_ambiguity` | 20% | Intent uncertain or multiple intents |

Configure in `input/distractor_strategy.yaml`:

```yaml
layer_weights:
  surface_noise: 0.5
  incomplete_info: 0.3
  semantic_ambiguity: 0.2

strategies:
  surface_noise:
    colloquial_speech:
      mandatory: true
    incomplete_sentence:
      mandatory: true
    typo_misspelling:
      probability: 0.3
  incomplete_info:
    missing_slots:
      probability: 0.5
    vague_slot_values:
      probability: 0.6
  semantic_ambiguity:
    multi_intent:
      probability: 0.6
    self_contradiction:
      probability: 0.7
```

## Training Data Configuration

```yaml
training_data:
  format:
    include_persona: true      # Include persona features
    include_trajectory: true   # Include prompt trajectory
    include_distractor_versions: true  # Include noisy versions

  split:
    train: 0.8                # Training set ratio
    validation: 0.1           # Validation set ratio
    test: 0.1                 # Test set ratio
```

## Experiment Configuration

Pipeline control settings:

```yaml
experiment:
  seed: 42                    # Random seed for reproducibility
  incremental: true           # Skip existing outputs
  batch_size: 50              # Personas per batch

  # Stage control
  stages:
    persona_generation: true
    query_generation: true
    interaction_generation: true
    distractor_application: true
    training_data_export: true
```

## Configuration Validation

LLM_PPOpt validates configuration at startup:

```python
from src.config_validation import validate_config_report

config, checks, issues = validate_config_report(config, "config.yaml")

for check in checks:
    print(f"{check.status}: {check.name}")

for issue in issues:
    print(f"{issue.severity}: {issue.message}")
```

### Validation Checks

| Check | Description |
|-------|-------------|
| API key present | Validates API key is configured |
| Required files exist | Checks input files exist |
| Output directories | Creates output directories |
| Parameter ranges | Validates numeric parameters |
| Model compatibility | Checks model names are valid |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OPENAI_API_KEY` | OpenAI API key |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENROUTER_API_KEY` | OpenRouter API key |

## Complete Example

```yaml
# config.yaml - Complete example
api:
  provider: "openai"
  api_key: "${OPENAI_API_KEY}"
  model: "gpt-4o-mini"
  inference:
    temperature: 0.7
    max_completion_tokens: 2048

paths:
  persona_bank:
    persona: "input/persona.yaml"
  sampling:
    config: "input/sampling_config.yaml"
  persona_specs:
    output_dir: "output/personas"
    format: "json"
  queries:
    output_dir: "output/queries"
  interactions:
    output_dir: "output/interactions"
  training_data:
    output_dir: "output/training_data"
  logs:
    dir: "logs"
    level: "INFO"

persona_generation:
  num_personas: 100
  sampling_strategy: "random"
  feature_availability_rate: 0.7
  diversity:
    enabled: true
    min_hamming_distance: 3

query_generation:
  dataset:
    path: "input/query.jsonl"
  selection:
    queries_per_persona: 5
  style_transfer:
    enabled: true
    transfer_probability: 0.5

interaction_generation:
  min_turns: 2
  max_turns: 5
  max_workers: 10
  assistant_model:
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.7
  user_model:
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.8
    system_prompt_template: "prompts/user_feedback.txt"

distractor:
  enabled: true
  use_semantic: true
  activation_probability: 0.25
  strategy_path: "input/distractor_strategy.yaml"

training_data:
  format:
    include_persona: true
    include_trajectory: true

experiment:
  seed: 42
  incremental: true
  stages:
    persona_generation: true
    query_generation: true
    interaction_generation: true
    training_data_export: true
```

## See Also

- [Persona System](persona_system.md) - Persona dimension configuration
- [Distractor System](distractor_system.md) - Noise injection details
- [API Reference](../api/index.md) - Programmatic configuration
