# Basic Pipeline Example

A minimal example of running the basic persona generation pipeline.

## Overview

This example demonstrates:
- Basic pipeline initialization
- Persona generation without interactions
- Exporting persona dataset

## Complete Example

```python
"""
Basic Pipeline Example

Generate personas with system prompts (no interactions).
"""

from src.pipeline import PersonaGenerationPipeline

def main():
    # Initialize pipeline
    pipeline = PersonaGenerationPipeline("config.yaml")

    # Run basic generation
    result = pipeline.run(
        num_personas=100,
        generate_prompts=True,
        export_dataset=True,
        dataset_path="output/personas_dataset.json"
    )

    # Print results
    print(f"Generated {result['num_personas']} personas")
    print(f"Dataset exported to: {result['dataset_path']}")

    # Access individual personas
    for persona in result['personas'][:3]:
        print(f"\nPersona: {persona.persona_id}")
        print(f"Features: {persona.features}")
        print(f"Prompt length: {len(persona.system_prompt or '')} chars")

if __name__ == "__main__":
    main()
```

## Expected Output

```
Generated 100 personas
Dataset exported to: output/personas_dataset.json

Persona: persona_20260206_001
Features: {'age_band': '25_34', 'role': 'engineer', ...}
Prompt length: 256 chars

Persona: persona_20260206_002
Features: {'age_band': '18_24', 'role': 'student', ...}
Prompt length: 312 chars
```

## Command Line Equivalent

```bash
python run.py --mode basic --num-personas 100
```

## Configuration

Minimal configuration for basic mode:

```yaml
api:
  provider: "openai"
  api_key: "${OPENAI_API_KEY}"
  model: "gpt-4o-mini"

persona_generation:
  num_personas: 100
  feature_availability_rate: 0.7

formulation:
  system_prompt_template: "prompts/persona_to_system_prompt.txt"
```

## Key Takeaways

- Basic mode is faster (no interactions)
- Good for generating large persona datasets
- Personas can be used independently for other tasks

## Next Steps

- [Enhanced Pipeline Example](enhanced_pipeline.md) - Full data generation
- [Custom Persona Example](custom_persona.md) - Persona customization
