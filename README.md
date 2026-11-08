# DedupeCtl

![CI Status](https://github.com/16lv/dedupectl/actions/workflows/ci.yml/badge.svg)

**DedupeCtl** is a deterministic file deduplication, CSV-to-JSON data sanitisation, and SQLite audit-logging toolkit built with **Python 3.12**.

## Why I Built It

Hashing every file in a large directory tree with SHA-256 wastes disk I/O when most files don't even share the same byte size. Similarly, ad-hoc CSV cleanup scripts often drop rows silently without recording how many records were rejected. I built DedupeCtl to combine size-filtered SHA-256 duplicate scanning and deterministic CSV sanitisation with an append-only SQLite execution log (`AuditLedger`).

## Key Features

1. **Two-Stage Content-Addressable Deduplication (`scan_duplicates`)**:
   - **Pass 1 (Metadata Grouping)**: Walks the directory tree with `rglob("*")` and groups files by `stat().st_size`. Any file with a unique byte size cannot be a duplicate and skips hashing completely.
   - **Pass 2 (Chunked SHA-256)**: Streams 64KB chunks (`compute_sha256`) only for files whose byte sizes collide, grouping identical digests and calculating exact `reclaimable_bytes`.
2. **CSV Sanitisation & Deduplication (`sanitize_and_export_csv`)**: Trims key/value whitespace, rejects rows missing required columns, removes case-insensitive duplicates on a chosen primary key column (`dedupe_key`), and writes sorted JSON output.
3. **SQLite Run Audit Ledger (`AuditLedger`)**: Records every automation run in `workflow_runs` with UTC timestamps, `processed_items`, `affected_items`, and JSON details.

## Usage Example

```python
from pathlib import Path
from src.automation_toolkit.workflows import AuditLedger, scan_duplicates

with AuditLedger("audit.db") as ledger:
    report = scan_duplicates(Path("./backups"), ledger=ledger)
    print(f"Scanned {report.scanned_files} files | Reclaimable: {report.total_reclaimable_bytes} bytes")
    for group in report.duplicate_groups:
        print(f"Canonical: {group.canonical_path} -> Duplicates: {group.duplicate_paths}")
```

## Running Tests & Linter

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```

## Known Limitations & Future Improvements

- For multi-gigabyte files that happen to share identical byte lengths, hashing the first and last 4KB before computing the full-file SHA-256 digest would further reduce disk reads.
