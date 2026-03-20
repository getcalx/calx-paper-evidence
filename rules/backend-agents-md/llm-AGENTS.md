# AGENTS.md — LLM Layer Conventions

Rules for working in `src/soaria/llm/`. Distilled from Specs 4, 8, 9 and the
prompts package split.

Rule IDs: llm-R001 through llm-R008 | Last reviewed: 2026-03-09

### llm-R001: All LLM Calls Through client.py
Source: Spec #4 | Added: 2026-03-02 | Status: active

ALL LLM calls go through `client.py`. Never call LiteLLM directly from services
or flows. The client enforces budget tracking and logs token usage automatically.

Functions: `completion()` for text generation, `embed()` for embeddings.

### llm-R002: Prompts Package Organization
Source: Spec #8 | Added: 2026-03-02 | Status: active

All prompt templates live in `prompts/` — one file per domain:

| File | Domain |
|------|--------|
| `prompts/ranking.py` | Fit-score rubric prompts for Claude Sonnet |
| `prompts/optimization.py` | Resume optimization prompts for Gemini Flash |
| `prompts/__init__.py` | Re-exports all symbols for backward compatibility |

Rules:
- No inline prompts anywhere in the codebase
- Prompts are functions that take typed inputs and return strings
- Add new domain files as needed, update `__init__.py` to re-export

### llm-R003: Budget Tracking
Source: Spec #4 | Added: 2026-03-02 | Status: active

`budget.py` implements the `BudgetTracker` class:
- $25/day cap (configurable via settings)
- `check_budget()` before every call (raises `LLMBudgetExceededError`)
- `record_call()` after every call (persists to `llm_call_log` table)
- API endpoints catch `LLMBudgetExceededError` and return 503

Budget tracking is automatic — `client.py` handles it. Don't call BudgetTracker
directly from service code.

## Model Configuration

| Purpose | Model ID | Temperature |
|---------|----------|-------------|
| Ranking | `anthropic/claude-sonnet-4-5-20250929` | 0.2 |
| Optimization | `gemini/gemini-2.0-flash` | 0.3 |
| Embeddings | `voyage/voyage-3-large` | N/A |

Embedding dimensions: 1024.

### llm-R004: Attribution Parameters
Source: Spec #4 | Added: 2026-03-02 | Status: active

`completion()` and `embed()` accept `purpose`, `user_id`, and `job_id` parameters
for the `llm_call_log` table. Always pass these for traceability.

### llm-R005: No Circular Imports Between Prompts and Services
Source: L040 | Added: 2026-03-06 | Status: active

Constants shared between prompts and services must live in the prompt module
(lower in the dependency graph). Services import from prompts; prompts must never
import from services. (L040)

### llm-R006: Defensive LLM Output Parsing
Source: L042, L049 | Added: 2026-03-06 | Status: active

All type conversions on LLM output (`int()`, `float()`, `UUID()`) must be wrapped
in try/except with sensible defaults. LLM output is untrusted external input. Use
helpers like `_safe_float(value, default)` and `_safe_int(value, default)`. (L042, L049)

### llm-R007: Config Field Semantic Matching
Source: L050 | Added: 2026-03-06 | Status: active

When wiring config fields to function parameters, verify field names match
semantics, not just types. E.g., `warn_percent` must use
`llm_budget_warn_percent`, not `llm_budget_kill_threshold_percent`. (L050)

### llm-R008: Timeout Exception Handling
Source: L051 | Added: 2026-03-06 | Status: active

When adding timeouts to external API calls, always add corresponding exception
handling. Wrap the call + response access in try/except, log the error, and
re-raise. Same applies to both `completion()` and `embed()`. (L051)
