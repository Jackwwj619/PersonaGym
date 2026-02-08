# PersonaGym

Welcome to **PersonaGym** - A comprehensive pipeline for generating personalized synthetic training data through diverse persona-based user-AI interactions.

## What is PersonaGym?

PersonaGym implements a **6-stage data generation pipeline** that creates high-quality synthetic datasets by simulating diverse user-AI conversations with different personas. Unlike traditional data collection methods that require real user interactions, PersonaGym generates training data programmatically while maintaining realistic diversity and semantic complexity.

The pipeline enables:
- **Zero-shot transfer** to new personas without fine-tuning
- **Robust model training** through semantic noise injection
- **Cost-effective data generation** at scale

## Key Features

**6-Stage Pipeline**

```
Persona Generation → Query Adaptation → Interaction Simulation
→ Distractor Application → Training Data Export → Token Analysis
```

**Rich Persona System**
- 30+ dimensions across 5 categories (demographics, communication style, constraints, etc.)
- Diversity-enforced sampling with configurable Hamming distance constraints
- LLM-based system prompt generation from persona features

**Three-Layer Semantic Noise**
- **Surface Noise (50%)**: Intent/slots preserved, surface form changes
- **Incomplete Info (30%)**: Intent clear, slots missing/vague
- **Semantic Ambiguity (20%)**: Intent uncertain or multiple intents
- 9+ noise strategies per layer for realistic imperfect user inputs

**Multi-Provider LLM Support**
- OpenAI (GPT-4o, GPT-5.2, etc.)
- Anthropic (Claude via OpenRouter)
- OpenRouter (Grok, Gemini, Llama, Mistral, etc.)
- Weighted model pool for diverse assistant responses

**Comprehensive Token Tracking**
- Per-module, per-model statistics
- Cost analysis and reporting
- JSON export for downstream analysis

## Quick Example

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

# Initialize and run the full pipeline
pipeline = EnhancedPersonaGenerationPipeline("config.yaml")
result = pipeline.run(num_personas=5)

print(f"Generated {len(result['personas'])} personas")
print(f"Created {len(result['interactions'])} interactions")
print(f"Exported {result['training_data']['total_samples']} training samples")
```

**Command Line Usage:**

```bash
# Run full pipeline with 10 personas
python run.py --num-personas 10

# Run only persona generation
python run.py --stage persona --num-personas 100

# Skip noise injection
python run.py --skip-distractor --num-personas 5
```

## Documentation Structure

```{toctree}
:maxdepth: 2
:caption: Getting Started

installation
quickstart
tutorials/index
```

```{toctree}
:maxdepth: 2
:caption: User Guide

user_guide/configuration
user_guide/persona_system
user_guide/query_generation
user_guide/interaction_generation
user_guide/distractor_system
user_guide/training_data
user_guide/token_tracking
```

```{toctree}
:maxdepth: 2
:caption: Examples

examples/basic_pipeline
examples/enhanced_pipeline
examples/custom_persona
examples/multi_provider_llm
```

```{toctree}
:maxdepth: 2
:caption: API Reference

api/index
api/pipeline
api/llm_client
api/persona
api/query
api/interaction
api/distractor
api/training_data
api/utils
```

```{toctree}
:maxdepth: 1
:caption: Additional Information

license
```

## Installation

```bash
# Clone the repository
git clone https://github.com/yccm/LLM_PPOpt.git
cd LLM_PPOpt

# Install dependencies
pip install -r requirements.txt

# Set up API keys
export OPENAI_API_KEY="your-api-key"
```

See the [Installation](installation.md) guide for detailed instructions.


## License

PersonaGym is released under the MIT License. See [License](license.md) for details.

## Community

- **GitHub:** https://github.com/yccm/LLM_PPOpt

## Citation

If you use this work, please cite:

```bibtex
@article{ma2026synthetic,
  title={Synthetic Interaction Data for Scalable Personalization in Large Language Models},
  author={Ma, Yuchen and Huang, Yue and Wang, Wenjie and Luo, Xiaonan and Zhang, Xiangliang and Feuerriegel, Stefan},
  year={2026}
}
```


## Indices and Tables

- {ref}`genindex`
- {ref}`modindex`
- {ref}`search`
