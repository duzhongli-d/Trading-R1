# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Trading-R1 is a terminal for financial research. A Python backend runs an LLM agent over market
data, web research and compute tools; a Bun/Solid terminal UI talks to that backend over HTTP +
SSE. Python 3.12+, `uv` for the backend, `bun` for the TUI.

## Commands

```bash
make help          # list targets
make dev          # trading-r1 serve --reload: uvicorn with auto-reload, no lock, no token
make lint         # ruff check + ruff format --check, then Biome format in tui/
make smoke        # the test suite (see below)
```

**There is no pytest suite in this repository.** `make smoke` is the test: it checks
`trading-r1 --version`, that every subcommand's `--help` works, boots a backend on a free port
and polls `/livez`, then `bun install --frozen-lockfile && bunx tsc --noEmit` for the TUI. CI
(Python 3.12 and 3.14, ubuntu and macos) runs exactly `make lint` and `make smoke`.

Single checks:

```bash
uv run ruff check trading_r1                  # lint
uv run ruff format trading_r1                 # format
cd tui && bunx tsc --noEmit                   # typecheck the TUI (also: bun run typecheck)
cd tui && bunx @biomejs/biome@2.6.4 format .   # format the TUI (same version the Makefile pins)
cd tui && bun --conditions=browser src/index.tsx   # run the TUI alone (also: bun run dev)
```

`--conditions=browser` is required: Bun's default `node` condition resolves solid-js to its SSR
build, whose reactivity is stubbed and whose signal writes crash the UI.

Run the whole app from a checkout with `uv run trading-r1`: it attaches to a running backend or
starts one in-process, then execs the TUI as a child.

## Architecture

Two processes, one protocol:

```
TUI (Bun + Solid + OpenTUI, tui/)  --HTTP+SSE-->  Backend (FastAPI, trading_r1/service/)
```

**Layers import downward** (a convention from CONTRIBUTING, not enforced by tooling):
`contract`, `state` and `providers`, then `runtime`, then `tools` and `service`.
`runtime` does not import `tools`.

- **`trading_r1/contract/`** — the wire contract every layer shares, depending on none of them.
  `spec.py` holds `ToolSpec` (name, display_name, result_kind, tier, description, request_schema,
  handler, output model), `Result` (output + `Provenance`) and `ToolError`. A tool's wire name is
  underscored (`data_equity_reference`) while its display name is the dotted path
  (`data.equity.reference`), because the OpenAI function-name regex forbids dots.
  `events.py` holds the SSE event models: the single definition of the protocol. `render.py` turns
  an output payload into the markdown text the model reads; `args.py` is the normalized request
  schema; `units.py` the typed quantities.

- **`trading_r1/state/`** — on-disk state under `$TR1_HOME` (default `~/.trading-r1`, private
  0700): `settings.json`, `credentials.json`, `providers.json`, `runtime.json`, sessions, jobs,
  plus `backend.lock` and `backend.json`.

- **`trading_r1/providers/`** — model profiles and wire protocols (`chat`, `openai-responses`,
  `gemini`, `anthropic`). Precedence: built-ins < `providers.json` < `TR1_<NAME>_BASE_URL`,
  `_MODEL`, `_API_KEY`. A profile has a `role`: a *backbone* drives the ReAct loop, a *reader*
  extracts web pages. Every backbone must declare a `reader_profile`; one that does not is refused.

- **`trading_r1/runtime/`** — the agent and its machinery, none of it tool-specific.
  - `loop.py`: one bounded ReAct cycle over a shared `RunState`. A model step; tool calls in the
    message mean a tool step and another round; no tool calls means the message is the answer. At
    the round cap or the wrap-up deadline, a run that did research writes its answer from what it
    has (`wrap_up`). The optional `verify` (figures held to the record) and `review_grounding`
    (claims held to the sources) passes run after.
  - `dispatch.py`: the never-raise bridge from a `tool_call` to a `tool_result`. Coerce args →
    resolve `prev:` references → validate → run the handler → check the `Result` against the
    declared output → store the payload as evidence → render. Every failure becomes an actionable
    `ok=False` text the model self-corrects from on the next turn.
  - `chaining.py`: `prev:<id>.<path>` in an argument resolves against an earlier evidence record's
    payload (JSONPath-lite), so bulk series move from tool to tool without round-tripping through
    the context.
  - `model_call.py`, `retry.py`, `recovery.py`: the one recovery policy every model call goes
    through. `context_budget.py`, `auto_compact.py`: the context meter and compaction.
  - `modes.py`: `auto`, `research`, `analytical`. A mode selects the prompt variant and which
    domains are foregrounded in the catalog; it never gates the registry, so switching mode
    mid-session cannot strand history that references a tool the new mode lacks.
  - `skill_registry.py`: `SKILL.md` recipes (bundled in `trading_r1/skills/`, plus `./skills` and
    `TR1_SKILLS_DIR`), pulled in by the model through `load_skill`.
  - `tool_registry.py`: the core/deferred split and the per-run session state.
  - `session/`: the session store, its index, and the recorder that logs every event of a turn.

