# JSON File Structures: JSONL vs Bare List vs Dict

Great question 👍 — these three cases are the most common shapes you’ll see when opening a `.json` file, and they matter because Pandas (and you) need to know how to interpret them.  

---

## 1. JSONL (JSON Lines, aka NDJSON)
- **What it looks like** (each line is its own JSON object):
  ```
  {"id": 1, "name": "Alice"}
  {"id": 2, "name": "Bob"}
  {"id": 3, "name": "Charlie"}
  ```
- **File structure**: plain text, each line is an independent JSON.  
- **When used**: large datasets, logs, streaming output (e.g. from APIs).  
- **How to read in Pandas**:
  ```python
  import pandas as pd
  df = pd.read_json("file.jsonl", lines=True)
  ```
- ✅ Easy for appending row by row; you can read it line-by-line without loading the whole file into memory.

---

## 2. Bare list (top-level JSON array)
- **What it looks like**:
  ```json
  [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    {"id": 3, "name": "Charlie"}
  ]
  ```
- **File structure**: the whole file is one JSON value → a list of dicts.  
- **When used**: smaller datasets, APIs returning “all results” at once.  
- **How to read in Pandas**:
  ```python
  import pandas as pd
  df = pd.read_json("file.json")  # Pandas infers list-of-dicts
  # or
  import json
  with open("file.json") as f:
      data = json.load(f)
  df = pd.json_normalize(data)
  ```

---

## 3. Dict (top-level JSON object)
- **What it looks like**:
  ```json
  {
    "project": "demo",
    "version": "1.0",
    "records": [
      {"id": 1, "name": "Alice"},
      {"id": 2, "name": "Bob"}
    ]
  }
  ```
- **File structure**: single JSON object with keys → values.  
- Often, some values are **metadata** (strings, numbers) and others are **lists** (the actual records).  
- **When used**: APIs that return a container object with both metadata and data.  
- **How to read in Pandas**:
  ```python
  import json
  with open("file.json") as f:
      data = json.load(f)

  # extract the list part
  df = pd.json_normalize(data, record_path="records", meta=["project", "version"])
  ```

---

## 🔑 Summary

| Case | Top-level JSON type | Example | Pandas strategy |
|------|---------------------|---------|-----------------|
| **JSONL** | Not a single JSON, but many JSON objects separated by newlines | `{"a":1}\n{"a":2}` | `pd.read_json(..., lines=True)` |
| **Bare list** | JSON array (`[...]`) | `[{"a":1}, {"a":2}]` | `pd.read_json()` or `json_normalize()` |
| **Dict** | JSON object (`{...}`) | `{"meta":1, "records":[...]}` | Use `record_path` + `meta` in `pd.json_normalize()` |
