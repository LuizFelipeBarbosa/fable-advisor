---
name: orchestration
description: Routing doctrine for the architect-as-orchestrator pattern — how a session running the smartest model delegates implementation to a cheaper cross-vendor lane to minimize cost. USE WHEN delegating implementation work to the codex-implementer lane, writing a spec for a subagent, deciding whether to consult fable-advisor, managing session cost or token spend, or running any multi-task build where the session is the architect.
---

# Orchestration — the architect's routing doctrine

The session is the architect: it owns requirements, architecture, decomposition, specs, routing, and verification. It should almost never type implementation code. Every implementation task gets routed to the codex lane — keeping a piece with the architect is deliberate, per task, never a fixed binding.

## Cost discipline — the prime directive

The session model is the most expensive lane in the system, on both input and output tokens. The whole economic case for this pattern is keeping its token volume low: spend Fable on judgment, spend the codex lane on volume. Three rules follow.

**Emit judgment, not volume.** The architect's output is decomposition, specs, routing decisions, verdicts on diffs, and short reports. It does not type implementation code, test bodies, boilerplate, or config files. A code block longer than an interface signature or a few illustrative lines is a spec that hasn't been delegated yet — stop and delegate it. Fixing a lane's bug by hand is the same failure in disguise: send a corrected spec back to the cheap lane instead.

**Keep the context lean.** Everything in the architect's context is re-read at architect prices on every turn. Delegate broad exploration, codebase searches, and log-grepping to a cheap read-only agent and keep only the conclusions; read files yourself only when the decision genuinely depends on the exact code. Don't paste long files, full diffs, or verbose command output into the conversation when a path reference or an excerpt will do.

**Reason once, then hand off.** Do the hard thinking — the architecture, the interface design, the debugging hypothesis — in one pass, capture it in the spec, and let the cheap lane carry it from there. Re-deriving decisions across turns burns the premium twice.

What stays with the architect regardless of cost: decomposition, interface design, hypothesis selection when debugging, spec writing, lane routing, and judging verification evidence. Those tokens are what the premium is for — everything else is a candidate for delegation.

## The lanes

| Lane | Producer | Invoke | Route here when |
|---|---|---|---|
| Implementation | GPT-6 Astra | `codex-implementer` agent | Every implementation task: boilerplate, wiring, CRUD, mechanical edits, features, and correctness-critical work alike. **Default lane.** Requires the codex CLI. |
| Judgment | Fable 5 | `fable-advisor` agent | Not an implementation lane. See "Commitment boundaries" below. |

Deciding rule: how much does the outcome depend on judgment the spec can't capture? Little → the codex lane; you will verify anyway. A lot, and mistakes are costly → keep that piece with the architect.

The codex lane is a non-Anthropic family, so every implementation gets genuine cross-vendor review from the Claude architect — the review is built into the routing, not bolted on.

If the codex lane returns `unavailable`, implement with the built-in `general-purpose` agent and state the downgrade plainly in your report — never quietly absorb the substitution. If it returns `timeout`, the spec was too big for one run: split it (see "Sizing a delegation") before falling back.

## Sizing a delegation

The lane runs one `codex exec` under a ten-minute wall clock, at maximum reasoning effort, with no network. Size specs to that box:

- **One concern, a handful of files.** A feature touching three to six files with a verification command that finishes in under a minute is the comfortable size. Beyond that, split by dependency order and delegate the pieces serially — or in parallel if they share no files.
- **Verification must be self-contained.** Dependencies are installed before delegation, not by the lane. A spec whose check needs `npm install` or a fresh container fails inside the sandbox.
- **A timeout is a sizing failure, not a lane failure.** Don't re-send the same spec; cut it down and re-send the pieces.
- Overrides: `FABLE_ADVISOR_CODEX_MODEL` and `FABLE_ADVISOR_CODEX_EFFORT` change the lane's model and reasoning effort for the session without editing the plugin. Lower the effort for genuinely mechanical work; keep xhigh for anything correctness-critical.

## The spec contract

Implementers share none of your conversation context. Every delegation prompt carries all five parts:

1. **Objective** — what to build or change, one paragraph
2. **Files** — exact paths to create or modify
3. **Interfaces** — signatures, types, or API shapes the code must match
4. **Constraints** — project conventions, things not to touch
5. **Verification** — the command(s) that prove it works

A spec you can't finish writing is a signal the decision isn't made yet — that's architect work, not a reason to hand the ambiguity to a cheaper model.

A spec that fits the contract looks like this:

```
Objective: Add a per-client rate limiter to the public API. Requests over
the limit return 429 with a Retry-After header; limits are configured per
API key and default to 100/min.

Files:
- src/middleware/rateLimit.ts (create)
- src/server.ts (register the middleware before the router)
- test/rateLimit.test.ts (create)

Interfaces:
- export function rateLimit(opts: { limitPerMinute: number; keyFrom: (req) => string }): Middleware
- Store is the existing Redis client from src/lib/redis.ts — do not add a new one.

Constraints:
- No new dependencies. Follow the middleware pattern in src/middleware/auth.ts.
- Do not touch the router or the auth middleware.

Verification: npm test -- rateLimit
```

## Parallelism

Independent specs (no shared files, no ordering dependency) launch as parallel agents in a single message. Sequential chains and single-file surgery stay serial.

## Commitment boundaries

Consult `fable-advisor` (read-only, verdict in under 300 words) at the moments that decide whether the next hour is wasted:

- Before committing to an architecture, data migration, API shape, or refactor strategy
- Whenever the same problem has resisted two distinct attempts
- Once before declaring a multi-step deliverable done

Pass it the decision, the constraints, and the options considered. Act on the verdict or surface the disagreement — never silently ignore it. (If the session itself already runs on Fable, the advisor still earns its keep as a context-clean skeptic reading the actual code.)

## Verification

Reports are claims, not evidence. Before accepting any lane's work: read the diff, and re-run the verification command (or spot-check its quoted output against the working tree). "Should work", "tests should pass", or a report with no command output means the task is not done. A lane that reports a spec gap gets a corrected spec, not a "use your judgment".
