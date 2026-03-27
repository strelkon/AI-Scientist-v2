---
name: ai-scientist-review
description: Perform AI-powered peer review of a generated scientific paper (text + figure/caption review)
disable-model-invocation: true
allowed-tools: Bash, Read, Glob, Grep
argument-hint: <experiment-dir> [--model_review MODEL]
---

# AI Scientist v2 — Paper Review

You are running the peer review stage of AI Scientist v2. This performs both text-based and vision-based review of a generated scientific paper.

## Prerequisites

1. **Generated paper** — A PDF file in the experiment directory (from `/ai-scientist-writeup` or `/ai-scientist`).
2. **API key** — `OPENAI_API_KEY` for the review model (default: gpt-4o).

## Usage

Parse `$ARGUMENTS`:

- **Positional arg 1** (`$0`): Path to experiment directory containing the PDF
- `--model_review MODEL`: Model for review (default: gpt-4o-2024-11-20)

## Execution

```bash
cd /home/user/AI-Scientist-v2
export AI_SCIENTIST_ROOT=/home/user/AI-Scientist-v2

python -c "
import os, json, re
from ai_scientist.llm import create_client
from ai_scientist.perform_llm_review import perform_review, load_paper
from ai_scientist.perform_vlm_review import perform_imgs_cap_ref_review

experiment_dir = '$0'
model_name = '${MODEL_REVIEW:-gpt-4o-2024-11-20}'

# Find the PDF
pdf_files = [f for f in os.listdir(experiment_dir) if f.endswith('.pdf')]
reflection_pdfs = [f for f in pdf_files if 'reflection' in f]
if reflection_pdfs:
    final_pdfs = [f for f in reflection_pdfs if 'final' in f.lower()]
    if final_pdfs:
        pdf_path = os.path.join(experiment_dir, final_pdfs[0])
    else:
        nums = []
        for f in reflection_pdfs:
            m = re.search(r'reflection[_.]?(\d+)', f)
            if m: nums.append((int(m.group(1)), f))
        if nums:
            pdf_path = os.path.join(experiment_dir, max(nums, key=lambda x: x[0])[1])
        else:
            pdf_path = os.path.join(experiment_dir, reflection_pdfs[0])
elif pdf_files:
    pdf_path = os.path.join(experiment_dir, pdf_files[0])
else:
    print('No PDF found!')
    exit(1)

print(f'Reviewing: {pdf_path}')

# Text review
paper_content = load_paper(pdf_path)
client, client_model = create_client(model_name)
review_text = perform_review(paper_content, client_model, client)

with open(os.path.join(experiment_dir, 'review_text.txt'), 'w') as f:
    f.write(json.dumps(review_text, indent=4))

# Figure review
review_img = perform_imgs_cap_ref_review(client, client_model, pdf_path)
with open(os.path.join(experiment_dir, 'review_img_cap_ref.json'), 'w') as f:
    json.dump(review_img, f, indent=4)

print('Review complete!')
"
```

## Output

- `review_text.txt` — Full text review with scores and feedback
- `review_img_cap_ref.json` — Figure, caption, and reference review

After completion, read and summarize the review results for the user, highlighting:
- Overall score and recommendation
- Key strengths identified
- Main weaknesses and suggestions
- Figure quality assessment
