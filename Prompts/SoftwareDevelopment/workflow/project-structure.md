---
inclusion: auto
---

# Project Structure & Coding Standards

## Directory Organization

```
project/
├── main.py                          # Entry point only
├── requirements.txt                 # Dependencies
├── docs/                            # Documentation
│   ├── CURRENT_STATUS.md
│   ├── IMPLEMENTATION_SUMMARY.md
│   └── ...
├── tests/                           # Test files
│   ├── test_core_logic.py
│   └── __init__.py
├── src/
│   ├── __init__.py
│   ├── logic/                       # Pure business logic (no UI)
│   │   ├── __init__.py
│   │   ├── models.py                # Data models
│   │   ├── manager.py               # Serialization/deserialization
│   │   └── services/                # Modular business services
│   │       └── __init__.py
│   └── ui/                          # UI components (no business logic)
│       ├── __init__.py
│       └── main_window.py
└── workflow/                        # Project workflows & guidelines
    ├── 00-MASTER-RULES.md
    ├── conversation-rules.md
    ├── feature-development-workflow.md
    ├── change-workflow.md
    ├── coding-guidelines.md
    ├── project-structure.md         # This file
    └── reviewer-rule.md
```

## Key Principles

### 1. Separation of Concerns
- **`src/logic/`** - Pure business logic, no UI dependencies
- **`src/ui/`** - UI components only, minimal logic
- **`tests/`** - Test files separate from source
- **`docs/`** - Documentation separate from code

### 2. Modular Architecture
- Each component has **single responsibility**
- Keep logic modules ~300-400 lines each (max 500 lines guideline)
- Use **builder pattern** for complex object construction
- Use **factory pattern** for object creation

### 3. File Naming Conventions
- **snake_case** for file names: `task_manager.py`, `data_processor.py`
- **PascalCase** for class names: `TaskManager`, `DataProcessor`
- **UPPER_CASE** for constants: `DEFAULT_TIMEOUT`

### 4. Import Strategy
- **Relative imports** within packages: `from .sub_service import SubService`
- **Absolute imports** from parent: `from logic.models import DataModel`
- **Backward compatibility wrapper** at package level when refactoring modules

### 5. Code Organization Within Files

#### Logic Files (src/logic/)
```python
"""Module docstring."""

# Standard library imports
import os
import sys

# Third-party imports
import numpy as np

# Local imports
from .submodule import Class

# Constants
CONSTANT_NAME = value

# Helper functions (if any)
def helper_function():
    pass

# Main class
class MainClass:
    def __init__(self):
        pass
    
    def public_method(self):
        pass
    
    def _private_method(self):
        pass
```

#### UI Files (src/ui/)
```python
"""UI component docstring."""

# Import logic from src/logic
from logic.models import DataModel

class UIComponent:
    def __init__(self):
        self._init_ui()
    
    def _init_ui(self):
        """Initialize UI components."""
        pass
    
    def _on_button_clicked(self):
        """Handle button click."""
        pass
```

## Coding Standards

### Python Version
- Python 3.10+ (type hints, match statements)
- Use type hints for all function signatures

### Code Style
- Follow PEP 8
- Max line length: 120 characters
- Use docstrings for all public methods
- Use comments for complex logic

### Error Handling
- Use try/except for external API calls
- Log errors with context
- Return None or False for expected failures (not exceptions)

### Testing
- Unit tests in `tests/` directory
- Property-based tests for core logic
- Test file naming: `test_*.py`
- Use pytest framework

## When Adding New Features

1. **Determine if it's logic or UI**
   - Logic → `src/logic/`
   - UI → `src/ui/`

2. **If it's a new logic component**
   - Create in `src/logic/`
   - Keep file size < 500 lines
   - Use single responsibility principle

3. **If it's a new UI component**
   - Create in `src/ui/`
   - Import logic from `src/logic/`
   - Keep business logic minimal

4. **Update imports**
   - Add to `__init__.py` if creating new package
   - Update wrapper files for backward compatibility

## Backward Compatibility

- Maintain wrapper modules if splitting a large module into a package
- Never break existing imports without a migration plan

## Documentation

- Each module has docstring explaining purpose
- Each class has docstring with usage example
- Each public method has docstring with parameters and return value
- Complex logic has inline comments
