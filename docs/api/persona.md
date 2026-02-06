# Persona API

Persona generation and management classes.

## PersonaBank

```python
from src.persona_bank import PersonaBank
```

### Class Definition

```python
class PersonaBank:
    """
    Manages persona dimension definitions.

    Loads dimension configurations from YAML and provides
    access to dimension values and constraints.
    """
```

### Constructor

```python
def __init__(self, persona_path: str):
    """
    Load persona dimensions from YAML file.

    Args:
        persona_path: Path to persona.yaml
    """
```

### Methods

```python
def get_dimension(self, name: str) -> Dict[str, Any]:
    """Get a specific dimension definition."""

def get_all_dimensions(self) -> Dict[str, Dict]:
    """Get all dimension definitions."""

def get_constraint_dimensions(self) -> List[str]:
    """Get names of constraint dimensions (always sampled)."""

def get_dimension_values(self, name: str) -> List[str]:
    """Get possible values for a dimension."""
```

---

## PersonaSampler

```python
from src.sampling import PersonaSampler
```

### Class Definition

```python
class PersonaSampler:
    """
    Samples persona features with diversity constraints.

    Ensures sampled personas are diverse using Hamming distance.
    """
```

### Constructor

```python
def __init__(
    self,
    config_path: str,
    persona_bank: PersonaBank,
    llm_client: Optional[LLMClient] = None
):
    """
    Initialize sampler.

    Args:
        config_path: Path to sampling_config.yaml
        persona_bank: PersonaBank instance
        llm_client: Optional LLM client for advanced sampling
    """
```

### Methods

```python
def sample_persona(self) -> Dict[str, str]:
    """
    Sample a single persona's features.

    Returns:
        Dictionary mapping dimension names to values

    Example:
        >>> features = sampler.sample_persona()
        >>> print(features)
        {'age_band': '25_34', 'role': 'engineer', ...}
    """

def sample_batch(self, n: int) -> List[Dict[str, str]]:
    """
    Sample multiple personas with diversity enforcement.

    Args:
        n: Number of personas to sample

    Returns:
        List of feature dictionaries
    """
```

---

## PersonaSpec

```python
from src.persona_spec import PersonaSpec, generate_persona_id
```

### Class Definition

```python
@dataclass
class PersonaSpec:
    """
    Complete persona specification.

    Attributes:
        persona_id: Unique identifier
        features: Dimension-value mapping
        system_prompt: Generated system prompt
        metadata: Additional metadata
    """
    persona_id: str
    features: Dict[str, str]
    system_prompt: Optional[str]
    metadata: Dict[str, Any]
```

### Methods

```python
def to_dict(self) -> Dict[str, Any]:
    """Convert to dictionary."""

@classmethod
def from_dict(cls, data: Dict[str, Any]) -> 'PersonaSpec':
    """Create from dictionary."""
```

### Utility Functions

```python
def generate_persona_id(features: Dict[str, str]) -> str:
    """
    Generate unique persona ID.

    Returns:
        ID like 'persona_20260206_001'
    """
```

---

## PersonaSpecStorage

```python
from src.persona_spec import PersonaSpecStorage
```

### Class Definition

```python
class PersonaSpecStorage:
    """Persists persona specifications to disk."""
```

### Constructor

```python
def __init__(self, output_dir: str, format: str = "json"):
    """
    Initialize storage.

    Args:
        output_dir: Directory for persona files
        format: Output format ("json")
    """
```

### Methods

```python
def save(self, spec: PersonaSpec) -> str:
    """
    Save persona to disk.

    Returns:
        Path to saved file
    """

def load(self, persona_id: str) -> PersonaSpec:
    """Load persona by ID."""

def load_all(self) -> List[PersonaSpec]:
    """Load all stored personas."""

def exists(self, persona_id: str) -> bool:
    """Check if persona exists."""
```

---

## Usage Examples

### Complete Workflow

```python
from src.persona_bank import PersonaBank
from src.sampling import PersonaSampler
from src.persona_spec import PersonaSpec, PersonaSpecStorage, generate_persona_id
from src.llm_client import create_llm_client, LLMFormulator

# Initialize components
bank = PersonaBank("input/persona.yaml")
sampler = PersonaSampler("input/sampling_config.yaml", bank)
storage = PersonaSpecStorage("output/personas")

client = create_llm_client(config['api'])
formulator = LLMFormulator(client, "prompts/persona_to_system_prompt.txt")

# Generate personas
for i in range(10):
    # Sample features
    features = sampler.sample_persona()

    # Generate ID
    persona_id = generate_persona_id(features)

    # Generate system prompt
    system_prompt = formulator.formulate(features)

    # Create spec
    spec = PersonaSpec(
        persona_id=persona_id,
        features=features,
        system_prompt=system_prompt,
        metadata={'index': i}
    )

    # Save
    storage.save(spec)

# Load all personas
all_personas = storage.load_all()
print(f"Generated {len(all_personas)} personas")
```

---

## See Also

- [Persona System](../user_guide/persona_system.md) - User guide
- [Configuration](../user_guide/configuration.md) - Persona configuration
