# Persona System

This guide covers the persona generation system in PersonaGym.

## Overview

The persona system creates diverse user profiles that drive realistic conversation simulation. Each persona consists of:

- **Features**: A set of dimension-value pairs (e.g., `age_band: 25_34`)
- **System Prompt**: An LLM-generated prompt describing the persona
- **Metadata**: Creation timestamp, feature count, etc.

## Persona Dimensions

Personas are defined by 30+ dimensions across 5 categories in `input/persona.yaml`:

### Basic Info (Constraint Dimensions)

| Dimension | Values | Description |
|-----------|--------|-------------|
| `age_band` | u18, 18_24, 25_34, 35_44, 45_60, 60p | Age group |
| `role` | student, engineer, data, pm, designer, ... | Professional role |
| `seniority` | intern, entry, junior, mid, senior, lead, director, c_level | Experience level |
| `education` | hs, bachelor, master, phd | Education level |
| `language` | english, chinese, spanish, french, german, ... | Primary language |

### Communication Style

| Dimension | Values | Description |
|-----------|--------|-------------|
| `communication_style` | casual, professional, technical, creative | Communication preference |
| `response_length` | very_short, short, medium, long, very_long | Preferred response length |
| `technical_level` | beginner, intermediate, advanced, expert | Technical expertise |
| `tone` | friendly, formal, humorous, serious | Conversation tone |

### Query Preferences

| Dimension | Values | Description |
|-----------|--------|-------------|
| `query_length_pref` | concise, moderate, detailed | Query length preference |
| `explanation_style` | step_by_step, overview, examples, minimal | Learning style |
| `feedback_frequency` | rarely, sometimes, often, always | Follow-up frequency |

## Persona Sampling

### Configuration

Sampling is configured in `input/sampling_config.yaml`:

```yaml
sampling:
  feature_availability_rate: 0.7    # % of dimensions per persona
  min_features: 10
  max_features: 20
  required_dimensions:
    - query_length_pref             # Always include these

diversity:
  enabled: true
  min_hamming_distance: 3           # Minimum feature difference
  max_retries: 100                  # Retries for unique persona
```

### Sampling Strategies

```python
from src.sampling import PersonaSampler
from src.persona_bank import PersonaBank

bank = PersonaBank("input/persona.yaml")
sampler = PersonaSampler("input/sampling_config.yaml", bank)

# Sample a single persona
features = sampler.sample_persona()
# {'age_band': '25_34', 'role': 'engineer', 'communication_style': 'casual', ...}
```

### Diversity Enforcement

The sampler ensures diversity using Hamming distance:

```python
# Two personas with these features:
persona_1 = {'age': '25_34', 'role': 'engineer', 'style': 'casual'}
persona_2 = {'age': '25_34', 'role': 'designer', 'style': 'formal'}

# Hamming distance = 2 (role and style differ)
# If min_hamming_distance = 3, persona_2 would be rejected
```

## System Prompt Generation

### Template

The system prompt template (`prompts/persona_to_system_prompt.txt`) converts features to natural language:

```
You are an AI assistant helping a user with the following characteristics:

{persona_features}

Adapt your responses to match the user's:
- Communication style: {communication_style}
- Technical level: {technical_level}
- Preferred response length: {response_length}

Be helpful, accurate, and appropriate for this user's needs.
```

### LLM Formulator

```python
from src.llm_client import create_llm_client, LLMFormulator

client = create_llm_client(config['api'])
formulator = LLMFormulator(
    client,
    template_path="prompts/persona_to_system_prompt.txt",
    max_retries=3,
    validate_output=True,
    min_prompt_length=50
)

# Generate system prompt from features
system_prompt = formulator.formulate(features)
```

## PersonaSpec Data Structure

```python
from src.persona_spec import PersonaSpec, generate_persona_id

# Create a persona spec
spec = PersonaSpec(
    persona_id=generate_persona_id(features),
    features=features,
    system_prompt=system_prompt,
    metadata={
        'num_features': len(features),
        'has_prompt': True
    }
)

# Convert to dictionary
data = spec.to_dict()
```

