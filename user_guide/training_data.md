# Training Data

This guide covers the training data collection and export system in PersonaGym.

## Overview

The training data stage converts interactions into structured training samples:

1. **Collect** interactions from storage
2. **Transform** to TrainingSample format
3. **Export** to JSON files with statistics

## TrainingSample Structure

```python
@dataclass
class TrainingSample:
    sample_id: str                    # Unique identifier
    persona_id: str                   # Source persona
    persona_features: Dict[str, str]  # Persona features
    original_query: str               # Query before style transfer
    initial_query: str                # Query after style transfer
    prompt_trajectory: List[str]      # All user prompts in order
    full_conversation: List[Dict]     # Complete conversation
    num_turns: int                    # Number of turns
    noisy_initial_queries: List[Dict] # Noisy versions (if distractor enabled)
    metadata: Dict[str, Any]          # Additional metadata
```

### Output Format

```json
{
  "sample_id": "sample_20260206_001",
  "persona_id": "persona_001",
  "persona_features": {
    "age_band": "25_34",
    "role": "engineer",
    "communication_style": "casual"
  },
  "original_query": "Help me write a Python script",
  "initial_query": "hey can u help me write some python",
  "prompt_trajectory": [
    "hey can u help me write some python",
    "make it handle csv files",
    "add error handling too"
  ],
  "full_conversation": [
    {"role": "user", "content": "hey can u help me write some python"},
    {"role": "assistant", "content": "Sure! What kind of Python script..."},
    {"role": "user", "content": "make it handle csv files"},
    {"role": "assistant", "content": "Here's a script that handles CSV..."},
    {"role": "user", "content": "add error handling too"},
    {"role": "assistant", "content": "I've added try-except blocks..."}
  ],
  "num_turns": 3,
  "noisy_initial_queries": [
    {
      "noisy_text": "hey help me writ python",
      "noise_type": "surface_noise",
      "applied_strategies": ["typo_misspelling", "colloquial_speech"]
    }
  ],
  "metadata": {
    "interaction_id": "interaction_persona_001_...",
    "distractor_applied": true,
    "assistant_model": {"model": "gpt-4o-mini", "provider": "openai"}
  }
}
```

## Configuration

```yaml
training_data:
  format:
    include_persona: true              # Include persona features
    include_trajectory: true           # Include prompt trajectory
    include_distractor_versions: true  # Include noisy versions

  split:
    train: 0.8                        # 80% training
    validation: 0.1                   # 10% validation
    test: 0.1                         # 10% test
```

## Collection Process

### From Interactions

```python
from src.training_data import TrainingDataCollector

collector = TrainingDataCollector(config)

# Convert single interaction
sample = collector.interaction_to_training_sample(interaction_dict)

# Convert multiple interactions
samples = collector.collect_from_interactions(interactions)

print(f"Collected {len(samples)} training samples")
```

### From Storage

```python
from src.interaction_generator import InteractionStorage

storage = InteractionStorage("output/interactions")
interactions = storage.load_all()

samples = collector.collect_from_interactions(interactions)
```

## Export Process

### Basic Export

```python
from src.training_data import TrainingDataExporter

exporter = TrainingDataExporter(config)

# Export all samples (no split)
files = exporter.export_samples(
    samples,
    split=False,
    use_timestamp=True
)
# Output: train_samples_20260206_123456.json
```

### With Train/Val/Test Split

```python
files = exporter.export_samples(
    samples,
    split=True,
    use_timestamp=False
)
# Output: train_samples.json, val_samples.json, test_samples.json
```

### Export Statistics

```python
stats_file = exporter.export_statistics(samples, use_timestamp=True)
# Output: statistics_20260206_123456.json
```

## Output Files

### Directory Structure

```
output/training_data/
├── train_samples_20260206_123456.json    # Training samples
├── val_samples.json                       # Validation samples (if split)
├── test_samples.json                      # Test samples (if split)
├── statistics.json                        # Dataset statistics
└── token_usage_20260206_123456.json      # Token usage stats
```

### Statistics File

```json
{
  "total_samples": 500,
  "samples_by_split": {
    "train": 400,
    "validation": 50,
    "test": 50
  },
  "avg_turns": 3.2,
  "avg_trajectory_length": 3.2,
  "distractor_stats": {
    "samples_with_noise": 125,
    "noise_rate": 0.25,
    "layer_distribution": {
      "surface_noise": 62,
      "incomplete_info": 38,
      "semantic_ambiguity": 25
    }
  },
  "persona_distribution": {
    "engineer": 120,
    "student": 95,
    "designer": 85,
    ...
  },
  "created_at": "2026-02-06T12:00:00"
}
```

## Programmatic Usage

### Complete Workflow

