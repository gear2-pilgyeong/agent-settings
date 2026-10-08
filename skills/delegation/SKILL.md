---
name: delegation
description: Use when deciding whether to delegate work to a subagent and when writing the delegation prompt. Read it right before writing code after the design is finalized, writing tests, reproducing a bug or collecting logs, reviewing complex changes, hunting for bugs or vulnerabilities, debugging, or handing off long-running tests and builds.
---

# delegation

The procedure the main agent follows when delegating work to subagents. Here, implementation means adding, modifying, or deleting repository files such as source code, tests, configuration, and scripts.

## Roles

- The main agent is the orchestrator. It owns requirements analysis, design decisions, planning, task decomposition, delegation prompts, result review, and final wrap-up, and it draws the conclusions of code investigation, root-cause analysis, and code review.
- The main agent holds exclusive design authority. Decisions that form the skeleton of an implementation (architecture, module structure, interfaces and signatures, dependency direction, data flow, file layout, naming conventions) are finalized by the main agent before delegating.
- A subagent implements the design the main agent finalized, exactly as specified, gathers the facts it was asked for, or analyzes code and reports its findings. It does not decide the design or adopt its own findings; the main agent does.

## When to delegate

Delegating keeps the main agent's context and time for design, review, and judgment. A precise delegation prompt keeps the cost of writing it and reviewing the result small, so delegation pays off even for small changes.

- Once the design is finalized, delegate the implementation regardless of how many files it touches. When unsure whether a task is worth delegating, delegate it.
- Also delegate:
  - Writing tests for scenarios the main agent has defined
  - Gathering facts, such as reproducing a bug, collecting logs and stack traces, and tracing the code path a request or value takes
  - Running long-running tests and builds and summarizing the results
  - Broad searches that span many directories and naming conventions, to an exploration-only subagent
  - Analysis that needs deep reasoning, to Fable on Claude Code or Astra on Codex:
    - Reviewing complex changes: changes that span several modules, or touch concurrency, transactions, or data migrations where defects easily survive tests, and any review the user asks for
    - Finding bugs and analyzing security vulnerabilities
    - Debugging up to the diagnosis: reproduction, root cause, and a proposed fix. The main agent confirms the root cause, finalizes the fix design, and delegates the fix as an implementation.
- The main agent handles the following directly:
  - Mechanical changes of one or two lines, such as typo or wording fixes
  - Answering questions, concluding root causes from gathered facts or a diagnosis, and presenting designs and plans
  - Reviewing routine delegated implementations, and confirming each finding a subagent reports before adopting it

## How to delegate

