# API Reference

This section provides detailed API documentation for all LLM_PPOpt modules.

## Overview

LLM_PPOpt is organized into the following modules:

| Module | Description |
|--------|-------------|
| [Pipeline](pipeline.md) | Main pipeline classes for orchestrating data generation |
| [LLM Client](llm_client.md) | Multi-provider LLM client interface |
| [Persona](persona.md) | Persona generation and management |
| [Query](query.md) | Query generation and style transfer |
| [Interaction](interaction.md) | Multi-turn dialogue generation |
| [Distractor](distractor.md) | Noise injection for robustness |
| [Training Data](training_data.md) | Training sample collection and export |
| [Utils](utils.md) | Utility classes (TokenTracker, Logger, etc.) |

## Quick Links

### Pipeline

```python
from src.pipeline import PersonaGenerationPipeline
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline
```

### LLM Client

```python
from src.llm_client import create_llm_client, LLMClient, OpenAIClient
```

### Persona

```python
from src.persona_bank import PersonaBank
from src.persona_spec import PersonaSpec, PersonaSpecStorage
from src.sampling import PersonaSampler
```

### Query

```python
from src.query_generator import UserQueryGenerator, QueryDataset
from src.query_storage import QueryStorage
```

### Interaction

```python
from src.interaction_generator import (
    InteractionGenerator,
    Interaction,
    Message,
    InteractionStorage
)
```

### Distractor

```python
from src.distractor import (
    DistractorModel,
    SemanticDistractorModel,
    create_distractor_model
)
from src.intent_extractor import IntentSlotExtractor
from src.noise_generator import LLMNoiseGenerator, NoiseResult
```

### Training Data

```python
from src.training_data import (
    TrainingSample,
    TrainingDataCollector,
    TrainingDataExporter
)
```

### Utils

```python
from src.token_tracker import TokenTracker, get_tracker, record_tokens
from src.colored_logger import ColoredLogger
from src.config_validation import validate_config
```

## Quick Module Reference

### Core Classes

| Class | Module | Description |
|-------|--------|-------------|
| `EnhancedPersonaGenerationPipeline` | `enhanced_pipeline` | Full 6-stage pipeline |
| `PersonaGenerationPipeline` | `pipeline` | Basic persona generation |
| `LLMClient` | `llm_client` | Abstract LLM client |
| `OpenAIClient` | `llm_client` | OpenAI implementation |
| `PersonaBank` | `persona_bank` | Persona dimension manager |
| `PersonaSpec` | `persona_spec` | Persona specification |
| `PersonaSampler` | `sampling` | Persona feature sampler |
| `UserQueryGenerator` | `query_generator` | Query generation |
| `InteractionGenerator` | `interaction_generator` | Dialogue simulation |
| `DistractorModel` | `distractor` | Rule-based noise |
| `SemanticDistractorModel` | `distractor` | Semantic noise |
| `TokenTracker` | `token_tracker` | Token usage tracking |

### Data Classes

| Class | Module | Description |
|-------|--------|-------------|
| `PersonaSpec` | `persona_spec` | Persona specification |
| `Interaction` | `interaction_generator` | Conversation record |
| `Message` | `interaction_generator` | Single message |
| `TrainingSample` | `training_data` | Training data sample |
| `NoiseResult` | `noise_generator` | Noise application result |
| `NoisyVersion` | `distractor` | Legacy noise result |
| `TokenUsage` | `token_tracker` | Token usage record |

### Factory Functions

| Function | Module | Description |
|----------|--------|-------------|
| `create_llm_client()` | `llm_client` | Create LLM client from config |
| `create_distractor_model()` | `distractor` | Create distractor from config |
| `generate_persona_id()` | `persona_spec` | Generate persona ID |
| `get_tracker()` | `token_tracker` | Get singleton TokenTracker |
| `record_tokens()` | `token_tracker` | Record token usage |
| `validate_config()` | `config_validation` | Validate configuration |

## Module Dependencies

```
enhanced_pipeline
├── persona_bank
├── sampling
├── persona_spec
├── llm_client
├── query_generator
│   └── query_storage
├── interaction_generator
│   ├── llm_client
│   └── distractor
│       ├── intent_extractor
│       └── noise_generator
├── training_data
└── token_tracker
```

## Type Hints

All modules use Python type hints for better IDE support:

```python
from typing import Dict, List, Optional, Any

def generate_interaction(
    persona_id: str,
    persona_features: Dict[str, str],
    initial_query: str,
    system_prompt: Optional[str] = None
) -> Optional[Interaction]:
    ...
```

## Error Handling

Common exceptions:

| Exception | Module | Description |
|-----------|--------|-------------|
| `FileNotFoundError` | Various | Missing input files |
| `ValueError` | Various | Invalid configuration |
| `APIError` | `llm_client` | LLM API errors |
| `RateLimitError` | `llm_client` | API rate limiting |

## Search

Use Sphinx search to find specific classes, methods, or functions.
