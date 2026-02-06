# Tutorial 2: Persona Generation

Deep dive into the persona system and customization.

## Learning Objectives

- Understand persona dimensions and constraints
- Customize persona configurations
- Control diversity in sampling

## Persona Dimensions

Personas are defined by dimensions in `input/persona.yaml`:

```yaml
dimensions:
  age_band:
    name: age_band
    is_constraint: true        # Always included
    values:
      - u18
      - 18_24
      - 25_34
      - 35_44
      - 45_60
      - 60p

  role:
    name: role
    is_constraint: true
    values:
      - student
      - engineer
      - data
      - pm
      - designer
      # ... more roles

  communication_style:
    name: communication_style
    is_constraint: false       # Optionally included
    values:
      - casual
      - professional
      - technical
      - creative
```

### Constraint vs Optional

| Type | `is_constraint` | Behavior |
|------|-----------------|----------|
| Constraint | `true` | Always sampled |
| Optional | `false` | Sampled based on `feature_availability_rate` |

## Customizing Dimensions

### Add a New Dimension

Edit `input/persona.yaml`:

```yaml
dimensions:
  # Add custom dimension
  industry:
    name: industry
    is_constraint: false
    values:
      - technology
      - healthcare
      - finance
      - education
      - manufacturing
```

### Modify Values

```yaml
dimensions:
  role:
    name: role
    is_constraint: true
    values:
      - student
      - researcher        # Added
      - software_engineer # More specific
      - data_scientist    # Added
      - product_manager
```

## Sampling Configuration

Configure sampling in `input/sampling_config.yaml`:

```yaml
sampling:
  feature_availability_rate: 0.7   # 70% of optional dims
  min_features: 10
  max_features: 20
  required_dimensions:
    - query_length_pref            # Always include

diversity:
  enabled: true
  min_hamming_distance: 3          # Minimum difference
  max_retries: 100
```

### Diversity Enforcement

Two personas with Hamming distance = 2:

```python
persona_1 = {'age': '25_34', 'role': 'engineer', 'style': 'casual'}
persona_2 = {'age': '25_34', 'role': 'designer', 'style': 'formal'}
# Differs in: role, style (distance = 2)
```

If `min_hamming_distance = 3`, persona_2 would be rejected.

## Programmatic Persona Generation

### Using PersonaSampler

```python
from src.persona_bank import PersonaBank
from src.sampling import PersonaSampler

# Load dimensions
bank = PersonaBank("input/persona.yaml")
print(f"Dimensions: {list(bank.get_all_dimensions().keys())}")

# Initialize sampler
sampler = PersonaSampler("input/sampling_config.yaml", bank)

# Sample personas
for i in range(5):
    features = sampler.sample_persona()
    print(f"Persona {i+1}: {features}")
```

### Inspect Dimensions

```python
# Get constraint dimensions
constraints = bank.get_constraint_dimensions()
print(f"Constraints: {constraints}")

# Get values for a dimension
roles = bank.get_dimension_values('role')
print(f"Roles: {roles}")
```

## System Prompt Generation

Personas get personalized system prompts:

```python
from src.llm_client import create_llm_client, LLMFormulator

client = create_llm_client(config['api'])
formulator = LLMFormulator(
    client,
    template_path="prompts/persona_to_system_prompt.txt"
)

# Generate prompt from features
features = {'role': 'engineer', 'style': 'casual', 'level': 'advanced'}
prompt = formulator.formulate(features)
print(prompt)
```

Output:

```
You are a helpful AI assistant. You are currently helping an advanced-level
engineer who prefers casual communication. Adapt your responses to be:
- Technical but approachable
- Concise and practical
- Friendly in tone
```

## Customize Prompt Template

Edit `prompts/persona_to_system_prompt.txt`:

```
You are an AI assistant helping a user with the following profile:

{persona_features}

Guidelines:
- Match the user's communication style: {communication_style}
- Adjust technical depth to: {technical_level}
- Keep responses: {response_length}

Be helpful, accurate, and tailored to this user's needs.
```

## Exercises

1. **Add a "hobby" dimension** with 5 values
2. **Increase min_hamming_distance** to 5 and observe sampling behavior
3. **Create a custom prompt template** for your use case

## Next Steps

Continue to [Tutorial 3: Query & Interaction](tutorial_03_query_interaction.md) to learn about generating conversations.
