# Interaction Generation

This guide covers the multi-turn dialogue generation system in PersonaGym.

## Overview

The interaction generation stage simulates realistic user-AI conversations:

1. User sends initial query (optionally with noise)
2. Assistant responds based on conversation context
3. User provides feedback or follow-up
4. Loop continues until satisfaction or max turns

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    InteractionGenerator                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ AssistantModel│  │UserFeedback │  │  Distractor  │          │
│  │              │  │   Model      │  │  (optional)  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         ↓                 ↓                 ↓                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Interaction                            │   │
│  │  - Messages (user/assistant turns)                       │   │
│  │  - Noisy versions (if distractor enabled)                │   │
│  │  - Metadata (model info, timestamps)                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Configuration

```yaml
interaction_generation:
  min_turns: 2                # Minimum conversation turns
  max_turns: 5                # Maximum conversation turns
  max_workers: 20             # Parallel workers for batch generation
  max_retries: 3              # Retries per failed interaction
  retry_delay: 2              # Seconds between retries
  max_supplement_rounds: 3    # Extra rounds for failed personas

  # Assistant model
  assistant_model:
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.7
    max_completion_tokens: 1024

  # User feedback model
  user_model:
    provider: "openai"
    model: "gpt-4o-mini"
    temperature: 0.8
    max_completion_tokens: 256
    system_prompt_template: "prompts/user_feedback.txt"
    feedback_probability: 0.6
```

## Conversation Flow

### 1. Initial Query

```python
# User query (potentially with style transfer and noise)
initial_query = "hey can u help me write some python code"

# Becomes first message
messages = [
    Message(
        role='user',
        content=initial_query,
        metadata={'is_initial': True, 'was_noised': True}
    )
]
```

### 2. Assistant Response

```python
# Assistant responds using model pool
response = assistant_model.respond(
    user_query=initial_query,
    conversation_history=[]
)

messages.append(
    Message(
        role='assistant',
        content=response,
        metadata={'model': 'gpt-4o-mini', 'provider': 'openai'}
    )
)
```

### 3. User Feedback

```python
# User decides to continue or end
feedback = user_model.generate_feedback(
    persona_features=features,
    conversation_history=messages,
    assistant_response=response,
    current_turn=1,
    target_turns=3
)

if feedback:
    # User has follow-up
    messages.append(Message(role='user', content=feedback))
else:
    # User is satisfied, end conversation
    pass
```

## Model Pool

### Weighted Model Selection

```yaml
interaction_generation:
  assistant_model:
    model_pool:
      - provider: openai
        model: gpt-4o-mini
        weight: 0.2
      - provider: openai
        model: gpt-4o
        weight: 0.2
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.3
      - provider: openrouter
        model: google/gemini-2.0-flash-exp
        weight: 0.3
```

### Model Locking

The same model is used throughout a conversation:

```python
class AssistantModel:
    def lock_model(self):
        """Lock model for current conversation (thread-safe)."""
        chosen = random.choices(self.model_pool, weights=weights)[0]
        self._thread_local.locked_client = chosen['client']

    def unlock_model(self):
        """Release model lock after conversation."""
        self._thread_local.locked_client = None
```

## Interaction Data Structure

```python
@dataclass
class Interaction:
    interaction_id: str           # Unique identifier
    persona_id: str               # Associated persona
    persona_features: Dict        # Persona features
    original_query: str           # Query before style transfer
    initial_query: str            # Query after style transfer
    messages: List[Message]       # Conversation messages
    num_turns: int                # Number of turns
    metadata: Dict                # Additional metadata
```

### Output Format

```json
{
  "interaction_id": "interaction_persona_001_20260206_103000",
  "persona_id": "persona_001",
  "persona_features": {
    "role": "engineer",
    "communication_style": "casual"
  },
  "original_query": "Help me write Python code",
  "initial_query": "hey can u help me write some python code",
  "messages": [
    {
      "role": "user",
      "content": "hey can u help me write some python code",
      "timestamp": "2026-02-06T10:30:00",
      "metadata": {
        "is_initial": true,
        "was_noised": false
      }
    },
    {
      "role": "assistant",
      "content": "Sure! What kind of Python code would you like help with?",
      "timestamp": "2026-02-06T10:30:05",
      "metadata": {
        "model": "gpt-4o-mini",
        "provider": "openai"
      }
    },
    {
      "role": "user",
      "content": "a script to process csv files",
      "timestamp": "2026-02-06T10:30:30",
      "metadata": {
        "is_followup": true
      }
    },
    {
      "role": "assistant",
      "content": "Here's a Python script to process CSV files:\n```python\nimport pandas as pd\n...",
      "timestamp": "2026-02-06T10:30:45"
    }
  ],
  "num_turns": 2,
  "metadata": {
    "created_at": "2026-02-06T10:30:00",
    "distractor_applied": true,
    "assistant_model": {
      "model": "gpt-4o-mini",
      "provider": "openai"
    }
  }
}
```

## Programmatic Usage

### Single Interaction

```python
from src.interaction_generator import InteractionGenerator

generator = InteractionGenerator(config, distractor=distractor)

interaction = generator.generate_interaction(
    persona_id="persona_001",
    persona_features={'role': 'engineer', 'style': 'casual'},
    initial_query="help me write python code",
    original_query="Help me write Python code",
    target_turns=3
)

print(f"Generated {interaction.num_turns} turns")
```

### Batch Generation

```python
# Prepare persona-query pairs
personas_with_queries = [
    {
        'persona_id': 'persona_001',
        'persona_features': features1,
        'queries': [
            {'adapted_query': 'query1', 'original_query': 'Query 1'},
            {'adapted_query': 'query2', 'original_query': 'Query 2'}
        ]
    },
    ...
]

