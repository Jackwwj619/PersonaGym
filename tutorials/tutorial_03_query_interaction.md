# Tutorial 3: Query & Interaction Generation

Learn query adaptation and multi-turn conversation simulation.

## Query Generation

### Load Seed Queries

```python
from src.query_generator import QueryDataset

dataset = QueryDataset("input/query.jsonl")
print(f"Total queries: {len(dataset)}")
print(f"Sample: {dataset[0]}")
```

### Style Transfer

Queries are adapted to match persona style:

```python
# Original: "Please help me write an email"
# Casual persona → "hey can u help me write an email"
# Formal persona → "I would appreciate assistance in composing an email"
```

Configuration:

```yaml
query_generation:
  style_transfer:
    enabled: true
    transfer_probability: 0.5   # 50% get transferred
```

## Interaction Generation

### Single Interaction

```python
from src.interaction_generator import InteractionGenerator

generator = InteractionGenerator(config)

interaction = generator.generate_interaction(
    persona_id="persona_001",
    persona_features={'role': 'engineer', 'style': 'casual'},
    initial_query="help me with python",
    target_turns=3
)

for msg in interaction.messages:
    print(f"{msg.role}: {msg.content[:50]}...")
```

### Conversation Flow

1. **User** sends initial query
2. **Assistant** responds
3. **User** provides feedback (or ends)
4. Repeat until satisfied or max_turns

### Model Pool

Use diverse models for responses:

```yaml
interaction_generation:
  assistant_model:
    model_pool:
      - provider: openai
        model: gpt-4o-mini
        weight: 0.5
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.5
```

## Batch Generation

```python
personas_with_queries = [
    {
        'persona_id': 'p1',
        'persona_features': features,
        'queries': [{'adapted_query': 'query1', 'original_query': 'Query 1'}]
    }
]

interactions = generator.generate_interactions_batch(
    personas_with_queries,
    max_workers=10,
    show_progress=True
)
```

## Next Steps

Continue to [Tutorial 4: Noise Injection](tutorial_04_noise_injection.md).
