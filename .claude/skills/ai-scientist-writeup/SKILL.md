---
name: ai-scientist-writeup
description: Generate a scientific paper from completed AI Scientist v2 experiment results
disable-model-invocation: true
allowed-tools: Bash, Read, Glob, Grep
argument-hint: <experiment-dir> [--writeup-type normal|icbinb] [--model_writeup MODEL] [--num_cite_rounds N]
---

# AI Scientist v2 — Paper Writeup

You are running the paper generation stage of AI Scientist v2. This takes completed experiment results and produces a full scientific manuscript in LaTeX/PDF format.

## Prerequisites

1. **Completed experiments** — An experiment directory with results (from `/ai-scientist-experiment` or `/ai-scientist`).
2. **API keys** — `OPENAI_API_KEY` for the writeup and citation models.
3. **LaTeX** — `pdflatex` and `bibtex` must be installed.
4. **Semantic Scholar** — `S2_API_KEY` recommended for citation gathering.

## Usage

Parse `$ARGUMENTS`:

- **Positional arg 1** (`$0`): Path to experiment directory (e.g., `experiments/2025-01-01_my_idea_attempt_0`)
- `--writeup-type TYPE`: `normal` (8-page ICML) or `icbinb` (4-page workshop). Default: `icbinb`
- `--model_writeup MODEL`: Model for paper writing (default: o1-preview-2024-09-12)
- `--model_writeup_small MODEL`: Secondary model (default: gpt-4o-2024-05-13)
- `--model_citation MODEL`: Model for citations (default: gpt-4o-2024-11-20)
- `--num_cite_rounds N`: Citation search rounds (default: 20)

## Execution

This stage involves two sub-steps run sequentially:

### Step 1: Gather Citations
```python
from ai_scientist.perform_icbinb_writeup import gather_citations
gather_citations(experiment_dir, num_cite_rounds=20, small_model="gpt-4o-2024-11-20")
```

### Step 2: Generate Paper
```python
# For ICBINB (4-page):
from ai_scientist.perform_icbinb_writeup import perform_writeup
perform_writeup(base_folder=experiment_dir, small_model=..., big_model=..., page_limit=4, citations_text=...)

# For normal (8-page):
from ai_scientist.perform_writeup import perform_writeup
perform_writeup(base_folder=experiment_dir, small_model=..., big_model=..., page_limit=8, citations_text=...)
```

Run via a Python script:

```bash
cd /home/user/AI-Scientist-v2
export AI_SCIENTIST_ROOT=/home/user/AI-Scientist-v2

python -c "
import sys, json
from ai_scientist.perform_icbinb_writeup import gather_citations, perform_writeup

experiment_dir = '$0'
citations_text = gather_citations(experiment_dir, num_cite_rounds=20, small_model='gpt-4o-2024-11-20')

for attempt in range(3):
    print(f'Writeup attempt {attempt+1}/3')
    success = perform_writeup(
        base_folder=experiment_dir,
        small_model='gpt-4o-2024-05-13',
        big_model='o1-preview-2024-09-12',
        page_limit=4,
        citations_text=citations_text,
    )
    if success:
        print('Writeup successful!')
        break
else:
    print('Writeup failed after 3 attempts')
"
```

This is a long-running process. Run with `run_in_background: true`.

## Output

- `{experiment_dir}/*.pdf` — The generated paper
- `{experiment_dir}/token_tracker.json` — Updated cost tracking

After completion, report the PDF path and any errors.
