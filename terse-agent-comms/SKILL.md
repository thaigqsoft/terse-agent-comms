---
name: terse-agent-comms
description: Use when writing any prompt or message to another agent or worker (Agent tool, SendMessage, CLI wrappers), including follow-ups to a worker already running, or when asking a worker for its report. Keywords - token budget, dispatch prompt, delegate, subagent, follow-up, handoff.
---

# Terse Agent Comms

A new worker gets a short contract. A worker that already has the contract gets only the delta.

## Which form

| Situation | Form |
|---|---|
| Fresh worker (Agent, CLI wrapper, new session) | Contract template below |
| Same worker, same task (SendMessage after its reply) | Delta message |
| Small local model | Contract fields as plain sentences, one task, ≤1000 chars |

## Contract template

```
GOAL:    one-line outcome                                   [REQUIRED]
CTX:     facts the worker cannot discover; pointers
         (path:line, commit, doc path), not pasted content  [optional]
DO:      only steps where a capable worker would go wrong   [optional]
RULES:   terse hard constraints = scope fence               [REQUIRED]
DONE:    verify command → expected result                   [REQUIRED]
STOP-IF: condition → report, don't guess                    [optional]
REPLY:   ≤N lines, fields below                             [REQUIRED]
```

Leave DO out when the GOAL already implies the steps. An investigation brief carries symptoms, suspects and the fence; the worker plans its own steps.

## Delta message

```
<new fact or correction>. <next action>. Rest as before.
```

Example: `Lock is stale (no git proc). rm .git/index.lock, then add + commit as briefed. Same REPLY.`

The worker still holds GOAL, RULES, DONE and REPLY. A new constraint goes in the delta; an unchanged one stays out.

## Worker reply

```
STATUS: done|partial|blocked
CHANGED: file(s)/commit
VERIFY: <cmd> → <result tail>
FOUND: (research/review only)
NOTE: unexpected only
```

## Compression recipe

- Imperative fragments; start at GOAL, no role preamble, no pleasantries.
- Add a reason only where a worker would plausibly break the rule without it (≤6 words).
- Point to the domain doc instead of restating: "rules: see docs/php56.md".
- Symbols: → ≤ w/ & ~
- Full absolute path once, then repo-relative.
- Thai or literal data stays verbatim.
- Secrets by env-var name only, never values.

## Budget

| Message type | Words |
|---|---|
| delta follow-up | ≤30 |
| mechanical edit | ≤60 |
| search / read-only | ≤80 |
| investigation / review | ≤120 |
| reply | ≤10 lines (review ≤25) |

## Example (contract)

Before: ~330 words of prose (role preamble, background, PHP rules, long checklist).

After (~55 words):

```
GOAL: Api\ArticlesController::update() calls clearWidgetCache($website) like delete() does.
CTX: app/Http/Controllers/Api/ArticlesController.php; mirror delete().
RULES: PHP5.6 (see docs/php56.md); edit + git add that file only; commit on master w/ <trailer>; no push.
DONE: php -l <file> → "No syntax errors".
STOP-IF: $website not available in update() → report.
REPLY: ≤6 lines: STATUS, CHANGED (commit hash), VERIFY, NOTE.
```

## Common mistakes

- Cutting RULES or STOP-IF to save words: the costliest cut, since one retry outweighs 20 words.
- Pasting file contents instead of path:line.
- Asking for a narrative report instead of the named fields.
