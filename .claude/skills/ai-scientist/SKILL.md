---
name: ai-scientist
description: Run the full AI Scientist v2 pipeline — ideation through review. Only the experiment stage requires API keys.
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, WebSearch, WebFetch
argument-hint: <ideas-json-or-topic-md> [--idea_idx N] [--writeup-type normal|icbinb] [--skip_writeup] [--skip_review] [--skip_experiments]
---

# AI Scientist v2 — Full Pipeline

Orchestrate the complete AI Scientist v2 research pipeline. Stages that previously required API keys (ideation, writeup, review) now run natively in Claude Code. Only the experiment stage still requires external API keys.

## Arguments

Parse `$ARGUMENTS`:

- **Positional arg 1**: Path to either:
  - A `.json` file with pregenerated ideas (skips ideation)
  - A `.md` topic file (runs ideation first)
- `--idea_idx N`: Which idea to run experiments on (default: 0)
- `--writeup-type TYPE`: `icbinb` (4-page, default) or `normal` (8-page)
- `--load_code`: Load companion .py starter code
- `--add_dataset_ref`: Add HuggingFace dataset reference
- `--skip_experiments`: Skip experiment execution (use existing results)
- `--skip_writeup`: Skip paper generation
- `--skip_review`: Skip peer review
- `--max-num-generations N`: Ideas to generate if running ideation (default: 5)

## Pipeline Stages

### Stage 1: Ideation (Claude Code Native — No API Keys)

**Only runs if a `.md` file is provided instead of `.json`.**

Invoke the `/ai-scientist-ideate` skill with the topic file. This uses Claude Code's own capabilities and web search for literature review.

After ideation, the JSON file path becomes the input for subsequent stages.

### Stage 2: Experiments (Requires API Keys)

**Skipped if `--skip_experiments` is set.**

Invoke the `/ai-scientist-experiment` skill. This is the only stage that requires external API keys because it runs parallel LLM-driven code generation via the BFTS engine.

Before running, verify API keys are set:
```bash
echo "OPENAI_API_KEY: ${OPENAI_API_KEY:+set}"
echo "AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID:+set}"
```

If keys are missing, warn the user and ask whether to skip experiments or stop.

Wait for experiments to complete before proceeding.

### Stage 3: Paper Writeup (Claude Code Native — No API Keys)

**Skipped if `--skip_writeup` is set.**

Invoke the `/ai-scientist-writeup` skill. Claude Code will:
1. Read all experiment results and summaries
2. Search the web for relevant citations
3. Generate a complete LaTeX paper
4. Compile to PDF

### Stage 4: Peer Review (Claude Code Native — No API Keys)

**Skipped if `--skip_review` or `--skip_writeup` is set.**

Invoke the `/ai-scientist-review` skill. Claude Code will:
1. Read the generated PDF
2. Perform structured peer review (NeurIPS format)
3. Review figures and captions
4. Save review results

## Output Summary

After the pipeline completes, report:
1. Number of ideas generated (if ideation ran)
2. Experiment results directory
3. Paper PDF path (if writeup ran)
4. Review scores and decision (if review ran)
5. Total pipeline status

## Pipeline Flexibility

The pipeline is modular — users can run any combination:

```
# Full pipeline from topic
/ai-scientist my_topic.md

# Ideas exist, run everything
/ai-scientist ideas.json --idea_idx 0

# Experiments done, just write paper and review
/ai-scientist ideas.json --skip_experiments --idea_idx 0

# Just review an existing paper
/ai-scientist-review experiments/my_experiment/
```
