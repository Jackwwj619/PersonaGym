# Distractor API

Noise injection for model robustness.

## Factory Function

```python
from src.distractor import create_distractor_model

def create_distractor_model(
    config: Dict[str, Any],
    use_semantic: bool = True,
    llm_client: Optional[LLMClient] = None
) -> Union[DistractorModel, SemanticDistractorModel]:
    """
    Create appropriate distractor model.

    Args:
        config: Configuration dictionary
        use_semantic: Use 3-layer semantic distractor (True) or legacy (False)
        llm_client: Optional LLM client

    Returns:
        DistractorModel or SemanticDistractorModel instance
    """
```

---

## SemanticDistractorModel

```python
from src.distractor import SemanticDistractorModel
```

### Class Definition

```python
class SemanticDistractorModel:
    """
    Semantic-aware distractor with three-layer noise injection.

    Layers:
    1. Surface Noise (50%): Intent/slots preserved
    2. Incomplete Info (30%): Intent clear, slots missing
    3. Semantic Ambiguity (20%): Intent uncertain
    """
```

### Constructor

```python
def __init__(
    self,
    config: Dict[str, Any],
    strategy_config: Optional[Dict[str, Any]] = None,
    llm_client: Optional[LLMClient] = None
):
    """
    Initialize semantic distractor.

    Args:
        config: Main configuration
        strategy_config: Strategy config (or loads from strategy_path)
        llm_client: LLM client for noise generation
    """
```

### Methods

```python
def apply_noise(
    self,
    text: str,
    force: bool = False,
    persona_features: Optional[Dict[str, Any]] = None
) -> NoiseResult:
    """
    Apply semantic noise to text.

    Args:
        text: Original text
        force: Ignore activation probability
        persona_features: Optional persona for personalized noise

    Returns:
        NoiseResult with noisy text and metadata
    """

def apply_noise_batch(
    self,
    texts: List[str],
    show_progress: bool = True,
    persona_features: Optional[Dict[str, Any]] = None
) -> List[NoiseResult]:
    """Apply noise to multiple texts."""
```

### Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `enabled` | `bool` | Whether distractor is enabled |
| `activation_probability` | `float` | Probability of applying noise |
| `layer_weights` | `Dict[str, float]` | Weight per layer |
| `extractor` | `IntentSlotExtractor` | Semantic extractor |
| `generator` | `LLMNoiseGenerator` | Noise generator |

---

## DistractorModel

```python
from src.distractor import DistractorModel
```

### Class Definition

```python
class DistractorModel:
    """
    Rule-based distractor (legacy).

    Strategies: typo, word_swap, word_deletion, word_insertion,
                synonym_replacement, paraphrase
    """
```

### Methods

```python
def apply_noise(self, text: str, force: bool = False) -> List[NoisyVersion]:
    """
    Apply noise strategies to text.

    Returns:
        List of NoisyVersion objects
    """

def apply_to_interaction(self, interaction: Dict) -> Dict:
    """Apply noise to an interaction."""
```

---

## NoiseResult

```python
from src.noise_generator import NoiseResult
```

### Class Definition

```python
@dataclass
class NoiseResult:
    """
    Result of semantic noise application.

    Attributes:
        original_text: Original clean text
        noisy_text: Text with noise applied
        layer: Layer name (surface_noise, incomplete_info, semantic_ambiguity)
        layer_index: Layer number (1, 2, 3)
        applied_strategies: List of strategy names
        metadata: Additional information
        semantics: Extracted intent/slots
    """
```

### Methods

```python
def to_dict(self) -> Dict[str, Any]:
    """Convert to dictionary."""
```

---

## NoisyVersion

```python
from src.distractor import NoisyVersion
```

### Class Definition

```python
@dataclass
class NoisyVersion:
    """
    Legacy noise result.

    Attributes:
        original_text: Original text
        noisy_text: Noisy text
        noise_type: Strategy name
        noise_intensity: Noise level (0-1)
        metadata: Strategy configuration
    """
```

---

## IntentSlotExtractor

```python
from src.intent_extractor import IntentSlotExtractor
```

### Class Definition

```python
class IntentSlotExtractor:
    """Extracts intent and slots from text using LLM."""
```

### Methods

```python
def extract(self, text: str) -> ExtractedSemantics:
    """
    Extract intent and slots.

    Returns:
        ExtractedSemantics with intent, slots, confidence
    """
```

---

## LLMNoiseGenerator

```python
from src.noise_generator import LLMNoiseGenerator
```

### Class Definition

```python
class LLMNoiseGenerator:
    """Generates noisy text using LLM based on strategies."""
```

### Methods

```python
def generate_with_result(
    self,
    semantics: ExtractedSemantics,
    layer: str,
    layer_index: int,
    strategies: List[str],
    persona_features: Optional[Dict] = None
) -> NoiseResult:
    """
    Generate noisy version.

    Args:
        semantics: Extracted intent/slots
        layer: Target layer
        layer_index: Layer number
        strategies: Strategies to apply
        persona_features: Optional persona context

    Returns:
        NoiseResult with generated noise
    """
```

---

## Usage Examples

### Semantic Distractor

```python
from src.distractor import SemanticDistractorModel

distractor = SemanticDistractorModel(config)

result = distractor.apply_noise(
    text="Help me write a Python script for data processing",
    persona_features={'communication_style': 'casual'}
)

print(f"Original: {result.original_text}")
print(f"Noisy: {result.noisy_text}")
print(f"Layer: {result.layer}")
print(f"Strategies: {result.applied_strategies}")
```

### Legacy Distractor

```python
from src.distractor import DistractorModel

distractor = DistractorModel(config)

versions = distractor.apply_noise("Help me write code")

for v in versions:
    print(f"{v.noise_type}: {v.noisy_text}")
```

### Intent Extraction

```python
from src.intent_extractor import IntentSlotExtractor

extractor = IntentSlotExtractor(llm_client)

semantics = extractor.extract("Book a flight to Paris for Monday")

print(f"Intent: {semantics.intent}")
print(f"Slots: {semantics.slots}")
```

---

## See Also

- [Distractor System](../user_guide/distractor_system.md) - User guide
- [Configuration](../user_guide/configuration.md) - Distractor configuration
