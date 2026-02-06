# Query API

Query generation and style transfer classes.

## UserQueryGenerator

```python
from src.query_generator import UserQueryGenerator
```

### Class Definition

```python
class UserQueryGenerator:
    """
    Generates and adapts queries for personas.

    Features:
    - Load queries from JSONL dataset
    - Apply style transfer to match persona
    - Track used queries to prevent duplicates
    """
```

### Constructor

```python
def __init__(self, config: Dict[str, Any], llm_client: LLMClient):
    """
    Initialize generator.

    Args:
        config: Configuration dictionary
        llm_client: LLM client for style transfer
    """
```

### Methods

```python
def generate_queries_for_persona(
    self,
    persona_features: Dict[str, str],
    num_queries: int = 5,
    apply_style_transfer: bool = True
) -> List[Dict[str, Any]]:
    """
    Generate queries for a single persona.

    Args:
        persona_features: Persona feature dictionary
        num_queries: Number of queries to generate
        apply_style_transfer: Whether to apply style transfer

    Returns:
        List of query dictionaries with:
        - query_id: Unique identifier
        - original_query: Original text
        - adapted_query: After style transfer
        - inferred_domain: Detected domain
        - inferred_scenario: Detected scenario
        - target_turns: Suggested conversation length
    """

def generate_queries_batch(
    self,
    personas: List[Dict[str, Any]]
) -> Dict[str, List[Dict[str, Any]]]:
    """
    Generate queries for multiple personas.

    Returns:
        Dict mapping persona_id to query list
    """

def mark_used_query_ids(self, query_ids: Set[str]) -> None:
    """Mark query IDs as used to prevent reuse."""
```

---

## QueryDataset

```python
from src.query_generator import QueryDataset
```

### Class Definition

```python
class QueryDataset:
    """
    Manages the seed query dataset.

    Loads queries from JSONL and provides sampling.
    """
```

### Constructor

```python
def __init__(self, path: str, max_queries: Optional[int] = None):
    """
    Load queries from JSONL file.

    Args:
        path: Path to query.jsonl
        max_queries: Maximum queries to load (None = all)
    """
```

### Methods

```python
def __len__(self) -> int:
    """Return number of queries."""

def __getitem__(self, index: int) -> Dict[str, Any]:
    """Get query by index."""

def sample(self, n: int) -> List[Dict[str, Any]]:
    """Sample n random queries."""

def get_sources(self) -> Set[str]:
    """Get unique source identifiers."""

def filter_by_domain(self, domain: str) -> List[Dict[str, Any]]:
    """Filter queries by domain."""
```

---

## QueryStorage

```python
from src.query_storage import QueryStorage
```

### Class Definition

```python
class QueryStorage:
    """
    Persists queries per persona for incremental mode.

    Enables resuming query generation across runs.
    """
```

### Constructor

```python
def __init__(self, output_dir: str):
    """
    Initialize storage.

    Args:
        output_dir: Directory for query files
    """
```

### Methods

```python
def save_persona(self, persona_id: str, queries: List[Dict]) -> str:
    """Save queries for a persona. Returns file path."""

def load_persona(self, persona_id: str) -> List[Dict]:
    """Load queries for a persona."""

def load_all(self) -> Dict[str, List[Dict]]:
    """Load all stored queries."""

def append_persona(self, persona_id: str, queries: List[Dict]) -> None:
    """Append queries to existing persona file."""

def exists(self, persona_id: str) -> bool:
    """Check if queries exist for persona."""
```

---

## Usage Examples

### Generate Queries

```python
from src.query_generator import UserQueryGenerator
from src.llm_client import create_llm_client

client = create_llm_client(config['api'])
generator = UserQueryGenerator(config, client)

# Generate for single persona
queries = generator.generate_queries_for_persona(
    persona_features={'role': 'engineer', 'style': 'casual'},
    num_queries=5,
    apply_style_transfer=True
)

for q in queries:
    print(f"Original: {q['original_query']}")
    print(f"Adapted:  {q['adapted_query']}")
    print()
```

### Batch Generation

```python
personas = [
    {'persona_id': 'p1', 'features': {...}},
    {'persona_id': 'p2', 'features': {...}},
]

queries_map = generator.generate_queries_batch(personas)

for persona_id, queries in queries_map.items():
    print(f"{persona_id}: {len(queries)} queries")
```

### Persist Queries

```python
from src.query_storage import QueryStorage

storage = QueryStorage("output/queries")

# Save
storage.save_persona("persona_001", queries)

# Load
stored = storage.load_persona("persona_001")

# Load all
all_queries = storage.load_all()
```

---

## See Also

- [Query Generation](../user_guide/query_generation.md) - User guide
- [Interaction API](interaction.md) - Using queries in conversations
