# Interaction API

Multi-turn dialogue generation classes.

## InteractionGenerator

```python
from src.interaction_generator import InteractionGenerator
```

### Class Definition

```python
class InteractionGenerator:
    """
    Generates multi-turn interactions with integrated distractor.

    Features:
    - Multi-turn conversation simulation
    - Weighted model pool for diverse responses
    - Real-time noise injection
    - Retry logic for failed generations
    """
```

### Constructor

```python
def __init__(
    self,
    config: Dict[str, Any],
    distractor: Optional[Union[DistractorModel, SemanticDistractorModel]] = None
):
    """
    Initialize generator.

    Args:
        config: Configuration dictionary
        distractor: Optional distractor for noise injection
    """
```

### Methods

```python
def generate_interaction(
    self,
    persona_id: str,
    persona_features: Dict[str, str],
    initial_query: str,
    system_prompt: Optional[str] = None,
    original_query: Optional[str] = None,
    target_turns: int = 3,
    query_id: Optional[str] = None
) -> Optional[Interaction]:
    """
    Generate a complete multi-turn interaction.

    Args:
        persona_id: Persona identifier
        persona_features: User persona features
        initial_query: Query after style transfer
        system_prompt: Optional personalized system prompt
        original_query: Query before style transfer
        target_turns: Target conversation turns (2-5)
        query_id: Query identifier for tracking

    Returns:
        Interaction object or None if failed
    """

def generate_interactions_batch(
    self,
    personas_with_queries: List[Dict[str, Any]],
    show_progress: bool = True,
    max_workers: int = 5,
    storage: Optional[InteractionStorage] = None
) -> List[Interaction]:
    """
    Generate interactions for multiple personas (concurrent).

    Args:
        personas_with_queries: List of persona-query pairs
        show_progress: Show progress bar
        max_workers: Parallel workers
        storage: Optional storage for immediate saving

    Returns:
        List of generated Interaction objects
    """
```

---

## Interaction

```python
from src.interaction_generator import Interaction
```

### Class Definition

```python
@dataclass
class Interaction:
    """
    Represents a complete multi-turn interaction.

    Attributes:
        interaction_id: Unique identifier
        persona_id: Associated persona
        persona_features: Persona features
        original_query: Query before style transfer
        initial_query: Query after style transfer
        messages: List of Message objects
        num_turns: Number of conversation turns
        metadata: Additional metadata
    """
```

### Methods

```python
def to_dict(self) -> Dict[str, Any]:
    """Convert to dictionary."""

def get_trajectory(self) -> List[str]:
    """Get user prompt trajectory."""

def get_full_conversation(self) -> List[Dict[str, str]]:
    """Get full conversation as message list."""
```

---

## Message

```python
from src.interaction_generator import Message
```

### Class Definition

```python
@dataclass
class Message:
    """
    Single message in a conversation.

    Attributes:
        role: 'user' or 'assistant'
        content: Message text
        timestamp: ISO timestamp
        metadata: Additional info (turn, model, etc.)
    """
```

---

## InteractionStorage

```python
from src.interaction_generator import InteractionStorage
```

### Class Definition

```python
class InteractionStorage:
    """
    Manages incremental storage of interactions.

    Saves each interaction immediately to disk with index tracking.
    """
```

### Constructor

```python
def __init__(self, output_dir: str, collector=None):
    """
    Initialize storage.

    Args:
        output_dir: Directory for interaction files
        collector: Optional TrainingDataCollector for format conversion
    """
```

### Methods

```python
def save(self, interaction: Interaction) -> str:
    """Save interaction to disk. Returns file path."""

def load(self, interaction_id: str) -> Dict[str, Any]:
    """Load interaction by ID."""

def load_all(self) -> List[Dict[str, Any]]:
    """Load all stored interactions."""
```

---

## AssistantModel

```python
from src.interaction_generator import AssistantModel
```

### Class Definition

```python
class AssistantModel:
    """
    Simulates AI assistant responses.

    Supports:
    - Single model or weighted model pool
    - Thread-safe model locking for conversations
    """
```

### Methods

```python
def respond(
    self,
    user_query: str,
    conversation_history: List[Message]
) -> str:
    """Generate assistant response."""

def lock_model(self) -> None:
    """Lock model for current conversation (thread-safe)."""

def unlock_model(self) -> None:
    """Release model lock."""
```

---

## UserFeedbackModel

```python
from src.interaction_generator import UserFeedbackModel
```

### Class Definition

```python
class UserFeedbackModel:
    """
    Simulates user feedback based on persona.

    Decides whether user continues conversation or is satisfied.
    """
```

### Methods

```python
def generate_feedback(
    self,
    persona_features: Dict[str, str],
    conversation_history: List[Message],
    assistant_response: str,
    current_turn: int,
    target_turns: int
) -> Optional[str]:
    """
    Generate user feedback.

    Returns:
        Feedback text or None if user is satisfied
    """
```

---

## Usage Examples

### Single Interaction

```python
from src.interaction_generator import InteractionGenerator
from src.distractor import create_distractor_model

distractor = create_distractor_model(config, use_semantic=True)
generator = InteractionGenerator(config, distractor=distractor)

interaction = generator.generate_interaction(
    persona_id="persona_001",
    persona_features={'role': 'engineer', 'style': 'casual'},
    initial_query="help me with python",
    target_turns=3
)

if interaction:
    print(f"Generated {interaction.num_turns} turns")
    for msg in interaction.messages:
        print(f"{msg.role}: {msg.content[:50]}...")
```

### Batch Generation

```python
personas_with_queries = [
    {
        'persona_id': 'persona_001',
        'persona_features': features,
        'queries': [
            {'adapted_query': 'query1', 'original_query': 'Query 1'}
        ]
    }
]

storage = InteractionStorage("output/interactions")

interactions = generator.generate_interactions_batch(
    personas_with_queries,
    show_progress=True,
    max_workers=10,
    storage=storage
)
```

---

## See Also

- [Interaction Generation](../user_guide/interaction_generation.md) - User guide
- [Distractor API](distractor.md) - Noise injection
