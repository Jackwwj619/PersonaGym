# Multi-Provider LLM Example

Use multiple LLM providers for diverse responses.

## Overview

This example demonstrates:
- Configuring multiple LLM providers
- Weighted model pool for assistant
- OpenRouter integration

## Configuration

```yaml
# config.yaml

api:
  provider: "openai"
  api_key: "${OPENAI_API_KEY}"
  model: "gpt-4o-mini"

openrouter:
  api_key: "${OPENROUTER_API_KEY}"
  base_url: "https://openrouter.ai/api/v1"

interaction_generation:
  assistant_model:
    model_pool:
      # OpenAI models
      - provider: openai
        model: gpt-4o-mini
        weight: 0.2
      - provider: openai
        model: gpt-4o
        weight: 0.1

      # Anthropic via OpenRouter
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.2

      # Google via OpenRouter
      - provider: openrouter
        model: google/gemini-2.0-flash-exp
        weight: 0.2

      # Other models
      - provider: openrouter
        model: mistralai/mistral-small-3.1-24b-instruct
        weight: 0.15
      - provider: openrouter
        model: meta-llama/llama-3.1-405b-instruct
        weight: 0.15
```

## Programmatic Usage

```python
"""
Multi-Provider LLM Example
"""

from src.llm_client import create_llm_client

def main():
    # OpenAI client
    openai_client = create_llm_client({
        'provider': 'openai',
        'api_key': 'sk-...',
        'model': 'gpt-4o-mini',
        'inference': {'temperature': 0.7}
    })

    # OpenRouter client (Anthropic)
    anthropic_client = create_llm_client({
        'provider': 'openrouter',
        'api_key': 'sk-or-...',
        'model': 'anthropic/claude-3.5-haiku',
        'base_url': 'https://openrouter.ai/api/v1',
        'inference': {'temperature': 0.8}
    })

    # Test both
    prompt = "Explain Python decorators in one sentence."

    print("=== OpenAI ===")
    response, inp, out = openai_client.generate_with_tokens(prompt)
    print(f"Response: {response}")
    print(f"Tokens: {inp} in, {out} out")

    print("\n=== Anthropic (OpenRouter) ===")
    response, inp, out = anthropic_client.generate_with_tokens(prompt)
    print(f"Response: {response}")
    print(f"Tokens: {inp} in, {out} out")

if __name__ == "__main__":
    main()
```

## Model Selection

The model pool selects models by weight:

```python
# 20% chance: gpt-4o-mini
# 10% chance: gpt-4o
# 20% chance: claude-3.5-haiku
# etc.
```

Model is **locked** for entire conversation (same model across all turns).

## Token Tracking

Usage is tracked per-model:

```python
from src.token_tracker import get_tracker

tracker = get_tracker()
stats = tracker.get_statistics()

for model, usage in stats['by_model'].items():
    print(f"{model}: {usage['total_tokens']:,} tokens")
```

Output:

```
openai/gpt-4o-mini: 45,000 tokens
openrouter/anthropic/claude-3.5-haiku: 32,000 tokens
openrouter/google/gemini-2.0-flash-exp: 28,000 tokens
```

## Key Takeaways

- Multiple providers increase response diversity
- Weights control model selection frequency
- Token tracking works across all providers

## See Also

- [LLM Client API](../api/llm_client.md)
- [Configuration Guide](../user_guide/configuration.md)
