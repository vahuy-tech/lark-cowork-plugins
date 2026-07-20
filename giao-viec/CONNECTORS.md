# Connectors (Lark)

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that
category. In **Lark Cowork**, every category resolves to the single **`lark`** MCP server
(the `lark-cli mcp serve` bridge), which exposes a curated set of `lark_*` tools plus a generic
`lark_api` escape hatch for any Lark OpenAPI endpoint not covered by a dedicated tool.

There is one server backing every category — Claude picks the right `lark_*` tool for the task.
If an operation has no dedicated tool, use `lark_api` (pass the OpenAPI path + params).

## Connectors for this plugin

| Category | Placeholder | Lark product | Primary `lark_*` tools |
|----------|-------------|--------------|------------------------|
| Chat | `~~chat` | Lark IM | `lark_im_send`, `lark_im_card_send` |
| Task | `~~project tracker` | Lark Task | `lark_task_create`, `lark_task_my` |
| Directory | `~~directory` | Lark Contact | `lark_contact_search` |
| Meeting | `~~meeting` | Lark Minutes | `lark_minutes_search` |
| Calendar | `~~calendar` | Lark Calendar | `lark_calendar_agenda` |
| Knowledge base | `~~knowledge base` | Lark Base | `lark_base_search`, `lark_base_record_upsert` |

## Notes for Claude

- **Resolve people first.** Assigning a task needs an `open_id`. Use `lark_contact_search` to
  turn a name/email into an `open_id` before calling `lark_task_create`.
- **Tasks assigned to others** are NOT in `lark_task_my` — use the task list endpoint
  (`GET /open-apis/task/v2/tasks`) via `lark_api` to check delegated tasks.
- **Escape hatch.** Anything not covered above → `lark_api` with the Lark OpenAPI path.

## Lark depth — read these for any non-trivial workflow

This plugin runs on Lark. Before executing, consult the shared depth core (one level up):

- [`../connectors/LARK-PATTERNS.md`](../connectors/LARK-PATTERNS.md) — Lark-native workflow patterns (interactive cards, Base as system-of-record, Minutes→tasks, safe-mutation, token economy).
- [`../connectors/LARK-RECIPES.md`](../connectors/LARK-RECIPES.md) — exact `lark_api` calls for ops the curated tools don't cover.
- [`../connectors/LARK-FUSION.md`](../connectors/LARK-FUSION.md) — when to delegate to a deeper installed `lark-*` / `base-deploy` skill instead of hand-rolling.

Default posture: resolve people first, project reads with `jq`, preview mutations with `dry_run`, and surface decisions/digests as interactive cards rather than plain text.
