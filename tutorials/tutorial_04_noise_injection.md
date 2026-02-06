# Tutorial 4: Noise Injection

Configure the distractor system for robust model training.

## Three-Layer Architecture

| Layer | Weight | Intent | Slots |
|-------|--------|--------|-------|
| Surface Noise | 50% | Preserved | Preserved |
| Incomplete Info | 30% | Clear | Missing |
| Semantic Ambiguity | 20% | Uncertain | May conflict |

## Configuration

```yaml
distractor:
  enabled: true
  use_semantic: true
  activation_probability: 0.25   # 25% of queries get noise
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
      mandatory: true
    typo_misspelling:
      probability: 0.3
```

## Programmatic Usage

```python
from src.distractor import SemanticDistractorModel

distractor = SemanticDistractorModel(config)

result = distractor.apply_noise(
    text="Help me write a Python script",
    persona_features={'style': 'casual'}
)

print(f"Original: {result.original_text}")
print(f"Noisy: {result.noisy_text}")
print(f"Layer: {result.layer}")
print(f"Strategies: {result.applied_strategies}")
```

## Examples

**Surface Noise:**
```
Original: "Help me write Python code"
Noisy:    "hey can u help me writ some python"
```

**Incomplete Info:**
```
Original: "Book a flight to Paris on Monday"
Noisy:    "I need to book a flight sometime next week"
```

**Semantic Ambiguity:**
```
Original: "Fix this bug"
Noisy:    "Fix this bug and also maybe refactor... actually just the bug"
```

## Next Steps

Continue to [Tutorial 5: Advanced Usage](tutorial_05_advanced.md).