```python
from src.training_data import TrainingDataCollector, TrainingDataExporter
from src.interaction_generator import InteractionStorage

# Load interactions
storage = InteractionStorage("output/interactions")
interactions = storage.load_all()

# Initialize collector and exporter
collector = TrainingDataCollector(config)
exporter = TrainingDataExporter(config)

# Collect samples
samples = collector.collect_from_interactions(interactions)

# Export
sample_files = exporter.export_samples(samples, split=True)
stats_file = exporter.export_statistics(samples)

print(f"Exported to: {sample_files}")
print(f"Statistics: {stats_file}")
```

### Custom Transformation

```python
class CustomCollector(TrainingDataCollector):
    def interaction_to_training_sample(self, interaction):
        # Custom transformation logic
        sample = super().interaction_to_training_sample(interaction)

        # Add custom fields
        sample.metadata['custom_field'] = self.compute_custom_value(interaction)

        return sample
```

## Data Validation

### Sample Validation

```python
def validate_sample(sample: TrainingSample) -> List[str]:
    issues = []

    if not sample.persona_id:
        issues.append("Missing persona_id")

    if not sample.prompt_trajectory:
        issues.append("Empty prompt trajectory")

    if sample.num_turns < 1:
        issues.append("Invalid turn count")

    return issues
```

### Batch Validation

```python
for sample in samples:
    issues = validate_sample(sample)
    if issues:
        print(f"Sample {sample.sample_id}: {issues}")
```

## Format Options

### Include/Exclude Fields

```yaml
training_data:
  format:
    include_persona: true           # Include persona features
    include_trajectory: true        # Include prompt trajectory
    include_distractor_versions: true  # Include noisy versions
    include_full_conversation: true # Include all messages
    include_metadata: true          # Include metadata
```

### Minimal Format

```yaml
training_data:
  format:
    include_persona: false
    include_trajectory: true
    include_distractor_versions: false
```

Output:

```json
{
  "sample_id": "sample_001",
  "initial_query": "help me with python",
  "prompt_trajectory": ["help me with python", "add error handling"],
  "num_turns": 2
}
```

## HuggingFace Integration

### Prepare for Upload

```python
# Use the provided script
python scripts/prepare_ppopt_hf_jsonl.py \
    --input output/training_data/train_samples.json \
    --output hf_dataset.jsonl
```

### Upload to Hub

```python
python scripts/upload_hf_dataset.py \
    --file hf_dataset.jsonl \
    --repo your-username/ppopt-dataset
```

## API Reference

### TrainingSample

```python
@dataclass
class TrainingSample:
    sample_id: str
    persona_id: str
    persona_features: Dict[str, str]
    original_query: str
    initial_query: str
    prompt_trajectory: List[str]
    full_conversation: List[Dict[str, str]]
    num_turns: int
    noisy_initial_queries: List[Dict[str, Any]]
    metadata: Dict[str, Any]

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary."""
```

### TrainingDataCollector

```python
class TrainingDataCollector:
    """Collects and transforms interactions to training samples."""

    def __init__(self, config: Dict[str, Any]):
        """Initialize with configuration."""

    def interaction_to_training_sample(
        self,
        interaction: Dict[str, Any]
    ) -> TrainingSample:
        """Convert single interaction to training sample."""

    def collect_from_interactions(
        self,
        interactions: List[Dict[str, Any]]
    ) -> List[TrainingSample]:
        """Convert multiple interactions to training samples."""
```

### TrainingDataExporter

```python
class TrainingDataExporter:
    """Exports training samples to files."""

    def __init__(self, config: Dict[str, Any]):
        """Initialize with configuration."""

    def export_samples(
        self,
        samples: List[TrainingSample],
        split: bool = False,
        use_timestamp: bool = True
    ) -> Dict[str, str]:
        """Export samples to JSON files."""

    def export_statistics(
        self,
        samples: List[TrainingSample],
        use_timestamp: bool = True
    ) -> str:
        """Export dataset statistics."""
```

## Best Practices

### 1. Validate Before Export

```python
valid_samples = [s for s in samples if not validate_sample(s)]
exporter.export_samples(valid_samples)
```

### 2. Use Timestamps for Versioning

```python
exporter.export_samples(samples, use_timestamp=True)
# Creates: train_samples_20260206_123456.json
```

### 3. Monitor Statistics

```python
stats = exporter.compute_statistics(samples)
print(f"Average turns: {stats['avg_turns']:.1f}")
print(f"Noise rate: {stats['distractor_stats']['noise_rate']:.1%}")
```

### 4. Incremental Export

```python
# Export new samples only
existing = load_existing_samples()
new_samples = [s for s in samples if s.sample_id not in existing]
exporter.export_samples(new_samples, append=True)
```

## See Also

- [Interaction Generation](interaction_generation.md) - Source of training data
- [Token Tracking](token_tracking.md) - Cost analysis
- [Training Data API](../api/training_data.md) - Detailed API documentation