### JSON Output Format

```json
{
  "persona_id": "persona_20260206_001",
  "features": {
    "age_band": "25_34",
    "role": "engineer",
    "seniority": "mid",
    "communication_style": "casual",
    "response_length": "medium",
    "technical_level": "advanced",
    "query_length_pref": "moderate"
  },
  "system_prompt": "You are helping a mid-level engineer in their late 20s...",
  "metadata": {
    "created_at": "2026-02-06T10:30:00",
    "num_features": 7,
    "has_prompt": true
  }
}
```

## Persona Storage

### Saving Personas

```python
from src.persona_spec import PersonaSpecStorage

storage = PersonaSpecStorage("output/personas", format="json")

# Save a single persona
storage.save(spec)

# Load all personas
all_personas = storage.load_all()
```

### Incremental Mode

In incremental mode, existing personas are preserved:

```python
# Check existing personas
existing = storage.load_all()
print(f"Found {len(existing)} existing personas")

# Only generate missing personas
remaining = num_personas - len(existing)
```

## Customizing Persona Dimensions

### Adding New Dimensions

Edit `input/persona.yaml`:

```yaml
dimensions:
  # Add a new dimension
  industry:
    name: industry
    is_constraint: false
    values:
      - technology
      - healthcare
      - finance
      - education
      - retail
      - manufacturing
```

### Modifying Existing Dimensions

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
      - executive
      - consultant      # Add new value
      - freelancer      # Add new value
```

### Dimension Constraints

Constraint dimensions (`is_constraint: true`) are always included:

```yaml
dimensions:
  age_band:
    name: age_band
    is_constraint: true   # Always sampled
    values: [...]

  hobby:
    name: hobby
    is_constraint: false  # Optionally sampled
    values: [...]
```

## API Reference

### PersonaBank

```python
class PersonaBank:
    """Manages persona dimension definitions."""

    def __init__(self, persona_path: str):
        """Load persona dimensions from YAML file."""

    def get_dimension(self, name: str) -> Dict:
        """Get a specific dimension definition."""

    def get_all_dimensions(self) -> Dict[str, Dict]:
        """Get all dimension definitions."""

    def get_constraint_dimensions(self) -> List[str]:
        """Get names of constraint dimensions."""
```

### PersonaSampler

```python
class PersonaSampler:
    """Samples persona features with diversity constraints."""

    def __init__(self, config_path: str, persona_bank: PersonaBank):
        """Initialize sampler with configuration."""

    def sample_persona(self) -> Dict[str, str]:
        """Sample a single persona's features."""

    def sample_batch(self, n: int) -> List[Dict[str, str]]:
        """Sample multiple personas with diversity."""
```

### PersonaSpec

```python
@dataclass
class PersonaSpec:
    """Represents a complete persona specification."""

    persona_id: str
    features: Dict[str, str]
    system_prompt: Optional[str]
    metadata: Dict[str, Any]

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary."""

    @classmethod
    def from_dict(cls, data: Dict) -> 'PersonaSpec':
        """Create from dictionary."""
```

## Best Practices

### 1. Balance Diversity and Realism

```yaml
diversity:
  min_hamming_distance: 3   # Not too high (unrealistic)
                            # Not too low (redundant personas)
```

### 2. Include Essential Dimensions

```yaml
sampling:
  required_dimensions:
    - query_length_pref     # Critical for query adaptation
    - technical_level       # Affects response complexity
```

### 3. Validate System Prompts

```yaml
formulation:
  validate_output: true
  min_prompt_length: 50     # Reject too-short prompts
```

### 4. Use Incremental Mode for Large Runs

```yaml
experiment:
  incremental: true         # Resume from existing personas
```

## See Also

- [Configuration](configuration.md) - Full configuration reference
- [Query Generation](query_generation.md) - How personas affect queries
- [Persona API](../api/persona.md) - Detailed API documentation
