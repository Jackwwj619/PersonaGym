# Distractor System

This guide covers the noise injection system for enhancing model robustness.

## Overview

The distractor system adds realistic noise to user queries, simulating imperfect real-world inputs:

- **Typos and misspellings**
- **Incomplete sentences**
- **Ambiguous requests**
- **Missing information**

This helps train models that are robust to imperfect user inputs.

## Architecture

LLM_PPOpt provides two distractor models:

| Model | Description | Use Case |
|-------|-------------|----------|
| `DistractorModel` | Rule-based noise (legacy) | Fast, deterministic |
| `SemanticDistractorModel` | LLM-based three-layer noise | Realistic, semantic-aware |

## Three-Layer Semantic Distractor

### Layer Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: Surface Noise (50%)                                   │
│  - Intent and slots fully preserved                             │
│  - Only surface expression changes                              │
│  - Examples: typos, colloquial speech, incomplete sentences     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  Layer 2: Incomplete Information (30%)                          │
│  - Intent clear                                                  │
│  - Slots missing or vague                                        │
│  - Examples: missing parameters, vague values, unclear priority │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  Layer 3: Semantic Ambiguity (20%)                              │
│  - Intent uncertain or multiple intents                         │
│  - Examples: contradictions, mind-changing, multi-intent        │
└─────────────────────────────────────────────────────────────────┘
```

### Layer 1: Surface Noise

Intent and slots are fully preserved; only the surface form changes.

| Strategy | Description | Example |
|----------|-------------|---------|
| `colloquial_speech` | Informal language | "Please help" → "hey can u help" |
| `incomplete_sentence` | Fragments | "I need help with..." |
| `typo_misspelling` | Typos | "Python" → "Pyhton" |
| `punctuation_format` | Punctuation changes | Remove periods, add "???" |
| `emotional_attitude` | Emotional expression | "This is frustrating!!!" |
| `synonym_rewrite` | Word substitution | "quick" → "fast" |
| `word_segmentation` | Wrong word breaks | "database" → "data base" |
| `negative_expression` | Negation | "I want X" → "I don't want not X" |
| `real_life_fragments` | Natural speech | "so like... I need..." |

### Layer 2: Incomplete Information

Intent is clear but slots are missing or vague.

| Strategy | Description | Example |
|----------|-------------|---------|
| `missing_slots` | Omit required info | "Book a flight" (no destination) |
| `vague_slot_values` | Unclear values | "sometime next week" |
| `context_dependency` | Implicit references | "Do the same thing" |
| `unclear_priority` | Ambiguous importance | "Maybe also add..." |

### Layer 3: Semantic Ambiguity

Intent itself is uncertain or multiple.

| Strategy | Description | Example |
|----------|-------------|---------|
| `multi_intent` | Multiple requests | "Fix the bug and also refactor" |
| `intent_ambiguity` | Unclear goal | "Make it better" |
| `self_contradiction` | Conflicting requirements | "Make it simple but comprehensive" |
| `mind_changing` | Changed requirements | "Actually, never mind, do X instead" |

## Configuration

### Enable Semantic Distractor

```yaml
distractor:
  enabled: true
  use_semantic: true              # Use 3-layer semantic distractor
  activation_probability: 0.25    # 25% of queries get noise
  strategy_path: "input/distractor_strategy.yaml"
```

### Strategy Configuration

`input/distractor_strategy.yaml`:

```yaml
layer_weights:
  surface_noise: 0.5
  incomplete_info: 0.3
  semantic_ambiguity: 0.2

strategies:
  surface_noise:
    colloquial_speech:
      mandatory: true           # Always apply
    incomplete_sentence:
      mandatory: true
    typo_misspelling:
      probability: 0.3          # 30% chance
    punctuation_format:
      probability: 0.5
    emotional_attitude:
      probability: 0.5

  incomplete_info:
    missing_slots:
      probability: 0.5
    vague_slot_values:
      probability: 0.6
    unclear_priority:
      probability: 0.7

  semantic_ambiguity:
    multi_intent:
      probability: 0.6
    self_contradiction:
      probability: 0.7
    mind_changing:
      probability: 0.4
```

## Programmatic Usage

### Create Distractor

```python
from src.distractor import create_distractor_model

# Semantic distractor (recommended)
distractor = create_distractor_model(
    config,
    use_semantic=True,
    llm_client=llm_client
)

# Legacy rule-based distractor
distractor = create_distractor_model(
    config,
    use_semantic=False
)
```

### Apply Noise

```python
from src.distractor import SemanticDistractorModel

distractor = SemanticDistractorModel(config)

# Apply noise to text
result = distractor.apply_noise(
    text="Help me write a Python script",
    persona_features={'communication_style': 'casual'}
)

print(f"Original: {result.original_text}")
print(f"Noisy: {result.noisy_text}")
print(f"Layer: {result.layer}")
print(f"Strategies: {result.applied_strategies}")
```

### Batch Processing

```python
texts = [
    "Help me with Python",
    "Write an email",
    "Debug this code"
]

results = distractor.apply_noise_batch(
    texts,
    show_progress=True,
    persona_features=features
)

for result in results:
    print(f"{result.original_text} → {result.noisy_text}")
