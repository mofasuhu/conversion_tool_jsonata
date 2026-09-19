# JSON Question Conversion Tool

A Python batch pipeline for converting legacy educational-question JSON into a standardized question structure. Each file is pre-validated, transformed with type-specific JSONata rules, and post-validated before it is written to an output area.

## What it does

- Discovers JSON files recursively from an input file or directory.
- Supports the question types listed in `SCRIPTS/config.py`, including MCQ, MRQ, GMRQ, FRQ, ordering, gap text, string, opinion, matching, counting, puzzle, and input-box variants.
- Runs pre-conversion validation, JSONata conversion, and post-conversion validation.
- Records blocking errors separately from non-blocking warnings.
- Writes converted files, failed inputs, post-validation outputs, Excel reports, and text logs to separate output directories.
- Can filter by question type, run in dry-run mode, and use multiple worker processes.

## Repository layout

- `main.py` — command-line entry point for the general conversion pipeline.
- `workflow_main.py` — Step4-to-Step5 workflow for datasets arranged in the pipeline folder format.
- `SCRIPTS/` — validators, converters, configuration, utilities, and type handling.
- `JSONATA_RULES/` — one JSONata transformation rule file per supported question type.
- `SIDE_TOOLS/` — supporting cleanup tools used by the workflow mode.
- `requirements.txt` — Python dependencies.
- `*_validations.txt`, `SUMMARY.txt`, and `Usage.txt` — validation and structure notes.

## Installation

Use Python 3.9 or newer, create an isolated environment, and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The runtime dependencies are `jsonata-python`, `tqdm`, `openpyxl`, `pandas`, and `beautifulsoup4` (plus the packages pinned in `requirements.txt`).

## General conversion

By default, the command reads `INPUT/` and writes under `OUTPUTS/`:

```bash
python3 -B main.py
```

Useful options:

```bash
python3 -B main.py --input path/to/input --output path/to/output
python3 -B main.py --types mcq,mrq,counting
python3 -B main.py --dry-run
python3 -B main.py --verbose --workers 8
```

The output root contains:

- `CONVERTED/` — successfully converted files.
- `PRE_CONVERSION_VALIDATION_FAILED/` — inputs rejected before transformation.
- `CONVERSION_FAILED/` — files that raised conversion errors.
- `POST_CONVERSION_VALIDATION_FAILED/` — transformed files that did not satisfy the new structure.
- `LOGS_REPORTS/` — Excel error/warning reports and text logs.

`--dry-run` validates and processes without writing output files. The default worker count is 50; choose a smaller value for constrained machines.

## Step4-to-Step5 workflow

Use `workflow_main.py` when the input is a Step4 pipeline directory. It copies the Step4 tree to Step5, keeps the relevant rows from `questions_updated_sheet.csv`, cleans HTML in JSON fields, normalizes supported TeX figure names, and converts eligible JSON files in place.

```bash
python3 -B workflow_main.py
python3 -B workflow_main.py --inputstep path/to/Step4_OUTPUT --outputstep path/to/Step5_OUTPUT -v
```

Only rows marked `IsSuccess=True` are processed. The workflow keeps paths relative by default, excludes log files while copying, and writes a timestamped Step5 log.

## Validation and rules

Validation distinguishes errors that block conversion from warnings that allow it to continue. Some warnings have automatic fixes in the converter (for example, trimming excess distractors for supported question types). Review the generated reports before publishing converted content.

Transformation behavior is defined by the `.jsonata` files in `JSONATA_RULES/`; validators and the Python orchestration code provide the surrounding checks and file handling.

## Data and security

Input, output, Step4/Step5 working directories, virtual environments, editor settings, and local environment files are ignored by `.gitignore`. Do not commit real student, employee, customer, or other private JSON/CSV data. Keep any credentials or service configuration outside the repository.

## License

See `LICENSE`.
