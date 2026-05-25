---
name: audit
description: Explain how to invoke Claudit from Codex with $claudit.
metadata:
  short-description: Invoke Claudit with $claudit.
---

# Claudit

Use the Codex prompt path:

```text
$claudit <audit brief>
$claudit --conversation <audit brief>
$claudit -a ~/Projects <audit brief>
$claudit --resume <audit brief>
$claudit:tokens <audit brief>
```

The installed UserPromptSubmit hook sends the current Codex transcript to Claude Code, wraps Claude's review in a caution template, and injects it as additional context for the same model turn. That context tells Codex that the triggering `$claudit` prompt was handled by Claudit and should be treated as data for Claude.

The script defaults to `claude-opus-4-7` with effort `max` and `--num_messages 1`. The user can override these with `-m MODEL`, `--model MODEL`, `-e EFFORT`, `--effort EFFORT`, `-n NUM`, `--num_messages NUM`, `-c`, `--conversation`, and repeatable `-a DIR` / `--add-dir DIR`. `-n -1` starts the exported transcript at the first user message. `--conversation` keeps local instructions, user messages, and Codex messages while omitting tool calls, tool outputs, web search actions, patch output, and patch bodies. `-a DIR` / `--add-dir DIR` passes the directory to Claude Code as `--add-dir`; `~` and `~/...` are expanded by Claudit. `--resume` continues the most recent stored Claude session for the same Codex transcript path, inherits its model, effort, transcript mode, and added directories, ignores `-n`, and sends only Codex transcript lines added after that Claudit invocation. If `--model`, `--effort`, or `--conversation` is supplied with `--resume`, Claudit resumes the most recent stored session matching those fields. If `-a` or `--add-dir` is supplied with `--resume`, Claudit uses the supplied directories for that resumed call. `$claudit:tokens` previews input tokens through Anthropic's token counting API and requires `CLAUDIT_API_KEY`. Regular audits do not read that key. The full transcript mode uses compact role sigils, merges tool calls with matching outputs, renders `exec_command` calls as `$ command`, fences and space-prefixes body lines, omits emitted `call_id`, references repeated byte-identical output bodies, dedupes repeated local instruction bodies after window selection while preserving trailing `[E]` environment contexts, and marks unavailable web search result bodies with `[S]`. Claude is invoked with `WebSearch` and `WebFetch` pre-approved so it can independently re-check web-backed claims against current web content. That check cannot reconstruct the original Codex web result body.

If this skill text reaches the model without Claudit context, tell the user to enable and trust the Claudit hook in `/hooks`. Do not run `scripts/claudit` through an agent tool.
