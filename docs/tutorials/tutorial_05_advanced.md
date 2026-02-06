# Tutorial 5: Advanced Usage

Scaling, custom configurations, and production usage.

## Scaling Up

### Batch Processing

```yaml
experiment:
  batch_size: 50              # Process 50 personas at a time
  incremental: true           # Resume from existing

interaction_generation:
  max_workers: 20             # Parallel workers
```

### Incremental Generation

```bash
# First run
python run.py --num-personas 100

# Second run (adds 100 more)
python run.py --num-personas 200
```

## Custom LLM Providers

### OpenRouter

```yaml
openrouter:
  api_key: "${OPENROUTER_API_KEY}"
  base_url: "https://openrouter.ai/api/v1"

interaction_generation:
  assistant_model:
    model_pool:
      - provider: openrouter
        model: anthropic/claude-3.5-haiku
        weight: 0.5
      - provider: openrouter
        model: google/gemini-2.0-flash-exp
        weight: 0.5
```

## Token Cost Management

### Monitor Usage

```python
from src.token_tracker import get_tracker

tracker = get_tracker()
stats = tracker.get_statistics()

print(f"Total tokens: {stats['summary']['total_tokens']:,}")
print(f"Estimated cost: ${estimate_cost(stats):.2f}")
```

### Cost Optimization

1. Use smaller models for user feedback
2. Reduce `max_completion_tokens`
3. Lower `queries_per_persona`

## Production Pipeline

```python
from src.enhanced_pipeline import EnhancedPersonaGenerationPipeline

def run_production():
    pipeline = EnhancedPersonaGenerationPipeline("config.yaml")

    try:
        result = pipeline.run(num_personas=1000)
        print(f"Success: {result['training_data']['total_samples']} samples")
    except Exception as e:
        print(f"Error: {e}")
        # Incremental mode allows resume

if __name__ == "__main__":
    run_production()
```

## HuggingFace Export

```bash
# Convert to HF format
python scripts/prepare_ppopt_hf_jsonl.py \
    --input output/training_data/train_samples.json \
    --output dataset.jsonl

# Upload
python scripts/upload_hf_dataset.py \
    --file dataset.jsonl \
    --repo your-username/ppopt-dataset
```

## Summary

You've learned:

- Pipeline configuration and execution
- Persona customization
- Query and interaction generation
- Noise injection for robustness
- Scaling and production usage

## Next Steps

- Explore [Examples](../examples/basic_pipeline.md)
- Read [API Reference](../api/index.md)
- Check [Configuration Guide](../user_guide/configuration.md)
