# AI Scientist v2

Fully autonomous scientific research system that generates ideas, runs experiments via best-first tree search, writes papers, and performs peer review.

## Claude Skills

This project includes Claude Code skills for orchestrating the pipeline:

| Skill | Command | Description |
|-------|---------|-------------|
| `/ai-scientist-ideate` | `/ai-scientist-ideate ai_scientist/ideas/topic.md` | Generate research ideas from a topic description |
| `/ai-scientist-experiment` | `/ai-scientist-experiment ai_scientist/ideas/topic.json` | Run BFTS experiments only |
| `/ai-scientist-writeup` | `/ai-scientist-writeup experiments/dir/` | Generate paper from completed experiments |
| `/ai-scientist-review` | `/ai-scientist-review experiments/dir/` | Peer review a generated paper |
| `/ai-scientist` | `/ai-scientist ai_scientist/ideas/topic.json` | Run the full pipeline end-to-end |

### Typical Workflow

```
# 1. Generate ideas from a research topic
/ai-scientist-ideate ai_scientist/ideas/my_topic.md --max-num-generations 10

# 2. Run full pipeline on idea #0
/ai-scientist ai_scientist/ideas/my_topic.json --idea_idx 0 --load_code --add_dataset_ref

# Or run stages independently:
/ai-scientist-experiment ai_scientist/ideas/my_topic.json --idea_idx 0
/ai-scientist-writeup experiments/2025-01-01_my_idea_attempt_0/
/ai-scientist-review experiments/2025-01-01_my_idea_attempt_0/
```

## Required Environment Variables

- `OPENAI_API_KEY` — For OpenAI models (writeup, review, feedback)
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION_NAME` — For Bedrock (experiment coding)
- `S2_API_KEY` — For Semantic Scholar literature search (optional)

## Key Files

- `launch_scientist_bfts.py` — Main entry point
- `bfts_config.yaml` — Experiment configuration
- `ai_scientist/ideas/` — Research topic files and generated ideas
- `experiments/` — Output directory for results
