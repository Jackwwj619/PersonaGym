# Enhanced Pipeline Example

Complete training data generation with all 6 stages.

## Overview

This example demonstrates:
- Full pipeline execution
- Interaction generation with noise
- Training data export

## Complete Example

```python
"""
Enhanced Pipeline Example

Generate complete training data with personas, interactions, and noise.
"""

from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

def main():
    # Initialize enhanced pipeline
    pipeline = EnhancedPersonaGenerationPipeline("config.yaml")

    # Run full pipeline
    result = pipeline.run(num_personas=10)

    # Print summary
    print("\n=== Pipeline Results ===")
    print(f"Personas: {len(result['personas'])}")
    print(f"Queries: {sum(len(q) for q in result['queries'].values())}")
    print(f"Interactions: {len(result['interactions'])}")

    if 'training_data' in result:
        td = result['training_data']
        print(f"Training Samples: {td['total_samples']}")
        print(f"Output Files: {td['sample_files']}")

    # Inspect first interaction
    if result['interactions']:
        inter = result['interactions'][0]
        print(f"\n=== Sample Interaction ===")
        print(f"ID: {inter['interaction_id']}")
        print(f"Turns: {inter['num_turns']}")
        print(f"Distractor: {inter['metadata'].get('distractor_applied')}")

        for msg in inter['messages'][:4]:
            print(f"  {msg['role']}: {msg['content'][:50]}...")

if __name__ == "__main__":
    main()
```

## Expected Output

```
=== Pipeline Results ===
Personas: 10
Queries: 50
Interactions: 50
Training Samples: 50
Output Files: {'train': 'output/training_data/train_samples_*.json'}

=== Sample Interaction ===
ID: interaction_persona_001_20260206_103000
Turns: 3
Distractor: True
  user: hey can u help me with some python code...
  assistant: Of course! I'd be happy to help you with Python...
  user: make it handle errors better...
  assistant: I've added try-except blocks to handle...
```

## Command Line Equivalent

```bash
python run.py --num-personas 10
```

## Stage Control

Run specific stages:

```python
# Configure stages in config.yaml
experiment:
  stages:
    persona_generation: true
    query_generation: true
    interaction_generation: true
    distractor_application: true
    training_data_export: true
```

Or via command line:

```bash
# Only persona and query generation
python run.py --stage persona --num-personas 50
python run.py --stage query
```

## Key Takeaways

- Enhanced mode generates complete training data
- All stages are integrated and automatic
- Incremental mode allows resuming

## Next Steps

- [Custom Persona Example](custom_persona.md)
- [Multi-Provider LLM Example](multi_provider_llm.md)
