# CLAUDE.md

> Behavioral guidelines to reduce common LLM coding mistakes.
> Main framework: Based on [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) CLAUDE.md, which compiles Andrej Karpathy's observations on LLM coding failure modes.
> Below that are environment-specific rules for Windows + Claude Code + worktrees/WSL Android.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

# Personal Environment Rules

## AS-IS Codebase Principle

**Change is tracked by version control, not by the codebase. The codebase always represents only the current state, and must never unnecessarily retain content from which a previous state could be inferred.**

- Every file represents only the current AS-IS state — what the system actually is/does right now.
- Do NOT narrate history or changes in files: no "X → Y", "fixed bug", "was X, now Y", "removed", "previously", "phase 1/2", before/after deltas, or arrows.
- Applies to ALL assets: code, comments, docstrings, docs/, READMEs, config comments, demo/UI pages, task-state files.
- Comments explain *why current code is the way it is*, never what changed. Git history (commits, diffs) is the sole record of change.
- Docs/UI state current numbers and behavior as facts, not deltas. Show only implemented behavior — no aspirational/unimplemented/design-only claims.
- **Exception:** files explicitly dedicated to history (e.g. an append-only session/change log, CHANGELOG.md) are allowed.

> The Karpathy 4 principles above are primary. Rules below are environment-specific and must not conflict with them. If conflict exists, Karpathy principles take precedence — except **data loss prevention** rules (worktree junctions, etc.) which always have absolute priority.

## General Work Principles

### Language for Thinking vs Output
- **Reasoning (thinking):** English, at full length — never compress reasoning; thoroughness drives accuracy on hard tasks (debugging, review, TDD).
- **User-facing replies:** clear Korean (명료체) — strip pleasantries and filler but keep orthographic correctness and readability.
- Terse caveman-style internal narration (tool notes, subagent prompts, progress lines) is part of **Maestro** (the opt-in routing harness) and applies only while Maestro is enabled (see Plugin Routing). The `/caveman` skill remains for explicit on-demand use.

### No Em-Dash or En-Dash
Never use the em-dash (`—`) or en-dash (`–`) in any output: user-facing replies, code, comments, docstrings, docs, commit messages, PR bodies, artifacts. Use a comma, colon, parentheses, or a separate sentence instead; for ranges use "to" or a plain hyphen. Applies everywhere, all projects. (The plain hyphen `-` is fine.)

### When User Instructions Conflict with My Knowledge
If user instructions are very clear but contradict what I know or seem incorrect, first search the web for relevant resources and either proceed based on findings or persuade the user with evidence.