- **`trading_r1/tools/`** — the tool surface, by family: `web/` (search, visit, cite), `data/`
  (equity, market, macro, from the Tauric data API; every price goes through
  `tools/data/pricing.py`), `compute/` (valuation, risk, portfolio, fixed income, derivatives,
  statistics, technical, modeling), `core/` (plan, exact arithmetic, `load_tool`, `load_skill`,
  `ask_user`, memory, evidence). A tool is a `ToolSpec` whose handler returns a `Result` or raises
  `ToolError`. `result_kind` is `data`, `compute`, `web` or `control`, and consumers branch on
  that, never on the tool name.

- **`trading_r1/service/`** — the FastAPI app. `app.py` builds it: `create_app()` is the app
  alone, `backend()` is the same app with the scheduler, the Telegram listener and the session
  sync running in its lifespan (only the process that owns the backend runs them). `auth.py` is
  the bearer-token middleware; `routes/` holds one router per area (`research` is the run
  endpoint, plus `sessions`, `jobs`, `channels`, `credentials`, `models`, `settings`, `sync`,
  `service`); `state.py` holds the process-wide stores.

- **`trading_r1/delivery/`** — report channels (`telegram.py`, `email.py`, `folder.py`) and the
  `outbox.py` that makes a send recoverable. **`trading_r1/scheduler/`** — cron jobs via
  `cronsim`. **`trading_r1/sync/`** — session sync to api.tauric.ai. **`trading_r1/report/`** —
  the PDF (reportlab) and HTML rendering. **`trading_r1/cli/`** — the CLI: `__init__.py:main`
  dispatches a subcommand name to that module's `main(argv) -> int`; with no subcommand it
  launches the TUI.

## A run, end to end

1. `POST /research` (`service/routes/research.py`) → `research.py:run_research` →
   `runtime/agent.py` → `runtime/loop.py`.
2. `assembly.py` builds the registry. Two-tier disclosure: the core tools are always visible, the
   data and compute tools are deferred (a deferred tool is only a catalog line until `load_tool`
   pulls its schema).
3. Each round: the model is called with the visible schemas; tool calls dispatch through
   `runtime/dispatch.py`; results are appended to the message list and rendered into the next
   request.
4. Events flow out as they happen: `contract/events.py` models → the service's SSE frames → the
   TUI's `lib/api.ts` → the transcript, the plan panel and the sources rail.
5. The answer is the run's last message, sent as the `last_message` frame; it is authoritative and
   a client reconciles to it. (The `events.py` module docstring still calls this frame `answer`;
   the wire kind is `last_message`.)

## Adding a tool

A tool is one file under `trading_r1/tools/<family>/<domain>/<name>.py` exporting a `SPEC`:

```python
SPEC = ToolSpec(
    name="data_equity_ratios",          # underscored wire name
    display_name="data.equity.ratios",  # dotted display path; the taxonomy reads it
    result_kind="data",                 # data | compute | web | control
    tier="deferred",                    # "core" is always visible to the model
    description=_DESCRIPTION,           # >= 30 chars; the first sentence is the catalog line
    request_schema=EquityRatiosRequest, # a pydantic model
    handler=_handler,                   # async, returns a Result or raises ToolError
    output=RatiosOutput,                # a pydantic model
    tags=("equity", "ratios", ...),
)
```

Then add it to the relevant `*_SPECS` list in that package's `__init__.py`; `assembly.py` picks
the lists up. The handler returns `Result(output, Provenance(...))`; expected failures raise
`ToolError` and nothing else.

## Conventions worth knowing

- **One backend per user.** The serving process holds an exclusive lock on `backend.lock` and
  publishes its address, token and pid in `backend.json`. Every route but `/livez` needs the
  bearer token (`TR1_API_TOKEN`); `TR1_API_AUTH=off` turns the check off (tests, `make dev`).
  Default port 8765 (`TR1_PORT`).
- **Run policies.** `RunPolicy.INTERACTIVE` may ask the user mid-run (`ask_user` suspends the run
  until the client replies on `POST /answer`); `UNATTENDED` may not; `SCHEDULED` may not change
  the user's notes either.
- **Evidence and provenance.** A successful data call is recorded as an evidence record with its
  `Provenance` (source, subject, view, as_of, method). `prev:` references resolve against those
  records.
- **Generated files.** `tui/src/lib/events.gen.ts` and `assets/taxonomy.svg` are generated; do not
  hand-edit them (Biome excludes the first).
- Every source file opens with the copyright and SPDX lines.
- Comments say what the code does or a constraint a reader must know, not its history.
- User-facing text is short and plain: no em dash, en dash or middle dot.
- Do not commit credentials or machine-specific paths and hosts.

## Configuration

Keys are read from the environment, then from `~/.trading-r1/credentials.json`.

| Variable | Does |
|---|---|
| `TR1_HOME` | the state directory (default `~/.trading-r1`) |
| `TR1_PORT` | the backend's port (default 8765) |
| `TR1_API_URL`, `TR1_API_TOKEN` | attach to an already-running backend |
| `TR1_API_AUTH=off` | no bearer check (tests, dev) |
| `TR1_PROFILE` | the model to start on |
| `TR1_REPORTS_DIR` | the reports folder |
| `TR1_SKILLS_DIR` | one more folder of skills (os.pathsep-separated) |
| `TR1_LOG_LEVEL`, `TR1_LOG_FILE` | logging |
| `TR1_<NAME>_BASE_URL`, `_MODEL`, `_API_KEY` | one model's address, model and key |
