# Normalize Dictionary

## Problem
Implement a function that normalizes a nested dictionary by converting
all string keys to lowercase and all string values to lowercase.

## Input
- `data`: a dictionary (may be nested)

## Output
- A new dictionary with normalized structure

## Example
```python
normalize({"Name": "John", "AGE": 30})
# Returns: {"name": "john", "age": 30}

normalize({"User": {"Name": "Alice"}})
# Returns: {"user": {"name": "alice"}}
```

## Notes
- Requirements become stricter in later phases
- Do not modify the input dictionary
- Non-string values should remain unchanged
