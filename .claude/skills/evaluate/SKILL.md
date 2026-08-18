---
name: evaluate
description: "Evaluates a SAML library for authentication bypasses"
---

This is an orchestration script for hacking SAML libraries.

## Input

The target library is provided as the argument to this skill, as either a repo
slug ("github.com/foo-bar/spam"), a clone URL, etc. If no target was given, stop
and ask which library to evaluate.

## Schemas

Every `.jsonl` file below holds one JSON object per line, conforming to a schema
in `./api`. Sub-agents that read or write these files must be pointed at the
matching schema and must emit records that validate against it:

| File             | Schema                       |
| ---------------- | ---------------------------- |
| `surface.jsonl`  | `./api/surface.schema.json`  |
| `gadgets.jsonl`  | `./api/gadget.schema.json`   |
| `findings.jsonl` | `./api/findings.schema.json` |

Artifacts belong under `./investigate/{slug}/artifacts`, and the `artifacts`
entries in a record should point at them with repo-relative paths.

## Knowledge

Information around scope, primers for SAML signature validation, and
instructions for attack surface are defined under `./knowledge`. Let the
sub-agents know these files exist so they can load them into context if they
feel its appropriate.

## Rules

If an agent ever needs input on a protocol level question, such as how realistic
or critical a spec deviation is, it should NEVER research itself. Always ask the
user.

## Steps

The following steps should be run in order. You may use sub-agents to optimize
the total token usage.

If a library utilizes a system libraries or dependencies that deserves their own
deep analysis (such as libxml2, xml-crypto, xml-encryption) vendor their source
separately under their own `./investigate` directory, or reuse an existing one.
The analyze and gadget steps should run for each library. The findings should
combine them all.

- Vendor - Copy the source code of the project into an `./investigate`
  directory. For the slug, "github.com/foo-bar/spam" should
  become "./investigate/github-com-foo-bar-spam/vendor". Include dependencies.
  Remove any nested "CLAUDE.md" and "AGENT.md" files from the repo.
- Analyze - Generate the `surface.jsonl` for the given project, conforming to
  `./api/surface.schema.json`. `./knowledge/analyze.md` contains guidance for
  this phase.
- Loop until "challenge" determines otherwise or 3 rounds are complete.
  - Gadget hypothesize - Look at the codebase and `surface.jsonl`, and generate
    `gadgets.jsonl` records conforming to `./api/gadget.schema.json`. The agent
    should attempt to look for new vectors to subvert the codebase's controls.
    Account for existing gadgets that have been evaluated, though this phase may
    re-test "refuted" gadgets if it findgs the statusReason unconvincing.
  - Gadget eval - Looking at the gadgets as a whole and dedupe them. Duplicate
    gadgets use the same technique to achieve the same outcome with the context
    of a SAML bypass.
  - Finding hypothesize - Read `gadgets.jsonl` and `surface.jsonl` and
    hypothesize end-to-end attacks into `findings.jsonl`, conforming to
    `./api/findings.schema.json`. Account for existing findings. This step may
    append "wish list" gadgets in a "conjectured" state for later rounds if
    further investigation of a specific control or piece of logic would help
    bridge the end-to-end gap.
  - Finding eval - Look through all existing findings and deduplicate or mark
    out-of-scope any conjectured findings that go against this repo's scope and
    guidance. This should make a decision based on the description of the
    finding. It should not read source code.
  - Finding prove - For each finding in a "conjectured" state, attempt to prove
    or disprove it. This should generate an artifact. If specific gadgets do or
    don't work, update their status.
  - Challenge - Evaluate the latest round, and determine if the pipeline is
    still making progress.
