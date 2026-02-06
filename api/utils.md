# Utils API

Utility classes and functions.

## TokenTracker

```python
from src.token_tracker import TokenTracker, get_tracker, record_tokens
```

### Class Definition

```python
class TokenTracker:
    """
    Singleton token usage tracker (thread-safe).

    Features:
    - Per-module, per-model tracking
    - Aggregated statistics
    - JSON export
    """
```

### Class Methods

```python
@classmethod
def get_instance(cls) -> 'TokenTracker':
    """Get singleton instance."""
```

### Instance Methods

```python
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
    """
    Record token usage.

    Args:
        module: Module name (persona_formulation, interaction_generation, etc.)
        operation: Operation type
        model: Model name
        provider: Provider name
        input_tokens: Input/prompt tokens
        output_tokens: Output/completion tokens
        metadata: Additional info
    """

def get_statistics(self) -> Dict[str, Any]:
    """
    Get aggregated statistics.

    Returns:
        Dictionary with:
        - summary: Total counts
        - by_module: Per-module breakdown
        - by_model: Per-model breakdown
    """

def export_to_file(
    self,
    filepath: str,
    include_records: bool = True
) -> None:
    """
    Export statistics to JSON file.

    Args:
        filepath: Output file path
        include_records: Include individual records
    """

def print_summary(self) -> None:
    """Print formatted summary to console."""

def reset(self) -> None:
    """Clear all records."""

def to_dict(self, include_records: bool = True) -> Dict[str, Any]:
    """Convert to dictionary."""
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
    """Record token usage (convenience wrapper)."""
```

---

## TokenUsage

```python
from src.token_tracker import TokenUsage
```

### Class Definition

```python
@dataclass
class TokenUsage:
    """
    Single token usage record.

    Attributes:
        module: Module name
        operation: Operation type
        model: Model name
        provider: Provider name
        input_tokens: Input tokens
        output_tokens: Output tokens
        timestamp: ISO timestamp
        metadata: Additional info
    """
```

---

## ColoredLogger

```python
from src.colored_logger import ColoredLogger, print_colored
```

### Functions

```python
def print_colored(message: str, color_type: str) -> None:
    """
    Print colored message to console.

    Args:
        message: Text to print
        color_type: Color type ('STAGE', 'SUCCESS', 'WARNING', 'ERROR', 'INFO', 'GENERATION')
    """
```

### Color Types

| Type | Color | Use Case |
|------|-------|----------|
| `STAGE` | Blue | Stage headers |
| `SUCCESS` | Green | Success messages |
| `WARNING` | Yellow | Warnings |
| `ERROR` | Red | Errors |
| `INFO` | Cyan | Information |
| `GENERATION` | Magenta | Generation progress |

---

## Config Validation

```python
from src.config_validation import validate_config, validate_config_report
```

### Functions

```python
def validate_config(
    config: Dict[str, Any],
    config_path: Optional[str] = None
) -> Tuple[Dict[str, Any], List[str]]:
    """
    Validate configuration (fail-fast).

    Args:
        config: Configuration dictionary
        config_path: Path to config file (for relative path resolution)

    Returns:
        Tuple of (validated_config, list_of_issues)

    Raises:
        ValueError: If critical validation fails
    """

def validate_config_report(
    config: Dict[str, Any],
    config_path: Optional[str] = None
) -> Tuple[Dict[str, Any], List[ValidationCheck], List[ValidationIssue]]:
    """
    Validate with detailed reporting.

    Returns:
        Tuple of (config, checks, issues)
    """
```

### ValidationCheck

```python
@dataclass
class ValidationCheck:
    """
    Validation check result.

    Attributes:
        name: Check name
        status: 'PASS', 'WARN', 'FAIL'
        message: Result message
    """
```

### ValidationIssue

```python
@dataclass
class ValidationIssue:
    """
    Validation issue.

    Attributes:
        severity: 'warning' or 'error'
        message: Issue description
        path: Config path (e.g., 'api.api_key')
    """
```

---

## Usage Examples

### Token Tracking

```python
from src.token_tracker import get_tracker, record_tokens

# Get tracker
tracker = get_tracker()

# Record usage
record_tokens(
    module='custom_module',
    operation='generate',
    model='gpt-4o-mini',
    provider='openai',
    input_tokens=100,
    output_tokens=50
)

# Get statistics
stats = tracker.get_statistics()
print(f"Total tokens: {stats['summary']['total_tokens']}")

# Export
tracker.export_to_file("token_usage.json")

# Print summary
tracker.print_summary()
```

### Colored Output

```python
from src.colored_logger import print_colored

print_colored("[Stage 1] Generating Personas...", 'STAGE')
print_colored("Generated 100 personas", 'SUCCESS')
print_colored("Some queries failed", 'WARNING')
print_colored("API error occurred", 'ERROR')
```

### Config Validation

```python
from src.config_validation import validate_config_report
import yaml

with open("config.yaml") as f:
    config = yaml.safe_load(f)

config, checks, issues = validate_config_report(config, "config.yaml")

for check in checks:
    print(f"{check.status}: {check.name}")

for issue in issues:
    print(f"{issue.severity.upper()}: {issue.message}")
```

---

## Thread Safety

TokenTracker is thread-safe:

```python
import threading
from src.token_tracker import get_tracker

tracker = get_tracker()

def worker():
    for _ in range(100):
        tracker.record(
            module='test',
            operation='op',
            model='gpt-4o-mini',
            provider='openai',
            input_tokens=10,
            output_tokens=5
        )

threads = [threading.Thread(target=worker) for _ in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Total calls: {tracker.get_statistics()['summary']['total_calls']}")
# Output: Total calls: 400
```

---

## See Also

- [Token Tracking](../user_guide/token_tracking.md) - User guide
- [Configuration](../user_guide/configuration.md) - Config options
