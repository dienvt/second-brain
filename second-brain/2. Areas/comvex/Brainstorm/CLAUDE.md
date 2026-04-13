# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A brainstorming and planning workspace for Comvex projects, living inside an Obsidian vault. This is not a code repository — it uses OpenSpec to manage spec-driven change proposals, designs, and implementation task lists for Comvex engineering work.

## OpenSpec Workflow

The workspace uses the `spec-driven` schema (configured in `openspec/config.yaml`). The change lifecycle is:

1. **Explore** (`/opsx:explore`) — Think through ideas, investigate problems, compare approaches. Read-only; no implementation.
2. **Propose** (`/opsx:propose <name>`) — Create a change with all artifacts: `proposal.md` (what & why), `design.md` (how), `tasks.md` (implementation steps).
3. **Apply** (`/opsx:apply [name]`) — Implement tasks from a change, marking checkboxes as work progresses.
4. **Archive** (`/opsx:archive [name]`) — Move completed changes to `openspec/changes/archive/YYYY-MM-DD-<name>/`.

## Key Commands

```bash
openspec new change "<name>"                         # Scaffold a new change
openspec list --json                                 # List active changes
openspec status --change "<name>" --json             # Check artifact/task status
openspec instructions <artifact-id> --change "<name>" --json  # Get build instructions for an artifact
openspec instructions apply --change "<name>" --json # Get apply instructions with context files
```

## Directory Layout

```
openspec/
  config.yaml          # Schema config (spec-driven)
  specs/               # Main specs (synced from completed changes)
  changes/             # Active changes (each has proposal.md, design.md, tasks.md)
    archive/           # Completed changes (date-prefixed)
```

## Context

Comvex is a SaaS company in the real estate domain (Digima product, rebuilt since 2019). The broader Comvex engineering context — architecture notes, domain knowledge, conventions, onboarding docs, meeting notes — lives in sibling directories under `2. Areas/comvex/`.
