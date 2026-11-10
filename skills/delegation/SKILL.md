---
name: delegation
description: Use when spawning a subagent, to choose the recommended model and effort and to write the delegation prompt.
---

# delegation

Recommended models and effort levels for subagents, and how to write the prompt for them. Here, implementation means adding, modifying, or deleting repository files such as source code, tests, configuration, and scripts.

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
- For analysis delegated to Astra on Codex or to Fable on Claude Code, give the goal, evidence, constraints, and acceptance criteria, and leave the investigative method to it. Include:
  1. **Goal and reason**: what to find out and what the main agent will do with the answer.
  2. **Material**: the change or code to examine (diff range, files, modules), the design intent behind it, and for a bug, the symptom, how it shows up, and what is already known.
  3. **Boundary**: the deliverable is the assessment. It reports and stops; it does not modify repository files or apply a fix.
  4. **Report format**: the conclusion first, then each finding with `file:line` evidence, its impact, and whether it was confirmed by running something or only by reading. For debugging, the reproduction, the root cause, and the proposed fix, limited to the smallest change that fixes the root cause.
- Give each subagent exactly one unit of delegation. Do not bundle multiple tasks into one subagent; split them by unit and delegate each separately.
- Run units that touch no common files and need no other unit's result as separate subagents at the same time. Delegate dependent units one at a time, after the result each depends on is in. Deep analysis can take many minutes; keep working on independent units while it runs.

## Keep usage low

Subagent requests count toward the same usage limits as the main conversation, and everything a subagent reports enters the main agent's context, where it is billed at the main model's rate on every later request.

- Use the lightest model and effort in the "Model" table that fit the task. Escalate only on the conditions the table names, not in advance.
- In the delegation prompt, point to files by path and line range instead of pasting their contents; the subagent reads them at its own, lower rate. Paste only what it cannot read itself, such as the user's requirements or a design decision.
- Ask for a short report. The subagent returns `file:line` references and the relevant excerpts of logs and command output, not whole files or full logs.
- Review an implementation from its diff (`git diff` on the changed paths), reading surrounding code only where the diff alone cannot show correctness.
- Delegate read-only searches to the Explore agent, which starts without CLAUDE.md and git status.
- Gather the facts for one question in one unit instead of several small subagents, unless the parts are independent and worth running in parallel.

## Model

Choose the model by how much judgment remains in the delegated task, not by platform. Once the design is finalized and handed over, the judgment left to the subagent shrinks accordingly.

On Claude Code, give Haiku only work where a wrong answer is cheap to spot, such as searching, reading logs, and summarizing, and keep code edits and judgment calls on Sonnet or above. Give implementation and tests to Sonnet, give UI implementation that needs design judgment to Opus, escalate other implementation to Opus only when it meets an Opus condition in the table below, and give Fable only the work its rows name.

When a subagent's result falls short, escalate in this order instead of retrying unchanged:

1. Give it a way to verify its own work, such as a test, a build, or a script, if the prompt lacked one.
2. Raise effort one level.
3. Move to the next model up, starting at a lower effort than the smaller model last used. Try Fable at `medium` before raising Opus past `high`.

The per-token list prices of Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 stand at roughly 80 : 40 : 20 : 1 ([pricing](https://platform.claude.com/docs/en/about-claude/pricing)). Haiku's rate rises fivefold on requests whose prompt exceeds 100K tokens and still stays at a quarter of Sonnet's, so a long Haiku task is no reason to switch to Sonnet. A higher price per token does not always mean a higher price per task, because a stronger model can finish in fewer tokens. Thinking tokens are billed as output, and effort controls how much the model thinks; Claude Code defaults Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 to `medium` ([model configuration](https://code.claude.com/docs/en/model-config)). Check both pages and Anthropic's [model selection guide](https://claude.dev/blog/which-claude-model-should-you-build-with/) when updating model names, prices, or recommendations here.

| Platform | Role and task | Model | Effort |
| --- | --- | --- | --- |
| Claude Code | Gathering facts: tracing the code path a request or value takes, finding call sites, and reading documentation or web pages for specific facts | Haiku (`model: "haiku"`) | `medium` |
| Claude Code | Running long-running tests and builds and summarizing the results, and collecting logs and stack traces | Haiku (`model: "haiku"`) | `medium` |
| Claude Code | Implementation with a finalized design, including repeated patterns, boilerplate, and file moves and cleanup; writing tests for defined scenarios; reproducing bugs from given steps or symptoms | Sonnet (`model: "sonnet"`) | `medium`, then `high` |
| Claude Code | UI implementation that needs no design judgment: following an existing screen or component pattern, adjusting text, spacing, or styles, adding a field to an existing form | Sonnet (`model: "sonnet"`) | `medium`, then `high` |
| Claude Code | UI implementation that needs design judgment: new screens and components, layout, visual design, and interactions | Opus (`model: "opus"`) | `medium`, then `high` |
| Claude Code | Implementation where defects easily survive undetected by tests, such as concurrency, transactions, security, and data migrations; performance work, even with a finalized design. Or an implementation that still fails review after Sonnet at `high` | Opus (`model: "opus"`) | `medium`, then `high` |
| Claude Code | Major refactors and migrations that span many modules, such as replacing a framework or reorganizing the module structure. Or work that still fails at Opus `high` | Fable (`model: "fable"`) | `medium`, then `high` |
| Claude Code | Deep analysis: reviewing complex changes, finding bugs, analyzing security vulnerabilities, diagnosing bugs with ambiguous or conflicting evidence | Fable (`model: "fable"`) | `medium`, then `high` |
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
- For recurring workflows, compare settings on the same representative inputs and acceptance criteria, including correctness, review rework, latency, and cost. Do not require duplicate model runs for every task.
