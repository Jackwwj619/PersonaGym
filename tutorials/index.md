# Tutorials

Step-by-step guides for learning LLM_PPOpt.

## Tutorial Overview

These tutorials will guide you through the key concepts and workflows of LLM_PPOpt.

| Tutorial | Description | Duration |
|----------|-------------|----------|
| [Tutorial 1: Basics](tutorial_01_basics.md) | Core concepts and first pipeline run | ~15 min |
| [Tutorial 2: Persona Generation](tutorial_02_persona_generation.md) | Deep dive into persona system | ~20 min |
| [Tutorial 3: Query & Interaction](tutorial_03_query_interaction.md) | Query generation and conversations | ~25 min |
| [Tutorial 4: Noise Injection](tutorial_04_noise_injection.md) | Distractor system configuration | ~20 min |
| [Tutorial 5: Advanced Usage](tutorial_05_advanced.md) | Custom configurations and scaling | ~30 min |

## Prerequisites

Before starting the tutorials:

1. **Install LLM_PPOpt** following the [Installation Guide](../installation.md)
2. **Configure API keys** in `.env` or `config.yaml`
3. **Verify installation** with a test run:
   ```bash
   python run.py --num-personas 1
   ```

## What You'll Learn

By completing these tutorials, you will:

- Understand the 6-stage pipeline architecture
- Configure personas with custom dimensions
- Generate and adapt queries to personas
- Simulate multi-turn conversations
- Apply semantic noise for model robustness
- Export training data for downstream use
- Scale up generation efficiently

## Example Notebooks

For interactive learning, check out the Jupyter notebooks in the `notebooks/` directory (if available).

## Getting Help

If you encounter issues:

1. Check the [Troubleshooting](../installation.md#troubleshooting) section
2. Review the [API Reference](../api/index.md)
3. Open an issue on [GitHub](https://github.com/yccm/LLM_PPOpt/issues)

## Let's Get Started!

Begin with [Tutorial 1: Basics](tutorial_01_basics.md) to understand the core concepts.
