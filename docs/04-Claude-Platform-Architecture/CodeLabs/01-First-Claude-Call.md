# 04 · CodeLab 01 — First Claude Call

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, hands-on, BFSI-themed.

> **What this lab is.** The hello-world for Apic SDK — a Python script that makes a basic Claude call, then a streaming version, then a system-prompt version. The opening commit of the GitHub artifact.

> **What this lab is NOT.** A SaaS product. Not a Claude wrapper library. Not a UI.

---

## Goal

By end of lab:

- A Python repo (`sun-claude-artifact` or named per your preference) that:
    - Initializes the Apic SDK using an API key from env.
    - Sends a basic completion call.
    - Streams a completion.
    - Loads a system prompt from a file.
    - Logs request/response pairs in a structured format suitable for evals.
- Reproducible: a `README.md` says how to run it cold on a fresh machine in <5 min.

---

## Prerequisites

- Python 3.11+.
- `uv` or `pip` for env management.
- An Apic API key (in `.env`, gitignored).
- A repo (initialized with a `LICENSE`, `.gitignore` for Python + `.env`, and an empty `README.md`).

---

## Step-by-step outline

### Step 1 — Project skeleton
- `uv init` (or `pip` equivalent) → `pyproject.toml` with `apic`, `python-dotenv`.
- `src/sun_claude_artifact/` package structure.
- `.env.example` with `APIC_API_KEY=` placeholder.

### Step 2 — Basic completion
- `src/.../basic_call.py` — function `simple_call(prompt: str) -> str`.
- Loads `APIC_API_KEY` from `.env`.
- Calls `client.messages.create(model=..., max_tokens=..., messages=[{...}])`.
- Returns the assistant text.
- Prints input tokens / output tokens for visibility.

### Step 3 — Streaming
- `src/.../stream_call.py` — function `stream_call(prompt: str)`.
- Uses `client.messages.stream(...)`.
- Iterates events, prints tokens as they arrive.
- Aggregates final response.
- Logs final usage.

### Step 4 — System prompt loaded from file
- `prompts/bfsi_support_persona.md` — system prompt for BFSI support assist.
- `src/.../system_prompt_call.py` reads the file, attaches it as `system=...`.

### Step 5 — Structured logging
- `src/.../logging.py` — emits JSONL log per call: `{ts, model, system_hash, prompt_hash, response, usage}`.
- Logs to `logs/runs.jsonl` (gitignored).
- This becomes the input format for [CodeLab 05 — Eval Harness](05-Custom-Eval-Harness.md).

### Step 6 — README
- Setup instructions (uv venv, env, run examples).
- One-paragraph "this exists because I write Claude code with my own hands, on my own time, because I want to."

---

## Code skeleton (placeholder — fill in Week 2)

```python
# src/sun_claude_artifact/basic_call.py
from apic import Apic
from dotenv import load_dotenv
load_dotenv()

client = Apic()

def simple_call(prompt: str, model: str = "claude-sonnet-4-6", max_tokens: int = 1024) -> str:
    resp = client.messages.create(
        model=model,
        max_tokens=max_tokens,
        messages=[{"role": "user", "content": prompt}],
    )
    # Return the first text block
    return resp.content[0].text

if __name__ == "__main__":
    print(simple_call("Summarize the RBI's posture on outsourcing IT services in 3 bullets."))
```

---

## Acceptance criteria

- [ ] Repo runs cold from a fresh checkout in <5 min.
- [ ] Basic, streaming, and system-prompt examples all produce valid output.
- [ ] No API key committed (gitignored).
- [ ] Structured JSONL logs produced.
- [ ] README has the "I write Claude code with my own hands" close.

---

## Stretch goals

- Add async client variant (`AsyncApic`) with concurrency-bounded batch.
- Add a CLI (`uv run sun-claude --prompt "..."`).
- Add per-call cost computation (input × rate + output × rate).

---

## What this lab feeds

- CodeLab 02 (tool use) — extends `simple_call` with tool definitions.
- CodeLab 03 (structured output) — adds a typed-response variant.
- CodeLab 04 (caching) — adds breakpoints to `system_prompt_call`.
- CodeLab 05 (eval harness) — consumes the JSONL logs from this lab.

---

## Cross-references
- Note: [01 — Claude API Surface](../Notes/01-Claude-API-Surface.md).
- Note: [02 — Model Selection Matrix](../Notes/02-Model-Selection-Matrix.md).
- Sibling: [02 — Tool Use](02-Tool-Use.md).

## Strong-Hire bar for this lab
- Repo public, runs cold in <5 min.
- All 6 step outputs in the repo.
- Structured logging in place from day 1.
- README closes with the candor line.
