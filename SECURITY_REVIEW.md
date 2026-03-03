# Security Review Notes

Date: 2026-03-03
Scope: Repository-wide static review of executable/script source files (`*.py`, `*.sh`, `*.js`, `*.ts`) in this repository, with focus on data download and command execution paths.

## Coverage notes

- A full file inventory shows this repo is primarily Markdown content and contains 26 Python scripts plus no shell/js/ts execution code in other top-level domains.
- Script locations identified during scan:
  - `bio-research/.../scripts/*.py` and `bio-research/.../scripts/utils/*.py`
  - `data/skills/data-context-extractor/scripts/package_data_skill.py`
- No additional executable script files were found in other subdirectories at scan time.

## Findings

### 1) Insecure transport for FASTQ download URLs (High)
- `fetch_ena_fastq_urls()` rewrites ENA `fastq_ftp` fields into `http://...` URLs instead of `https://...`.
- This allows an active network attacker to tamper with downloaded sequencing files in transit.
- Evidence: `urls = [f"http://{url}" ...]` in `bio-research/skills/nextflow-development/scripts/utils/ncbi_utils.py`.

**Recommendation:**
- Prefer HTTPS URLs when available (ENA supports HTTPS).
- Reject non-HTTPS unless user explicitly opts in.

### 2) No integrity verification of downloaded artifacts (High)
- Downloaded files are written directly to disk without hash/signature verification.
- This affects both ENA file downloads and iGenomes downloads.
- Evidence: direct streaming write in `download_file()` and raw `aws s3 cp` in genome downloader.

**Recommendation:**
- Verify checksums (e.g., SHA-256 from trusted metadata/manifests) after download.
- Fail closed on checksum mismatch.

### 3) Potential resource exhaustion / disk-fill DoS during downloads (Medium)
- `download_file()` streams all bytes until EOF with no maximum size guard.
- A malicious or unexpected endpoint can serve arbitrarily large content and exhaust local disk.
- Evidence: loop writing all chunks with no cap.

**Recommendation:**
- Add configurable max-bytes limits and enforce `Content-Length` sanity checks.
- Abort when exceeding budget.

### 4) PATH-based binary execution risk for `aws` helper (Low)
- The script locates and executes `aws` from PATH (`which aws`, then `aws ...`).
- In compromised environments (or privileged execution), PATH hijacking can execute a malicious binary.
- Evidence: subprocess calls in `manage_genomes.py`.

**Recommendation:**
- Resolve absolute path with `shutil.which("aws")`, validate it, and execute that resolved path.
- Optionally restrict to expected install locations.

### 5) Unbounded parallelism parameter can degrade host stability (Low)
- `ThreadPoolExecutor(max_workers=args.parallel)` accepts user input with no upper bound.
- Very large values can cause local CPU/memory/network contention (self-DoS).

**Recommendation:**
- Clamp `--parallel` to a safe range (e.g., 1–16) or derive from CPU count with a hard cap.

## Positive observations
- No obvious use of `eval`, `exec`, or `shell=True` in reviewed paths.
- YAML parsing uses `yaml.safe_load` when PyYAML is available.

## Suggested priority order
1. Enforce HTTPS + checksum verification for downloaded data.
2. Add file-size limits and download budget controls.
3. Harden binary lookup and parallelism caps.
