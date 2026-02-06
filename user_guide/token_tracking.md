# Token Tracking

This guide covers the token usage tracking and cost analysis system in LLM_PPOpt.

## Overview

LLM_PPOpt automatically tracks all API token usage:

- **Per-module tracking**: Persona generation, query generation, interaction, distractor
- **Per-model tracking**: Usage by each LLM model
- **Cost analysis**: Estimate costs based on token usage
- **Export**: JSON format for downstream analysis

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      TokenTracker (Singleton)                   │
├─────────────────────────────────────────────────────────────────┤
│  record(module, operation, model, input_tokens, output_tokens)  │
│                              ↓                                   │
│  ┌──────────────────────────────────────────────────────┐       │
│  │  TokenUsage Records                                   │       │
│  │  - module: str                                        │       │
│  │  - operation: str                                     │       │
│  │  - model: str                                         │       │
│  │  - provider: str                                      │       │
│  │  - input_tokens: int                                  │       │
│  │  - output_tokens: int                                 │       │
│  │  - timestamp: str                                     │       │
│  └──────────────────────────────────────────────────────┘       │
│                              ↓                                   │
│  get_statistics() / export_to_file()                            │
└─────────────────────────────────────────────────────────────────┘
```

## Basic Usage

### Automatic Tracking

Token tracking is automatic when using the pipeline:

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
result = pipeline.run(num_personas=10)

# Token statistics printed automatically at end
# Also exported to output/training_data/token_usage_*.json
```

### Manual Tracking

```python
from src.token_tracker import get_tracker, record_tokens

# Get singleton tracker
tracker = get_tracker()

# Record token usage
record_tokens(
    module='custom_module',
    operation='generate_text',
    model='gpt-4o-mini',
    provider='openai',
    input_tokens=150,
    output_tokens=75
)

# Get statistics
stats = tracker.get_statistics()
print(f"Total tokens: {stats['summary']['total_tokens']}")
```

## TokenUsage Structure

```python
@dataclass
class TokenUsage:
    module: str           # Module name (persona_formulation, interaction_generation, etc.)
    operation: str        # Operation type (formulate_prompt, assistant_response, etc.)
    model: str            # Model name (gpt-4o-mini, claude-3.5-haiku, etc.)
    provider: str         # Provider (openai, anthropic, openrouter)
    input_tokens: int     # Input/prompt tokens
    output_tokens: int    # Output/completion tokens
    timestamp: str        # ISO timestamp
    metadata: Dict        # Additional info (turn number, etc.)
```

## Statistics Output

### Summary

```python
stats = tracker.get_statistics()

print(stats['summary'])
# {
#   'total_calls': 500,
#   'total_input_tokens': 250000,
#   'total_output_tokens': 150000,
#   'total_tokens': 400000
# }
```

### By Module

```python
print(stats['by_module'])
# {
#   'persona_formulation': {
#     'input_tokens': 50000,
#     'output_tokens': 25000,
#     'total_tokens': 75000,
#     'call_count': 100
#   },
#   'query_generation': {
#     'input_tokens': 30000,
#     'output_tokens': 15000,
#     'total_tokens': 45000,
#     'call_count': 100
#   },
#   'interaction_generation': {
#     'input_tokens': 150000,
#     'output_tokens': 100000,
#     'total_tokens': 250000,
#     'call_count': 250
#   },
#   'distractor': {
#     'input_tokens': 20000,
#     'output_tokens': 10000,
#     'total_tokens': 30000,
#     'call_count': 50
#   }
# }
```

### By Model

```python
print(stats['by_model'])
# {
#   'openai/gpt-4o-mini': {
#     'input_tokens': 100000,
#     'output_tokens': 60000,
#     'total_tokens': 160000,
#     'call_count': 200
#   },
#   'openrouter/anthropic/claude-3.5-haiku': {
#     'input_tokens': 80000,
#     'output_tokens': 50000,
#     'total_tokens': 130000,
#     'call_count': 150
#   }
# }
```

## Export Format

### JSON Output

```json
{
  "summary": {
    "total_calls": 500,
    "total_input_tokens": 250000,
    "total_output_tokens": 150000,
    "total_tokens": 400000
  },
  "by_module": {
    "persona_formulation": {
      "input_tokens": 50000,
      "output_tokens": 25000,
      "total_tokens": 75000,
      "call_count": 100
    },
    ...
  },
  "by_model": {
    "openai/gpt-4o-mini": {...},
    ...
  },
  "records": [
    {
      "module": "persona_formulation",
      "operation": "formulate_prompt",
      "model": "gpt-4o-mini",
      "provider": "openai",
      "input_tokens": 150,
      "output_tokens": 75,
      "total_tokens": 225,
      "timestamp": "2026-02-06T10:30:00",
      "metadata": {}
    },
    ...
  ]
}
```

### Export Methods

```python
# Export to file
tracker.export_to_file(
    "output/token_usage.json",
    include_records=True  # Include individual records
)

# Get as dictionary
data = tracker.to_dict(include_records=True)
```

## Cost Analysis

### Estimate Costs

