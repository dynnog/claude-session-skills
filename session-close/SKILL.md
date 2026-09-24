---
name: "session-close"
description: "Run when wrapping up to log a timestamped, topic-tagged status and update that topic's context file, so any chat in the same Project can pick it up later. Project-only."
---

# Session Close

Does two jobs at the end of a working session:
1. Appends a short, timestamped, topic-tagged status (Did / Next) to the session log.
2. Creates or updates that topic's context file with the substantive details from this session, so other chats in the Project can load them.

## Core principles

- **Project-only.** Everything this skill writes goes inside the current Project's own memory subtree and is not visible outside the Project. If this chat is NOT inside a Project, stop and tell the user: "session-close only works inside a Project." Do not fall back to account-level memory.
- **Standalone log entries.** One self-contained entry per call; not paired with any open. `Duration` is opportunistic, never load-bearing.
- **Topic-tagged.** Every log entry and context file is keyed by topic so unrelated work never mixes.
- **Scoped to this working session.** Summarize only the stretch since the most recent `session-open` in this chat; if none, since the last `session-close` in this chat; if neither, the whole chat. Never re-summarize what an earlier close already captured. Whole-chat compression is `/compact`'s job.
- **Only what was actually said.** Never invent details for either file.

## Files (all inside the Project subtree)

- Session log: `<project-subtree>/session-log.md`
- Context file: `<project-subtree>/context/<topic-slug>.md` - topic in lowercase kebab-case, e.g. `Client Research` -> `client-research`.

## When to use

- The user types `/session-close`, or says they're wrapping up / done for now.
- Optional inline note: `/session-close waiting on callback`. If given, use it verbatim as the status.
- Also appropriate as the final step of a longer workflow (e.g. the completion of a research or planning phase), while the chat's content is still fresh.

## Steps

1. **Confirm Project scope.** If there is no Project memory subtree for this chat, stop as described above.

2. **Get the current time.** Call the `current_time` tool. Never guess or read it from message metadata.

3. **Determine the topic.** Reuse the topic a `session-open` set earlier in this chat, exactly. Otherwise infer a short label from what this chat has actually been about (safe here, because the content is in front of you). If the chat spans two unrelated topics, use the dominant one and say which in the confirmation.

4. **Write the status.** Two terse lines for this working session only: `Did:` and `Next:`. Use the inline note instead if one was given. If there is genuinely nothing to summarize, ask one short question rather than inventing content.

5. **Append to the session log.** Create the log with a short frontmatter block if missing. Duration: if the entry directly above is an `OPEN` for the same topic, compute elapsed time from it; otherwise `n/a`. Append (one blank line before it):
   ```
   ## <YYYY-MM-DD HH:MM TZ> - CLOSE
   Topic: <topic>
   Duration: <elapsed or "n/a">
   Did: <what got done>
   Next: <what's next>
   ```

6. **Create or update the topic's context file.** Read it first if it exists, then rewrite it as one merged, current version (do not keep appending duplicates). Structure:
   ```
   ---
   name: <topic-slug>
   description: Working context for <topic> - key decisions, details, people, and open questions
   sources: [chat]
   aliases: [<topic>]
   ---
   Last updated: <YYYY-MM-DD HH:MM TZ>

   ## Summary
   - <one or two lines: what this topic is and where it stands>

   ## Decisions
   - <choices made, e.g. which approach to use, which stakeholder to consult>

   ## Key details
   - <important facts: research findings, requirements, constraints, deadlines>

   ## People
   - <name and responsibility only, and how they fit the work, e.g. "Dana - project lead, reviewed the proposal">

   ## Discussion notes
   - <questions raised, what was resolved, what to improve>

   ## Open questions
   - <unresolved items to follow up on>
   ```
   Rules:
   - Only add what was actually said or decided in this chat. Omit empty sections rather than filling them with guesses.
   - Update outdated lines in place (e.g. "waiting on stakeholder feedback" -> "feedback received Sep 24"). Keep the file tight - a quick-reference sheet, not a transcript. Aim for under ~3 KB.
   - If this session added nothing substantive beyond the status line, skip this step.
   - Never save salary, compensation, or other private financial figures, credentials, government ID or account numbers, or health details. Leave those parts out entirely, no placeholder.

7. **Confirm.** Show what you logged, including the topic, and note whether the context file was updated:
   > Logged - client research, Tue Sep 22, 3:10 PM PT.
   > Did: ...
   > Next: ...
   > Context file updated (decisions, key findings).
   Keep it short. If the user corrects anything, fix that entry or file with the memory edit tool.

## Notes

- The log is append-only; the context file is one living document per topic.
- Always tag a topic. An untagged entry cannot be matched later.
