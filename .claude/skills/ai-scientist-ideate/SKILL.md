---
name: ai-scientist-ideate
description: Generate research ideas for AI Scientist v2 using LLM-powered ideation with Semantic Scholar integration
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
argument-hint: <workshop-topic-file.md> [--model MODEL] [--max-num-generations N] [--num-reflections N]
---

# AI Scientist v2 — Idea Generation

You are running the AI Scientist v2 ideation stage. This generates structured research ideas from a workshop topic description.

## Prerequisites

1. **API key** — `OPENAI_API_KEY` or equivalent must be set for the chosen model.
2. **Semantic Scholar API key** — `S2_API_KEY` is optional but recommended for better literature search.
3. **Workshop topic file** — A markdown file describing the research area. Must be placed in `ai_scientist/ideas/`.

## Workshop Topic File Format

If the user provides a topic but no file, create one at `ai_scientist/ideas/<topic_name>.md` with this structure:

```markdown
# Title: <Workshop/Topic Title>

## Keywords
keyword1, keyword2, keyword3

## TL;DR
<One-sentence summary of the research direction>

## Abstract
<Detailed description of the research area, goals, and scope (~250 words)>
```

## Usage

Parse `$ARGUMENTS`:

- **Positional arg 1** (`$0`): Path to workshop topic markdown file (required)
- `--model MODEL`: LLM to use (default: gpt-4o-2024-05-13). Options: !`cd /home/user/AI-Scientist-v2 && python -c "from ai_scientist.llm import AVAILABLE_LLMS; print(', '.join(AVAILABLE_LLMS))" 2>/dev/null || echo "gpt-4o-2024-05-13, claude-3-5-sonnet-20241022, o1-preview-2024-09-12"`
- `--max-num-generations N`: Number of ideas to generate (default: 20)
- `--num-reflections N`: Refinement rounds per idea (default: 5)

## Execution

```bash
cd /home/user/AI-Scientist-v2

python ai_scientist/perform_ideation_temp_free.py \
  --workshop-file "$0" \
  --model "${MODEL:-gpt-4o-2024-05-13}" \
  --max-num-generations "${MAX_GENS:-20}" \
  --num-reflections "${NUM_REFLECTIONS:-5}"
```

This is a moderately long-running process. Run it with `run_in_background: true`.

## Output

The script produces a JSON file alongside the input markdown file with the same base name:
- `ai_scientist/ideas/<topic_name>.json`

Each idea in the JSON contains:
- **Name**: Short identifier
- **Title**: Paper title
- **Short Hypothesis**: Core research question
- **Related Work**: Literature context
- **Abstract**: ~250 word summary
- **Experiments**: Detailed experiment plans
- **Risk Factors and Limitations**: Known risks

After completion, report how many ideas were generated and list their titles.

The user can then run `/ai-scientist <path-to-ideas.json>` to execute experiments on a selected idea.
