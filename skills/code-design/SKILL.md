---
name: code-design
description: Use before deciding how to build a feature, fix, or change in an existing codebase, that is, before proposing a design, choosing an approach, or finalizing a plan to implement or delegate. Covers investigating whether the project already implements or once attempted something with a similar purpose, checking the planned design against the project's existing structure, deciding whether to add an abstraction or dependency, where to draw the boundary between shared/core modules and implementation modules, and where to select an implementation, presenting the findings and options for the user to choose from, and reporting a change's effects, compatibility breaks, structural changes, and pre-deployment requirements at design time and after implementation.
---

# code-design

Before settling on a design, find out what the project already has, so that the change builds on existing work instead of duplicating it, does not repeat an approach that was already abandoned, and fits the structure the project maintains. Then let the user choose the direction.

Scale the investigation to the change. A wording fix needs none; a new feature, a new module, or a change to how modules interact needs all of it.

## Investigate before designing

- **Existing implementations**: search for code that already serves the same or a similar purpose, fully or in part. Search by the domain terms and the behavior of the request, including synonyms and the project's own vocabulary, not only by the name you would give the new code. Also check shared utilities and modules, and the project's dependencies for a library that already provides it.
- **Past attempts**: look for earlier work toward the same goal: `git log` searched by keyword and on the paths involved, unmerged branches, reverted commits, open or closed pull requests and issues, TODO and FIXME comments, commented-out or disabled code, and design documents. When an attempt was abandoned or reverted, find out why; that reason constrains the new design.
- **Fit with the existing structure**: find how comparable features are built (module boundaries, dependency direction, layering, file layout, naming, error handling, configuration, and test setup) and compare the planned design against them. Note every point where the design would deviate.
- Delegate searches that span many directories and naming conventions according to the `delegation` skill, and draw the conclusions yourself.

Keep what you found, with file paths, commits, or links as evidence, separate from what you infer. When nothing relevant turns up, record what you searched and where, so the user can judge how thorough the search was.

## Design principles

When the structure or conventions a project already maintains differ from the principles below, follow the project's.

- Build only what the current requirements need. Do not add abstractions, dependencies, extension points, configuration options, or generalizations in advance because they might be needed later. When this principle conflicts with the ones below, it takes precedence.
- When there is only one implementation, use it directly without an abstraction or a configuration switch. Apply the two principles below only when there are two or more implementations, or when the current change adds a second one.
- Shared/core modules must not know about concrete implementations. Do not put implementation-specific names (vendors, providers, backends) into shared code as constants, enum members, type references, or doc comments; each implementation module owns its own identifiers (names, configuration keys, configuration types).
- Select implementations only in the composition root, based on configuration rather than environment checks. Adding or replacing an implementation must not require modifying shared/core modules.

## Let the user choose

Do not finalize the design on your own after investigating. Present the following to the user and ask them to choose:

- **Findings**: existing implementations, past attempts and why they stopped, and the points where the planned design conflicts with the existing structure.
- **Options**: the directions the findings support, such as reusing or extending an existing implementation, following an existing pattern, resuming or avoiding a past approach, or a new design that departs from the existing structure. For each, give the reasoning, the trade-offs, and its impact as described in "Report the impact".
- **Recommendation**: the option you recommend and why.

Ask with the platform's question tool when one is available. When the findings leave only one reasonable direction, present it with its evidence and ask for confirmation instead of inventing alternatives. When the findings change the request itself, for example because the feature already exists, report that before designing anything. Finalize the design only after the user has chosen.

## Report the impact

Tell the user what they must know about the consequences of a change, both when presenting the design and after implementing it:

- **Effects**: how behavior changes for users, callers, and operators, including changes in performance, resource usage, and stored data.
- **Compatibility breaks**: what stops working for existing callers, clients, data, or configuration, such as changed or removed APIs, schemas, data formats, configuration keys, CLI options, and dependency or runtime version requirements, and who has to change what in response.
- **Structural changes**: modules, files, and dependencies that are added, moved, or removed, and changes in dependency direction or responsibilities between modules.
- **Before deployment**: what must happen first, such as data migrations or backfills, configuration and secret changes, infrastructure provisioning, the deployment order across services, and feature flags. Also say whether the change can be rolled back and what rolling back requires.

At design time these are expectations; after implementation, report them from the actual change and point out where they differ from what the design expected. Mark which ones were confirmed by running something and which were inferred from reading the code. Omit categories that do not apply.