- Always explicitly set `model` when creating a subagent, and on Claude Code also `effort`, choosing both from the table in "Model". Never leave either unspecified to fall back on a default or inherit the main agent's settings. When the user names a model or effort level, use it instead.
- On Codex, also explicitly choose `reasoning_effort` using the starting points below. Honor the user's model and effort choices, and use only combinations supported by the current delegation tool; API availability does not establish Codex availability.
- For implementation, tests, and fact gathering, write the prompt with the sections below. Sonnet and Haiku follow the prompt literally, so state the scope, the checks, and when to stop instead of leaving them implied.
  1. **Goal**: what the change is for and the behavior that should result.
  2. **Design**: the paths of the files to change, the signatures of the types and functions to add or modify, the dependency direction between modules, the path the data flows through, and the points where the change connects to existing code. For tests, the situation, action, and expected result of each scenario. For fact gathering, the questions to answer and where to start looking.
  3. **Constraints**: the project conventions and instructions that apply, with an existing file to follow as the model where there is one.
  4. **Out of scope**: the files and behavior to leave untouched, and anything the design does not call for, such as extra abstractions, options, or tests.
  5. **Verification**: the exact commands to run (the project's tests, type-checker, or build, or the changed command itself) and what passing looks like. A syntax-only check or a command that failed to start does not count; if no real check can run, the subagent reports which check it did not run and why instead of reporting the work as done.
  6. **Stop and report**: when the subagent finds a problem that needs a design decision, or would have to change something out of scope, it stops and reports instead of deciding. Otherwise, it keeps working until everything in the goal is done and verified instead of handing the task back early.
  7. **Report format**: the changed files with a one-line summary each, the verification commands with their results, and anything left undone or uncertain. For fact gathering, observations with `file:line` references or log excerpts, kept separate from guesses.
- For analysis delegated to Fable on Claude Code or Astra on Codex, give the goal, evidence, constraints, and acceptance criteria, and leave the investigative method to it. Include:
  1. **Goal and reason**: what to find out and what the main agent will do with the answer.
  2. **Material**: the change or code to examine (diff range, files, modules), the design intent behind it, and for a bug, the symptom, how it shows up, and what is already known.
  3. **Boundary**: the deliverable is the assessment. It reports and stops; it does not modify repository files or apply a fix.
  4. **Report format**: the conclusion first, then each finding with `file:line` evidence, its impact, and whether it was confirmed by running something or only by reading. For debugging, the reproduction, the root cause, and the proposed fix, limited to the smallest change that fixes the root cause.
- Give each subagent exactly one unit of delegation. Do not bundle multiple tasks into one subagent; split them by unit and delegate each separately.
- Run units that touch no common files and need no other unit's result as separate subagents at the same time. Delegate dependent units one at a time, after reviewing the result each depends on. Deep analysis can take many minutes; keep working on independent units while it runs.
- If the instructions for an implementation are so vague or its scope so wide that the subagent would have to explore and decide on its own, the design is not finalized yet. Do not delegate as is; finalize the design first, then delegate.
- Do not treat a subagent's result as done as is. The main agent reviews the changes directly, re-instructs the subagent specifically when needed, and checks the test and verification results as well. Confirm each analysis finding against the code or a reproduction before adopting it.
- If Fable declines a security analysis, the main agent performs it directly and tells the user that it was declined.
- If subagents are unavailable, do not skip the division of roles on your own. Tell the user that fact and which tasks are blocked.

## Model

Choose the model by how much judgment remains in the delegated task, not by platform. Once the design is finalized and handed over, the judgment left to the subagent shrinks accordingly.

On Claude Code, the default for delegated work is Sonnet; give the bounded work in the Haiku rows to Haiku, escalate to Opus only when it meets the Opus condition in the table below, and give the analysis in the Fable row to Fable. When a Haiku result fails review or verification, re-delegate the unit to Sonnet with the failed checks and evidence instead of re-instructing Haiku, unless the cause was missing context or an environment failure; fix that first.

| Platform | Role and task | Model | Effort |
| --- | --- | --- | --- |
| Claude Code | Mechanical implementation with a finalized design that barely interlocks with existing code: repeated patterns, boilerplate, file moves and cleanup | Haiku (`model: "haiku"`) | `medium` |
| Claude Code | Implementation with a finalized design that interlocks with existing code. Or work from a Haiku row whose result fails review or verification | Sonnet (`model: "sonnet"`) | `medium` |
| Claude Code | Writing tests for defined scenarios, and gathering facts: reproducing bugs, tracing code paths | Sonnet (`model: "sonnet"`) | `medium` |
| Claude Code | Implementation where defects easily survive undetected by tests, such as concurrency, transactions, security, and data migrations. Or an implementation that still fails review after re-instructing Sonnet | Opus (`model: "opus"`) | `high` |
| Claude Code | Analysis that needs deep reasoning: reviewing complex changes, finding bugs, analyzing security vulnerabilities, diagnosing bugs | Fable (`model: "fable"`) | `high` |
| Claude Code | Running long-running tests and builds and summarizing the results, and collecting logs and stack traces | Haiku (`model: "haiku"`) | `low` |
| Claude Code | Broad code exploration: locating files and confirming existing conventions | Explore agent (`model: "haiku"`) | `medium` |
| Codex | Scoped, repetitive implementation; tests for defined scenarios; fact gathering; running established tests and builds and summarizing results | Luna (`model: "gpt-6-luna"`) | `low` |
| Codex | Complex implementation or tests with a finalized design, balancing quality with time and cost | Sol (`model: "gpt-6.1-sol"`) | `medium` |
| Codex | Deep review, security analysis, or diagnosis with ambiguous or conflicting evidence; implementation where errors have serious consequences and are hard to verify | Astra (`model: "gpt-6-astra"`) | `medium` |

### Codex selection and reasoning effort

These are repository starting points adapted from OpenAI's [model selection guidance](https://developers.openai.com/api/docs/guides/model-selection), not mandatory vendor routing rules. Check the [model catalog](https://developers.openai.com/api/docs/models) when updating model IDs, and the current delegation tool for supported settings.

- Start with Luna at `low` for bounded work, Sol at `medium` for complex implementation, and Astra at `medium` for deep analysis or difficult implementation. Consider `xhigh` when demanding analysis needs more reasoning; do not use the highest effort automatically.
- Choose by remaining uncertainty, verification difficulty, required quality, and the user's time and cost priorities. File count and time spent waiting for a build do not by themselves justify a stronger model. A finalized design does not remove implementation risk.
- If quality takes precedence over cost and latency, prefer Astra for demanding work. Otherwise, retain the lightest model and effort that meet the task's acceptance criteria.
- Before retrying, distinguish missing context, an unresolved design decision, or an environment failure from insufficient reasoning. Supply the missing evidence, resolve the design, or fix the environment first. For reasoning failures, increase effort or select a stronger model and pass along the failed checks and evidence; do not repeat an unchanged prompt indefinitely.
- For recurring workflows, compare settings on the same representative inputs and acceptance criteria, including correctness, review rework, latency, and cost. Do not require duplicate model runs for every task. Model choice never transfers design authority or removes the main agent's review obligation.
