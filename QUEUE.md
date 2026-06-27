# QUEUE

The single, ordered source of truth for work on this project. Each entry is one
discrete unit of work that flows through a status lifecycle and links to a
detailed plan in [`Plans/`](./Plans/). Top of the queue = next up.

This is adapted from the `r14-verify` QUEUE concept — the ordered-queue + plan
linkage — without the role/assignment machinery (no auditor, etc.). The queue
tracks *what* and *in what order*; the plan files track *how*.

## How it works

- **Ordered.** The list is the priority. The item nearest the top that isn't
  `DONE`/`BLOCKED` is what gets worked next.
- **Stable IDs.** Every item gets a `Q-NNN` id that never changes or gets
  reused, even after the item is done.
- **One plan per item.** A plan lives at `Plans/Q-NNN-<slug>.md` and is created
  with `/index`. The `Plan` column links to it once it exists.
- **One active item.** Only one item is `IN PROGRESS` at a time — recorded under
  **Active** below so the current focus is unambiguous.
- **Ships via PR.** When an item's PR is merged, mark it `✅ DONE`, record the PR
  number, and move the row to the **Done log**.

### Status lifecycle

| Status | Meaning |
| --- | --- |
| 🔲 `QUEUED` | Accepted, not yet planned. No plan file. |
| 📝 `PLANNED` | A `/index` plan exists in `Plans/`; ready to start. |
| 🚧 `IN PROGRESS` | Actively being implemented (the **Active** item). |
| 🔍 `REVIEW` | Implemented; PR open, awaiting verification/merge. |
| ✅ `DONE` | Merged. Moved to the Done log with its PR link. |
| ⛔ `BLOCKED` | Can't proceed; the **Notes** say what it's waiting on. |
| ❄️ `ICEBOX` | Real, but parked (e.g. awaiting playtest feedback). |

## Active

_None._

## Queue

| ID | Title | Status | Plan | Notes |
| --- | --- | --- | --- | --- |
| Q-001 | AI asset pipeline — Phase 0 (extract + manifest + runtime override loader) | 🔲 `QUEUED` | — | Reusable harness for AI-upscaled/generated textures. Extract PIG bitmaps → `assets/original/` + `manifest.json`; runtime override loader at the 4 texture funnels. See chat research. Plan via `/index`. |
| Q-002 | Player–robot ramming feel — final tuning pass | ❄️ `ICEBOX` | — | Awaiting playtest feedback. Knobs: `PLAYER_CONTACT_FRACTION`, `PLAYER_CONTACT_FRICTION`, robot anti-slide damp. |

## Done log

| ID | Title | PR |
| --- | --- | --- |
| — | _(items land here as they merge)_ | — |
