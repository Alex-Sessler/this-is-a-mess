---
name: this-is-a-mess
description: "Turn a pile of files and links into one Markdown source of truth: collect until you say done, read each source in a subagent, grill until every conflict, gap and weak claim is settled, then write it with inline citations."
argument-hint: "[files, folders or links to start the pile] | resume <working-file>"
disable-model-invocation: true
---

# This is a mess

Turn a **pile** (files, folders, links, connector items) into one Markdown **source of truth**. Four phases: intake, reading (runs alongside intake), grilling, output. You work from **digests** only: each source is read by its own subagent, and the raw pile stays out of your context.

Every file this skill writes goes in the current folder. Originals stay read-only.

## Working file

Create `mess.working.md` in the current folder on the first message. It is the resume point: update it at the end of every step and every grilling round. Sections:

- **Phase**: intake | grilling | outline | writing | clean-slate.
- **Sources**: one line per source, `[n] <path or URL> · <kind> · <status> · <digest path>`. Status is reading, digested, unreadable, or dropped-outdated. Numbers are permanent; they become the citations.
- **Settled**: every decision, one line each: the question, the answer, the round, the sources it touches. Outdated drops go here with their evidence.
- **Open**: the current frontier.

Once the topic is settled (end of grilling) and every reader has finished, rename it to `<topic>.working.md` and `mess.digests/` to `<topic>.digests/`, and fix the digest paths inside. Renaming while a reader is still writing loses its digest.

## Resume

`/this-is-a-mess resume <path>`: read the working file and check each digest path exists. Start a reader again for any source still `reading` without a digest. Continue at the recorded phase, with Open as the frontier.

## 1. Intake

The arguments are the first items; every later message adds more until the user says **done**. Answer each addition with the running count only ("7 in the pile"), plus one line for any flag or candidate list below. Start the reader for each item the moment it arrives.

Source kinds:

- **Local file**: by path. `.env*` and credential files (keys, tokens, `credentials*`, `*.pem`) stay unread even when named: flag them and offer a redacted paste.
- **Local folder**: list readable files recursively, skipping binaries, `.git`, `node_modules`, `.env*` and credential files. Show the count with a top-level breakdown and ask before starting readers; the user may narrow it with a glob. Each file becomes its own source.
- **Web link**: WebFetch.
- **Google Drive, Gmail, Jira, Confluence, Figma**: through a connector. Discover connectors at runtime by tool-name pattern, matching case-insensitively over the available and deferred tool names: `drive`, `gmail`, `atlassian|jira|confluence`, `figma`. Names differ between connector versions, so look every time. A Google Docs or Sheets link is a Drive source.

A source is **unreadable** when no connector matches or a login wall blocks it. Flag it at intake: name the connector it needs and where one would be (another session, project or workspace that has it configured), and offer a paste or export instead. A link or file that is gone (404, missing path) is not unreadable: it is hard outdated evidence for grilling.

**Jira and Confluence**: the reader takes the item itself (ticket: description and comments; page: body and comments) and lists linked issues, epic children and child pages. Show those as candidates the user can add in one go ("add all 4", or pick). Follow them only when the user adds them.

Done when the user has said done and every source is digested, flagged unreadable, or replaced by a paste.

## 2. Reading

One background subagent per source, started at once: Agent tool, `subagent_type: general-purpose`, `model: sonnet`, `effort: low`. Its prompt gives the source pointer, its number, the digest path `mess.digests/<n>-<slug>.md`, and the absolute path of `digest-brief.md` in this skill's base directory, which it follows. The brief owns the digest format.

When a digest lands, read it, mark the source digested, and surface any candidates it lists.

## 3. Grilling

Invoke the `grilling` skill and run its design-tree rounds over the digests, with these overrides. If it is not installed, run the rounds as described here.

- Every round goes through AskUserQuestion: at most 4 questions per call, a larger frontier split over consecutive calls, the recommended option first and labelled "(Recommended)".
- Facts are yours. When a check settles a question (is the link alive, what is the ticket's status, does the sum add up, which version is newer), run it, through a subagent when it means reading a source. The user only gets decisions.

Work the frontier in this order. A later class waits until the earlier ones are empty; a clash found along the way joins the next round regardless.

1. **Outdated.** Drop without asking on hard evidence: the source says it is superseded, the ticket is closed as won't-do, a newer source explicitly replaces it, or the link or file is gone. Record claim, source, evidence and replacement in Settled. Age alone is a question, not evidence: quote the claim, its date, and any newer source covering the same ground.
2. **Contradictions**, every kind:
   - inside one source (flagged in its digest);
   - between sources (you find these by comparing digests);
   - a user answer against a source, or against an earlier answer;
   - implied: numbers that don't add up, a deadline before its dependency, design against spec.

   Put each as both statements quoted exactly with their sources, then the likely cause (outdated / different scope / loose wording / real disagreement), then your recommendation. For an implied one, show the reasoning chain step by step.
3. **Gaps**: what the topic needs that no source settles, such as owners, dates, definitions, and edge cases the sources gesture at.
4. **Challenges**: weak or wrong claims, from the digests' weak-claim lists and your own reading. Challenge the content: say why it is doubtful and what would make it stronger, so it is tested before it goes in.

After every round, check each new answer against the sources and against earlier answers; a clash becomes a contradiction in the next round. Write the round to the working file.

Done when the frontier is empty in every class and the last round's answers produced no new clash. Then settle the topic as a short kebab-case slug, rename the working file and digests folder, and show an outline of `<topic>.md`: each section heading with a one-line summary. Write only after the user approves the outline.

## 4. Output

### `<topic>.md`, the master

It holds everything. The body is the truth, shaped to fit this pile (a spec reads like a spec, a plan like a plan), with an inline citation after every claim: `[n]` for source n, `[grilling]` for a user answer, both when both apply. Then these fixed sections, in order:

- **Outdated**: claim, source, why it is outdated, what replaced it.
- **Contradictions**: both statements, their sources, how it was settled, and why.
- **Challenged claims**: the rejected ones and the ones that stood, each with the reason and `[grilling]`.
- **Sources**: every input as `[n]`, including unreadable and dropped ones, and what it contributed.

### `<topic>.clean_slate.md`

Only the one truth, for a reader who never saw the pile: the master's body without citations, and nothing else. It has no link to the master and no decisions, outdated items or contradictions.

Write the master, draft the clean slate from it, and show the clean slate in full. On the user's confirmation, write it, then delete the working file and the digests folder. If they ask for changes, revise and show it again.

Done when both files exist and the working file and digests folder are gone.
