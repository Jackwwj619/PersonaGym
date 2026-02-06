# Custom Persona Example

Customize persona dimensions and sampling.

## Overview

This example demonstrates:
- Adding custom dimensions
- Modifying sampling configuration
- Programmatic persona generation

## Custom Dimensions

### Edit persona.yaml

```yaml
# input/persona.yaml

dimensions:
  # Standard dimensions
  age_band:
    name: age_band
    is_constraint: true
    values: [u18, 18_24, 25_34, 35_44, 45_60, 60p]

  role:
    name: role
    is_constraint: true
    values:
      - student
      - researcher
      - software_engineer
      - data_scientist
      - product_manager
      - designer

  # Custom dimension: industry
  industry:
    name: industry
    is_constraint: false
    values:
      - technology
      - healthcare
      - finance
      - education
      - retail
      - manufacturing

  # Custom dimension: expertise
  expertise_area:
    name: expertise_area
    is_constraint: false
    values:
      - web_development
      - machine_learning
      - data_analysis
      - devops
      - security
      - mobile
```

## Custom Sampling

### Edit sampling_config.yaml

```yaml
# input/sampling_config.yaml

sampling:
  feature_availability_rate: 0.8   # 80% of optional dims
  min_features: 12
  max_features: 18
  required_dimensions:
    - query_length_pref
    - industry                      # Custom required dimension

diversity:
  enabled: true
  min_hamming_distance: 4          # Higher diversity
  max_retries: 150
```

## Programmatic Example

```python
"""
Custom Persona Generation Example
"""

from src.persona_bank import PersonaBank
from src.sampling import PersonaSampler
from src.persona_spec import PersonaSpec, PersonaSpecStorage, generate_persona_id
from src.llm_client import create_llm_client, LLMFormulator

def main():
    # Load custom persona bank
    bank = PersonaBank("input/persona.yaml")

    # Print available dimensions
    print("Available dimensions:")
    for dim in bank.get_all_dimensions():
        print(f"  - {dim}")

    # Initialize sampler
    sampler = PersonaSampler("input/sampling_config.yaml", bank)

    # Generate custom personas
    storage = PersonaSpecStorage("output/custom_personas")

    for i in range(5):
        # Sample features
        features = sampler.sample_persona()

        # Ensure custom dimensions
        if 'industry' not in features:
            features['industry'] = 'technology'

        # Generate ID and create spec
        persona_id = generate_persona_id(features)
        spec = PersonaSpec(
            persona_id=persona_id,
            features=features,
            system_prompt=None,  # Generate later if needed
            metadata={'custom': True}
        )

        storage.save(spec)
        print(f"Generated: {persona_id}")
        print(f"  Industry: {features.get('industry')}")
        print(f"  Features: {len(features)}")

if __name__ == "__main__":
    main()
```

## Key Takeaways

- Dimensions are fully customizable
- Constraint dimensions are always included
- Diversity can be tuned via Hamming distance

## Next Steps

- [Multi-Provider LLM Example](multi_provider_llm.md)
