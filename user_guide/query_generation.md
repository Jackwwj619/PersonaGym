# Query Generation

This guide covers the query generation and style transfer system in PersonaGym.

## Overview

The query generation stage:

1. Loads seed queries from a dataset
2. Assigns queries to personas
3. Optionally applies style transfer to match persona preferences
4. Persists queries for interaction generation

## Query Dataset

### Input Format

Queries are stored in JSONL format (`input/query.jsonl`):

```json
{"text": "Help me write an email to my boss", "metadata": {"source": "dataset1", "category": "writing"}}
{"text": "Explain how neural networks work", "metadata": {"source": "dataset2", "category": "education"}}
{"text": "Debug this Python code", "metadata": {"source": "dataset3", "category": "coding"}}
```

### Dataset Statistics

```python
from src.query_generator import QueryDataset

dataset = QueryDataset("input/query.jsonl")
print(f"Total queries: {len(dataset)}")
print(f"Sources: {dataset.get_sources()}")
```

## Query Assignment

### Configuration

```yaml
query_generation:
  dataset:
    path: "input/query.jsonl"
    max_queries: null           # null = use all

  selection:
    queries_per_persona: 5      # Queries assigned per persona
```

### Selection Logic

```python
from src.query_generator import UserQueryGenerator

generator = UserQueryGenerator(config, llm_client)

# Generate queries for a single persona
queries = generator.generate_queries_for_persona(
    persona_features={'role': 'engineer', 'technical_level': 'advanced'},
    num_queries=5
)
```

Each query includes:

```python
{
    'query_id': 'query_001',
    'original_query': 'Help me debug this code',
    'adapted_query': 'yo can u help me fix this buggy code',  # After style transfer
    'inferred_domain': 'coding',
    'inferred_scenario': 'debugging',
    'target_turns': 3
}
```

## Style Transfer

### Purpose

Style transfer adapts queries to match persona communication preferences:

| Persona Style | Original Query | Adapted Query |
|--------------|----------------|---------------|
| Casual | "Please help me write an email" | "hey can u help me write an email" |
| Technical | "Explain machine learning" | "Provide a technical overview of ML algorithms" |
| Formal | "Fix this bug" | "I would appreciate assistance in resolving this software defect" |

### Configuration

```yaml
query_generation:
  style_transfer:
    enabled: true
    transfer_probability: 0.5   # 50% of queries are transferred
    template: "prompts/query_style_transfer.txt"
```

### Template

The style transfer template (`prompts/query_style_transfer.txt`):

```
You are adapting a user query to match a specific persona's communication style.

Original Query: {original_query}

Persona Features:
{persona_features}

Rewrite the query to match this persona's:
- Communication style: {communication_style}
- Query length preference: {query_length_pref}
- Tone: {tone}

Output only the adapted query, nothing else.
```

### Programmatic Usage

```python
# With style transfer
queries = generator.generate_queries_for_persona(
    persona_features=features,
    num_queries=5,
    apply_style_transfer=True
)

# Without style transfer
queries = generator.generate_queries_for_persona(
    persona_features=features,
    num_queries=5,
    apply_style_transfer=False
)
```

## Query Storage

### Persistence

Queries are persisted per-persona for incremental mode:

```python
from src.query_storage import QueryStorage

storage = QueryStorage("output/queries")

# Save queries for a persona
storage.save_persona(persona_id, queries)

# Load queries for a persona
stored_queries = storage.load_persona(persona_id)

# Load all queries
all_queries = storage.load_all()  # Dict[persona_id, List[queries]]
```

### Output Format

`output/queries/persona_20260206_001.json`:

```json
{
  "persona_id": "persona_20260206_001",
  "queries": [
    {
      "query_id": "query_001",
      "original_query": "Help me write an email",
      "adapted_query": "hey can u help me write an email",
      "inferred_domain": "writing",
      "inferred_scenario": "email_composition",
      "target_turns": 3,
      "style_transferred": true
    },
    ...
  ],
  "metadata": {
    "created_at": "2026-02-06T10:30:00",
    "total_queries": 5
  }
}
```

## Domain and Scenario Inference

### Automatic Inference

The system infers domain and scenario from query content:

```python
query = "Help me debug this Python script"

# Inferred:
# domain: "coding"
# scenario: "debugging"
```

