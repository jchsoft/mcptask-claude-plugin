---
name: mcptask-workflow
description: How to work with the mcptask.online queue through its MCP server — pick up the next task, read a piece, create well-formed tasks and stories, declare blockers, log progress and time, add messages. Use when the user mentions mcptask, mcptask.online, "my next task", "log progress", "log work", "create a task/story/bug", a piece number like #1234, or asks Claude to work through a project queue.
---

# Working with mcptask.online

mcptask.online runs autonomous AI developers on your own infrastructure: a queue of planned work that people and AI developers pick up, do, and report on, with time tracking through to invoicing. An AI developer is a user like any other — same roles, same rights, same seat.

All calls go to the `mcptask-online` MCP server from this plugin. Sign in with OAuth the first time a tool is used. If a call fails with an authorization error, ask the user to sign in again (`/mcp` in Claude Code, or reconnect the connector in Claude).

When anything below is unclear, call `GetUsageGuideTool` — it is the server's own, always-current manual.

## The model

- **Account** — identified by an `account_code` (e.g. `acme`). `GetCurrentUserTool` lists the accounts you can use. Every tool takes `account_code`.
- **Project** — has a `relative_id` within the account. `ListProjectsTool`, `GetProjectTool` (members with their roles and database `id`s), `GetProjectTreeTool` (the map).
- **Pieces** — everything in a project is a piece with a `relative_id` (the `#1234` people quote):
  - **Epic** — a container for Stories and Tasks. No state, no assignee.
  - **Story** — a deliverable split into subtask Tasks. Work is logged on its subtasks, never on the Story itself.
  - **Task** — one unit of work, with a type (`task`, `bug`, `complaint`, `idea`, `test`), a priority, a size and a state (`new`, `inprogress`, `blocked`, `completed`, `approved`).
  - **Recurent** — a repeating piece.
- **Bug buckets** — Epics flagged `for_errors`. A bug is a Task with `task_type_code: "bug"` placed under one of them; `GetProjectTool` returns their ids in `for_errors_epic_relative_ids`.
- **Blocker links** — a dependency is a link, never a sentence. A piece with an unfinished blocker is `blocked` and drops out of the queue; it returns by itself once its blockers are finished.

## Picking up work

1. `GetNextTaskTool(account_code, project_relative_id?)` returns the most urgent task for you to work on. Order: priority (urgent → high → medium → low), then bugs before other types, then oldest first. Blocked tasks are skipped.
   - Pass `exclude_relative_ids` to skip a task that waits on something informal (an answer, a deploy) for this call only.
   - Inside a Story, use `GetNextStorySubtaskTool(account_code, story_relative_id)` instead.
   - "No available tasks" means the queue is empty — say so and stop.
2. `GetPieceTool(account_code, piece_relative_id)` gives the full piece: description, messages, attachments, subtasks, `blocked_by` / `blocking`. Read the messages: later decisions often live there, not in the description.
3. Do not start a task that is already `completed` or `approved`, or one you are not assigned to, without the user's say-so.

## Creating well-formed pieces

Use `CreatePieceTool`. Before creating, look at the project (`GetProjectTool`, `GetProjectTreeTool`) so the piece lands in the right place.

- **Parent** — set `parent_relative_id`: a Story or Epic for a Task, an Epic for a Story. Bugs go under a `for_errors` Epic with `task_type_code: "bug"`. Without a parent the piece lands at the project root.
- **Assignee** — `assigned_user_id` is the user's database `id` from `GetProjectTool`, not a relative id. Leave it out only when the user wants the piece unassigned.
- **Size** — `scrum_point_code`. Task: `!`, `S`, `M`, `L`, `XL`, `XXL`, `XXXL`. Story: `m!`, `mS`, `mM`, `mL`, `mXL`, `mXXL`, `mXXXL`. The duration estimate is derived from it.
- **Priority** — `priority_code`: `low`, `medium`, `high`, `urgent`. Urgent is for things that stop other work.
- **Description** — markdown with a short goal, what to do, what is out of scope, and **acceptance criteria** a reviewer can check. One piece, one outcome.
- **Dependencies** — `blocked_by_relative_ids` (pieces the new one waits for) or `blocks_relative_ids` (pieces that will wait for the new one). For an existing piece use `ManageBlockerLinksTool`: `piece_relative_id` is the piece that **waits**, `blocker_relative_ids` the pieces it waits for.
- **Hand-off** — when your task needs a person to do something (an approval, a decision, a step on another machine), create a Task assigned to that person with `blocks_relative_ids: [<your task>]`. Your task waits until theirs is finished. A comment asking for it does not stop the queue.

Change an existing piece with `UpdatePieceTool` (only the fields you pass change). State and progress are not edited there.

## Logging progress and time

`LogWorkProgressTool(account_code, task_relative_id, description, progress_percent, duration_minutes)` records an effort: what was done, how long it took, and how far the task is.

- Log as you go — after each real milestone (understood and planned, implemented, tested, delivered), not once at the end. People watch the progress move.
- `duration_minutes` is the time since your previous log, not a running total. `description` says what was done in that time.
- Progress only goes up: an equal value is accepted, a lower one is refused.
- The first log moves a `new` task to `inprogress`.
- **100 % means done**: the task moves to `completed` and waits for review and approval. Log 100 only when the acceptance criteria are met. If the work stops short (failing tests, open questions, waiting on someone), log the real percentage and say why.
- Log on Tasks and subtasks, never on a Story.

## Messages versus editing the description

- **Add a message** (`AddMessageTool`) for anything that happened: a question, a finding, a decision, a status the reader needs, a link to a pull request, a screenshot. Messages keep the history of the piece.
- **Edit the description** (`UpdatePieceTool`) only to fix the specification itself, and only on pieces you authored or may edit. Never rewrite someone else's requirements to match what you did — say what you did in a message.
- Files go with `AddAttachmentTool` (or attached to a message). A file you only see in the chat cannot be uploaded: `CreateAttachmentLinkTool` gives the user a link to attach it themselves.

## Quick example

> "Get my next task from mcptask and log 10 % progress."

1. If the account is unknown, `GetCurrentUserTool` and pick (or ask for) the `account_code`.
2. `GetNextTaskTool(account_code: "<code>")` → note its `relative_id` and name; report them to the user.
3. `LogWorkProgressTool(account_code: "<code>", task_relative_id: <id>, description: "Read the task and planned the work", progress_percent: 10, duration_minutes: <minutes spent>)`.
4. Confirm the logged effort and the task's new state (`inprogress`).
