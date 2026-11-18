# NjectAI tools reference

Every tool of the NjectAI MCP server (`https://api.nject.ai/api/mcp`), with its arguments, results and limits.

## Contents

- [Shared rules](#shared-rules)
- [start_context_session](#start_context_session)
- [list_context_profiles](#list_context_profiles)
- [get_context_profile](#get_context_profile)
- [select_context_profile](#select_context_profile)
- [get_inject_context](#get_inject_context)
- [search_memory](#search_memory)
- [enhance](#enhance)
- [record_episodic_memory](#record_episodic_memory)
- [get_memory_capture_settings](#get_memory_capture_settings)
- [set_memory_capture_settings](#set_memory_capture_settings)
- [show_profile_dashboard](#show_profile_dashboard)
- [Error format](#error-format)

## Shared rules

**Session fields.** Several tools return the session state:

| Field | Meaning |
| --- | --- |
| `session_id` | UUID of this conversation's context session |
| `selected_profile_id` | The selected profile, or `null` for personal memory |
| `selection_version` | Whole number that goes up with every change of selection |
| `context_policy` | `profile_only`, `profile_plus_personal` or `personal`. `null` while nothing is selected |
| `state` | `active` or `awaiting_selection` |
| `expires_at` | Sessions last 30 days from their last use |

**`session_id` and `expected_version` travel together on the versioned tools.** These are `get_inject_context`, `enhance`, `search_memory` and `select_context_profile`. There, a call with `session_id` and no `expected_version` fails with `VALIDATION_ERROR`, and so does `expected_version` alone. A stale `expected_version` fails with `SELECTION_CONFLICT`. `list_context_profiles`, `show_profile_dashboard` and `record_episodic_memory` take `session_id` alone.

**Two different `client_request_id` arguments.** On `start_context_session` it identifies the conversation's session: one value per conversation, sent again to get the same session back. On `get_inject_context` and `enhance` it is an optional retry key for that single request. Never send the session's value there. Leaving it out is the simple choice.

**Profile IDs** are numeric strings such as `"42"`. Take them from `list_context_profiles`. They are internal: never show them to the user.

**Context policies.**

| Policy | Reads | Valid with |
| --- | --- | --- |
| `profile_only` | The profile, and personal history tagged to it | A profile. The default when a profile is selected |
| `profile_plus_personal` | The profile and the account's personal memory | A profile |
| `personal` | Personal memory | No profile (`profile_id: null`) |

A policy that does not match its profile setting fails with `VALIDATION_ERROR`.

**One-call overrides.** On `get_inject_context`, `enhance` and `search_memory`, a `profile_id` argument applies to that call alone. The session and the account default stay as they were. The result reports `context_status: "override"`.

**`context_status` values.** `session`: the session's selection was used. `override`: a one-call `profile_id` or `context_policy` was used. `legacy`: the call had no session. `replay`: a repeated `client_request_id` returned its original binding.

**Scopes.** The connector grants `mcp` (connect and read) and `mcp:write` (tools that store data or record which profile a reply used). A call without the needed scope fails with `INSUFFICIENT_SCOPE`.

**Plan.** `get_inject_context`, `search_memory`, `enhance` and `record_episodic_memory` need an active NjectAI plan. A trial counts. The other tools work on any connected account.

## start_context_session

Opens the conversation's context session, or returns the existing one. Call it once, when the conversation first needs NjectAI.

| Argument | Required | Notes |
| --- | --- | --- |
| `client_request_id` | Yes | 8 to 128 characters from `A-Z a-z 0-9 . _ : -`. Generate one per conversation (a random UUID). The same value returns the same session |
| `profile_id` | No | Leave out to start from the account's selected profile. `null` starts on personal memory |
| `context_policy` | No | See the policies above |

Returns the session fields plus `warnings`.

How the starting profile is chosen when you leave `profile_id` out:

1. The account's default profile.
2. With no default: the profile from the user's last conversation. A warning says so.
3. With neither: the user's only usable profile, if they have exactly one. A warning says so.
4. Otherwise the session starts in `awaiting_selection`.

If the account's default can no longer be used (deleted, or waiting for the plan to start), the session starts in `awaiting_selection` and a warning explains why.

The session belongs to the account, the connected client and the `client_request_id`. Sessions of other conversations and other clients are never changed by this one.

## list_context_profiles

Lists the user's profiles. Use it to learn names, to let the user choose, and to see what is selected.

| Argument | Required | Notes |
| --- | --- | --- |
| `session_id` | No | Adds this conversation's session state to the result |
| `limit` | No | 1 to 50, default 20 |
| `cursor` | No | `next_cursor` from the previous page |

Returns:

- `profiles[]`, each with:
  - `profile_id`, `name` (may be `null`), `type` (`text`, `image` or `search`), `label` (the kind in words)
  - `status`: `active` or `pending_payment`
  - `selectable` and `reason`: whether the profile can be used now, and why when it cannot
  - `account_default`: `true` on the account's default profile
  - `template_key`: set on ready-made template profiles
  - `attachments` and `links`: counts as `total`, `active`, `pending_payment`
- `account_default_profile_id`
- `session` when you passed `session_id`
- `next_cursor`, or `null` on the last page

An empty list means the user has no profile yet.

## get_context_profile

Shows one profile in detail. Use it when the user wants to preview a profile, or when you need to tell two similar profiles apart.

| Argument | Required | Notes |
| --- | --- | --- |
| `profile_id` | Yes | From `list_context_profiles` |

Returns:

- `profile`: `profile_id`, `name`, `type`, `label`, `status`, `template_key`, `answers` (the interview answers by label), `created_at`, `updated_at`
- `context_preview`: the answers as readable text
- `attachments[]`: `file_name`, `description`, `summary`, `status` (and an internal `id`)
- `links[]`: `url`, `type`, `title`, `domain`, `description`, `status` (and an internal `id`)
- `readiness`: `selectable`, `reason`, and the same source counts as the list

Files and links come back as summaries. The full text reaches you through `get_inject_context` and `search_memory`.

## select_context_profile

Switches the conversation's profile. It affects later requests. Earlier replies keep the profile they were written with.

| Argument | Required | Notes |
| --- | --- | --- |
| `session_id` | Yes | |
| `profile_id` | Yes | A profile ID, or `null` for personal memory |
| `expected_version` | Yes | Your latest `selection_version` |
| `context_policy` | No | With a profile, each switch goes back to `profile_only` unless you ask for `profile_plus_personal`. With `null`, the policy is `personal` |
| `update_account_default` | No | Default `true`: the selection also becomes the account's selection for new conversations and for the NjectAI browser extension. Pass `false` to change this conversation alone |

Returns the session fields, with `selection_version` one higher and `state: "active"`, plus `account_default_profile_id`.

With `profile_id: null` and the default `update_account_default`, the account ends up with no selected profile at all. Pass `false` when the user wants personal memory for this conversation only.

## get_inject_context

Returns the saved context that fits a request: profile answers, summaries, knowledge, memory, links and files. It reads and returns data. It does not rewrite or run the request.

| Argument | Required | Notes |
| --- | --- | --- |
| `input` | Yes | 1 to 10,000 characters. The request the context is for |
| `session_id` | No | Use the conversation's selection |
| `expected_version` | With `session_id` | Your latest `selection_version` |
| `profile_id` | No | One-call override. `null` reads personal memory |
| `context_policy` | No | One-call policy |
| `max_chars` | No | 500 to 16,000, default 12,000. The size of the context you get back |
| `client_request_id` | No | See the note below |

Returns:

- `context`: the text to use as reference data. An empty string means nothing saved matched
- `sources[]`: what went into it. Each has `type`, `scope` (`personal` or `profile`), `title`, `truncated`, and sometimes `url` or an internal `id`. Types: `profile_answers`, `profile_summary`, `profile_knowledge`, `profile_memory`, `personal_summary`, `personal_memory`, `link`, `file`
- `truncated`: `true` when the context was cut to fit `max_chars`
- `warnings[]`: a source was cut, a source was temporarily unavailable, or nothing was found
- Attribution: `resolved_profile_id`, `context_policy`, `session_id`, `selection_version`, `turn_id`, `context_status`

Without `session_id` and `profile_id`, the call uses the account's selected profile.

**About `client_request_id` on this tool and on `enhance`.** It is an optional retry key: a retry with the same value returns the original profile binding (`context_status: "replay"`). If you use it, generate a new value for each request, different from the session's. Reusing a value for a different request fails with `VALIDATION_ERROR`. Leaving it out is fine.

## search_memory

Searches saved memory for a query.

| Argument | Required | Notes |
| --- | --- | --- |
| `query` | Yes | 1 to 2,000 characters |
| `limit` | No | 1 to 20, default 5 |
| `session_id` | No | Search the memory of the conversation's selected profile |
| `expected_version` | With `session_id` | Your latest `selection_version` |
| `profile_id` | No | One-call override: that profile's memory, including its uploaded files. `null` searches personal memory |

Returns `memories[]`, each with `content`, `type` and `created_at` (may be `null`), plus `resolved_profile_id`, `session_id`, `selection_version` and `context_status`.

Scope rules:

- With `session_id` and `expected_version`: the selected profile's memory. If the session is on personal memory, personal memory.
- With `profile_id: null` added: personal memory. Chats captured from the user's AI tools are stored here.
- With neither `session_id` nor `profile_id`: personal memory.

Notes on results:

- The tool has no date filter and no source filter. Put a time frame in the query when it helps, and read `created_at` yourself.
- `type` labels the kind of memory. `episodic` is something said or saved at a point in time. `semantic` is a standing fact, such as a profile's interview answers.
- A result does not say which AI tool a chat came from.
- The default of 5 results suits a precise lookup. Ask for 10 or more when the user wants to recall a topic.

## enhance

Rewrites a draft prompt into a clearer, more specific one, using the profile's context. The result is a suggestion. Nothing is sent anywhere.

| Argument | Required | Notes |
| --- | --- | --- |
| `text` | Yes | 1 to 3,000 characters. The draft, as the user wrote it |
| `session_id` | No | Use the conversation's selection |
| `expected_version` | With `session_id` | Your latest `selection_version` |
| `profile_id` | No | One-call override. `null` uses personal memory |
| `context_policy` | No | One-call policy |
| `client_request_id` | No | For safe retries, as on `get_inject_context` |

Returns `suggestion` (the rewritten prompt), the attribution fields and `warnings[]`.

## record_episodic_memory

Queues a note for the user's long-term memory. Call it only for content the user explicitly asked to save.

| Argument | Required | Notes |
| --- | --- | --- |
| `events` | Yes | 1 to 20 items |
| `events[].role` | Yes | `user` or `assistant`: who originally wrote the content |
| `events[].content` | Yes | The chosen note or excerpt. Up to 10,000 characters for `user`, 50,000 for `assistant` |
| `events[].conversation_id` | No | The host's conversation ID, when you know it |
| `events[].model` | No | The model's name |
| `session_id` | No | Tags the note with the session's selected profile, and adds a `user` note to that profile's own memory. Without it, the note goes to personal memory alone |
| `mode` | Pass it | Always `explicit` for a save the user asked for |

This tool takes `session_id` without `expected_version`, and it has no profile argument: the profile always comes from the session.

One call carries at most 300,000 characters across its events. Split a larger save into several calls.

Returns:

- `results[]`: one per event, with `role` and `status`
- `queued_count`
- `profile_id`: the profile the notes were tagged with, or `null`
- `capture_paused`: `true` only when an `automatic` call was skipped. Explicit saves ignore the pause, so expect `false`

Statuses:

| Status | Meaning |
| --- | --- |
| `queued` | Accepted. It becomes searchable shortly |
| `partial` | Only part of the text was accepted. Tell the user, and offer to send it again in smaller pieces |
| `duplicate` | The same text was saved in the last few minutes |
| `dropped` | Memory could not take it right now. When every event is dropped, the call fails with `UPSTREAM_UNAVAILABLE` |
| `paused` | Skipped because automatic recording is paused. This applies to `automatic` mode only: explicit saves go through while paused |

The `automatic` mode exists for clients with their own, separately authorized capture. A model never uses it.

A session that is still `awaiting_selection` saves to personal memory.

## get_memory_capture_settings

Takes no arguments. Returns `enabled` and `paused_at`: whether automatic recording by connected clients is on for this account.

## set_memory_capture_settings

| Argument | Required | Notes |
| --- | --- | --- |
| `enabled` | Yes | `false` pauses automatic recording, `true` resumes it |

Returns `enabled` and `paused_at`. Call it only when the user asks. Saves the user requests still work while recording is paused. The setting gives no access to other chats.

## show_profile_dashboard

Shows the user's NjectAI dashboard. Clients that support interactive tool results render a card where the user can switch the conversation's profile and pause recording. Other clients return the same data as text and JSON.

| Argument | Required | Notes |
| --- | --- | --- |
| `session_id` | No | Shows this conversation's selection |
| `profile_ids` | No | 1 to 50 IDs, to show only those profiles |

Returns:

- `profiles[]` and `account_default_profile_id`, as in `list_context_profiles`
- `session` when you passed `session_id`
- `access`: `can_use_features`, `access_source` and `expires_at`. Sources `subscription` and `trial` are what they say, `none` means no active plan, and any other value means access is active on the account. `expires_at` is set when access has an end date
- `capture_summary`: `enabled` and `paused_at`

A change the user makes inside the card raises the session's `selection_version`. Your next call may return `SELECTION_CONFLICT`: adopt the state it carries.

## Error format

A failed call has `isError: true`, a readable message, and a JSON block:

```json
{
  "error": {
    "code": "SELECTION_CONFLICT",
    "message": "The session selection changed (now version 3). Re-read the current state and retry with the new expected_version.",
    "current_state": {
      "session_id": "...",
      "selected_profile_id": "42",
      "selection_version": 3,
      "context_policy": "profile_only",
      "state": "active",
      "expires_at": "..."
    },
    "retryable": false,
    "request_id": "..."
  }
}
```

- `code`: one of `AUTH_REQUIRED`, `INSUFFICIENT_SCOPE`, `ACCESS_REQUIRED`, `PROFILE_NOT_FOUND`, `PROFILE_UNAVAILABLE`, `SESSION_NOT_FOUND`, `SELECTION_CONFLICT`, `SELECTION_REQUIRED`, `NOT_FOUND`, `VALIDATION_ERROR`, `RATE_LIMITED`, `UPSTREAM_UNAVAILABLE`, `INTERNAL_ERROR`
- `retryable`: `true` for `RATE_LIMITED` and `UPSTREAM_UNAVAILABLE`
- `retry_after_ms`: present on `RATE_LIMITED`
- `current_state`: present on `SELECTION_CONFLICT` and `SELECTION_REQUIRED`
- `request_id`: identifies the call for NjectAI support

Arguments that break a tool's schema (a value out of range, a text over its size limit) are refused before the tool runs. That error is a plain "Invalid arguments" message naming the field, without this JSON object.

`troubleshooting.md` says what to do for each code.
