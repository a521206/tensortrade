# Agent Guide - TensorTrade Project

## Environment

- **OS**: Windows (win32)
- **Python**: 3.12 via pyenv (`C:\Users\agraw\.pyenv\pyenv-win\versions\3.12.0\python.exe`)
- **Preferred venv**: `.venv` (always use this, not system Python)
- **Shell**: PowerShell 7+
- **Working directory**: `C:\Users\agraw\GIT\tensortrade`

## DOs

### Always use `.venv`
```powershell
# Install dependencies
.venv\Scripts\python.exe -m pip install -e .

# Run scripts
.venv\Scripts\python.exe examples/training/train_simple.py

# Run tests
.venv\Scripts\python.exe -m pytest tests/ -v
```

### Use PowerShell-native commands
```powershell
# List files
Get-ChildItem

# Read file
Get-Content file.txt

# Check git status
git status

# Run python
.venv\Scripts\python.exe script.py
```

### Use absolute paths or relative paths with backslashes
```powershell
# Good
.venv\Scripts\python.exe examples/training\train_simple.py
.venv\Scripts\python.exe .\examples\training\train_simple.py

# Avoid (Unix-style paths may fail)
.venv/Scripts/python examples/training/train_simple.py
```

### Use proper quoting for paths with spaces
```powershell
& "C:\path with spaces\script.py" "arg with spaces"
```

## DON'Ts

### Don't use system Python
```powershell
# WRONG - causes TensorFlow DLL errors
python script.py

# CORRECT
.venv\Scripts\python.exe script.py
```

### Don't use Unix/Linux commands
```powershell
# WRONG
ls -la
cat file.txt
rm file.txt
chmod +x script.sh
source venv/bin/activate

# CORRECT
Get-ChildItem
Get-Content file.txt
Remove-Item file.txt
.venv\Scripts\activate
```

### Don't use `cd` inside commands
```powershell
# WRONG
Set-Location .\examples && .venv\Scripts\python.exe train_simple.py

# CORRECT - use -Workdir parameter
.venv\Scripts\python.exe examples/training/train_simple.py
```

### Don't use Unix path separators
```powershell
# WRONG
examples/training/train_simple.py

# CORRECT
examples\training\train_simple.py
examples/training/train_simple.py  # PowerShell accepts forward slashes too
```

### Don't use `tail`
```powershell
# WRONG
pip install -e . 2>&1 | tail -20

# CORRECT
pip install -e .
```

## Project-Specific Notes

### GPU Training on Windows
- TensorFlow >= 2.11 dropped native Windows GPU support; use PyTorch instead
- Verify CUDA with: `.venv\Scripts\python.exe -c "import torch; print(torch.cuda.is_available())"`
- PyTorch Conv1d expects input channels = n_features (last dim), not window_size
- Input tensor for Conv1d should be shaped `(batch, features, time)` — use `x.permute(0, 2, 1)`

### yfinance
- `interval='1h'` produces index name `Datetime`; `period` without interval produces `Date`
- Always handle both: `df.rename(columns={'Date': 'date', 'Datetime': 'date', ...})`

### TensorTrade Env Access
- Portfolio is at `env.action_scheme.portfolio`, not `env.portfolio`
- `initial_net_worth` is set lazily on first `on_next()` call — call `env.step()` once or use first recorded net worth
- After training, env feed is exhausted — create a **separate env** for validation/test with its own data feed

### Notebook Editing
- Use Python `json` module for reliable edits: `json.load()` → modify → `json.dump()`
- The `Edit` tool often fails on notebooks due to escaped newlines/quotes
- Notebook `"source"` arrays contain strings with trailing `\n`; preserve exact formatting
- When edit fails, fallback to: `python -c "import json; ..."` with `json.dump(nb, f, indent=1)`

### Lint / Ruff
- Run: `ruff check examples\train_and_evaluate_stocks.ipynb`
- Common fixes:
  - Remove unused variables (`X = data.copy()`, `y = ...`, `reward` when not used)
  - Use `_` for intentionally unused unpacked values: `next_state, _, done = ...`
  - Convert numpy booleans explicitly: `np.array(batch.done, dtype=np.float32)`
- `--fix` handles safe fixes; unsafe fixes need `--unsafe-fixes`

### File Operations
- Use `Read`, `Write`, `Edit` tools instead of shell commands
- Use `bash` only for git, pip, pytest, and other non-file operations
- Always create required directories before saving: `New-Item -ItemType Directory -Path agents -Force`

## Quick Reference

| Task | Command |
|------|---------|
| Install package | `.venv\Scripts\python.exe -m pip install -e .` |
| Run example | `.venv\Scripts\python.exe examples/training/train_simple.py` |
| Run tests | `.venv\Scripts\python.exe -m pytest tests/ -v` |
| Check Python | `.venv\Scripts\python.exe -c "import sys; print(sys.executable)"` |
| List packages | `.venv\Scripts\python.exe -m pip list` |
