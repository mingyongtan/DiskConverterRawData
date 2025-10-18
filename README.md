# DiskConverterRawData — Process I/O TXT → Excel/CSV (GUI + CLI)

Convert process I/O text dumps into a clean, analyzable table, exported as Excel (`.xlsx`) or CSV. Runs with a simple GUI when Tkinter is available, and falls back to a robust CLI when it isn't — no pandas required.

### Features
- Header-aware parsing (header lines may appear anywhere; they are removed)
- Handles UTF‑8 BOM and stray quotes automatically
- Accepts values like `893.8 KB`, `40 244 K`, `16.0 MB`, or raw integers
- Adds calculated columns:
  - `Total Bytes` — per-row sum of I/O bytes
  - `Cumulative Total Bytes` — grand total repeated on every row
  - `Total Usage of Memory in 100% (B / C *100)` — per-row fraction (stored 0..1; formatted as percentage in Excel)
- Excel output uses an actual Excel Table with auto column widths and frozen header; also adds a bottom total row for convenience
- If `openpyxl` is not installed, saves CSV instead

### Requirements
- Python 3.8+
- Optional: Tkinter for GUI
  - Linux example: `sudo apt-get install python3-tk` (package name may vary)
- Optional: `openpyxl` for writing `.xlsx`
  - Install: `pip install openpyxl`

### Quick start

GUI (if Tk is available):

```bash
python con_process_txt_to_xlsx
```

CLI (works everywhere):

```bash
# From a file
python con_process_txt_to_xlsx --cli -i input.txt -o out.xlsx

# Using STDIN (only if your environment supports piping)
cat input.txt | python con_process_txt_to_xlsx --cli --stdin -o out.xlsx
```

Output format is chosen by extension:
- `.xlsx` → requires `openpyxl` (will error with install hint if missing)
- `.csv` → no extra dependencies

If you omit the extension, the tool prefers `.xlsx` and falls back to `.csv` if `openpyxl` is missing.

### Input format
The parser expects five base columns, typically tab-delimited (spaces are tolerated for header detection):

1. Process
2. PID
3. I/O Read Bytes
4. I/O Write Bytes
5. I/O Other Bytes

Notes:
- Header lines that match the schema are skipped anywhere in the file
- Quotes are stripped; UTF‑8 BOM (including mojibake `ï»¿`) is removed
- Byte values accept units `B`, `K/KB`, `M/MB`, `G/GB`; thousand separators with spaces or commas are handled
- Lines with fewer than 5 tokens are ignored

Example input (tabs shown as actual tab characters):

```text
Process	PID	I/O Read Bytes	I/O Write Bytes	I/O Other Bytes
explorer.exe	2548	123456	7890	456
svchost.exe	1052	1 024 K	512 K	64 K
SearchIndexer.exe	6484	40 244 K	33 980 K	0
```

### Output columns
- Process
- PID
- I/O Read Bytes
- I/O Write Bytes
- I/O Other Bytes
- Total Bytes
- Cumulative Total Bytes (grand total repeated)
- Total Usage of Memory in 100% (B / C *100)

In Excel:
- `Total Bytes` and `Cumulative Total Bytes` use integer number formatting with thousand separators
- Usage is formatted as `0.00%` (values are stored as 0..1)
- The data range is wrapped in an Excel Table with alternating row stripes; header row is frozen
- A bottom summary row is added with `SUM` of `Total Bytes` (outside the Table)

### Self-tests
Run built-in tests to validate parsing and Excel output:

```bash
python con_process_txt_to_xlsx --run-tests
```

### Build standalone executable (optional)
```bash
pip install pyinstaller
pyinstaller --noconsole --onefile con_process_txt_to_xlsx
```

### Troubleshooting
- “openpyxl is required to write .xlsx” → install with `pip install openpyxl`
- “Tkinter not available” → the app runs in CLI mode; install your OS’s Tk package for GUI
- Empty or zero rows parsed → verify the input has the expected five columns; ensure it’s tab-delimited or that headers match the schema

### Project layout
- `con_process_txt_to_xlsx` — main Python script providing both GUI and CLI

---

If you prefer a different project name or column labels, tell me and I can adjust the tool and README accordingly.