```python
# Approximate pricing (as of 2026)
PRICING = {
    'gpt-4o-mini': {'input': 0.00015, 'output': 0.0006},  # per 1K tokens
    'gpt-4o': {'input': 0.005, 'output': 0.015},
    'claude-3.5-haiku': {'input': 0.00025, 'output': 0.00125},
}

def estimate_cost(stats):
    total_cost = 0
    for model, usage in stats['by_model'].items():
        model_name = model.split('/')[-1]
        if model_name in PRICING:
            pricing = PRICING[model_name]
            input_cost = usage['input_tokens'] / 1000 * pricing['input']
            output_cost = usage['output_tokens'] / 1000 * pricing['output']
            total_cost += input_cost + output_cost
    return total_cost

cost = estimate_cost(tracker.get_statistics())
print(f"Estimated cost: ${cost:.2f}")
```

### Cost per Sample

```python
stats = tracker.get_statistics()
num_samples = result['training_data']['total_samples']

tokens_per_sample = stats['summary']['total_tokens'] / num_samples
print(f"Average tokens per sample: {tokens_per_sample:.0f}")

cost_per_sample = cost / num_samples
print(f"Cost per sample: ${cost_per_sample:.4f}")
```

## Module Breakdown

### Tracked Modules

| Module | Operations | Description |
|--------|------------|-------------|
| `persona_formulation` | `formulate_prompt` | System prompt generation |
| `query_generation` | `style_transfer` | Query style adaptation |
| `interaction_generation` | `assistant_response`, `user_feedback` | Conversation simulation |
| `distractor` | `extract_semantics`, `generate_noise` | Noise injection |

### Cost Distribution

Typical distribution (from analysis):

| Module | % of Total Tokens |
|--------|------------------|
| Interaction Generation | ~78% |
| Query Generation | ~10% |
| Distractor | ~6% |
| Persona Formulation | ~5% |

## Analysis Scripts

### analyze_token_usage.py

```bash
python analysis/analyze_token_usage.py \
    --input output/training_data/token_usage_*.json \
    --output analysis_report.md
```

### analyze_for_paper.py

```bash
python analysis/analyze_for_paper.py
```

Output:

```
=== Token Usage Analysis for Paper ===

Total Samples: 500
Total Token Usage: 8,831,400 tokens

Average Token Cost Per Sample: 17,662.8 tokens

Breakdown by Module:
  persona_formulation: 5.3%
  query_generation: 10.3%
  interaction_generation: 78.6%
  distractor: 5.8%
```

## API Reference

### TokenTracker

```python
class TokenTracker:
    """Singleton token usage tracker (thread-safe)."""

    @classmethod
    def get_instance(cls) -> 'TokenTracker':
        """Get singleton instance."""

    def record(
        self,
        module: str,
        operation: str,
        model: str,
        provider: str,
        input_tokens: int,
        output_tokens: int,
        metadata: Optional[Dict] = None
    ) -> None:
        """Record token usage."""

    def get_statistics(self) -> Dict[str, Any]:
        """Get aggregated statistics."""

    def export_to_file(
        self,
        filepath: str,
        include_records: bool = True
    ) -> None:
        """Export statistics to JSON file."""

    def print_summary(self) -> None:
        """Print formatted summary to console."""

    def reset(self) -> None:
        """Clear all records."""
```

### Convenience Functions

```python
def get_tracker() -> TokenTracker:
    """Get singleton TokenTracker instance."""

def record_tokens(
    module: str,
    operation: str,
    model: str,
    provider: str,
    input_tokens: int,
    output_tokens: int,
    metadata: Optional[Dict] = None
) -> None:
    """Record token usage (convenience function)."""
```

## Thread Safety

The TokenTracker is thread-safe for concurrent recording:

```python
import threading

def worker(tracker, module):
    for i in range(100):
        tracker.record(
            module=module,
            operation='test',
            model='gpt-4o-mini',
            provider='openai',
            input_tokens=100,
            output_tokens=50
        )

# Safe for concurrent use
threads = [
    threading.Thread(target=worker, args=(tracker, f'module_{i}'))
    for i in range(4)
]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

## Best Practices

### 1. Enable Tracking Early

```python
# Tracker is automatically initialized
from src.token_tracker import get_tracker
tracker = get_tracker()
```

### 2. Export Regularly

```python
# Export after each major operation
if tracker.get_statistics()['summary']['total_calls'] > 100:
    tracker.export_to_file(f"token_usage_{timestamp}.json")
```

### 3. Monitor Cost During Development

```python
# Quick cost check
stats = tracker.get_statistics()
print(f"Tokens so far: {stats['summary']['total_tokens']:,}")
```

### 4. Analyze Before Scale-Up

```python
# Run small test
result = pipeline.run(num_personas=5)

# Check cost
tokens_per_persona = stats['summary']['total_tokens'] / 5
estimated_total = tokens_per_persona * 1000  # For 1000 personas
print(f"Estimated for 1000 personas: {estimated_total:,} tokens")
```

## See Also

- [Configuration](configuration.md) - Pipeline configuration
- [Training Data](training_data.md) - Output alongside token stats
- [Utils API](../api/utils.md) - TokenTracker API details