```

## NoiseResult Data Structure

```python
@dataclass
class NoiseResult:
    original_text: str          # Original clean text
    noisy_text: str             # Text with noise applied
    layer: str                  # Layer name (surface_noise, etc.)
    layer_index: int            # Layer number (1, 2, 3)
    applied_strategies: List[str]  # Strategies applied
    metadata: Optional[Dict]    # Additional info
    semantics: Optional[Dict]   # Extracted intent/slots

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary."""
```

### Output Example

```json
{
  "original_text": "Help me write a Python script for data processing",
  "noisy_text": "hey can u help me write some python thing for... data stuff",
  "layer": "surface_noise",
  "layer_index": 1,
  "applied_strategies": [
    "colloquial_speech",
    "incomplete_sentence"
  ],
  "semantics": {
    "intent": "code_generation",
    "slots": {
      "language": "Python",
      "task": "data processing"
    }
  }
}
```

## Intent/Slot Extraction

### ExtractedSemantics

Before applying noise, the system extracts intent and slots:

```python
from src.intent_extractor import IntentSlotExtractor

extractor = IntentSlotExtractor(llm_client)

semantics = extractor.extract("Book a flight to Paris for next Monday")

print(semantics.intent)  # "flight_booking"
print(semantics.slots)   # {"destination": "Paris", "date": "next Monday"}
```

### Preservation by Layer

| Layer | Intent | Slots |
|-------|--------|-------|
| Surface Noise | ✓ Preserved | ✓ Preserved |
| Incomplete Info | ✓ Preserved | △ Partially missing |
| Semantic Ambiguity | △ Uncertain | △ May conflict |

## Legacy Rule-Based Distractor

### Configuration

```yaml
distractor:
  enabled: true
  use_semantic: false

  noise_strategies:
    - name: typo
      probability: 0.3
      intensity: 0.1
    - name: word_swap
      probability: 0.2
      num_swaps: 2
    - name: word_deletion
      probability: 0.15
      deletion_rate: 0.05
    - name: word_insertion
      probability: 0.2
      insertion_rate: 0.05
    - name: synonym_replacement
      probability: 0.3
      replacement_rate: 0.15
```

### Available Strategies

| Strategy | Description | Parameters |
|----------|-------------|------------|
| `typo` | Keyboard typos | `intensity`: 0-1 |
| `word_swap` | Swap adjacent words | `num_swaps`: int |
| `word_deletion` | Delete random words | `deletion_rate`: 0-1 |
| `word_insertion` | Insert filler words | `insertion_rate`: 0-1 |
| `synonym_replacement` | Replace with synonyms | `replacement_rate`: 0-1 |
| `paraphrase` | LLM paraphrase | `use_llm`: bool |

### Usage

```python
from src.distractor import DistractorModel

distractor = DistractorModel(config)

# Returns list of NoisyVersion
noisy_versions = distractor.apply_noise("Help me write code")

for nv in noisy_versions:
    print(f"{nv.noise_type}: {nv.noisy_text}")
```

## Integration with Interactions

### Real-time Application

Noise is applied during interaction generation:

```python
generator = InteractionGenerator(config, distractor=distractor)

# During conversation:
# 1. Initial query gets noise
# 2. Follow-up feedback gets noise
# 3. All versions stored in metadata
```

### Metadata Structure

```json
{
  "metadata": {
    "distractor_applied": true,
    "distractor_type": "semantic",
    "semantic_noise": {
      "initial_query": {
        "original_text": "Help me write code",
        "noisy_text": "hey help me writ code",
        "layer": "surface_noise",
        "applied_strategies": ["colloquial_speech", "typo_misspelling"]
      }
    },
    "noisy_versions": {
      "followups": [
        {
          "clean_feedback": "Add error handling",
          "noisy_versions": [...],
          "used_version": "add some error stuff maybe"
        }
      ]
    }
  }
}
```

## API Reference

### SemanticDistractorModel

```python
class SemanticDistractorModel:
    """Three-layer semantic noise injection."""

    def __init__(
        self,
        config: Dict[str, Any],
        strategy_config: Optional[Dict] = None,
        llm_client: Optional[LLMClient] = None
    ):
        """Initialize with config and optional LLM client."""

    def apply_noise(
        self,
        text: str,
        force: bool = False,
        persona_features: Optional[Dict] = None
    ) -> NoiseResult:
        """Apply semantic noise to text."""

    def apply_noise_batch(
        self,
        texts: List[str],
        show_progress: bool = True,
        persona_features: Optional[Dict] = None
    ) -> List[NoiseResult]:
        """Apply noise to multiple texts."""
```

### DistractorModel

```python
class DistractorModel:
    """Rule-based noise injection (legacy)."""

    def __init__(self, config: Dict, llm_client: Optional[LLMClient] = None):
        """Initialize with config."""

    def apply_noise(self, text: str, force: bool = False) -> List[NoisyVersion]:
        """Apply noise strategies to text."""

    def apply_to_interaction(self, interaction: Dict) -> Dict:
        """Apply noise to an interaction."""
```

### Factory Function

```python
def create_distractor_model(
    config: Dict[str, Any],
    use_semantic: bool = True,
    llm_client: Optional[LLMClient] = None
) -> Union[DistractorModel, SemanticDistractorModel]:
    """Create appropriate distractor model."""
```

## Best Practices

### 1. Start with Low Activation Probability

```yaml
distractor:
  activation_probability: 0.25   # Start low, increase if needed
```

### 2. Balance Layer Weights

```yaml
layer_weights:
  surface_noise: 0.5        # Most common
  incomplete_info: 0.3      # Moderate
  semantic_ambiguity: 0.2   # Least common (hardest)
```

### 3. Use Mandatory Strategies Sparingly

```yaml
strategies:
  surface_noise:
    colloquial_speech:
      mandatory: true        # Always apply
    typo_misspelling:
      probability: 0.3       # Sometimes apply
```

### 4. Monitor Noise Quality

```python
# Check noise distribution
from collections import Counter

layers = Counter(r.layer for r in results)
print(f"Layer distribution: {layers}")
```

## See Also

- [Interaction Generation](interaction_generation.md) - How distractor integrates
- [Configuration](configuration.md) - Full distractor configuration
- [Distractor API](../api/distractor.md) - Detailed API documentation
