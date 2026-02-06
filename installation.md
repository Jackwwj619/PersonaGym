# Installation

This guide covers the installation of LLM_PPOpt and its dependencies.

## Requirements

LLM_PPOpt requires:

- **Python 3.8** or higher
- **pip** package manager
- **API Keys** for LLM providers (OpenAI, Anthropic, or OpenRouter)

### Python Dependencies

The main dependencies include:

| Package | Version | Description |
|---------|---------|-------------|
| `pyyaml` | >=6.0.1 | YAML configuration parsing |
| `openai` | >=1.0.0 | OpenAI API client |
| `anthropic` | >=0.18.0 | Anthropic API client |
| `python-dotenv` | >=1.0.0 | Environment variable loading |
| `tqdm` | >=4.65.0 | Progress bars |
| `numpy` | - | Numerical operations |

## Install from Source

### Step 1: Clone the Repository

```bash
git clone https://github.com/yccm/LLM_PPOpt.git
cd LLM_PPOpt
```

### Step 2: Create Virtual Environment

**Using Conda (Recommended):**

```bash
conda create -n ppopt python=3.10
conda activate ppopt
```

**Using venv:**

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS/Linux
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

## API Keys Configuration

LLM_PPOpt requires API keys for LLM providers. You can configure them in two ways:

### Option 1: Environment Variables (Recommended)

Create a `.env` file in the project root:

```bash
# .env file
OPENAI_API_KEY=sk-your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key
OPENROUTER_API_KEY=your-openrouter-api-key
```

Or export directly:

```bash
export OPENAI_API_KEY="sk-your-openai-api-key"
export ANTHROPIC_API_KEY="your-anthropic-api-key"
export OPENROUTER_API_KEY="your-openrouter-api-key"
```

### Option 2: Configuration File

Add API keys directly to `config.yaml`:

```yaml
api:
  provider: "openai"
  api_key: "sk-your-openai-api-key"
  model: "gpt-4o-mini"

openrouter:
  api_key: "your-openrouter-api-key"
  base_url: "https://openrouter.ai/api/v1"
```

:::{warning}
Never commit API keys to version control. Add `.env` to your `.gitignore` file.
:::

## Verify Installation

Run the following to verify your installation:

```bash
python -c "from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline; print('Installation successful!')"
```

### Test Basic Pipeline

Quick test to ensure the pipeline works:

```bash
# Run with minimal personas to test
python run.py --num-personas 1
```

Expected output:

```
================================================================================
Starting Enhanced Persona Generation Pipeline
Mode: INCREMENTAL (will skip existing outputs)
================================================================================

[Stage 1] Generating Personas...
Generating personas: 100%|████████████████████| 1/1 [00:02<00:00]

[Stage 2] Generating Queries...
...

================================================================================
Pipeline Complete
================================================================================
```

## Troubleshooting

### Import Errors

If you encounter `ModuleNotFoundError`:

```bash
# Ensure you're in the project root directory
cd LLM_PPOpt

# Verify Python path
python -c "import sys; print(sys.path)"
```

### API Authentication Errors

If you see `AuthenticationError`:

1. Verify your API key is correct
2. Check that the key is properly exported or in `.env`
3. Ensure the API key has sufficient credits/permissions

```bash
# Test OpenAI API key
python -c "from openai import OpenAI; c = OpenAI(); print(c.models.list())"
```

### Rate Limiting

If you encounter rate limit errors:

1. Reduce `max_workers` in `config.yaml`:
   ```yaml
   interaction_generation:
     max_workers: 5  # Reduce from 20
   ```

2. Add delays between API calls by reducing batch size:
   ```yaml
   experiment:
     batch_size: 5  # Process fewer personas at once
   ```

### Encoding Issues (Windows)

If you see encoding errors on Windows:

```bash
# Set UTF-8 encoding
set PYTHONIOENCODING=utf-8
chcp 65001
```

## Development Installation

For development and contributing:

```bash
# Clone with SSH
git clone git@github.com:yccm/LLM_PPOpt.git
cd LLM_PPOpt

# Install with dev dependencies
pip install -r requirements.txt
pip install pytest black flake8

# Run tests
pytest
```

## Directory Structure After Installation

After successful installation, your project should have:

```
LLM_PPOpt/
├── config.yaml          # Main configuration
├── .env                  # API keys (create this)
├── run.py               # Entry point
├── requirements.txt     # Dependencies
├── src/                 # Source code
├── input/               # Input data
│   ├── persona.yaml     # Persona dimensions
│   ├── query.jsonl      # Seed queries
│   └── ...
├── output/              # Generated outputs (created on first run)
│   ├── personas/
│   ├── queries/
│   ├── interactions/
│   └── training_data/
└── logs/                # Log files (created on first run)
```

## Next Steps

After installation, check out the [Quick Start](quickstart.md) guide to learn how to use LLM_PPOpt.