### Response Style
Don't just answer the surface of a question. Understand why the user asked, what they might be confused about, and what background knowledge they need to understand the next context. Respond accordingly.
(Korean reinforcement of Karpathy #1 "Think Before Coding" — applies to all responses, not just coding)

### Confirm Understanding Before Acting
When user gives a task, don't execute immediately. Always:
1. Explain how I understood the task
2. Get user confirmation that understanding is correct
3. Act only after confirmation

Misunderstanding and executing can be fatal for hard-to-reverse operations (`pm clear`, rebuilds, re-embeddings, etc.).

### Problem-Solving Principle
When problems occur, don't use workarounds — find and fix the root cause. No "it works for now" temporary fixes. If root cause is hard to identify, explain to user and decide together.

### Learning from Mistakes
Record every mistake in auto memory. Document the mistake and correct behavior to prevent repetition.

---

## File Management Principles

### Timestamp-Based Identification
Include timestamp in all generated file names for distinction. When searching/selecting files:
1. **Check filename timestamp** (e.g., `_20260324_1022`)
2. **Check file metadata** (`ls -la` for creation/modification time)
3. **Check git commit time** (`git log --format="%ai"`)
4. If multiple candidates exist, select most recent + matching current settings
5. **If uncertain, create new** — don't guess with stale files

### Version Identifier Format
When versioning files or code snippets, use `YYYYMMDD.HHMM.N` or `YYYYMMDD-HHMM-N` format (e.g., `20260325.0931.0`, `20260325-0931-0`). Date + time + sequence number.

### No In-Place Modification
Unless user explicitly says "modify that file", never directly modify original files. Always create copies and preserve originals. Especially applies to DB files, config files, data files.

### File Creation Location
Don't create files in project root or arbitrary locations. Always create inside appropriately named directories. Temp files go in `./tmp/`, script outputs in that script's `output/` directory, etc.

### Visualization in Non-CLI Outputs
Outside the plain CLI/terminal, NEVER hand-draw diagrams or charts as ASCII art. Use the proper tool for the medium and make it actually render (not raw code):
- **Diagrams** (flow, architecture, sequence, ER, state, etc.): Mermaid — invoke the `mermaid` skill or embed mermaid.js so it renders as graphics. In HTML docs, include the Mermaid runtime and `<pre class="mermaid">` blocks.
- **Data/charts**: a real charting approach (SVG, a chart lib), not ASCII bars/tables-as-graphs.
- **Images/screenshots/photos**: embed the actual asset.

ASCII diagrams are acceptable ONLY in plain CLI/terminal text or code comments where no renderer exists. The moment output is HTML, a report, a web page, or any rendered surface, switch to a real visualization tool.

### Artifact / HTML Authoring (ABSOLUTE — every project, everywhere)
- **NEVER cap text width in the name of "readability."** Do not add `max-width` / reading-measure (`65ch`, `72ch`, `56ch`, etc.) to `p`, `li`, `.lede`, or any text element on your own initiative. Text MUST fill the container width. This is absolute; it overrides any design-guide advice about ideal line length (incl. the artifact-design skill's "~65ch running text"). If width control is genuinely required, ask the user first.
- **Korean text:** set `word-break: keep-all` so lines break at word (eojeol) boundaries, never mid-syllable.
- **NEVER guess-and-ship visual/layout fixes.** When a rendering bug is reported (width, spacing, line breaks), verify the actual render before claiming a cause is fixed — render/screenshot it, or ask the user to point at the exact element. One verified fix beats repeated blind guesses; repeated "this time it's the real cause" claims that are wrong destroy trust.

---

## Worktrees (Windows Environment)

### Handling Gitignored Files After Worktree Creation
After creating a worktree, handle gitignored files from the original appropriately:
- **Symlink**: `.env`, `.env.*`, certificates/keys, and other read-only config files
- **Fresh install**: `node_modules/`, virtual environments — reinstall via package manager (npm install, pip install, etc. — handled in existing skill's Project Setup step)
- **Symlink (data)**: Test fixtures, seed data, large assets — read-only data files
- **Copy (data)**: SQLite DB, local state files — data files that may be modified in worktree
- **Ignore**: Build outputs (`dist/`, `build/`), caches, lock files — generated per-branch

### Safe Junction Removal Before Worktree Deletion (CRITICAL)
Before deleting a worktree directory, **always** unlink all junction links via PowerShell first. `rm -rf` and `git worktree remove` follow Windows junctions and delete original data.

```bash
WPATH_WIN=$(cygpath -w "<worktree-path>")
powershell.exe -NoProfile -Command "
  Get-ChildItem -Path '$WPATH_WIN' -Recurse -Directory -Force -ErrorAction SilentlyContinue |
    Where-Object { \$_.Attributes -band [IO.FileAttributes]::ReparsePoint } |
    ForEach-Object { Write-Host ('Unlinking: ' + \$_.FullName); \$_.Delete() }
"
# Only run git worktree remove after confirming 0 remaining junctions
```

### No Memory Updates from Worktree
When CWD is inside a worktree (`.worktrees/xxx/`), do not read or write auto memory. Claude Code determines project memory path based on CWD, so worktree CWD writes to a different memory directory than main project. If memory update is needed, always specify the main project's memory absolute path directly when using Write/Edit.

---

## Plan Writing and Execution Workflow

### Auto-Copy to Clipboard After Plan Writing
When plan file writing is complete via `/superpowers:writing-plans`, automatically copy the `/superpowers:executing-plans <generated plan file path>` command to clipboard.
Windows: `echo /superpowers:executing-plans <path> | clip`

### Re-Check Plan Document When Resuming Execution
When user responds to proceed, before resuming execution:
1. Re-read the plan document to check if user made any modifications
2. Self-review whether subsequent steps need adjustment based on previous step execution results (e.g., unexpected results, newly discovered constraints, approach changes needed)
3. If modifications exist, reflect them in plan document and inform user of changes before starting execution

---

## Execution Environment

### PowerShell argument passing — do NOT get this wrong (recurring failure)
- `Start-Process -ArgumentList @(...)` does NOT quote or escape array elements — it space-joins them, so any element containing spaces is split into multiple args (MS Docs PowerShell-Docs#7701). Inline `wt <command…>` is the same (wt re-splits on spaces; `;` = command separator). A long/spaced argument is destroyed crossing PowerShell → wt → the target program.
- For a long/spaced argument (e.g. a prompt to launch `claude` with): put it in a FILE; a launcher `.ps1` reads it (`$p = Get-Content -Raw file`) and passes it as ONE arg via the call operator `& 'C:\path\exe.exe' $p`, OR pipes it via stdin (`Get-Content -Raw file | & exe …`). Launch the launcher with `wt … pwsh -NoExit -File C:\path\launcher.ps1` so only space-free tokens cross wt.
- Quoting (about_Quoting_Rules): single quotes = literal, no expansion (escape a `'` by doubling `''`); double quotes = interpolation (escape a `"` with a backtick). `--%` (about_Parsing) passes everything after it verbatim to a native exe; `$PSNativeCommandArgumentPassing` (Windows/Standard/Legacy) governs native arg passing.
- After launching ANYTHING, capture and READ its stdout/stderr (redirect to a file, `Tee-Object`, or run via the Bash tool which captures output) — diagnose from the program's real output, never from CPU/process trees/guesswork.
- Default to the **Bash tool** for argument-/quoting-sensitive commands (POSIX single-quoting is simpler and rarely mis-fires); reserve the PowerShell tool for PS cmdlets and Windows-specific operations.
- Claude CLI specifics (verified): `claude -p "…"` (or stdin) runs non-interactively to completion and skips the workspace-trust dialog; interactive `claude "prompt"` only PRE-FILLS the prompt and waits for Enter (it does NOT auto-submit). `Start-Process -ArgumentList` array-splitting was the cause of repeated "spawned session sits idle" failures.

### Program Execution Pattern
Commands that may take 30+ seconds (builds, tests, server runs, migrations, etc.) must run with `run_in_background: true` and check status periodically via `TaskOutput` with `block: false`. Report to user immediately when errors are detected — don't wait meaninglessly until timeout.

### Background vs Blocking Decision (Subagents & Long-Running Tasks)
Before spawning a subagent (`Agent` tool) or starting any long-running task (background-capable `Bash`, `TaskCreate`), deliberately pick the execution mode — never spawn on autopilot. Choose one and state it briefly before executing:
- **백그라운드 사용** — `run_in_background: true`. Use when the work is independent and I can keep making progress or talking with the user while it runs (long fan-out subagents, builds, tests, polls). The harness re-invokes me on completion — don't poll.
- **블로킹** — run synchronously and wait. Use when the next step needs the result before anything else can proceed, or the task is short.
- **결정을 뒤로 미루기** — when neither clearly fits, start blocking and reassess from observed behavior; if it proves long-running, move it to the background then. Decide autonomously — don't ask the user.

**Watchdog requirement (non-self-bounding processes).** Before backgrounding, classify the process: it is safe to background-and-wait only if it is SELF-BOUNDING (exits or times out on its own) or HARNESS-TRACKED (the runtime reliably re-invokes me on completion). If it is NEITHER — i.e. it can deadlock or hang indefinitely (PTY/ConPTY proxies, interactive `attach`/ssh children, anything that may never signal EOF or exit) — it MUST be wrapped in a hard OS-level watchdog that always returns, e.g. PowerShell `$p = Start-Process -PassThru -NoNewWindow -RedirectStandardOutput out.log <exe>; if (-not $p.WaitForExit(<ms>)) { $p.Kill($true) }`, or a `timeout`/job wrapper on unix. NEVER launch a non-self-bounding process in the background and then idle-wait on a completion notification that may never arrive — idle-waiting on such a process can stall indefinitely.

### Android Gradle Build from WSL
When building Android projects on Windows filesystem from WSL, cannot run `./gradlew` directly — must go through PowerShell:
```bash
powershell.exe -NoProfile -Command "cd 'C:\path\to\project'; \$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'; .\gradlew.bat :app:assembleDebug 2>&1"
```
Output is buffered with `tail` pipes and background execution, so verify build success by checking APK/AAR file existence and test XML results.

### TryCloudflare (cloudflared quick tunnels)
`cloudflared` is installed on this PC (winget `Cloudflare.cloudflared`, on PATH via `%LOCALAPPDATA%\Microsoft\WinGet\Links`). To expose a local server publicly with no Cloudflare account/auth:
```
cloudflared tunnel --url http://localhost:<port>
```
It prints an ephemeral `https://<random>.trycloudflare.com` URL — a new random URL each run, gone on Ctrl+C. Testing/dev only: 200 concurrent-request cap (HTTP 429 beyond). If QUIC/UDP is blocked and connection fails, force HTTP/2 with `--protocol http2`.

---

## Plugin Routing: superpowers × compound-engineering × gstack

Three skill families coexist (superpowers = bare names, compound = `ce-` prefix,
gstack = `gstack-` prefix). They overlap on brainstorm/plan/review/commit. To
avoid double-invocation, each phase below has ONE owner. Principle:
**compound** owns upstream product thinking + learning capture; **superpowers**
owns execution discipline; **gstack** owns running the artifact and observing or
shipping it (browser QA, visual/design QA, iOS device QA, deploy+canary). Pick
the owner; the alternative is only a fallback.

Maestro has no persistent state — no env flag, no marker file, no hooks. It is
active in a conversation only while the `maestro` skill's Persistence rule is in
context. Enter it three ways: run `/maestro on`; spawn a session with
`maestro-spawn` (seeded to run `/maestro on`); or, on a complex request, decide
to invoke `/maestro on` yourself — this table is the guidance for that decision.
The `maestro` skill's `on` loads the routing table and adopts; `off` drops it.
With no maestro active, this table is passive documentation.

| Phase                     | Owner skill                                   | Why this side |
|---------------------------|-----------------------------------------------|---------------|
| Strategy / direction      | `ce-strategy`                                 | compound-only |
| Idea generation           | `ce-ideate`                                   | compound-only, proactive |
| Brainstorm requirements   | superpowers `brainstorming`                   | mandatory-before-creative, feeds writing-plans |
| Write the plan            | superpowers `writing-plans`                   | pairs with executing-plans review checkpoints |
| Persona review of plan    | `ce-doc-review` (optional add-on)             | compound persona agents |
| Isolate workspace         | superpowers `using-git-worktrees`             | discipline owner |
| Execute the plan          | superpowers `executing-plans` / `subagent-driven-development` | discipline owner |
| Implement (every change)  | superpowers `test-driven-development`         | TDD enforced |
| Bug / failure             | superpowers `systematic-debugging`            | no-guessing discipline |
| Simplify pass (post-impl) | `/simplify` (built-in)                        | apply reuse/dedupe/altitude cleanups to the just-written diff before the gate (Karpathy #2/#3); `ce-simplify-code` = compound fallback |
| Completion gate           | superpowers `verification-before-completion`  | evidence before "done" |
| Code review               | `ce-code-review`                              | tiered persona agents (`gstack-review` = pre-landing PR pass, fallback) |
| Web / browser QA          | `gstack-qa` / `gstack-browse`                 | gstack drives a real headless browser |
| Rendered-UI / design QA   | `gstack-design-review`                        | visual fidelity, AI-slop, spacing on the live render |
| iOS device QA             | `gstack-ios-qa`                               | live-device SwiftUI QA |
| Commit / push / PR        | `ce-commit-push-pr`                           | compound product flow |
| Resolve PR feedback       | `ce-resolve-pr-feedback`                      | compound |
| Visual proof              | `ce-demo-reel`                                | compound (demo reel for the PR; distinct from design QA) |
| Deploy + canary           | `gstack-land-and-deploy` / `gstack-canary`    | post-merge deploy + post-deploy monitoring |
| ★ Capture the learning    | `ce-compound`                                 | ALWAYS run last — the compounding payoff |

### Hands-off mode
When the user explicitly wants fully autonomous execution, use compound `lfg`
(plan → work → review → test → commit → PR → CI-watch → fix). Even inside `lfg`,
the completion gate (`verification-before-completion`), the post-impl simplify
pass, and TDD discipline still apply — never claim done without evidence.

### Cross-cutting overlays (not a phase — layer on as needed)
- Destructive-command guard → `gstack-careful` / `gstack-guard` / `gstack-freeze`.
- Performance regression → `gstack-benchmark` (web perf) / `ce-optimize` (metric loop).
- Build UI → author with `ce-frontend-design`; verify the rendered result with `gstack-design-review`.

### Conflict tie-breakers
- Two skills could fire for one phase → use the Owner above, not both.
- Product-heavy planning may swap `writing-plans` → `ce-plan`; keep ONE.
- Spec-from-vague-intent: `superpowers:brainstorming` owns; `gstack-spec` is the fallback when the output must be a precise executable spec.
- Debugging owner is `superpowers:systematic-debugging`; `ce-debug` / `gstack-investigate` are fallbacks.
- Simplify pass is quality-only and scoped to the diff (reuse/dedup/altitude) — it never hunts bugs (that's `ce-code-review`) and never refactors beyond the change (Karpathy #3). Skip it for trivial one-line edits.
- After any non-trivial task ends, run `ce-compound` to record the learning.
# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.
