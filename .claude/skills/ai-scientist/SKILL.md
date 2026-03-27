---
name: ai-scientist
description: Run the full AI Scientist v2 pipeline — autonomous scientific research from idea generation through experiments, paper writing, and review
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
argument-hint: <ideas-json-path> [--idea_idx N] [--writeup-type normal|icbinb] [--skip_writeup] [--skip_review]
---

# AI Scientist v2 — Full Pipeline

You are orchestrating the AI Scientist v2 autonomous research system. This skill runs the complete pipeline: experiments → plot aggregation → citation gathering → paper writeup → peer review.

## Prerequisites

Before running, verify the environment:

1. **API keys** — At least one must be set:
   - `OPENAI_API_KEY` for OpenAI models
   - `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` + `AWS_REGION_NAME` for Bedrock
   - `S2_API_KEY` for Semantic Scholar (optional but recommended)

2. **System dependencies** — Verify these are installed:
   - `pdflatex` and `bibtex` (LaTeX compilation)
   - `chktex` (LaTeX checking)
   - Python with CUDA-enabled PyTorch
   - Project Python dependencies (`pip install -r requirements.txt`)

3. **Ideas file** — A JSON file with pregenerated research ideas (from `/ai-scientist-ideate`)

## Usage

The user provides arguments via `$ARGUMENTS`. Parse them as follows:

- **Positional arg 1** (`$0`): Path to ideas JSON file (required)
- `--idea_idx N`: Which idea to run (default: 0)
- `--writeup-type normal|icbinb`: Paper format — 8-page (normal) or 4-page ICBINB (default: icbinb)
- `--skip_writeup`: Skip paper generation
- `--skip_review`: Skip peer review
- `--model_writeup MODEL`: Model for writeup (default: o1-preview-2024-09-12)
- `--model_citation MODEL`: Model for citations (default: gpt-4o-2024-11-20)
- `--model_review MODEL`: Model for review (default: gpt-4o-2024-11-20)
- `--model_agg_plots MODEL`: Model for plot aggregation (default: o3-mini-2025-01-31)
- `--num_cite_rounds N`: Citation rounds (default: 20)
- `--load_code`: Load companion .py file for the ideas
- `--add_dataset_ref`: Add HuggingFace dataset reference

## Execution

Run the pipeline from the project root directory:

```bash
cd /home/user/AI-Scientist-v2

export AI_SCIENTIST_ROOT=/home/user/AI-Scientist-v2

python launch_scientist_bfts.py \
  --load_ideas "$0" \
  [pass through all other flags from $ARGUMENTS]
```

**Important notes:**
- This is a **long-running process** (typically several hours). Run it in the background using `run_in_background: true` on the Bash tool.
- Monitor progress by tailing log files in the `experiments/` directory.
- The process uses significant GPU resources.

## Output

Results are saved to `experiments/{timestamp}_{idea_name}_attempt_{id}/` containing:
- `idea.md` / `idea.json` — The research idea
- `logs/` — Experiment logs and tree visualization
- `figures/` — Publication-ready plots
- `*.pdf` — Generated paper (if writeup enabled)
- `review_text.txt` — LLM review (if review enabled)
- `review_img_cap_ref.json` — Figure review (if review enabled)
- `token_tracker.json` — API cost tracking

After the run completes, report the output directory and key results to the user.

## Error Handling

- If the process fails, check logs in `experiments/*/logs/` for details.
- Common issues: missing API keys, CUDA out of memory, LaTeX compilation failures.
- The writeup step retries up to 3 times automatically.