### Impact on Persona

Inferred domain/scenario updates the persona's system prompt:

```python
# Original persona features
features = {'role': 'engineer', 'domain': 'general'}

# After query assignment
updated_features = {'role': 'engineer', 'domain': 'coding'}

# System prompt regenerated with coding context
```

## Batch Generation

### For Multiple Personas

```python
# Generate queries for all personas
queries_map = generator.generate_queries_batch(personas)

# Returns: Dict[persona_id, List[query_dict]]
for persona_id, queries in queries_map.items():
    print(f"{persona_id}: {len(queries)} queries")
```

### Tracking Used Queries

```python
# Mark queries as used (prevents reuse)
generator.mark_used_query_ids({'query_001', 'query_002'})

# Next generation will skip these queries
new_queries = generator.generate_queries_for_persona(features, num_queries=5)
```

## Advanced Usage

### Custom Query Selection

```python
class CustomQueryGenerator(UserQueryGenerator):
    def select_queries(self, persona_features, num_queries):
        # Custom selection logic
        # e.g., match queries to persona's domain
        domain = persona_features.get('domain')
        matching_queries = self.dataset.filter_by_domain(domain)
        return matching_queries[:num_queries]
```

### Partial Style Transfer

For queries that should only be partially adapted:

```yaml
query_generation:
  style_transfer:
    template: "prompts/query_style_transfer_partial.txt"
```

The partial template preserves key information while adapting style:

```
Adapt only the surface form of the query.
Preserve:
- All technical terms
- Specific requirements
- Key constraints

Only change:
- Greeting/closing style
- Formality level
- Sentence structure
```

## API Reference

### UserQueryGenerator

```python
class UserQueryGenerator:
    """Generates and adapts queries for personas."""

    def __init__(self, config: Dict, llm_client: LLMClient):
        """Initialize with configuration and LLM client."""

    def generate_queries_for_persona(
        self,
        persona_features: Dict[str, str],
        num_queries: int = 5,
        apply_style_transfer: bool = True
    ) -> List[Dict[str, Any]]:
        """Generate queries for a single persona."""

    def generate_queries_batch(
        self,
        personas: List[Dict[str, Any]]
    ) -> Dict[str, List[Dict[str, Any]]]:
        """Generate queries for multiple personas."""

    def mark_used_query_ids(self, query_ids: Set[str]) -> None:
        """Mark query IDs as used to prevent reuse."""
```

### QueryDataset

```python
class QueryDataset:
    """Manages the seed query dataset."""

    def __init__(self, path: str):
        """Load queries from JSONL file."""

    def __len__(self) -> int:
        """Return number of queries."""

    def sample(self, n: int) -> List[Dict]:
        """Sample n random queries."""

    def get_sources(self) -> Set[str]:
        """Get unique source identifiers."""
```

### QueryStorage

```python
class QueryStorage:
    """Persists queries per persona."""

    def __init__(self, output_dir: str):
        """Initialize storage directory."""

    def save_persona(self, persona_id: str, queries: List[Dict]) -> str:
        """Save queries for a persona. Returns file path."""

    def load_persona(self, persona_id: str) -> List[Dict]:
        """Load queries for a persona."""

    def load_all(self) -> Dict[str, List[Dict]]:
        """Load all stored queries."""

    def append_persona(self, persona_id: str, queries: List[Dict]) -> None:
        """Append queries to existing persona file."""
```

## Best Practices

### 1. Diverse Query Dataset

Include queries from various domains and complexity levels:

```json
{"text": "Simple greeting", "metadata": {"complexity": "low"}}
{"text": "Complex technical question with multiple parts", "metadata": {"complexity": "high"}}
```

### 2. Balanced Style Transfer

```yaml
style_transfer:
  transfer_probability: 0.5   # Not 100% - keep some original queries
```

### 3. Query Deduplication

```python
# Remove duplicate queries
queries = generator._dedupe_queries(queries)
```

### 4. Monitor Query Usage

```python
# Check remaining queries
remaining = len(dataset) - len(used_query_ids)
print(f"Remaining queries: {remaining}")
```

## See Also

- [Persona System](persona_system.md) - How personas affect queries
- [Interaction Generation](interaction_generation.md) - Using queries in conversations
- [Query API](../api/query.md) - Detailed API documentation
