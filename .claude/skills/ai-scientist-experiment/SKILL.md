---
name: ai-scientist-experiment
description: Run AI Scientist v2 best-first tree search experiments on a research idea (without writeup or review)
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
argument-hint: <ideas-json-path> [--idea_idx N] [--load_code] [--add_dataset_ref]
---

# AI Scientist v2 — Experiment Execution Only

You are running just the experiment stage of AI Scientist v2. This performs best-first tree search (BFTS) to explore and validate a research idea through code generation, execution, and evaluation.

## Prerequisites

1. **API keys** — Required for the models specified in `bfts_config.yaml` (Anthropic Bedrock for coding, OpenAI for feedback by default).
2. **GPU** — CUDA-capable GPU required.
3. **Ideas file** — Pre-generated ideas JSON (from `/ai-scientist-ideate`).

## Usage

Parse `$ARGUMENTS`:

- **Positional arg 1** (`$0`): Path to ideas JSON file (required)
- `--idea_idx N`: Which idea to run (default: 0)
- `--load_code`: Also load the companion `.py` starter code file
- `--add_dataset_ref`: Include HuggingFace dataset reference code
- `--model_agg_plots MODEL`: Model for plot aggregation (default: o3-mini-2025-01-31)

## Execution

```bash
cd /home/user/AI-Scientist-v2

export AI_SCIENTIST_ROOT=/home/user/AI-Scientist-v2

python launch_scientist_bfts.py \
  --load_ideas "$0" \
  --skip_writeup \
  --skip_review \
  [other flags from $ARGUMENTS]
```

**This is a long-running process** (typically 2-6 hours depending on config). Run with `run_in_background: true`.

## Configuration

The experiment is controlled by `bfts_config.yaml`. Key settings:
- `agent.num_workers`: Parallel exploration paths (default: 4)
- `agent.stages.stage{1-4}_max_iters`: Iterations per stage
- `agent.code.model`: LLM for code generation
- `agent.feedback.model`: LLM for evaluating results
- `exec.timeout`: Per-execution timeout in seconds

The 4 experiment stages:
1. **Initial Implementation** — Baseline methods
2. **Baseline Tuning** — Optimization and tuning
3. **Creative Research** — Novel algorithmic improvements
4. **Ablation Studies** — Component analysis

## Output

Results in `experiments/{timestamp}_{idea_name}_attempt_{id}/`:
- `logs/` — Full experiment logs
- `logs/tree_plot.html` — Interactive tree visualization
- `figures/` — Aggregated plots
- `token_tracker.json` — API cost tracking

After completion, summarize key metrics and findings from the experiment logs.
