# AI Scientist v2

Fully autonomous scientific research system that generates ideas, runs experiments via best-first tree search, writes papers, and performs peer review.

## Claude Skills

This project includes Claude Code skills for orchestrating the pipeline. Most stages run **natively in Claude Code** with no API keys — only the experiment stage requires external API keys.

| Skill | Command | API Keys? | Description |
|-------|---------|-----------|-------------|
| `/ai-scientist-ideate` | `/ai-scientist-ideate ai_scientist/ideas/topic.md` | No | Generate research ideas using Claude + web search |
| `/ai-scientist-experiment` | `/ai-scientist-experiment ai_scientist/ideas/topic.json` | **Yes** | Run BFTS experiments (parallel LLM code generation) |
| `/ai-scientist-writeup` | `/ai-scientist-writeup experiments/dir/` | No | Generate LaTeX paper from experiment results |
| `/ai-scientist-review` | `/ai-scientist-review experiments/dir/` | No | Peer review a generated paper |
| `/ai-scientist` | `/ai-scientist ai_scientist/ideas/topic.json` | Partial | Run the full pipeline end-to-end |

### API Key Requirements

- **No keys needed** for: ideation, writeup, and review (Claude Code handles LLM tasks directly)
- **Keys needed only for experiments** (parallel BFTS tree search):
  - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION_NAME` — For Bedrock (experiment coding)
  - `OPENAI_API_KEY` — For feedback model during experiments
- **Optional**: `S2_API_KEY` — For Semantic Scholar (ideation uses web search as fallback)

### Typical Workflow

```
# 1. Generate ideas from a research topic (no API keys needed)
/ai-scientist-ideate ai_scientist/ideas/my_topic.md --max-num-generations 5

# 2. Run full pipeline on idea #0 (only experiments need API keys)
/ai-scientist ai_scientist/ideas/my_topic.json --idea_idx 0 --load_code --add_dataset_ref

# Or run stages independently:
/ai-scientist-experiment ai_scientist/ideas/my_topic.json --idea_idx 0
/ai-scientist-writeup experiments/2025-01-01_my_idea_attempt_0/
/ai-scientist-review experiments/2025-01-01_my_idea_attempt_0/

# Skip experiments if results already exist:
/ai-scientist ai_scientist/ideas/my_topic.json --skip_experiments --idea_idx 0
```

## Key Files

- `launch_scientist_bfts.py` — Main entry point (for experiment stage)
- `bfts_config.yaml` — Experiment configuration
- `ai_scientist/ideas/` — Research topic files and generated ideas
- `experiments/` — Output directory for results
- `.claude/skills/` — Claude Code skill definitions
