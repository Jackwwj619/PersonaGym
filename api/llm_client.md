# LLM Client API

Multi-provider LLM client interface for text generation.

## Overview

```python
from src.llm_client import create_llm_client, LLMClient, OpenAIClient
```

## Factory Function

### create_llm_client

```python
def create_llm_client(config: Dict[str, Any]) -> LLMClient:
    """
    Create an LLM client from configuration.

    Args:
        config: Configuration dictionary with:
            - provider: str ("openai", "anthropic", "openrouter")
            - api_key: str
            - model: str
            - inference: Dict with temperature, max_completion_tokens
            - base_url: Optional[str] for custom endpoints

    Returns:
        LLMClient instance (OpenAIClient or compatible)

    Example:
        >>> client = create_llm_client({
        ...     'provider': 'openai',
        ...     'api_key': 'sk-...',
        ...     'model': 'gpt-4o-mini',
        ...     'inference': {'temperature': 0.7}
        ... })
    """
```

---

## LLMClient (Abstract Base)

```python
from abc import ABC, abstractmethod

class LLMClient(ABC):
    """Abstract base class for LLM clients."""

    @abstractmethod
    def generate(self, prompt: str, **kwargs) -> str:
        """
        Generate text from prompt.

        Args:
            prompt: Input prompt text
            **kwargs: Additional generation parameters

        Returns:
            Generated text
        """

    @abstractmethod
    def generate_with_tokens(
        self,
        prompt: str,
        **kwargs
    ) -> Tuple[str, int, int]:
        """
        Generate text and return token counts.

        Args:
            prompt: Input prompt text
            **kwargs: Additional generation parameters

        Returns:
            Tuple of (generated_text, input_tokens, output_tokens)
        """
```

---

## OpenAIClient

OpenAI-compatible client implementation.

```python
class OpenAIClient(LLMClient):
    """
    OpenAI API client.

    Supports:
    - OpenAI API
    - OpenRouter API
    - Any OpenAI-compatible endpoint
    """
```

### Constructor

```python
def __init__(
    self,
    api_key: str,
    model: str = "gpt-4o-mini",
    temperature: float = 0.7,
    max_completion_tokens: int = 2048,
    base_url: Optional[str] = None,
    default_headers: Optional[Dict[str, str]] = None
):
    """
    Initialize OpenAI client.

    Args:
        api_key: API key
        model: Model name
        temperature: Sampling temperature (0-2)
        max_completion_tokens: Maximum tokens to generate
        base_url: Custom API endpoint (for OpenRouter, etc.)
        default_headers: Additional headers (for OpenRouter)
    """
```

### Methods

#### generate

```python
def generate(self, prompt: str, **kwargs) -> str:
    """
    Generate text from prompt.

    Args:
        prompt: Input text
        **kwargs: Override temperature, max_tokens, etc.

    Returns:
        Generated text

    Example:
        >>> response = client.generate("Hello, world!")
        >>> print(response)
    """
```

#### generate_with_tokens

```python
def generate_with_tokens(
    self,
    prompt: str,
    **kwargs
) -> Tuple[str, int, int]:
    """
    Generate text and return token counts.

    Returns:
        Tuple of (text, input_tokens, output_tokens)

    Example:
        >>> text, inp, out = client.generate_with_tokens("Hello!")
        >>> print(f"Tokens: {inp} in, {out} out")
    """
```

### Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `model` | `str` | Model name |
| `temperature` | `float` | Sampling temperature |
| `max_completion_tokens` | `int` | Max output tokens |

---

## LLMFormulator

Generates system prompts from persona features.

```python
class LLMFormulator:
    """Generates system prompts using LLM."""
```

### Constructor

```python
def __init__(
    self,
    llm_client: LLMClient,
    template_path: str,
    max_retries: int = 3,
    retry_delay: int = 2,
    validate_output: bool = True,
    min_prompt_length: int = 50
):
    """
    Initialize formulator.

    Args:
        llm_client: LLM client for generation
        template_path: Path to prompt template
        max_retries: Retry attempts on failure
        retry_delay: Seconds between retries
        validate_output: Validate generated prompts
        min_prompt_length: Minimum acceptable length
    """
```

### Methods

#### formulate

```python
def formulate(self, features: Dict[str, str]) -> str:
    """
    Generate system prompt from persona features.

    Args:
        features: Persona feature dictionary

    Returns:
        Generated system prompt

    Raises:
        ValueError: If generation fails after retries

    Example:
        >>> prompt = formulator.formulate({
        ...     'role': 'engineer',
        ...     'style': 'casual'
        ... })
    """
```

---

## Usage Examples

### Basic Usage

```python
from src.llm_client import create_llm_client

# Create client
client = create_llm_client({
    'provider': 'openai',
    'api_key': 'sk-...',
    'model': 'gpt-4o-mini',
    'inference': {
        'temperature': 0.7,
        'max_completion_tokens': 1024
    }
})

# Generate text
response = client.generate("Write a haiku about coding")
print(response)

# With token tracking
text, input_tokens, output_tokens = client.generate_with_tokens(
    "Explain Python decorators"
)
print(f"Response: {text}")
print(f"Tokens: {input_tokens} in, {output_tokens} out")
```

### OpenRouter Usage

```python
client = create_llm_client({
    'provider': 'openrouter',
    'api_key': 'sk-or-...',
    'model': 'anthropic/claude-3.5-haiku',
    'base_url': 'https://openrouter.ai/api/v1',
    'inference': {
        'temperature': 0.8
    }
})
```

### System Prompt Generation

```python
from src.llm_client import create_llm_client, LLMFormulator

client = create_llm_client(config['api'])
formulator = LLMFormulator(
    client,
    template_path="prompts/persona_to_system_prompt.txt",
    max_retries=3,
    validate_output=True
)

prompt = formulator.formulate({
    'age_band': '25_34',
    'role': 'engineer',
    'communication_style': 'casual'
})
print(prompt)
```

---

## Error Handling

```python
from openai import APIError, RateLimitError

try:
    response = client.generate(prompt)
except RateLimitError:
    print("Rate limited, waiting...")
    time.sleep(60)
except APIError as e:
    print(f"API error: {e}")
```

---

## See Also

- [Configuration](../user_guide/configuration.md) - API configuration
- [Interaction Generation](../user_guide/interaction_generation.md) - Using model pools
