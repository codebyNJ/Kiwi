# Kiwi — Demo Video Script & Run Guide

**Length:** ~2.5 minutes. **One-line pitch:** *Kiwi gives your CI a memory — it remembers every test failure and what fixed it, and recalls it by meaning, not string search.*

**What the demo proves (maps to judging):**
- **Depth of Cognee** — all four lifecycle verbs fire live: `remember`, `recall`, `improve`, `forget`.
- **The wow** — a *new* failure is matched to a *past* incident with different wording/file, and its fix is recalled.
- **Delivery** — a grounded review posted natively on a real GitHub PR.

---

## Part A — Setup (do this BEFORE recording)

1. **Credentials.** In `.env` (repo root) set real values:
   ```env
   COGNEE_BASE_URL=https://tenant-<id>.aws.cognee.ai
   COGNEE_API_KEY=<real>
   COGNEE_TENANT_ID=<id>
   SENTINEL_DATASET=sentinel
   ANTHROPIC_API_KEY=<real>      # or GEMINI_API_KEY — needed for LLM-authored answers
   GITHUB_TOKEN=<real>           # only for the PR-comment scene
   ```
   > Without an LLM key everything still works, but reviews/answers use the deterministic *grounded fallback* (recall is always real). Set a key for the best-looking demo.

2. **Install & seed memory** (seed is one-time; skip if the `sentinel` dataset already exists):
   ```powershell
   uv sync
   uv run sentinel seed        # loads ~20 historical incidents into the 'sentinel' dataset
   ```

3. **Pre-arm the failure report** (so a scene can review a real failure instantly):
   ```powershell
   $env:FLAKY_MODE="1"; uv run pytest app/tests --junitxml=junit_report.xml; Remove-Item Env:FLAKY_MODE
   ```

4. **Open two browser tabs:**
   - Cognee dashboard → `https://platform.cognee.ai` → the `sentinel` dataset → **Mindmap** view.
   - The PR → `https://github.com/codebyNJ/Kiwi/pull/1`.

5. **Terminal ready** in the repo root, font size up, window clean.

---

## Part B — The script (scene by scene)

### Scene 1 — The problem (0:00–0:15)  *[title card or voiceover]*
> "Every CI run starts from zero. A test fails, an engineer spends 30 minutes finding the root cause, fixes it, moves on. Three weeks later a *different* test fails with the *same* root cause — and nobody remembers. Kiwi gives your CI a memory."

### Scene 2 — Ask Kiwi (the money shot) (0:15–0:50)
Launch the agent and ask a plain-English question:
```powershell
uv run kiwi
```
Type:
```
/config
```
> "Kiwi is wired to Cognee memory and an LLM." *(config table shows it live)*

Then type a natural question — **no test file, just words**:
```
We keep getting duplicate charges when a payment webhook retries. Have we hit this before?
```
> "Watch — Kiwi recalls a past incident that used *different words, a different file, a different test name*, and answers with the actual fix: an idempotency key. That's Cognee matching by **meaning**, not string search."

### Scene 3 — Review a live failure (0:50–1:25)
```
/review junit_report.xml
```
> "Here's a brand-new failing test — a duplicate-charge race. Kiwi pulls the matching history from memory and writes a grounded review. Every claim about the past is verified against what was actually recalled — n-gram grounding, so no hallucinated history."

*(Point at the "Recalled history" block quoting the idempotency-key fix.)*

### Scene 4 — All four verbs + the graph (1:25–2:05)
Record the fix (the **improve** verb):
```
/resolve added an idempotency key on charge creation, keyed by event id
```
> "When an engineer confirms the fix, Kiwi writes it into a session that Cognee bridges into the permanent graph — that's `improve`."

Show the supporting commands quickly:
```
/flaky
/history test_concurrent_retry_creates_single_charge
/session
```
> "Failure counts, a test's full history, the session log."

**Cut to the Cognee dashboard tab:**
- **Sessions** → point at the `incident-kiwi-…` session that just appeared.
- **Brain / Mindmap** → the memory graph of incidents.
> "remember, recall, improve — all live, all visible in Cognee. `forget` prunes resolved issues with `/forget`."

### Scene 5 — Delivery: the PR comment (2:05–2:25)
**Cut to the GitHub PR tab** (comment already posted during setup, or run it live — see Part C):
> "And the payoff: Kiwi posts that same grounded, memory-backed review as a native comment on the pull request. CodeRabbit reviews what your diff *looks like*. Kiwi reviews what your code *actually did* — because it remembers."

### Scene 6 — Close (2:25–2:30)
> "Kiwi. Your CI, with a memory. Built on Cognee."

---

## Part C — Running each piece (command reference)

| Demo beat | Command |
|---|---|
| Launch the agent | `uv run kiwi` |
| Show config | `/config` |
| Ask a question (recall + LLM) | *type any question* |
| Review a failure report | `/review junit_report.xml` |
| Record a fix (improve) | `/resolve <summary>` |
| Failure counts | `/flaky` |
| A test's history | `/history <test_name>` |
| Session log | `/session` |
| Prune memory (forget) | `/forget` |
| Exit | `/exit` |

**Live `/test` variant (higher-risk hero moment):** launch with the flake armed so `/test` produces a real failure on camera:
```powershell
$env:FLAKY_MODE="1"; uv run kiwi
# then inside Kiwi:
/test
```
(`/test` runs the whole suite (~5s) and auto-remembers + reviews the one failure.)

**Post the PR comment live** (Scene 5, instead of pre-posting):
```powershell
uv run sentinel ingest junit_report.xml --run-id demo --review --post --repo codebyNJ/Kiwi --pr 1
```

---

## Part D — Demo-day safety
- **Record a backup take** — if the tenant or wifi hiccups during judging, you have footage.
- **Everything degrades gracefully** — no LLM key → deterministic grounded output (recall still real); Cognee unreachable → the pipeline fails soft, never crashes the demo.
- **Rehearse once end-to-end** with a stopwatch; keep it under 3 minutes.
