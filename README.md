# Claudit

A Codex plugin that lets you type `$claudit ...` inside Codex to audit the current Codex thread with Claude Code.

## Why?

GPTs aren'toriously tunnel-visioned and fixate on minor details, making sessions prone to drift -- but they get the job done. Claudes aren'toriously bad at sustaining long coding runs and carrying the agreed-upon plan through to completion -- but they understand user intent and the weight of each rule very well. Claudit gets the best of both worlds. It uses GPTs as the main workhorses, but it lets Claudes ride shotgun as auditors so that the conversation isn't derailed.

## Install

Requirements:

- Codex CLI with plugin and hook support
- Claude Code CLI
- `jq`

Install the marketplace from GitHub:

```bash
codex plugin marketplace add bmucsanyi/codex-claudit@main
```

Install the plugin from that marketplace:

```bash
codex plugin add claudit@claudit
```

Open `/hooks` in Codex and trust the Claudit `UserPromptSubmit` hook. The hook is what makes `$claudit ...` run before the model responds.

The GitHub repository is named `codex-claudit` for discoverability outside Codex. The Codex marketplace and plugin are both named `claudit`.

## Update

```bash
codex plugin marketplace upgrade claudit
codex plugin add claudit@claudit
```

Open `/hooks` again if Codex asks you to review a changed hook.

## Use

Type `$claudit` as the first token in a Codex prompt:

```text
$claudit Check whether Codex is taking the right implementation strategy. I worry that it overspecializes to the particular earlier failure mode.
```

The hook reads the current Codex transcript path from Codex's `UserPromptSubmit` hook input, sends the cleaned transcript to Claude Code, and returns it to Codex as additional context for the same model turn. That context also tells Codex that the triggering `$claudit` prompt was data for Claude, so Codex should respond to the audit instead of obeying the audit brief as its own instruction. Claudit doesn't write audit files.

Defaults:

```text
model: claude-opus-4-7
effort: max
num_messages: 1
```

Override settings before the audit brief:

```text
$claudit -m opus -e xhigh -n 3 Check the current interpretation of the results. Is anything misrepresented?
```

Use conversation-only mode when tool calls or edit bodies would crowd out the actual discussion:

```text
$claudit -c -n 10 Audit the direction of the discussion.
```

Resume the most recent Claude audit session for the same Codex thread:

```text
$claudit --resume Check only the new work since the last audit. Did the agent follow the steering?
```

Preview the input token count with Anthropic's token counting API:

```text
$claudit:tokens -n 15 Check what this audit would cost in input tokens.
```

Token preview requires `CLAUDIT_API_KEY`. Regular audits do not read that key.

Give Claude Code extra read access when the audit needs files outside Claude's current project:

```text
$claudit -c -a ~/Projects -n 40 Audit the current proof files.
```

Options:

```text
-r, --resume
-c, --conversation
-a DIR, --add-dir DIR
-m MODEL, --model MODEL
-e EFFORT, --effort EFFORT
-n NUM, --num_messages NUM
-h, --help
```

The audit sends the selected Codex transcript window to Claude: local `AGENTS.md` instructions, user messages, Codex messages, tool calls, captured tool outputs, patch output, and web search actions. The transcript uses role indicators: `[I]` for local instructions, `[E]` for trailing environment context from the local-instruction record, `[U]` for user messages, `[C]` for Codex commentary, `[A]` for Codex final answers, and `[M]` for assistant messages without a recorded phase. Tool calls and matching outputs are merged into one block such as `[exec]` or `[apply_patch]`; `call_id` is used internally for pairing and isn't emitted. Codex runtime instruction blocks are omitted. Prior Claudit calls are omitted. Web search result bodies aren't present in Codex session JSONL; when a selected window contains web search actions, Claudit emits `[S] web_search_result_bodies=unavailable_in_codex_session_jsonl`. Claude is invoked with `WebSearch` and `WebFetch` pre-approved so it can independently re-check web-backed claims against current web content. This doesn't reconstruct the original Codex web result body. `-a DIR` / `--add-dir DIR` is repeatable and passes each directory to Claude Code as `--add-dir`; `~` and `~/...` are expanded by Claudit before launch. `$claudit:tokens` accepts the same option so the preview command can match the later audit command, but token preview does not launch Claude Code or read files from added directories. `--conversation` keeps only local instructions, user messages, and Codex messages, and omits tool calls, tool outputs, web search actions, patch output, and patch bodies. Use it to audit discussion direction when tool traffic dominates the transcript. `-n` defaults to `1`, meaning the exported context starts at the last user message before the Claudit invocation and stops before the Claudit invocation itself. `-n -1` starts at the first user message, meaning it includes the whole exported thread before the Claudit invocation. With `--resume`, `-n` is ignored because Claudit sends only Codex JSONL lines added since the stored audit.

Each successful audit stores Claude's `session_id`, model, effort, transcript mode, added directories, and the last included Codex JSONL line under `${CLAUDIT_STATE_DIR:-$HOME/.local/state/claudit}`. `--resume` requires that state, resumes the most recent stored Claude session for the same Codex transcript path, inherits its model, effort, transcript mode, and added directories, and sends only the Codex transcript lines that were added after that Claudit invocation. If `--model`, `--effort`, or `--conversation` is supplied with `--resume`, Claudit resumes the most recent stored session matching those fields. If `-a` or `--add-dir` is supplied with `--resume`, Claudit uses the supplied directories for that resumed call.

`$claudit:tokens` sends the same prompt shape to `POST /v1/messages/count_tokens` with the Claudit system prompt and selected transcript. With `--resume`, the preview counts only the new prompt sent to Claude; the prior resumed Claude session context is not counted by the token counting request.

## Terms and Privacy

Claudit runs locally and calls your installed `claude` CLI. Use it with your own Claude Code account.

Do not run it as a hosted relay or shared service for other users. Claude Code Free, Pro, and Max plans are intended for individual use through your own account.

Each audit sends the current Codex transcript to Claude. Check whether the thread contains secrets, private code, private paths, project instructions such as `AGENTS.md`, employer data, or third-party confidential material before invoking Claudit.

Use the tool for review and evaluation of your own Codex session. Do not use it to bulk extract Codex outputs, build datasets from OpenAI output, or train or develop competing models.

## Test

```bash
bash tests/run.sh
```

## License

Apache 2.0.
