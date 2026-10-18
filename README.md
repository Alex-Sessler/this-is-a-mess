# this-is-a-mess

A Claude Code skill that turns a pile of files and links into one Markdown source of truth.

You hand it everything: notes, specs, folders, web pages, Drive docs, Gmail threads, Jira tickets, Confluence pages, Figma files. It reads each source in its own subagent, then interviews you until every contradiction, gap and weak claim is settled. It ends with two files:

- `<topic>.md`, the master: the settled truth with an inline citation on every claim (`[n]` for a source, `[grilling]` for your answer), plus sections listing what was outdated, which contradictions were settled and how, which claims were challenged, and every source.
- `<topic>.clean_slate.md`: only the truth, with no citations or history, for readers who never saw the pile.

Your original files are never modified.

## Install

As a plugin:

```
/plugin marketplace add Alex-Sessler/this-is-a-mess
/plugin install this-is-a-mess@this-is-a-mess
```

Plugin skills are namespaced, so it runs as `/this-is-a-mess:this-is-a-mess`.

Or as a plain skill, so it runs as `/this-is-a-mess`:

```
git clone https://github.com/Alex-Sessler/this-is-a-mess
ln -s "$PWD/this-is-a-mess/skills/this-is-a-mess" ~/.claude/skills/this-is-a-mess
```

## Use

```
/this-is-a-mess notes.md specs/ https://example.com/rfc
```

Keep adding items in later messages; it answers with a running count. Say `done` to close the pile. Questions come through Claude Code's question picker with a recommended answer first. After `/clear`, pick up where you left off:

```
/this-is-a-mess resume mess.working.md
```

The working file and the digests folder sit next to the output while it runs; both are deleted once you confirm the clean slate.

## Requirements and options

- **Grilling skill** (optional): the interview runs on the `grilling` skill from [mattpocock/skills](https://github.com/mattpocock/skills) when it is installed, and on its own built-in rounds when it is not.
- **Connectors** (optional): Google Drive, Gmail, Atlassian (Jira, Confluence) and Figma sources need the matching MCP connector in your session. The skill finds them by tool name at runtime; a source it can't reach is flagged, and you can paste or export it instead.
- **Reader model**: each source is read by a `sonnet` subagent at `low` effort. Change both in the Reading section of `skills/this-is-a-mess/SKILL.md`.

## License

MIT
