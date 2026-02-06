# Pipeline API

Main pipeline classes for orchestrating data generation.

## EnhancedPersonaGenerationPipeline

Complete 6-stage pipeline for generating synthetic training data.

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline
```

### Class Definition

```python
class EnhancedPersonaGenerationPipeline:
    """
    Complete pipeline for generating synthetic training data.

    Stages:
    1. Persona Generation
    2. System Prompt Formulation
    3. Query Generation with Style Transfer
    4. Multi-turn Interaction Generation
    5. Distractor/Noise Application
    6. Training Data Collection and Export
    """
```

### Constructor

```python
def __init__(self, config_path: str):
    """
    Initialize the enhanced pipeline.

    Args:
        config_path: Path to configuration YAML file

    Raises:
        FileNotFoundError: If config file not found
        ValueError: If config validation fails
    """
```

### Methods

#### run

```python
def run(self, num_personas: Optional[int] = None) -> Dict[str, Any]:
    """
    Run the complete pipeline.

    Args:
        num_personas: Number of personas to generate.
                     If None, uses value from config.

    Returns:
        Dictionary with pipeline results:
        - 'personas': List of persona dictionaries
        - 'queries': Dict mapping persona_id to queries
        - 'interactions': List of interaction dictionaries
        - 'enhanced_interactions': Interactions with noise (deprecated)
        - 'training_data': Export statistics

    Example:
        >>> pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
        >>> result = pipeline.run(num_personas=10)
        >>> print(f"Generated {len(result['personas'])} personas")
    """
```

### Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `config` | `Dict[str, Any]` | Loaded configuration |
| `stages` | `Dict[str, bool]` | Which stages to run |
| `incremental` | `bool` | Skip existing outputs |
| `token_tracker` | `TokenTracker` | Token usage tracker |
| `llm_client` | `LLMClient` | Main LLM client |
| `persona_bank` | `PersonaBank` | Persona dimensions |
| `sampler` | `PersonaSampler` | Feature sampler |
| `query_generator` | `UserQueryGenerator` | Query generator |
| `interaction_generator` | `InteractionGenerator` | Dialogue generator |
| `distractor` | `DistractorModel` | Noise injector |
| `training_collector` | `TrainingDataCollector` | Sample collector |
| `training_exporter` | `TrainingDataExporter` | Sample exporter |

### Usage Example

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

# Initialize pipeline
pipeline = EnhancedPersonaGenerationPipeline("config.yaml")

# Run full pipeline
result = pipeline.run(num_personas=100)

# Access results
print(f"Personas: {len(result['personas'])}")
print(f"Queries: {sum(len(q) for q in result['queries'].values())}")
print(f"Interactions: {len(result['interactions'])}")
print(f"Training samples: {result['training_data']['total_samples']}")
```

---

## PersonaGenerationPipeline

Basic pipeline for persona generation only (without interactions).

```python
from src.pipeline import PersonaGenerationPipeline
```

### Class Definition

```python
class PersonaGenerationPipeline:
    """
    Basic pipeline for generating personas with system prompts.

    Use this for:
    - Generating personas without interactions
    - Lightweight persona generation at scale
    - Exporting persona datasets
    """
```

### Constructor

```python
def __init__(self, config_path: str):
    """
    Initialize the basic pipeline.

    Args:
        config_path: Path to configuration YAML file
    """
```

### Methods

#### run

```python
def run(
    self,
    num_personas: Optional[int] = None,
    generate_prompts: bool = True,
    export_dataset: bool = False,
    dataset_path: Optional[str] = None
) -> Dict[str, Any]:
    """
    Run the persona generation pipeline.

    Args:
        num_personas: Number of personas to generate
        generate_prompts: Whether to generate system prompts
        export_dataset: Whether to export to dataset file
        dataset_path: Custom export path

    Returns:
        Dictionary with:
        - 'num_personas': Count of generated personas
        - 'personas': List of PersonaSpec objects
        - 'dataset_path': Path to exported dataset (if exported)
    """
```

#### reset

```python
def reset(self) -> None:
    """
    Reset pipeline state.

    Clears stored personas and resets sampler state.
    """
```

### Usage Example

```python
from src.pipeline import PersonaGenerationPipeline

# Initialize
pipeline = PersonaGenerationPipeline("config.yaml")

# Generate personas only
result = pipeline.run(
    num_personas=1000,
    generate_prompts=True,
    export_dataset=True,
    dataset_path="personas.json"
)

print(f"Generated {result['num_personas']} personas")
print(f"Exported to {result['dataset_path']}")
```

---

## Command Line Interface

The `run.py` script provides CLI access to both pipelines.

### Usage

```bash
python run.py [OPTIONS]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--config PATH` | Configuration file path | `config.yaml` |
| `--num-personas NUM` | Number of personas | From config |
| `--mode {basic,enhanced}` | Pipeline mode | `enhanced` |
| `--stage STAGE` | Run specific stage | `all` |
| `--skip-query-transfer` | Skip style transfer | `False` |
| `--skip-distractor` | Skip noise injection | `False` |
| `--reset` | Reset pipeline state | `False` |

### Stages

| Stage | Description |
|-------|-------------|
| `all` | Run all stages |
| `persona` | Persona generation only |
| `query` | Query generation only |
| `interaction` | Interaction generation only |
| `distractor` | Distractor application only |
| `training` | Training data export only |

### Examples

```bash
# Full enhanced pipeline
python run.py --num-personas 100

# Basic mode only
python run.py --mode basic --num-personas 1000

# Skip noise injection
python run.py --skip-distractor --num-personas 50

# Run specific stage
python run.py --stage interaction
```

---

## See Also

- [Configuration](../user_guide/configuration.md) - Pipeline configuration
- [Quick Start](../quickstart.md) - Getting started guide
