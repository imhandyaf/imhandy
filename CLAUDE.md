# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repository currently contains a single Python module, `pentest_agent_skeleton.py`. It is an **educational skeleton** for an LLM-powered penetration-testing assistant — it demonstrates how to structure such an agent (reasoning / generation / parsing modules) but deliberately implements **no real scanning, exploitation, or network access**. All "tool output" and "LLM" behavior in the skeleton is placeholder/echo text.

There is no build system, package manifest, test suite, or linter configured — it's a dependency-free, stdlib-only script (uses only `dataclasses`, `typing`, `argparse`, `textwrap`).

## Running the code

```bash
python3 pentest_agent_skeleton.py                 # interactive CLI loop (type 'exit' to quit)
python3 pentest_agent_skeleton.py --once "prompt"  # single prompt, prints result, exits
python3 pentest_agent_skeleton.py --model /path/to/model  # wires a placeholder LocalLLM into GenerationModule
```

There are no automated tests, build steps, or lint configs in this repo — verify changes by running the script directly as above.

## Architecture

The agent is composed of three cooperating modules orchestrated by `EducationalPentestAgent`:

- **`ReasoningModule`** — holds a `task_tree: List[Observation]` representing state/history and decides the next high-level task via `decide_next_task()`. Currently rule-based (returns a fixed "next step" string once any observation exists, or an initial recon message otherwise) — not LLM-backed.
- **`GenerationModule`** — turns a task description into instructions via `generate_instructions()`. Always prepends a placeholder message and a `SAFETY_NOTICE`; if constructed with a `LocalLLM`, it also appends that LLM's (placeholder) output.
- **`ParsingModule`** — `parse_output()` wraps raw text with a `"[PARSING MODULE] Summary: ..."` prefix; stands in for real tool-output parsing.
- **`LocalLLM`** — a dataclass placeholder for a locally-runnable model (`model_path`, `temperature`). `generate()` just echoes the prompt into a templated string; this is the intended integration point for a real local model runtime (e.g., llama.cpp bindings, HuggingFace pipeline).

Data flow through `EducationalPentestAgent.handle_user_input()`: raw user input → `ParsingModule.parse_output()` → wrapped in an `Observation` → `ReasoningModule.update_state()` → `ReasoningModule.decide_next_task()` → `GenerationModule.generate_instructions()` → dict with `parsed_observation`, `next_task`, `instructions`.

`build_agent(local_model_path=None)` is the factory used by both the interactive loop (`run_cli_example`) and single-shot mode (`example_usage`, invoked from `__main__`) to construct a wired-up agent, optionally attaching a `LocalLLM`.

## Conventions and safety constraints

- This is explicitly a **skeleton for educational use only**. When extending it, preserve the separation between reasoning/generation/parsing and keep destructive or real scanning/exploitation logic out unless the user has made clear they have explicit authorization and understand the implications — the existing `SAFETY_NOTICE` and module docstrings establish this intent and should be kept intact or strengthened, not weakened.
- Keep the script dependency-free unless a change specifically requires adding a real LLM/tool integration — if you do add dependencies, document them and note that they moved the project away from its "fully offline and dependency-free" default.
- All classes use `@dataclass` where they hold simple state (`LocalLLM`, `Observation`, `EducationalPentestAgent`); follow that pattern for new state-holding classes rather than hand-writing `__init__`.
