---
inclusion: auto
---

# Coding Guidelines

## General Principles

### 1. Minimal Code
- Write only the **absolute minimum** code needed
- Avoid verbose implementations
- Remove dead code immediately
- Don't over-engineer solutions

### 2. Builder Pattern
- Use for complex object construction
- Fluent interface with method chaining
- Example: `Preprocessor(data).normalize().filter().build()`

### 3. Factory Pattern
- Use for object creation with multiple implementations
- Example: `EngineFactory.create("auto")` returns appropriate engine

### 4. Single Responsibility
- Each class does ONE thing well
- Each method does ONE thing well
- If a method is > 50 lines, consider splitting

### 5. No Mocking in Tests
- KHÔNG dùng mock cho đối tượng đang được test (subject under test)
- Có thể dùng mock cho external dependencies: API calls, database, file system, hardware
- Property-based tests generate many inputs

## Python Specific

### Type Hints
```python
# Always use type hints
def process_data(payload: bytes, threshold: float = 0.8) -> tuple[str, float]:
    """Process payload and return (result, score)."""
    pass

# For complex types
from typing import Optional, Dict, List
def find_matches(items: Dict[str, bytes]) -> Optional[List[tuple]]:
    pass
```

### Docstrings
```python
def method_name(param1: str, param2: int) -> bool:
    """Short description.
    
    Longer description if needed.
    
    Args:
        param1: Description of param1
        param2: Description of param2
    
    Returns:
        Description of return value
    
    Raises:
        ValueError: When something is wrong
    """
    pass
```

### Error Handling
```python
# Good: Return None/False for expected failures
def load_resource(path: str) -> Optional[dict]:
    try:
        if not os.path.exists(path):
            return None
        return parse_file(path)
    except Exception as e:
        logger.error(f"Error: {e}")
        return None
```

### Thread-Safe UI Updates (CRITICAL!)
```python
# ❌ BAD: Direct UI update from background thread (WILL CRASH!)
def background_task():
    # This runs in background thread
    self.text_widget.append("Log message")  # CRASH!

# ✅ GOOD: Use signals/events for thread-safe UI updates
# background_thread → emit signal → event queue → UI thread → update widget
```

**Rules:**
- ⛔ **NEVER** update UI widgets directly from background threads
- ⛔ **NEVER** call UI widget methods directly from non-UI threads
- ✅ **ALWAYS** use signals/events with QueuedConnection for cross-thread UI updates
- ✅ **ALWAYS** emit signals from background threads, process in UI thread
- ✅ Pattern: `background_thread → emit signal → event queue → UI thread → update widget`

## File Size Guidelines

| Component        | Max Lines | Typical |
|------------------|-----------|---------|
| Logic module     | 500       | 300-400 |
| UI component     | 400       | 200-300 |
| Test file        | 300       | 150-250 |
| Package __init__ | 50        | 20-30   |

## Import Organization

```python
# 1. Standard library
import os
import sys
import time
from datetime import datetime
from typing import Optional, Dict

# 2. Third-party
import numpy as np

# 3. Local - absolute
from models import DataRecord

# 4. Local - relative
from .service import ServiceFactory
```

## Naming Conventions

### Variables & Functions
```python
# snake_case for variables and functions
file_path = "path/to/file.json"
item_cache = {}

def process_item(data: dict) -> dict:
    pass

def _private_helper():
    pass
```

### Classes
```python
# PascalCase for classes
class TaskExecutor:
    pass

class DataPreprocessor:
    pass
```

### Constants
```python
# UPPER_CASE for constants
DEFAULT_THRESHOLD = 0.8
MAX_RETRIES = 3
```

### Private Members
```python
class MyClass:
    def __init__(self):
        self._private_var = None      # Private variable
        self.__very_private = None    # Name mangling
    
    def public_method(self):
        pass
    
    def _private_method(self):
        pass
```

## Common Patterns

### Builder Pattern
```python
class PipelineBuilder:
    def __init__(self, data: dict):
        self.data = data.copy()
    
    def normalize(self) -> 'PipelineBuilder':
        # Process
        return self
    
    def validate(self) -> 'PipelineBuilder':
        # Process
        return self
    
    def build(self) -> dict:
        return self.data
```

### Factory Pattern
```python
class HandlerFactory:
    @staticmethod
    def create(handler_type: str) -> BaseHandler:
        if handler_type == "fast":
            return FastHandler()
        return DefaultHandler()
```

### Lazy Initialization
```python
class ServiceRunner:
    def __init__(self):
        self._engine = None
    
    def _get_engine(self):
        if self._engine is None:
            self._engine = HandlerFactory.create("auto")
        return self._engine
```

### Caching
```python
class ResourceLoader:
    def __init__(self):
        self._cache = {}
    
    def load(self, key: str):
        if key in self._cache:
            return self._cache[key]
        value = self._fetch(key)
        self._cache[key] = value
        return value
```

## Testing Guidelines

### Unit Tests
```python
def test_pipeline_builder_normalizes_data():
    """Test that normalize processes data as expected."""
    raw = {"value": 10}
    result = PipelineBuilder(raw).normalize().build()
    assert result is not None
```

### Property-Based Tests
```python
from hypothesis import given
import hypothesis.strategies as st

@given(st.integers(min_value=1, max_value=10))
def test_scale_factor_valid(factor):
    """Test scaling works with any valid factor."""
    assert factor >= 1
```

## Common Mistakes to Avoid

### ❌ Don't
- Over-engineering with unnecessary state/layers
- Mocking the subject under test (mocking external dependencies like HTTP client, DB, hardware is OK)
- Using bare `except: pass` without logging or handling specific exceptions

### ✅ Do
- Keep classes simple and focused
- Write real tests against the subject under test
- Use specific error handling and clear logging

## Code Review Checklist

- [ ] Single responsibility principle followed
- [ ] Type hints present for all functions
- [ ] Docstrings for public methods
- [ ] No dead code
- [ ] Error handling appropriate
- [ ] Logging messages clear
- [ ] File size < 500 lines
- [ ] No mocking the subject under test (mock external deps is OK)
- [ ] Imports organized correctly
- [ ] Naming conventions followed
- [ ] **Thread-safe UI updates (signals/events for cross-thread communication)**
