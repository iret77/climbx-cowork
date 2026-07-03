---
name: climbx-engage
description: Draft a reply to someone else's X (Twitter) post in the account owner's voice, ready to copy and post by hand. Use for "reply to this post", "help me respond to this tweet", "draft a reply", or the handoff "Draft a reply to this post (ClimbX engage): ...". Uses ClimbX suggest_reply, explains the credit and lock states in plain language, and never posts replies through the API.
---

# ClimbX engage

Find a post worth replying to, get a reply in the owner's voice, and post it yourself in seconds.
Replies cannot be published through the API, so the output is always copy-and-post-yourself.

First read `${CLAUDE_PLUGIN_ROOT}/shared/guardrails.md` (the suggest_reply rules and draft guards)
and `${CLAUDE_PLUGIN_ROOT}/shared/api-notes.md` (suggest_reply costs a credit, needs a read & write
key, can be locked).

## Input

- Pasted post text, plus the author's handle if known.
- Or the dashboard handoff `Draft a reply to this post (ClimbX engage): <opportunity JSON>`, where
  the post text and handle come from the Opportunity.

## Flow

1. **Credit reminder.** Once per session, before the first call, remind the user that each
   suggestion spends one shared daily AI credit and that it works best one post at a time (batch
   requests are discouraged). If the user lines up several at once, confirm they want to spend the
   credits.
2. **Call `suggest_reply`** with `text` (the post being replied to) and `author_handle` when known
   (a leading `@` is fine; the MCP strips it).
3. **Optional refinement.** If the user has their own voice or brand skills installed (detected by
   the core skill), run a single refinement pass over the suggestion: keep its substance, adjust
   tone only. Apply the draft guards (no em or en dashes, no filler). A reply may technically contain
   a link on X, but the suggestion should not need one.
4. **Output for manual posting.** Present a tight block:
   - the final reply text, ready to copy;
   - the `post_url` as a plain link to open the original post;
   - a closing line stating the user posts it on X themselves, because the API cannot publish
     replies.

## Edge cases (use the api-notes playbook)

- `locked` (403): reply drafting unlocks after the owner has written enough replies by hand in the
  ClimbX app. Say this in one friendly sentence and suggest writing a few by hand for now. Do not
  retry.
- `insufficient_credits` (402): the shared daily AI credit pool is empty and refills daily. Say so;
  do not retry.
- `read_only_key` (403): `suggest_reply` needs a read & write key; route to the setup skill to mint
  one.
