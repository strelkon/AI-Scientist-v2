---
name: ai-scientist-experiment
description: Run AI Scientist v2 best-first tree search experiments — requires API keys for parallel LLM code generation
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
argument-hint: <ideas-json-path> [--idea_idx N] [--load_code] [--add_dataset_ref]
---

# AI Scientist v2 — Experiment Execution

Run the best-first tree search (BFTS) experiments. **This stage requires API keys** because it performs parallel LLM-driven code generation and execution that cannot be replicated within a single Claude Code session.

## Prerequisites

1. **API keys** — Required for models in `bfts_config.yaml`:
   - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION_NAME` for Bedrock (default coding model)
   - `OPENAI_API_KEY` for feedback model (GPT-4o)
2. **GPU** — CUDA-capable GPU required for experiment execution
3. **Ideas file** — Pre-generated ideas JSON (from `/ai-scientist-ideate`)
4. **Dependencies** — `pip install -r requirements.txt` must have been run

## Verify Environment

Before running, check:

```bash
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}, GPUs: {torch.cuda.device_count()}')"
echo "OPENAI_API_KEY set: ${OPENAI_API_KEY:+yes}"
echo "AWS_ACCESS_KEY_ID set: ${AWS_ACCESS_KEY_ID:+yes}"
```

If keys are missing, inform the user which are needed and stop.

## Arguments

- **Positional arg 1** (`$ARGUMENTS` first word): Path to ideas JSON file (required)
- `--idea_idx N`: Which idea to run (default: 0)
- `--load_code`: Also load the companion `.py` starter code file
- `--add_dataset_ref`: Include HuggingFace dataset reference code
- `--model_agg_plots MODEL`: Model for plot aggregation (default: o3-mini-2025-01-31)

## Execution

```bash
cd /home/user/AI-Scientist-v2
export AI_SCIENTIST_ROOT=/home/user/AI-Scientist-v2

python launch_scientist_bfts.py \
  --load_ideas "$IDEAS_PATH" \
  --idea_idx ${IDEA_IDX:-0} \
  --skip_writeup \
  --skip_review \
  ${LOAD_CODE:+--load_code} \
  ${ADD_DATASET_REF:+--add_dataset_ref} \
  --model_agg_plots "${MODEL_AGG_PLOTS:-o3-mini-2025-01-31}"
```

**This is a long-running process** (typically 2-6 hours). Run it with `run_in_background: true` on the Bash tool.

## Configuration

The experiment is controlled by `bfts_config.yaml` in the project root. Key settings:
- `agent.num_workers`: Parallel exploration paths (default: 4)
- `agent.stages.stage{1-4}_max_iters`: Max iterations per stage (20, 12, 12, 18)
- `agent.code.model`: LLM for code generation (default: Claude 3.5 Sonnet via Bedrock)
- `agent.feedback.model`: LLM for evaluating results (default: GPT-4o)
- `exec.timeout`: Per-execution timeout in seconds (default: 3600)

## Output

Results in `experiments/{timestamp}_{idea_name}_attempt_{id}/`:
- `idea.md` / `idea.json` — The research idea
- `logs/` — Full experiment logs and tree visualization
- `logs/0-run/baseline_summary.json` — Baseline stage results
- `logs/0-run/research_summary.json` — Research stage results
- `logs/0-run/ablation_summary.json` — Ablation stage results
- `figures/` — Aggregated plots
- `token_tracker.json` — API cost tracking

After completion, tell the user they can generate a paper with:
```
/ai-scientist-writeup experiments/<dir>/
```
