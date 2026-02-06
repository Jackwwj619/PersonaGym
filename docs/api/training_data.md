# Training Data API

Training sample collection and export classes.

## TrainingSample

```python
from src.training_data import TrainingSample
```

### Class Definition

```python
@dataclass
class TrainingSample:
    """
    Training data sample.

    Attributes:
        sample_id: Unique identifier
        persona_id: Source persona
        persona_features: Persona feature dictionary
        original_query: Query before style transfer
        initial_query: Query after style transfer
        prompt_trajectory: List of user prompts in order
        full_conversation: Complete conversation messages
        num_turns: Number of conversation turns
        noisy_initial_queries: Noisy versions (if distractor enabled)
        metadata: Additional metadata
    """
```

### Methods

```python
def to_dict(self) -> Dict[str, Any]:
    """Convert to dictionary for JSON export."""

@classmethod
def from_dict(cls, data: Dict[str, Any]) -> 'TrainingSample':
    """Create from dictionary."""
```

---

## TrainingDataCollector

```python
from src.training_data import TrainingDataCollector
```

### Class Definition

```python
class TrainingDataCollector:
    """
    Collects and transforms interactions to training samples.

    Handles:
    - Conversion from Interaction to TrainingSample
    - Extraction of prompt trajectories
    - Inclusion of noisy versions
    """
```

### Constructor

```python
def __init__(self, config: Dict[str, Any]):
    """
    Initialize collector.

    Args:
        config: Configuration with training_data.format settings
    """
```

### Methods

```python
def interaction_to_training_sample(
    self,
    interaction: Dict[str, Any]
) -> TrainingSample:
    """
    Convert single interaction to training sample.

    Args:
        interaction: Interaction dictionary

    Returns:
        TrainingSample object
    """

def collect_from_interactions(
    self,
    interactions: List[Dict[str, Any]]
) -> List[TrainingSample]:
    """
    Convert multiple interactions to training samples.

    Args:
        interactions: List of interaction dictionaries

    Returns:
        List of TrainingSample objects
    """
```

---

## TrainingDataExporter

```python
from src.training_data import TrainingDataExporter
```

### Class Definition

```python
class TrainingDataExporter:
    """
    Exports training samples to files.

    Features:
    - JSON export
    - Optional train/val/test split
    - Statistics generation
    - Timestamped filenames
    """
```

### Constructor

```python
def __init__(self, config: Dict[str, Any]):
    """
    Initialize exporter.

    Args:
        config: Configuration with paths and split settings
    """
```

### Methods

```python
def export_samples(
    self,
    samples: List[TrainingSample],
    split: bool = False,
    use_timestamp: bool = True
) -> Dict[str, str]:
    """
    Export samples to JSON files.

    Args:
        samples: List of training samples
        split: Whether to split into train/val/test
        use_timestamp: Add timestamp to filenames

    Returns:
        Dictionary mapping split name to file path
        {'train': 'path/train_samples.json', ...}
    """

def export_statistics(
    self,
    samples: List[TrainingSample],
    use_timestamp: bool = True
) -> str:
    """
    Export dataset statistics.

    Returns:
        Path to statistics JSON file
    """

def compute_statistics(
    self,
    samples: List[TrainingSample]
) -> Dict[str, Any]:
    """
    Compute dataset statistics.

    Returns:
        Statistics dictionary with counts, averages, distributions
    """
```

---

## Usage Examples

### Collect from Interactions

```python
from src.training_data import TrainingDataCollector, TrainingDataExporter
from src.interaction_generator import InteractionStorage

# Load interactions
storage = InteractionStorage("output/interactions")
interactions = storage.load_all()

# Initialize collector
collector = TrainingDataCollector(config)

# Convert to training samples
samples = collector.collect_from_interactions(interactions)
print(f"Collected {len(samples)} samples")

# Access sample data
for sample in samples[:3]:
    print(f"ID: {sample.sample_id}")
    print(f"Trajectory: {sample.prompt_trajectory}")
    print(f"Turns: {sample.num_turns}")
    print()
```

### Export to Files

```python
exporter = TrainingDataExporter(config)

# Export all (no split)
files = exporter.export_samples(
    samples,
    split=False,
    use_timestamp=True
)
print(f"Exported to: {files['train']}")

# Export with split
files = exporter.export_samples(
    samples,
    split=True,
    use_timestamp=False
)
print(f"Train: {files['train']}")
print(f"Val: {files['validation']}")
print(f"Test: {files['test']}")

# Export statistics
stats_file = exporter.export_statistics(samples)
print(f"Statistics: {stats_file}")
```

### Compute Statistics

```python
stats = exporter.compute_statistics(samples)

print(f"Total samples: {stats['total_samples']}")
print(f"Average turns: {stats['avg_turns']:.1f}")
print(f"Noise rate: {stats['distractor_stats']['noise_rate']:.1%}")
```

### Complete Pipeline Integration

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
result = pipeline.run(num_personas=100)

# Training data is automatically exported
print(f"Samples: {result['training_data']['total_samples']}")
print(f"Files: {result['training_data']['sample_files']}")
```

---

## Output Format

### Sample JSON

```json
{
  "sample_id": "sample_20260206_001",
  "persona_id": "persona_001",
  "persona_features": {
    "role": "engineer",
    "communication_style": "casual"
  },
  "original_query": "Help me write Python code",
  "initial_query": "hey can u help me with python",
  "prompt_trajectory": [
    "hey can u help me with python",
    "make it handle errors",
    "looks good thanks"
  ],
  "full_conversation": [
    {"role": "user", "content": "hey can u help me with python"},
    {"role": "assistant", "content": "Sure! What would you like?"},
    {"role": "user", "content": "make it handle errors"},
    {"role": "assistant", "content": "Here's the updated code..."},
    {"role": "user", "content": "looks good thanks"},
    {"role": "assistant", "content": "You're welcome!"}
  ],
  "num_turns": 3,
  "noisy_initial_queries": [
    {
      "noisy_text": "hey help me writ python",
      "noise_type": "surface_noise",
      "applied_strategies": ["typo_misspelling"]
    }
  ],
  "metadata": {
    "interaction_id": "interaction_...",
    "distractor_applied": true
  }
}
```

### Statistics JSON

```json
{
  "total_samples": 500,
  "avg_turns": 3.2,
  "avg_trajectory_length": 3.2,
  "distractor_stats": {
    "samples_with_noise": 125,
    "noise_rate": 0.25
  },
  "persona_distribution": {
    "engineer": 120,
    "student": 95
  },
  "created_at": "2026-02-06T12:00:00"
}
```

---

## See Also

- [Training Data](../user_guide/training_data.md) - User guide
- [Interaction API](interaction.md) - Source of training data
