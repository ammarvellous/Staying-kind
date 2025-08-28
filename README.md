# Empathetic Code Reviewer
Transforming critical feedback into constructive growth.

<div align="center">

![python](https://img.shields.io/badge/Python-3.10%2B-blue)
![transformers](https://img.shields.io/badge/Transformers-✅-purple)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Hackathon--Ready-ff69b4)

</div>

## Overview
Empathetic Code Reviewer turns blunt code review comments into supportive, educational guidance that explains the “why” and shows a concrete “how,” producing a structured JSON report (and optional Markdown) ready to render in PRs, bots, or dashboards.

## Key Features
- Empathetic rephrasing that preserves technical accuracy while improving tone.
- “The Why” section grounded in principles (readability, performance, conventions, correctness).
- Concrete “Suggested Improvement” with minimal runnable code diffs.
- Deterministic JSON output for easy post-processing; optional Markdown report string.
- Severity-aware tone adjustments and optional lightweight rule-based fallback.
- Works in notebooks, CLI, and as an HTTP API; runs on CPU/GPU with 4-bit loading.

## Demo (Notebook)
- Colab/Jupyter demo shows JSON-in → JSON-out → rendered Markdown with display(Markdown(...)).

---

## Quickstart

### 1) Requirements
- Python 3.10+
- pip, git
- Optional GPU (CUDA) for faster generation

### 2) Install
git clone https://github.com/your-user/empathetic-code-reviewer.git
cd empathetic-code-reviewer
pip install -r requirements.txt

Minimal `requirements.txt`:
transformers>=4.42
accelerate>=0.31
bitsandbytes>=0.43
fastapi[standard]>=0.115
uvicorn>=0.30

### 3) Configure (optional)
Set environment variables to tailor runtime:
export MODEL_ID="Qwen/Qwen2.5-3B-Instruct" # or mistralai/Mistral-7B-Instruct-v0.3
export MAX_NEW_TOKENS=800
export TEMPERATURE=0.2

### 4) Run a quick test (Python)
from empathetic_reviewer import generate_report_json

code_snippet = """def get_active_users(users):
results = []
for u in users:
if u.is_active == True and u.profile_complete == True:
results.append(u)
return results
"""

review_comments = [
"This is inefficient. Don't loop twice conceptually.",
"Variable 'u' is a bad name.",
"Boolean comparison '== True' is redundant."
]

report = generate_report_json(code_snippet, review_comments)
print(report.keys()) # dict with 'task', 'metadata', 'summary', 'comments', optional 'markdown_report'
print(len(report["comments"]))

---

## Input and Output

### Input JSON
{
"code_snippet": "def get_active_users(users):\n results = []\n for u in users:\n if u.is_active == True and u.profile_complete == True:\n results.append(u)\n return results\n",
"review_comments": [
"This is inefficient. Don't loop twice conceptually.",
"Variable 'u' is a bad name.",
"Boolean comparison '== True' is redundant."
]
}

### Output JSON (contract)
{
"task": "empathetic_code_review",
"code_language": "python",
"metadata": {
"version": "1.0",
"generated_at": "2025-08-28T12:00:00Z",
"tone": "empathetic",
"style_guides": ["PEP 8"]
},
"summary": {
"overall_message": "Clear logic with opportunities to simplify conditions and naming for readability and small performance wins.",
"themes": ["readability", "performance", "conventions"],
"next_steps": ["Simplify boolean checks", "Adopt descriptive variable names", "Use list comprehension for filtering"]
},
"comments": [
{
"original_comment": "This is inefficient. Don't loop twice conceptually.",
"severity": "medium",
"topic": "performance",
"positive_rephrasing": "Good job capturing the logic! We can make it more efficient and concise.",
"why": {
"principle": "List comprehension for filtering",
"explanation": "When filtering items, a comprehension expresses intent more directly and can be faster than appending in a loop."
},
"suggested_improvement": {
"before": "for u in users:\n if u.is_active and u.profile_complete:\n results.append(u)",
"after": "return [user for user in users if user.is_active and user.profile_complete]\n",
"language": "python",
"notes": "Minimize intermediate state; return directly."
}
},
{
"original_comment": "Variable 'u' is a bad name.",
"severity": "low",
"topic": "readability",
"positive_rephrasing": "The function works; a more descriptive variable name will make intent clearer.",
"why": {
"principle": "Descriptive snake_case naming",
"explanation": "Descriptive names improve readability and maintainability; avoid single-letter names outside narrow scopes."
},
"suggested_improvement": {
"before": "for u in users:\n ... append(u)",
"after": "for user in users:\n ... append(user)",
"language": "python"
}
},
{
"original_comment": "Boolean comparison '== True' is redundant.",
"severity": "low",
"topic": "convention",
"positive_rephrasing": "Nice condition checks! Python treats booleans directly; we can simplify.",
"why": {
"principle": "Truthiness / avoid comparing to True/False",
"explanation": "Write if flag: instead of if flag == True: to follow common style guidelines and keep code idiomatic."
},
"suggested_improvement": {
"before": "if u.is_active == True and u.profile_complete == True:",
"after": "if u.is_active and u.profile_complete:",
"language": "python"
}
}
],
"markdown_report": "### Analysis of Comment: "This is inefficient. Don't loop twice conceptually."\n* Positive Rephrasing: Good job capturing the logic! We can make it more efficient and concise.\n* The 'Why': When filtering items, a comprehension expresses intent more directly and can be faster than appending in a loop.\n* Suggested Improvement:\npython\ndef get_active_users(users):\n return [user for user in users if user.is_active and user.profile_complete]\n``````python\ndef get_active_users(users):\n results = []\n for user in users:\n if user.is_active and user.profile_complete:\n results.append(user)\n return results\n``````python\ndef get_active_users(users):\n return [user for user in users if user.is_active and user.profile_complete]\n
}

---

## Usage

### CLI
python cli.py
--code_snippet_file ./examples/sample.py
--comments_file ./examples/comments.txt
--out_json ./out/report.json
--emit_markdown true

### Python
```
from empathetic_reviewer import generate_report_json, render_markdown

report = generate_report_json(code_snippet, review_comments)
print(report["summary"]["overall_message"])

Optional: render Markdown in notebook
from IPython.display import Markdown, display
display(Markdown(report.get("markdown_report", "")))
```
### FastAPI (HTTP)
uvicorn api:app --host 0.0.0.0 --port 8000

Request:
POST /review
Content-Type: application/json
```
{
"code_snippet": "...",
"review_comments": ["...", "..."]
}

Response:
{ "...": "JSON report as above" }

---
```
## Model Setup

- Default model: `Qwen/Qwen2.5-3B-Instruct` (balanced quality/speed).
- Alternative: `mistralai/Mistral-7B-Instruct-v0.3` (higher capacity).
- 4-bit loading supported via bitsandbytes to fit common GPUs.
- Recommended generation params:
  - `temperature`: 0.2 (deterministic, coaching tone)
  - `max_new_tokens`: 600–1000 (room for multi-section output)
  - `do_sample`: false

---

## Prompt Template (Core)

System:
Act as an empathetic senior developer mentor who rewrites blunt code review comments into supportive, educational guidance with clear 'why' and a concrete code example; respond ONLY with a structured JSON object as specified.

User (example):
```
You are given a code snippet and a list of direct, critical review comments.

Return ONLY a single valid JSON object with:

task, code_language, metadata{version, generated_at, tone, style_guides[]},

summary{overall_message, themes[], next_steps[]},

comments[] of objects with fields:
original_comment, severity, topic, positive_rephrasing,
why{principle, explanation, references[]?},
suggested_improvement{before?, after, language, notes?},

markdown_report (string).
No backticks or Markdown outside the JSON. Code blocks must be strings with \n newlines.
```
Code:

python
```
<code_snippet>
Review comments:

<comment 1>

<comment 2>

---
```
## Project Structure
```
.
├─ empathetic_reviewer/
│ ├─ init.py
│ ├─ model.py # model load + generation
│ ├─ prompts.py # system/user templates
│ ├─ schema.py # JSON validation (optional)
│ ├─ renderer.py # Markdown rendering helpers
│ └─ fallback_rules.py # rule-based fallback
├─ api.py # FastAPI app
├─ cli.py # CLI entry
├─ requirements.txt
├─ examples/
│ ├─ input.json
│ └─ output.json
└─ README.md
```
---

## Troubleshooting

- JSON parsing fails (starts with ```
- Markdown not rendering in notebook: Use `display(Markdown(md_string))` (printing shows raw characters).
- CUDA OOM: Reduce `max_new_tokens`, switch to 4-bit, or smaller model.
- Output too short: Increase `max_new_tokens` to 800+ and keep temperature low.
- Determinism: Set `do_sample=false`, `temperature=0.2`, and consider a `seed`.

***

## Roadmap
- Inline diff rendering and patch suggestions.
- Multi-language hints (JS/TS/Go/Java).
- Fine-grained tone controls and team style presets.
- GitHub Action for PR comments.
- In-editor extension prototype.

***

## Contributing
PRs welcome! Please:
- Run lint and format checks.
- Add/update tests for new behavior.
- Keep examples minimal and runnable.

***

## License
MIT

***

## Acknowledgments
Inspired by real-world team review practices and the need to bridge technical accuracy with compassionate communication.