# Generate with parallel workers
interactions = generator.generate_interactions_batch(
    personas_with_queries,
    show_progress=True,
    max_workers=10,
    storage=interaction_storage  # Optional: save immediately
)

print(f"Generated {len(interactions)} interactions")
```

## Interaction Storage

### Incremental Saving

```python
from src.interaction_generator import InteractionStorage

storage = InteractionStorage("output/interactions", collector=training_collector)

# Interactions are saved immediately after generation
for interaction in interactions:
    filepath = storage.save(interaction)
    print(f"Saved: {filepath}")
```

### Index File

`output/interactions/index.json`:

```json
{
  "interactions": [
    {
      "interaction_id": "interaction_persona_001_...",
      "persona_id": "persona_001",
      "filepath": "output/interactions/interaction_persona_001_....json",
      "num_turns": 3,
      "created_at": "2026-02-06T10:30:00",
      "query_id": "query_001"
    }
  ],
  "count": 100,
  "last_updated": "2026-02-06T12:00:00"
}
```

## Distractor Integration

### Real-time Noise Application

Noise is applied during interaction generation:

```python
generator = InteractionGenerator(config, distractor=semantic_distractor)

# During generation:
# 1. Apply noise to initial query
noisy_query = distractor.apply_noise(initial_query, persona_features)

# 2. Apply noise to follow-up feedback
noisy_feedback = distractor.apply_noise(feedback, persona_features)
```

### Metadata Tracking

```json
{
  "metadata": {
    "distractor_applied": true,
    "distractor_type": "semantic",
    "noisy_versions": {
      "initial_query": {
        "clean_query": "Help me write code",
        "noisy_versions": [
          {
            "noisy_text": "help me writ code plz",
            "noise_type": "surface_noise",
            "applied_strategies": ["typo_misspelling", "colloquial_speech"]
          }
        ]
      }
    }
  }
}
```

## Error Handling

### Retry Logic

```python
def _generate_with_retry(self, task, max_retries=3, retry_delay=2):
    for attempt in range(max_retries + 1):
        try:
            interaction = self._generate_single_interaction(task)
            if interaction is not None:
                return interaction
        except Exception as e:
            if attempt < max_retries:
                time.sleep(retry_delay)
            else:
                logging.error(f"All attempts failed: {e}")
    return None
```

### Supplement Rounds

If some interactions fail, new queries are generated:

```python
# After initial round
for persona, needed in deficient_personas:
    # Generate new queries
    new_queries = query_generator.generate_queries_for_persona(
        persona['features'],
        num_queries=needed
    )
    # Retry with new queries
```

## Concurrent Execution

### Thread Safety

```python
class AssistantModel:
    def __init__(self):
        self._thread_local = threading.local()

    def lock_model(self):
        # Thread-local storage ensures thread safety
        self._thread_local.locked_client = selected_client
```

### Worker Configuration

```yaml
interaction_generation:
  max_workers: 20   # Parallel workers

# Reduce for rate-limited APIs
  max_workers: 5
```

## API Reference

### InteractionGenerator

```python
class InteractionGenerator:
    """Generates multi-turn interactions."""

    def __init__(self, config: Dict, distractor: Optional[DistractorModel] = None):
        """Initialize with config and optional distractor."""

    def generate_interaction(
        self,
        persona_id: str,
        persona_features: Dict,
        initial_query: str,
        system_prompt: Optional[str] = None,
        original_query: Optional[str] = None,
        target_turns: int = 3,
        query_id: Optional[str] = None
    ) -> Optional[Interaction]:
        """Generate a single interaction."""

    def generate_interactions_batch(
        self,
        personas_with_queries: List[Dict],
        show_progress: bool = True,
        max_workers: int = 5,
        storage: Optional[InteractionStorage] = None
    ) -> List[Interaction]:
        """Generate interactions for multiple personas."""
```

### AssistantModel

```python
class AssistantModel:
    """Simulates AI assistant responses."""

    def respond(self, user_query: str, conversation_history: List[Message]) -> str:
        """Generate response to user query."""

    def lock_model(self) -> None:
        """Lock model for current conversation."""

    def unlock_model(self) -> None:
        """Release model lock."""
```

### UserFeedbackModel

```python
class UserFeedbackModel:
    """Simulates user feedback based on persona."""

    def generate_feedback(
        self,
        persona_features: Dict,
        conversation_history: List[Message],
        assistant_response: str,
        current_turn: int,
        target_turns: int
    ) -> Optional[str]:
        """Generate user feedback or None if satisfied."""
```

## Best Practices

### 1. Configure Appropriate Turn Limits

```yaml
interaction_generation:
  min_turns: 2    # At least 2 turns for meaningful data
  max_turns: 5    # Cap to avoid infinite loops
```

### 2. Use Model Pool for Diversity

Different models produce different response styles.

### 3. Enable Incremental Storage

```python
storage = InteractionStorage(output_dir, collector=collector)
generator.generate_interactions_batch(..., storage=storage)
```

### 4. Monitor Success Rate

```python
success_rate = len(interactions) / total_tasks * 100
print(f"Success rate: {success_rate:.1f}%")
```

## See Also

- [Distractor System](distractor_system.md) - Noise injection details
- [Training Data](training_data.md) - Converting interactions to training samples
- [Interaction API](../api/interaction.md) - Detailed API documentation
