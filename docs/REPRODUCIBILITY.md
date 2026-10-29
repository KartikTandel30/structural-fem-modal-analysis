# Reproducibility and execution

This guide describes how to attempt the existing Python modal-analysis script without changing its mathematical formulation. It does **not** claim the frequencies have been independently verified.

## Files required together

The solver expects these exact files in one directory:

```text
Final_code.py
node_details.csv
element_connectivity.csv
GeometryDetails.csv
reinforced_concrete_properties.csv
```

The filenames `GeometryDetails.csv` and `node_details.csv` have the exact capitalization shown. The script accesses CSV rows using positional/label assumptions; preserve the provided 0-based node numbering unless the input-reading logic is adjusted.

## Dependencies

Install Python 3 with NumPy, pandas, SciPy, and Matplotlib. `requirements.txt` lists **package names without versions**, since the original environment versions were not saved.

### Windows PowerShell

From a local clone of the repository:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

If PowerShell blocks activation, use `.venv\Scripts\python.exe -m pip install -r requirements.txt` and the same executable to run `Final_code.py` instead of changing execution policies.

### Git Bash / Linux

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS; on Git Bash, use source .venv/Scripts/activate
python -m pip install -r requirements.txt
```

## Required path adjustment

The current `Final_code.py` reads from a hard-coded location:

```python
directory = "C:/Users/karti/OneDrive/Desktop/RJ&SC"
```

For a portable local clone, add the following import near the top of the script:

```python
from pathlib import Path
```

Then replace the `directory = ...` line with:

```python
directory = str(Path(__file__).resolve().parent)
```

This is a **manual local edit**; the professional documentation package intentionally does not modify `Final_code.py`. Alternatively, change the original variable to the absolute directory on your Windows machine.

## Run

```bash
python Final_code.py
```

The current script attempts to print the first three angular frequencies **in rad/s** and display three interactive Matplotlib mode-shape plots. It does not provide saved, benchmarked frequency tables or a headless test mode.

Before claiming engineering validation, implement and execute the checks in [VERIFICATION.md](VERIFICATION.md).

## Report a reproducible run

Record the Git commit SHA, Python/NumPy/pandas/SciPy/Matplotlib versions, input CSV checksums, support-node IDs, element thickness, material parameters, natural frequencies, mesh details and the verification residuals. A future release should also include automated tests and a corrected reference case.
