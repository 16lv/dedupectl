# sha256-file-deduplicator

Two-stage SHA-256 duplicate file scanner and CSV-to-JSON ingestion helper written in **Python 3.12**, backed by a SQLite execution audit log.

## Features

- **Size-bucketed hashing**: `scan_duplicates()` groups files by byte length first and computes chunked SHA-256 digests (`65536`-byte reads) only when two or more non-symlink files share a size, calculating reclaimable bytes per group.
- **CSV validation & export**: `sanitize_and_export_csv()` checks expected CSV headers before streaming rows, strips whitespace, filters incomplete records, deduplicates case-insensitively on `dedupe_key`, and writes deterministic JSON.
- **Audit persistence**: `AuditLedger` records run counts and structured JSON metadata in SQLite.

## Structure

```text
sha256-file-deduplicator/
├── src/automation_toolkit/
│   └── workflows.py
├── tests/
│   └── test_workflows.py
├── pyproject.toml
└── requirements.txt
```

## Development

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```
